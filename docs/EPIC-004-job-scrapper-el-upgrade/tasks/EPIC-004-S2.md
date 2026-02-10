# EPIC-004-S2: Normalization ETL

## Overview
**Epic**: EPIC-004
**Repository**: `modules/brightdata`
**Feature Branch**: `feature/epic-004-job-scraper-elt-upgrade`
**Agent**: Backend

## Context
This story implements the normalization pipeline using GLM-4. It replaces the old `unified_etl.py` with a new GLM-4 based normalization that outputs to `normalized_jobs` database.

Reference:
- PRD: Section 2.1 (F3: Normalization ETL)
- Solution Architecture: Section 2.1 (Normalization ETL)

## Tasks

### TASK-1: Create Normalization Prompts

**Description**: Design prompts for job normalization

**Detailed Steps**:

1. Create `src/job_scraper/llm/prompts.py`:

```python
"""
LLM Prompts for job processing
"""

# Normalization Prompts
NORMALIZATION_SYSTEM_PROMPT = """You are a job data normalizer for Australian job postings.
Your task is to extract and normalize job information from raw job postings.

Follow these rules:
1. employment_type must be one of: full_time, part_time, contract, casual, internship
2. work_arrangement must be one of: onsite, remote, hybrid
3. job_location format: "City, State" (e.g., "Canberra, ACT")
4. State must be Australian state abbreviation: ACT, NSW, NT, QLD, SA, TAS, VIC, WA
5. posted_date must be in ISO 8601 format (YYYY-MM-DD)
6. Remove all HTML tags from job_description
7. If a field cannot be determined, use null

Output JSON only, no additional text."""

NORMALIZATION_USER_PROMPT_TEMPLATE = """Normalize the following job postings and return a JSON array.

For each job, provide:
- index: the original index in the input array
- job_title: normalized job title
- company_name: normalized company name
- job_description: clean description with HTML removed
- job_location: "City, State" format
- employment_type: one of: full_time, part_time, contract, casual, internship
- work_arrangement: one of: onsite, remote, hybrid
- posted_date: ISO date format (YYYY-MM-DD)

Jobs to normalize:
{jobs_text}

Return JSON format:
{{"jobs": [{{"index": 0, "job_title": "...", "company_name": "...", ...}}]}}"""

# JD Analysis Prompts
JD_ANALYSIS_SYSTEM_PROMPT = """You are an expert job matcher analyzing job descriptions against a candidate profile.
Focus on DevOps, SRE, Cloud Engineering, and Platform Engineering roles.

Evaluate:
1. Visa compatibility: Does the JD require citizenship/PR? (491 visa holders cannot apply)
2. Security clearance: Does the JD require NV1/NV2/Baseline clearance?
3. Skill match: How well do the candidate's skills match the JD requirements? (0-100 score)
4. Experience match: Does the candidate have relevant experience?

Output JSON only, no additional text."""

JD_ANALYSIS_USER_PROMPT_TEMPLATE = """Analyze this job description against the candidate profile.

Job Description:
{job_description}

Candidate Profile:
Name: {name}
Title: {title}
Years of Experience: {years_of_experience}
Visa Status: {visa_status}

Skills:
{skills}

Work Experience:
{experience}

Certifications:
{certifications}

Exclusion Criteria:
- Visa Requirements: {visa_requirements}
- Security Clearance: {security_clearance}

Return JSON:
{{
  "visa_compatible": true/false,
  "security_clearance_required": true/false,
  "skill_match_score": 0-100,
  "matched_skills": ["skill1", "skill2"],
  "missing_skills": ["skill1", "skill2"],
  "experience_match": true/false,
  "reason": "brief explanation"
}}"""


def format_jobs_for_normalization(jobs: list) -> str:
    """Format jobs list for LLM normalization prompt"""
    formatted = []
    for i, job in enumerate(jobs):
        formatted.append(f"[{i}]")
        formatted.append(f"Title: {job.get('job_title', 'N/A')}")
        formatted.append(f"Company: {job.get('company_name', 'N/A')}")
        formatted.append(f"Location: {job.get('job_location', 'N/A')}")
        formatted.append(f"Description: {job.get('job_description', 'N/A')[:500]}...")
        formatted.append(f"Posted: {job.get('job_posted_time', 'N/A')}")
        formatted.append(f"Type: {job.get('job_employment_type', 'N/A')}")
        formatted.append("")
    return "\n".join(formatted)


def format_resume_for_analysis(resume: dict) -> dict:
    """Format resume dict for JD analysis prompt"""
    # Flatten skills
    all_skills = resume.get("technical_skills", {})
    skill_list = []
    for category, skills in all_skills.items():
        if isinstance(skills, list):
            skill_list.extend(skills)

    # Format experience
    exp_list = []
    for exp in resume.get("work_experience", []):
        exp_list.append(f"- {exp.get('title')} at {exp.get('company')}: {', '.join(exp.get('key_skills', []))}")

    return {
        "name": resume.get("profile", {}).get("name", ""),
        "title": resume.get("profile", {}).get("title", ""),
        "years_of_experience": resume.get("profile", {}).get("years_of_experience", 0),
        "visa_status": resume.get("profile", {}).get("visa_status", ""),
        "skills": ", ".join(skill_list[:50]),  # Limit to 50 skills
        "experience": "\n".join(exp_list[:5]),  # Limit to 5 experiences
        "certifications": ", ".join(resume.get("certifications", [])),
        "visa_requirements": ", ".join(resume.get("exclusion_criteria", {}).get("visa_requirements", [])),
        "security_clearance": ", ".join(resume.get("exclusion_criteria", {}).get("security_clearance", [])),
    }
```

