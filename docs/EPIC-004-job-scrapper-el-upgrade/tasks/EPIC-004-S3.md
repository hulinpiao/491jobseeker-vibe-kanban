# EPIC-004-S3: JD Analysis

## Overview
**Epic**: EPIC-004
**Repository**: `modules/brightdata`
**Feature Branch**: `feature/epic-004-job-scraper-elt-upgrade`
**Agent**: Backend

## Context
This story implements JD analysis with three-stage filtering: visa/PR requirements, security clearance, and skill/experience matching.

Reference:
- PRD: Section 2.1 (F4: JD Analysis)
- Solution Architecture: Section 2.1 (JD Analyzer)

## Tasks

### TASK-1: Create JD Analyzer Module

**Description**: Implement JD analysis with three-stage filtering

**Detailed Steps**:

1. Create `src/job_scraper/etl/jd_analyzer.py`:

```python
"""
JD Analysis Pipeline
Filters jobs by visa requirements, security clearance, and skill matching
"""
import json
import logging
from datetime import datetime
from pathlib import Path
from typing import Optional

from pymongo import MongoClient, UpdateOne

from src.job_scraper.llm.glm_client import GLMClient
from src.job_scraper.llm.prompts import (
    format_resume_for_analysis,
    JD_ANALYSIS_SYSTEM_PROMPT,
    JD_ANALYSIS_USER_PROMPT_TEMPLATE,
)

logger = logging.getLogger(__name__)


class JDAnalyzer:
    """Analyze JDs for visa, clearance, and skill compatibility"""

    # Visa requirement keywords
    VISA_KEYWORDS = [
        "citizenship",
        "citizen",
        "pr",
        "permanent resident",
        "permanent residency",
        "australian citizen",
        "must be australian",
        "must be pr",
        "must have permanent",
        "working rights in australia",
        " unrestricted",
    ]

    # Security clearance keywords
    CLEARANCE_KEYWORDS = [
        "nv1",
        "nv2",
        "baseline",
        "security clearance",
        "defence clearance",
        "government clearance",
        "negative vetting",
        "secret clearance",
        "top secret",
    ]

    def __init__(
        self,
        mongo_client: MongoClient,
        glm_client: GLMClient,
        resume_path: Optional[Path] = None,
        skill_threshold: int = 60,
    ):
        """
        Initialize JDAnalyzer.

        Args:
            mongo_client: MongoDB client
            glm_client: GLM-4 API client
            resume_path: Path to resume.json
            skill_threshold: Minimum skill match score (0-100)
        """
        self.mongo = mongo_client
        self.glm = glm_client
        self.skill_threshold = skill_threshold
        self.resume = self._load_resume(resume_path)

    def _load_resume(self, resume_path: Optional[Path]) -> dict:
        """Load resume configuration"""
        if resume_path is None:
            base_dir = Path(__file__).parent.parent.parent
            resume_path = base_dir / "config" / "resume.json"

        with open(resume_path) as f:
            return json.load(f)

    def check_visa_requirements(self, job_description: str) -> dict:
        """
        Rule-based check for citizenship/PR requirements.

        Args:
            job_description: Job description text

        Returns:
            Dict with {compatible: bool, reason: str}
        """
        text_lower = job_description.lower()

        # Check for exclusion keywords
        for keyword in self.VISA_KEYWORDS:
            if keyword in text_lower:
                return {
                    "compatible": False,
                    "reason": f"Visa requirement found: '{keyword}'",
                    "keyword": keyword,
                }

        return {"compatible": True, "reason": "No visa restrictions found"}

    def check_security_clearance(self, job_description: str) -> dict:
        """
        Rule-based check for security clearance requirements.

        Args:
            job_description: Job description text

        Returns:
            Dict with {required: bool, reason: str}
        """
        text_lower = job_description.lower()

        for keyword in self.CLEARANCE_KEYWORDS:
            if keyword in text_lower:
                return {
                    "required": True,
                    "reason": f"Security clearance required: '{keyword}'",
                    "keyword": keyword,
                }

        return {"required": False, "reason": "No clearance requirements found"}

    def match_skills(self, job: dict) -> dict:
        """
        GLM-4 based skill and experience matching.

        Args:
            job: Normalized job dict

        Returns:
            Dict with match analysis
        """
        resume_formatted = format_resume_for_analysis(self.resume)
        prompt = JD_ANALYSIS_USER_PROMPT_TEMPLATE.format(
            job_description=job.get("job_description", ""),
            **resume_formatted,
        )

        try:
            response = self.glm.chat_json(JD_ANALYSIS_SYSTEM_PROMPT, prompt)

            return {
                "score": response.get("skill_match_score", 0),
                "matched_skills": response.get("matched_skills", []),
                "missing_skills": response.get("missing_skills", []),
                "experience_match": response.get("experience_match", False),
                "reason": response.get("reason", ""),
            }
        except Exception as e:
            logger.error(f"GLM skill matching failed: {e}")
            return {
                "score": 0,
                "matched_skills": [],
                "missing_skills": [],
                "experience_match": False,
                "reason": f"Analysis failed: {e}",
            }

    def analyze_job(self, job: dict) -> dict:
        """
        Run full analysis on a job.

        Args:
            job: Normalized job dict

        Returns:
            Job dict with analysis_result added
        """
        job_desc = job.get("job_description", "")

        # Stage 1: Visa check
        visa_check = self.check_visa_requirements(job_desc)

        # Stage 2: Security clearance check
        clearance_check = self.check_security_clearance(job_desc)

        # Stage 3: Skill matching (only if visa compatible)
        skill_match = None
        if visa_check["compatible"] and not clearance_check["required"]:
            skill_match = self.match_skills(job)

        # Determine if job passes
        passes = (
            visa_check["compatible"]
            and not clearance_check["required"]
            and skill_match is not None
            and skill_match["score"] >= self.skill_threshold
        )

        job["analysis_result"] = {
            "visa_compatible": visa_check["compatible"],
            "visa_reason": visa_check["reason"],
            "security_clearance_required": clearance_check["required"],
            "clearance_reason": clearance_check["reason"],
            "skill_match_score": skill_match["score"] if skill_match else 0,
            "skill_match_details": skill_match if skill_match else None,
            "passed": passes,
            "analyzed_at": datetime.utcnow().isoformat(),
        }

        return job

    def filter_jobs(
        self,
        jobs: list,
    ) -> tuple[list[dict], list[dict], dict]:
        """
        Filter jobs, return (passed, failed, stats).

        Args:
            jobs: List of normalized job dicts

        Returns:
            Tuple of (passed_jobs, failed_jobs, statistics)
        """
        passed = []
        failed = []
        stats = {
            "total_analyzed": len(jobs),
            "visa_incompatible": 0,
            "security_clearance_required": 0,
            "skill_mismatch": 0,
            "passed": 0,
        }

        for job in jobs:
            analyzed = self.analyze_job(job)
            result = analyzed["analysis_result"]

            if result["passed"]:
                passed.append(analyzed)
                stats["passed"] += 1
            else:
                failed.append(analyzed)
                if not result["visa_compatible"]:
                    stats["visa_incompatible"] += 1
                elif result["security_clearance_required"]:
                    stats["security_clearance_required"] += 1
                elif result["skill_match_score"] < self.skill_threshold:
                    stats["skill_mismatch"] += 1

        logger.info(
            f"Filter results: {stats['passed']} passed, "
            f"{stats['visa_incompatible']} visa incompatible, "
            f"{stats['security_clearance_required']} clearance required, "
            f"{stats['skill_mismatch']} skill mismatch"
        )

        return passed, failed, stats

    def load_filtered_jobs(self, jobs: list, date: Optional[str] = None):
        """
        Load filtered (passed) jobs to filtered_jobs database.

        Args:
            jobs: List of passed job dictionaries
            date: Date string for collection name (YYYY_MM_DD)
        """
        if not jobs:
            logger.warning("No filtered jobs to load")
            return

        if date is None:
            date = datetime.now().strftime("%Y_%m_%d")

        collection_name = date
        collection = self.mongo.filtered_jobs[collection_name]

        # Add timestamp
        for job in jobs:
            job["filtered_at"] = datetime.utcnow()

        # Bulk upsert
        operations = [
            UpdateOne(
                {"job_id": job["job_id"]},
                {"$set": job},
                upsert=True
            )
            for job in jobs
        ]

        result = collection.bulk_write(operations)

        logger.info(
            f"Loaded to filtered_jobs.{collection_name}: "
            f"{result.upserted_count} jobs"
        )

    def run(self, jobs: list) -> dict:
        """
        Run full JD analysis pipeline.

        Args:
            jobs: List of normalized job dicts from NormalizationETL

        Returns:
            Summary dict with statistics
        """
        logger.info("Starting JD Analysis")

        passed, failed, stats = self.filter_jobs(jobs)

        # Load passed jobs
        self.load_filtered_jobs(passed)

        logger.info(f"JD Analysis complete: {stats['passed']}/{stats['total_analyzed']} passed")

        return stats
```

