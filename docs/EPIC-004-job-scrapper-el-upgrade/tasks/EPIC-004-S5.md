# EPIC-004-S5: Testing & Documentation

## Overview
**Epic**: EPIC-004
**Repository**: `modules/brightdata`
**Feature Branch**: `feature/epic-004-job-scraper-elt-upgrade`
**Agent**: Backend

## Context
This story completes the epic with comprehensive testing and documentation updates.

Reference:
- PRD: Section 3.2 (Code Quality)
- Solution Architecture: Section 7 (Future Enhancements)

## Tasks

### TASK-1: Write E2E Test

**Description**: Create end-to-end test for full pipeline

**Detailed Steps**:

1. Create `tests/test_e2e_pipeline.py`:

```python
"""
End-to-End Pipeline Test
Tests the complete pipeline with mocked external dependencies
"""
from unittest.mock import Mock, MagicMock, patch
from datetime import datetime
import pytest
from pymongo import MongoClient

from src.job_scraper.orchestrator import PipelineOrchestrator


@pytest.fixture
def mock_mongo_client():
    """Mock MongoDB client with all databases"""
    client = MagicMock(spec=MongoClient)

    # job_scraper database
    client.job_scraper = MagicMock()
    client.job_scraper.linkedin_jobs = MagicMock()
    client.job_scraper.indeed_jobs = MagicMock()
    client.job_scraper.seek_jobs = MagicMock()

    # normalized_jobs database
    client.normalized_jobs = MagicMock()
    client.normalized_jobs.__getitem__ = Mock(return_value=MagicMock())

    # filtered_jobs database
    client.filtered_jobs = MagicMock()
    client.filtered_jobs.__getitem__ = Mock(return_value=MagicMock())

    # job_statistics database
    client.job_statistics = MagicMock()
    client.job_statistics.daily_stats = MagicMock()

    # Client property
    client.client = client

    return client


@pytest.fixture
def mock_glm_client():
    """Mock GLM client"""
    client = Mock()
    client.chat_json = Mock(side_effect=[
        # Normalization response
        {
            "jobs": [
                {
                    "index": 0,
                    "job_title": "DevOps Engineer",
                    "company_name": "Test Company",
                    "job_description": "Job description here",
                    "job_location": "Canberra, ACT",
                    "employment_type": "full_time",
                    "work_arrangement": "remote",
                    "posted_date": "2026-02-10",
                }
            ]
        },
        # Skill match response
        {
            "visa_compatible": True,
            "security_clearance_required": False,
            "skill_match_score": 85,
            "matched_skills": ["Docker", "Kubernetes"],
            "missing_skills": [],
            "experience_match": True,
            "reason": "Good match",
        }
    ])
    client.close = Mock()
    return client


@pytest.fixture
def sample_raw_jobs():
    """Sample raw jobs from scraping"""
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


@pytest.mark.e2e
def test_complete_pipeline(
    mock_mongo_client,
    mock_glm_client,
    sample_raw_jobs,
):
    """Test the complete pipeline end-to-end"""
    # Mock scrapers
    with patch("src.job_scraper.orchestrator.LinkedInScraper") as mock_linkedin, \
         patch("src.job_scraper.orchestrator.IndeedScraper") as mock_indeed, \
         patch("src.job_scraper.orchestrator.SeekScraper") as mock_seek, \
         patch("src.job_scraper.orchestrator.get_mongo_client", return_value=mock_mongo_client), \
         patch("src.job_scraper.orchestrator.GLMClient", return_value=mock_glm_client), \
         patch("src.job_scraper.config.Config.linkedin_enabled", True), \
         patch("src.job_scraper.config.Config.indeed_enabled", True), \
         patch("src.job_scraper.config.Config.seek_enabled", True):

        # Mock scraper returns
        mock_linkedin.return_value.run.return_value = {"count": 1}
        mock_indeed.return_value.run.return_value = {"count": 1}
        mock_seek.return_value.run.return_value = {"count": 1}

        # Mock extract_today_jobs
        mock_mongo_client.job_scraper.linkedin_jobs.find.return_value = sample_raw_jobs
        mock_mongo_client.job_scraper.indeed_jobs.find.return_value = []
        mock_mongo_client.job_scraper.seek_jobs.find.return_value = []

        # Mock collection operations
        mock_collection = MagicMock()
        mock_collection.bulk_write.return_value = MagicMock(upserted_count=1, modified_count=0)
        mock_mongo_client.normalized_jobs.__getitem__.return_value = mock_collection
        mock_mongo_client.filtered_jobs.__getitem__.return_value = mock_collection

        # Mock find returns normalized jobs
        mock_collection.find.return_value = [
            {
                "job_id": "123",
                "job_title": "DevOps Engineer",
                "job_description": "Job description here",
            }
        ]

        # Run pipeline
        from src.job_scraper.orchestrator import PipelineOrchestrator
        orchestrator = PipelineOrchestrator()
        result = orchestrator.run_pipeline()

        # Verify results
        assert "date" in result
        assert "stages" in result
        assert "scraping" in result["stages"]
        assert "normalization" in result["stages"]
        assert "jd_analysis" in result["stages"]
        assert "statistics" in result["stages"]


@pytest.mark.e2e
def test_pipeline_handles_graceful_degradation(
    mock_mongo_client,
    mock_glm_client,
):
    """Test pipeline handles failures gracefully"""
    with patch("src.job_scraper.orchestrator.get_mongo_client", return_value=mock_mongo_client), \
         patch("src.job_scraper.orchestrator.GLMClient", return_value=mock_glm_client):

        # Mock no jobs found
        mock_mongo_client.job_scraper.linkedin_jobs.find.return_value = []
        mock_mongo_client.job_scraper.indeed_jobs.find.return_value = []
        mock_mongo_client.job_scraper.seek_jobs.find.return_value = []

        from src.job_scraper.orchestrator import PipelineOrchestrator
        orchestrator = PipelineOrchestrator()
        result = orchestrator.run_pipeline()

        # Should complete without errors
        assert "stages" in result
        # Normalization should report 0 jobs
        assert result["stages"]["normalization"]["normalized_count"] == 0
```