2. Create tests `tests/test_prompts.py`:

```python
"""
Tests for LLM prompts
"""
from src.job_scraper.llm.prompts import (
    format_jobs_for_normalization,
    format_resume_for_analysis,
    NORMALIZATION_SYSTEM_PROMPT,
    JD_ANALYSIS_SYSTEM_PROMPT,
)


def test_format_jobs_for_normalization():
    jobs = [
        {
            "job_title": "DevOps Engineer",
            "company_name": "Test Company",
            "job_location": "Canberra, ACT",
            "job_description": "We are looking for...",
            "job_posted_time": "2 days ago",
            "job_employment_type": "Full-time",
        }
    ]
    result = format_jobs_for_normalization(jobs)

    assert "[0]" in result
    assert "DevOps Engineer" in result
    assert "Test Company" in result


def test_format_resume_for_analysis(resume_data):
    result = format_resume_for_analysis(resume_data)

    assert result["name"] == "Hulin Piao (Eric)"
    assert "DevOps" in result["skills"]
    assert "AWS" in result["skills"]
    assert result["visa_status"] == "491"


@pytest.fixture
def resume_data():
    return {
        "profile": {
            "name": "Hulin Piao (Eric)",
            "title": "Senior DevSecOps Engineer",
            "years_of_experience": 9,
            "visa_status": "491",
        },
        "technical_skills": {
            "programming_languages": ["Python", "Bash"],
            "cloud": ["AWS", "Azure"],
        },
        "work_experience": [
            {
                "title": "DevOps Engineer",
                "company": "Test Corp",
                "key_skills": ["Docker", "Kubernetes"],
            }
        ],
        "certifications": ["AWS DevOps Professional"],
        "exclusion_criteria": {
            "visa_requirements": ["Citizenship", "PR"],
            "security_clearance": ["NV1", "NV2"],
        },
    }
```

**Acceptance Criteria**:
- [ ] Prompts module created
- [ ] Normalization prompts defined
- [ ] JD analysis prompts defined
- [ ] Helper functions for formatting
- [ ] Tests pass

---

### TASK-2: Create Normalization ETL Module

**Description**: Implement normalization using GLM-4

**Detailed Steps**:

1. Create `src/job_scraper/etl/normalization_etl.py`:

```python
"""
Normalization ETL using GLM-4
"""
import logging
from datetime import datetime, timedelta
from typing import Optional

from pymongo import MongoClient, UpdateOne
from src.job_scraper.llm.glm_client import GLMClient
from src.job_scraper.llm.prompts import (
    format_jobs_for_normalization,
    NORMALIZATION_SYSTEM_PROMPT,
    NORMALIZATION_USER_PROMPT_TEMPLATE,
)

logger = logging.getLogger(__name__)


class NormalizationETL:
    """Normalize raw jobs from all platforms using GLM-4"""

    def __init__(self, mongo_client: MongoClient, glm_client: GLMClient):
        """
        Initialize NormalizationETL.

        Args:
            mongo_client: MongoDB client
            glm_client: GLM-4 API client
        """
        self.mongo = mongo_client
        self.glm = glm_client
        self.batch_size = 8

    def extract_today_jobs(self) -> list:
        """
        Extract jobs created today from all platform collections.

        Returns:
            List of raw job dictionaries
        """
        today = datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)
        tomorrow = today + timedelta(days=1)

        jobs = []
        platforms = ["linkedin_jobs", "indeed_jobs", "seek_jobs"]

        for platform in platforms:
            collection = self.mongo.job_scraper[platform]
            cursor = collection.find({
                "created_at": {"$gte": today, "$lt": tomorrow}
            })

            for job in cursor:
                job["_platform"] = platform.replace("_jobs", "")
                jobs.append(job)

            logger.info(f"Extracted {cursor.count()} jobs from {platform}")

        logger.info(f"Total jobs extracted today: {len(jobs)}")
        return jobs

    def normalize_jobs(self, jobs: list) -> list:
        """
        Normalize jobs using GLM-4 in batches.

        Args:
            jobs: List of raw job dictionaries

        Returns:
            List of normalized job dictionaries
        """
        if not jobs:
            return []

        normalized = []
        total_batches = (len(jobs) + self.batch_size - 1) // self.batch_size

        for i in range(0, len(jobs), self.batch_size):
            batch = jobs[i:i + self.batch_size]
            batch_num = i // self.batch_size + 1

            logger.info(f"Normalizing batch {batch_num}/{total_batches} ({len(batch)} jobs)")

            try:
                batch_normalized = self._normalize_batch(batch)
                normalized.extend(batch_normalized)
            except Exception as e:
                logger.error(f"Failed to normalize batch {batch_num}: {e}")
                # Add jobs with minimal normalization as fallback
                normalized.extend(self._fallback_normalize(batch))

        return normalized

    def _normalize_batch(self, batch: list) -> list:
        """Normalize a single batch using GLM-4"""
        jobs_text = format_jobs_for_normalization(batch)
        prompt = NORMALIZATION_USER_PROMPT_TEMPLATE.format(jobs_text=jobs_text)

        response = self.glm.chat_json(NORMALIZATION_SYSTEM_PROMPT, prompt)

        # Map response back to original jobs
        normalized = []
        for item in response.get("jobs", []):
            idx = item.get("index")
            if idx is not None and 0 <= idx < len(batch):
                original = batch[idx]
                normalized_job = {
                    "job_id": original.get("job_posting_id", f"unknown_{idx}"),
                    "job_title": item.get("job_title"),
                    "company_name": item.get("company_name"),
                    "job_description": item.get("job_description"),
                    "job_location": item.get("job_location"),
                    "employment_type": item.get("employment_type"),
                    "work_arrangement": item.get("work_arrangement"),
                    "posted_date": item.get("posted_date"),
                    "sources": [{
                        "platform": original.get("_platform"),
                        "job_posting_id": original.get("job_posting_id"),
                        "url": original.get("apply_link", ""),
                    }],
                    "created_at": datetime.utcnow(),
                }
                normalized.append(normalized_job)

        return normalized

    def _fallback_normalize(self, batch: list) -> list:
        """Fallback normalization when LLM fails"""
        normalized = []
        for job in batch:
            normalized.append({
                "job_id": job.get("job_posting_id", ""),
                "job_title": job.get("job_title", ""),
                "company_name": job.get("company_name", ""),
                "job_description": job.get("job_description", ""),
                "job_location": job.get("job_location", ""),
                "employment_type": "full_time",
                "work_arrangement": "remote",
                "posted_date": datetime.now().strftime("%Y-%m-%d"),
                "sources": [{
                    "platform": job.get("_platform"),
                    "job_posting_id": job.get("job_posting_id"),
                    "url": job.get("apply_link", ""),
                }],
                "created_at": datetime.utcnow(),
            })
        return normalized

    def deduplicate_jobs(self, jobs: list) -> list:
        """
        Remove duplicate jobs (same company, title, location).

        Args:
            jobs: List of normalized job dictionaries

        Returns:
            Deduplicated list
        """
        seen = {}
        deduplicated = []

        for job in jobs:
            # Create dedup key
            key = self._create_dedup_key(job)

            if key not in seen:
                seen[key] = job
                deduplicated.append(job)
            else:
                # Merge sources if duplicate
                seen[key]["sources"].extend(job["sources"])

        logger.info(f"Deduplicated: {len(jobs)} -> {len(deduplicated)} jobs")
        return deduplicated

    def _create_dedup_key(self, job: dict) -> str:
        """Create deduplication key from job"""
        company = job.get("company_name", "").lower().strip()
        title = job.get("job_title", "").lower().strip()
        location = job.get("job_location", "").lower().strip()

        # Normalize
        company = company.replace(" ", "").replace("-", "").replace(".", "")
        title = title.replace(" ", "").replace("-", "").replace(".", "")

        return f"{company}::{title}::{location}"

    def load_normalized_jobs(self, jobs: list, date: Optional[str] = None):
        """
        Load normalized jobs to normalized_jobs database.

        Args:
            jobs: List of normalized job dictionaries
            date: Date string for collection name (YYYY_MM_DD). Defaults to today.
        """
        if not jobs:
            logger.warning("No jobs to load")
            return

        if date is None:
            date = datetime.now().strftime("%Y_%m_%d")

        collection_name = date
        collection = self.mongo.normalized_jobs[collection_name]

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
            f"Loaded to normalized_jobs.{collection_name}: "
            f"{result.upserted_count} upserted, {result.modified_count} modified"
        )

    def run(self) -> dict:
        """
        Run full normalization pipeline.

        Returns:
            Summary dict with counts
        """
        logger.info("Starting Normalization ETL")

        # Extract
        raw_jobs = self.extract_today_jobs()
        if not raw_jobs:
            logger.warning("No jobs found for today")
            return {"raw_count": 0, "normalized_count": 0}

        # Normalize
        normalized = self.normalize_jobs(raw_jobs)

        # Deduplicate
        deduplicated = self.deduplicate_jobs(normalized)

        # Load
        self.load_normalized_jobs(deduplicated)

        return {
            "raw_count": len(raw_jobs),
            "normalized_count": len(deduplicated),
        }
```