2. Create tests `tests/test_jd_analyzer.py`:

```python
"""
Tests for JD Analyzer
"""
import json
from unittest.mock import Mock, MagicMock
from pathlib import Path
import pytest

from pymongo import MongoClient

from src.job_scraper.etl.jd_analyzer import JDAnalyzer


@pytest.fixture
def resume_file(tmp_path):
    """Create temporary resume file"""
    resume = {
        "profile": {
            "name": "Test User",
            "title": "DevOps Engineer",
            "years_of_experience": 5,
            "visa_status": "491",
        },
        "technical_skills": {
            "cloud": ["AWS", "Azure"],
            "devops": ["Docker", "Kubernetes", "Terraform"],
        },
        "work_experience": [
            {
                "title": "DevOps Engineer",
                "company": "Tech Corp",
                "key_skills": ["AWS", "Docker"],
            }
        ],
        "certifications": ["AWS DevOps Professional"],
        "exclusion_criteria": {
            "visa_requirements": ["Citizenship", "PR"],
            "security_clearance": ["NV1", "NV2"],
        },
    }
    path = tmp_path / "resume.json"
    path.write_text(json.dumps(resume))
    return path


@pytest.fixture
def mongo_client():
    """Mock MongoDB client"""
    client = MagicMock(spec=MongoClient)
    client.filtered_jobs = MagicMock()
    return client


@pytest.fixture
def glm_client():
    """Mock GLM client"""
    client = Mock()
    client.chat_json = Mock(return_value={
        "visa_compatible": True,
        "security_clearance_required": False,
        "skill_match_score": 85,
        "matched_skills": ["Docker", "Kubernetes", "AWS"],
        "missing_skills": [],
        "experience_match": True,
        "reason": "Strong match on core DevOps skills",
    })
    return client


@pytest.fixture
def sample_job():
    return {
        "job_id": "123",
        "job_title": "DevOps Engineer",
        "company_name": "Test Company",
        "job_description": "We are looking for a DevOps Engineer with AWS and Docker experience.",
        "job_location": "Canberra, ACT",
        "sources": [{"platform": "linkedin"}],
    }


class TestJDAnalyzer:
    """Test JDAnalyzer class"""

    def test_init(self, mongo_client, glm_client, resume_file):
        analyzer = JDAnalyzer(mongo_client, glm_client, resume_file)
        assert analyzer.mongo == mongo_client
        assert analyzer.glm == glm_client
        assert analyzer.skill_threshold == 60
        assert analyzer.resume["profile"]["visa_status"] == "491"

    def test_check_visa_requirements_compatible(self, mongo_client, glm_client, resume_file):
        analyzer = JDAnalyzer(mongo_client, glm_client, resume_file)
        result = analyzer.check_visa_requirements("No visa requirements mentioned")

        assert result["compatible"] is True

    def test_check_visa_requirements_incompatible(self, mongo_client, glm_client, resume_file):
        analyzer = JDAnalyzer(mongo_client, glm_client, resume_file)
        result = analyzer.check_visa_requirements("Must be Australian Citizen or PR")

        assert result["compatible"] is False
        assert "citizen" in result["reason"].lower()

    def test_check_security_clearance_not_required(self, mongo_client, glm_client, resume_file):
        analyzer = JDAnalyzer(mongo_client, glm_client, resume_file)
        result = analyzer.check_security_clearance("No clearance mentioned")

        assert result["required"] is False

    def test_check_security_clearance_required(self, mongo_client, glm_client, resume_file):
        analyzer = JDAnalyzer(mongo_client, glm_client, resume_file)
        result = analyzer.check_security_clearance("Must have NV1 security clearance")

        assert result["required"] is True
        assert "nv1" in result["reason"]

    def test_match_skills(self, mongo_client, glm_client, resume_file, sample_job):
        analyzer = JDAnalyzer(mongo_client, glm_client, resume_file)
        result = analyzer.match_skills(sample_job)

        assert "score" in result
        assert "matched_skills" in result
        assert "missing_skills" in result

    def test_analyze_job_passing(self, mongo_client, glm_client, resume_file, sample_job):
        analyzer = JDAnalyzer(mongo_client, glm_client, resume_file)
        analyzed = analyzer.analyze_job(sample_job)

        assert "analysis_result" in analyzed
        assert analyzed["analysis_result"]["visa_compatible"] is True
        assert analyzed["analysis_result"]["passed"] is True

    def test_analyze_job_failing_visa(self, mongo_client, glm_client, resume_file):
        analyzer = JDAnalyzer(mongo_client, glm_client, resume_file)
        job = {
            "job_id": "456",
            "job_description": "Must be Australian Citizen or Permanent Resident",
        }
        analyzed = analyzer.analyze_job(job)

        assert analyzed["analysis_result"]["visa_compatible"] is False
        assert analyzed["analysis_result"]["passed"] is False

    def test_filter_jobs(self, mongo_client, glm_client, resume_file):
        analyzer = JDAnalyzer(mongo_client, glm_client, resume_file)

        jobs = [
            {
                "job_id": "1",
                "job_description": "DevOps role, no restrictions",
            },
            {
                "job_id": "2",
                "job_description": "Must be Australian Citizen",
            },
            {
                "job_id": "3",
                "job_description": "Requires NV1 clearance",
            },
        ]

        passed, failed, stats = analyzer.filter_jobs(jobs)

        assert stats["total_analyzed"] == 3
        assert stats["visa_incompatible"] >= 1
        assert stats["security_clearance_required"] >= 1
```

**Acceptance Criteria**:
- [ ] JDAnalyzer class created
- [ ] Visa filter checks for citizenship/PR keywords
- [ ] Security clearance filter checks for NV1/NV2/Baseline
- [ ] Skill matching uses GLM-4 with resume profile
- [ ] Filter threshold: 60 points minimum
- [ ] `filter_jobs()` returns (passed, failed, stats)
- [ ] `load_filtered_jobs()` saves to `filtered_jobs.YYYY_MM_DD`
- [ ] Tests pass

---

## Git Workflow

### Commits
```bash
git add src/job_scraper/etl/jd_analyzer.py tests/test_jd_analyzer.py
git commit -m "feat(etl): add JD analysis pipeline with visa, clearance, and skill filtering"
```

---

## Completion Checklist
- [ ] JDAnalyzer implemented
- [ ] Three-stage filtering working
- [ ] Statistics tracked
- [ ] Tests pass
- [ ] Commits pushed

---
**Version**: 1.0
**Created**: 2026-02-10