**Acceptance Criteria**:
- [ ] E2E test runs successfully
- [ ] All stages validated
- [ ] Graceful degradation tested

---

### TASK-2: Update Documentation

**Description**: Update project documentation

**Detailed Steps**:

1. Update `README.md`:

```markdown
# Job Scraper with GLM-4 Analysis

Automated job scraping and analysis pipeline for DevOps/SRE roles.

## Features

- **Multi-platform Scraping**: LinkedIn, Indeed, Seek
- **LLM Normalization**: GLM-4 based job data normalization
- **Smart Filtering**:
  - Visa/PR requirement detection
  - Security clearance requirement detection
  - Skill/experience matching
- **Statistics Tracking**: Daily metrics on pipeline performance

## Architecture

```
Scraping → Normalization (GLM-4) → JD Analysis → Statistics
```

## Setup

### Prerequisites

- Python 3.12+
- MongoDB 7.0+
- Zhipu AI GLM-4 API key

### Installation

```bash
cd modules/brightdata
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Configuration

1. Copy `.env.example` to `.env`
2. Set your API keys:

```bash
BRIGHTDATA_API_TOKEN=your_token
GLM_API_KEY=your_glm_key
MONGODB_URI=mongodb://localhost:27017
```

3. Configure search parameters in `config/*.yaml` files

## Usage

### Run Once

```bash
python scripts/run_pipeline.py
```

### Scheduled Mode (Daily at 1:00 PM)

```bash
python scripts/run_pipeline.py --cron
```

## Databases

- `job_scraper`: Raw platform data (indeed_jobs, seek_jobs, linkedin_jobs)
- `normalized_jobs`: Normalized jobs (collections: YYYY_MM_DD)
- `filtered_jobs`: Filtered jobs matching criteria (collections: YYYY_MM_DD)
- `job_statistics`: Daily pipeline statistics (collection: daily_stats)

## Development

### Run Tests

```bash
pytest tests/ -v
```

### Test Coverage

```bash
pytest tests/ --cov=src/job_scraper --cov-report=html
```

## License

MIT
```

**Acceptance Criteria**:
- [ ] README reflects new architecture
- [ ] GLM-4 setup documented
- [ ] Database structure documented

---

### TASK-3: Code Review

**Description**: Run pragmatic code review

**Detailed Steps**:

1. Run tests with coverage:
```bash
cd modules/brightdata
pytest tests/ --cov=src/job_scraper --cov-report=term-missing
```

2. Check for any linting issues:
```bash
ruff check src/
```

3. Verify coverage > 80%

**Acceptance Criteria**:
- [ ] All tests passing
- [ ] Coverage > 80%
- [ ] No critical linting issues

---

## Git Workflow

### Commits
```bash
git add tests/test_e2e_pipeline.py
git commit -m "test(e2e): add end-to-end pipeline test"

git add README.md
git commit -m "docs: update for GLM-4 architecture"

# After code review
git add -u
git commit -m "refactor: address code review feedback"
```

---

## Completion Checklist
- [ ] E2E test passing
- [ ] Documentation updated
- [ ] Test coverage > 80%
- [ ] All commits pushed
- [ ] Ready for PR

---
**Version**: 1.0
**Created**: 2026-02-10