2. Create tests `tests/test_normalization_etl.py`:

```python
"""
Tests for Normalization ETL
"""
from unittest.mock import Mock, patch, MagicMock
from datetime import datetime

import pytest
from pymongo import MongoClient

from src.job_scraper.etl.normalization_etl import NormalizationETL


@pytest.fixture
def mongo_client():
    """Mock MongoDB client"""
    client = MagicMock(spec=MongoClient)
    client.job_scraper = MagicMock()
    client.normalized_jobs = MagicMock()
    return client


@pytest.fixture
def glm_client():
    """Mock GLM client"""
    client = Mock()
    client.chat_json = Mock(return_value={
        "jobs": [
            {
                "index": 0,
                "job_title": "DevOps Engineer",
                "company_name": "Test Company",
                "job_description": "We are looking for...",
                "job_location": "Canberra, ACT",
                "employment_type": "full_time",
                "work_arrangement": "remote",
                "posted_date": "2026-02-10",
            }
        ]
    })
    return client


@pytest.fixture
def sample_jobs():
    return [
        {
            "job_posting_id": "123",
            "job_title": "DevOps Engineer",
            "company_name": "Test Company",
            "job_description": "We are looking for a DevOps Engineer...",
            "job_location": "Canberra, ACT",
            "apply_link": "https://example.com/job/123",
            "created_at": datetime.now(),
        }
    ]


class TestNormalizationETL:
    """Test NormalizationETL class"""

    def test_init(self, mongo_client, glm_client):
        etl = NormalizationETL(mongo_client, glm_client)
        assert etl.mongo == mongo_client
        assert etl.glm == glm_client
        assert etl.batch_size == 8

    def test_extract_today_jobs(self, mongo_client, glm_client, sample_jobs):
        mock_collection = MagicMock()
        mock_collection.find.return_value = sample_jobs
        mongo_client.job_scraper.linkedin_jobs = mock_collection
        mongo_client.job_scraper.indeed_jobs = MagicMock()
        mongo_client.job_scraper.seek_jobs = MagicMock()

        etl = NormalizationETL(mongo_client, glm_client)
        jobs = etl.extract_today_jobs()

        assert len(jobs) == 1
        assert jobs[0]["_platform"] == "linkedin"

    def test_normalize_jobs(self, mongo_client, glm_client, sample_jobs):
        etl = NormalizationETL(mongo_client, glm_client)
        normalized = etl.normalize_jobs(sample_jobs)

        assert len(normalized) == 1
        assert normalized[0]["job_title"] == "DevOps Engineer"
        assert normalized[0]["company_name"] == "Test Company"
        assert "sources" in normalized[0]

    def test_deduplicate_jobs(self, mongo_client, glm_client):
        etl = NormalizationETL(mongo_client, glm_client)
        jobs = [
            {
                "job_id": "1",
                "company_name": "Test Company",
                "job_title": "DevOps Engineer",
                "job_location": "Canberra, ACT",
                "sources": [{"platform": "linkedin"}],
            },
            {
                "job_id": "2",
                "company_name": "Test Company",
                "job_title": "DevOps Engineer",
                "job_location": "Canberra, ACT",
                "sources": [{"platform": "indeed"}],
            },
            {
                "job_id": "3",
                "company_name": "Different Company",
                "job_title": "SRE",
                "job_location": "Sydney, NSW",
                "sources": [{"platform": "seek"}],
            },
        ]
        deduplicated = etl.deduplicate_jobs(jobs)

        assert len(deduplicated) == 2
        assert len(deduplicated[0]["sources"]) == 2  # Merged sources

    def test_create_dedup_key(self, mongo_client, glm_client):
        etl = NormalizationETL(mongo_client, glm_client)
        job = {
            "company_name": "Test-Company Pty Ltd",
            "job_title": "Senior DevOps Engineer",
            "job_location": "Canberra, ACT",
        }
        key = etl._create_dedup_key(job)

        assert "::" in key
        assert "testcompanyptyltd" in key
```

**Acceptance Criteria**:
- [ ] NormalizationETL class created
- [ ] Extracts today's jobs from 3 platform collections
- [ ] Normalizes using GLM-4 in batches of 8
- [ ] Deduplicates by company+title+location
- [ ] Loads to `normalized_jobs.YYYY_MM_DD`
- [ ] Tests pass

---

### TASK-3: Delete Old Ollama Code

**Description**: Remove Ollama-related code

**Detailed Steps**:

```bash
cd modules/brightdata
git rm src/job_scraper/llm/ollama_client.py
git rm src/job_scraper/etl/unified_etl.py
```

Remove any imports in other files:
```bash
grep -r "ollama" src/
grep -r "unified_etl" src/
```

**Acceptance Criteria**:
- [ ] `ollama_client.py` deleted
- [ ] `unified_etl.py` deleted
- [ ] No broken imports

---

## Git Workflow

### Commits
```bash
git add src/job_scraper/llm/prompts.py tests/test_prompts.py
git commit -m "feat(llm): add normalization and JD analysis prompts"

git add src/job_scraper/etl/normalization_etl.py tests/test_normalization_etl.py
git commit -m "feat(etl): add GLM-4 normalization pipeline"

git rm src/job_scraper/llm/ollama_client.py src/job_scraper/etl/unified_etl.py
git commit -m "refactor: remove Ollama and old unified ETL"
```

---

## Completion Checklist
- [ ] Prompts module created
- [ ] NormalizationETL implemented
- [ ] Tests pass
- [ ] Old code removed
- [ ] Commits pushed

---
**Version**: 1.0
**Created**: 2026-02-10
