# EPIC-004-S4: Statistics & Orchestrator

## Overview
**Epic**: EPIC-004
**Repository**: `modules/brightdata`
**Feature Branch**: `feature/epic-004-job-scraper-elt-upgrade`
**Agent**: Backend

## Context
This story implements statistics collection and updates the orchestrator to run the complete pipeline.

Reference:
- PRD: Section 2.1 (F5: Statistics Tracking)
- Solution Architecture: Section 2.1 (Statistics Collector)

## Tasks

### TASK-1: Create Statistics Module

**Description**: Implement statistics collection and storage

**Detailed Steps**:

1. Create `src/job_scraper/etl/statistics.py`:

```python
"""
Statistics Collection for Pipeline Tracking
"""
import logging
from datetime import datetime

from pymongo import MongoClient, UpdateOne

logger = logging.getLogger(__name__)


class StatisticsCollector:
    """Collect and store pipeline statistics"""

    def __init__(self, mongo_client: MongoClient):
        """
        Initialize StatisticsCollector.

        Args:
            mongo_client: MongoDB client
        """
        self.mongo = mongo_client

    def collect_raw_counts(self) -> dict:
        """
        Count raw jobs by platform for today.

        Returns:
            Dict with counts by platform
        """
        today = datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)
        tomorrow = today.replace(hour=23, minute=59, second=59)

        counts = {
            "indeed": 0,
            "seek": 0,
            "linkedin": 0,
            "total": 0,
        }

        platforms = {
            "linkedin": "linkedin_jobs",
            "indeed": "indeed_jobs",
            "seek": "seek_jobs",
        }

        for platform, collection_name in platforms.items():
            collection = self.mongo.job_scraper[collection_name]
            count = collection.count_documents({
                "created_at": {"$gte": today, "$lte": tomorrow}
            })
            counts[platform] = count
            counts["total"] += count

        logger.info(f"Raw job counts: {counts}")
        return counts

    def collect_filter_stats(self, filter_results: dict) -> dict:
        """
        Aggregate filter results from JDAnalyzer.

        Args:
            filter_results: Dict returned by JDAnalyzer.run()

        Returns:
            Same dict (already aggregated)
        """
        return filter_results

    def collect_skill_scores(self, filtered_jobs: list) -> dict:
        """
        Calculate skill score statistics.

        Args:
            filtered_jobs: List of passed jobs with analysis_result

        Returns:
            Dict with min, max, avg scores
        """
        scores = [
            job["analysis_result"]["skill_match_score"]
            for job in filtered_jobs
            if job.get("analysis_result", {}).get("skill_match_score")
        ]

        if not scores:
            return {"min": 0, "max": 0, "avg": 0}

        return {
            "min": min(scores),
            "max": max(scores),
            "avg": round(sum(scores) / len(scores), 1),
        }

    def save_daily_stats(self, date: str, stats: dict):
        """
        Save daily statistics to job_statistics.daily_stats.

        Args:
            date: Date string (YYYY_MM_DD)
            stats: Statistics dict
        """
        collection = self.mongo.job_statistics.daily_stats

        document = {
            "date": date,
            "scraped_at": datetime.utcnow(),
            **stats,
        }

        collection.update_one(
            {"date": date},
            {"$set": document},
            upsert=True
        )

        logger.info(f"Saved statistics for {date}")

    def run(
        self,
        date: str,
        raw_counts: dict,
        normalized_count: int,
        filter_results: dict,
        filtered_jobs: list,
    ) -> dict:
        """
        Run full statistics collection.

        Args:
            date: Date string (YYYY_MM_DD)
            raw_counts: Dict from collect_raw_counts()
            normalized_count: Number of normalized jobs
            filter_results: Dict from JDAnalyzer.run()
            filtered_jobs: List of passed jobs

        Returns:
            Complete statistics dict
        """
        logger.info("Collecting statistics")

        skill_scores = self.collect_skill_scores(filtered_jobs)

        stats = {
            "date": date,
            "scraped_at": datetime.utcnow().isoformat(),
            "pipeline_run_id": f"{date}_{int(datetime.now().timestamp())}",
            "raw_counts": raw_counts,
            "normalized_count": normalized_count,
            "filter_results": filter_results,
            "skill_match_scores": skill_scores,
        }

        self.save_daily_stats(date, stats)

        logger.info(
            f"Statistics: {raw_counts['total']} raw -> {normalized_count} normalized -> "
            f"{filter_results.get('passed', 0)} passed"
        )

        return stats
```

2. Create tests `tests/test_statistics.py`:

```python
"""
Tests for Statistics Collector
"""
from unittest.mock import MagicMock
from datetime import datetime
import pytest

from pymongo import MongoClient

from src.job_scraper.etl.statistics import StatisticsCollector


@pytest.fixture
def mongo_client():
    """Mock MongoDB client"""
    client = MagicMock(spec=MongoClient)
    client.job_scraper = MagicMock()
    client.job_statistics = MagicMock()
    return client


class TestStatisticsCollector:
    """Test StatisticsCollector class"""

    def test_init(self, mongo_client):
        collector = StatisticsCollector(mongo_client)
        assert collector.mongo == mongo_client

    def test_collect_raw_counts(self, mongo_client):
        mock_collection = MagicMock()
        mock_collection.count_documents.return_value = 10
        mongo_client.job_scraper.linkedin_jobs = mock_collection
        mongo_client.job_scraper.indeed_jobs = MagicMock()
        mongo_client.job_scraper.seek_jobs = MagicMock()

        collector = StatisticsCollector(mongo_client)
        counts = collector.collect_raw_counts()

        assert counts["linkedin"] == 10
        assert counts["total"] == 10

    def test_collect_skill_scores(self, mongo_client):
        collector = StatisticsCollector(mongo_client)
        jobs = [
            {"analysis_result": {"skill_match_score": 70}},
            {"analysis_result": {"skill_match_score": 80}},
            {"analysis_result": {"skill_match_score": 90}},
        ]
        scores = collector.collect_skill_scores(jobs)

        assert scores["min"] == 70
        assert scores["max"] == 90
        assert scores["avg"] == 80.0

    def test_collect_skill_scores_empty(self, mongo_client):
        collector = StatisticsCollector(mongo_client)
        scores = collector.collect_skill_scores([])

        assert scores["min"] == 0
        assert scores["max"] == 0
        assert scores["avg"] == 0

    def test_save_daily_stats(self, mongo_client):
        mock_collection = MagicMock()
        mongo_client.job_statistics.daily_stats = mock_collection

        collector = StatisticsCollector(mongo_client)
        collector.save_daily_stats("2026_02_10", {"test": "data"})

        mock_collection.update_one.assert_called_once()
```

**Acceptance Criteria**:
- [ ] StatisticsCollector class created
- [ ] Counts raw jobs by platform
- [ ] Aggregates filter results
- [ ] Calculates skill score statistics
- [ ] Saves to `job_statistics.daily_stats`
- [ ] Tests pass

---

### TASK-2: Update MongoDB Client

**Description**: Add support for multiple databases

**Detailed Steps**:

1. Update `src/job_scraper/db/mongo_client.py`:

```python
"""
MongoDB Client Singleton
"""
import os
from typing import Optional

from pymongo import MongoClient
from pymongo.database import Database

from src.job_scraper.config import Config


class MongoDBClient:
    """MongoDB client singleton"""

    _instance: Optional["MongoDBClient"] = None
    _client: Optional[MongoClient] = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        if self._client is None:
            self._client = MongoClient(Config.MONGODB_URI)

    @property
    def client(self) -> MongoClient:
        """Get raw MongoClient"""
        return self._client

    @property
    def job_scraper(self) -> Database:
        """Get job_scraper database"""
        return self._client[Config.MONGODB_DATABASE]

    @property
    def normalized_jobs(self) -> Database:
        """Get normalized_jobs database"""
        return self._client["normalized_jobs"]

    @property
    def filtered_jobs(self) -> Database:
        """Get filtered_jobs database"""
        return self._client["filtered_jobs"]

    @property
    def job_statistics(self) -> Database:
        """Get job_statistics database"""
        return self._client["job_statistics"]

    def close(self):
        """Close connection"""
        if self._client is not None:
            self._client.close()
            self._client = None

    def __getattr__(self, name):
        """Proxy to client for direct database access"""
        return getattr(self._client, name)


def get_mongo_client() -> MongoDBClient:
    """Get MongoDB client singleton instance"""
    return MongoDBClient()
```

**Acceptance Criteria**:
- [ ] Properties for normalized_jobs, filtered_jobs, job_statistics
- [ ] Tests pass

---

### TASK-3: Update Orchestrator

**Description**: Integrate new ETL pipelines into orchestrator

**Detailed Steps**:

1. Update `src/job_scraper/orchestrator.py`:

```python
"""
Pipeline Orchestrator
Coordinates scraping, normalization, JD analysis, and statistics
"""
import logging
from datetime import datetime
from typing import Optional

from src.job_scraper.config import Config, get_resume_path
from src.job_scraper.db.mongo_client import get_mongo_client
from src.job_scraper.llm.glm_client import GLMClient
from src.job_scraper.etl.normalization_etl import NormalizationETL
from src.job_scraper.etl.jd_analyzer import JDAnalyzer
from src.job_scraper.etl.statistics import StatisticsCollector

# Import scrapers
from src.job_scraper.scrapers.linkedin import LinkedInScraper
from src.job_scraper.scrapers.indeed import IndeedScraper
from src.job_scraper.scrapers.seek import SeekScraper

logger = logging.getLogger(__name__)


class PipelineOrchestrator:
    """Orchestrates the complete job scraping and analysis pipeline"""

    def __init__(self):
        """Initialize orchestrator with all components"""
        Config.validate()

        self.mongo = get_mongo_client()
        self.glm = GLMClient(**Config.get_glm_config())

        # Scrapers
        self.linkedin_scraper = LinkedInScraper(self.mongo)
        self.indeed_scraper = IndeedScraper(self.mongo)
        self.seek_scraper = SeekScraper(self.mongo)

        # ETL Pipelines
        self.normalization_etl = NormalizationETL(self.mongo.client, self.glm)
        self.jd_analyzer = JDAnalyzer(
            self.mongo.client,
            self.glm,
            resume_path=get_resume_path(),
            skill_threshold=60,
        )
        self.statistics = StatisticsCollector(self.mongo.client)

        logger.info("Pipeline Orchestrator initialized")

    def run_scraping(self) -> dict:
        """
        Run scraping for all enabled platforms.

        Returns:
            Dict with scrape results
        """
        logger.info("Starting scraping phase")
        results = {}

        if Config.linkedin_enabled:
            try:
                results["linkedin"] = self.linkedin_scraper.run()
                logger.info(f"LinkedIn scraped: {results['linkedin']} jobs")
            except Exception as e:
                logger.error(f"LinkedIn scraping failed: {e}")
                results["linkedin"] = {"error": str(e)}

        if Config.indeed_enabled:
            try:
                results["indeed"] = self.indeed_scraper.run()
                logger.info(f"Indeed scraped: {results['indeed']} jobs")
            except Exception as e:
                logger.error(f"Indeed scraping failed: {e}")
                results["indeed"] = {"error": str(e)}

        if Config.seek_enabled:
            try:
                results["seek"] = self.seek_scraper.run()
                logger.info(f"Seek scraped: {results['seek']} jobs")
            except Exception as e:
                logger.error(f"Seek scraping failed: {e}")
                results["seek"] = {"error": str(e)}

        return results

    def run_pipeline(self) -> dict:
        """
        Run complete pipeline: Scraping -> Normalization -> JD Analysis -> Statistics

        Returns:
            Summary dict with all results
        """
        start_time = datetime.now()
        date_str = start_time.strftime("%Y_%m_%d")

        logger.info(f"Starting pipeline for {date_str}")

        summary = {
            "date": date_str,
            "started_at": start_time.isoformat(),
            "stages": {},
        }

        try:
            # Stage 1: Scraping
            logger.info("=" * 50)
            logger.info("Stage 1: Scraping")
            scrape_results = self.run_scraping()
            summary["stages"]["scraping"] = scrape_results

            # Stage 2: Normalization
            logger.info("=" * 50)
            logger.info("Stage 2: Normalization")
            norm_results = self.normalization_etl.run()
            summary["stages"]["normalization"] = norm_results

            if norm_results["normalized_count"] == 0:
                logger.warning("No normalized jobs, skipping analysis")
                return summary

            # Get normalized jobs for next stage
            normalized_jobs = list(self.mongo.normalized_jobs[date_str].find())

            # Stage 3: JD Analysis
            logger.info("=" * 50)
            logger.info("Stage 3: JD Analysis")
            filter_results = self.jd_analyzer.run(normalized_jobs)
            summary["stages"]["jd_analysis"] = filter_results

            # Get filtered jobs for statistics
            filtered_jobs = list(self.mongo.filtered_jobs[date_str].find())

            # Stage 4: Statistics
            logger.info("=" * 50)
            logger.info("Stage 4: Statistics")
            raw_counts = self.statistics.collect_raw_counts()

            stats = self.statistics.run(
                date_str=date_str,
                raw_counts=raw_counts,
                normalized_count=norm_results["normalized_count"],
                filter_results=filter_results,
                filtered_jobs=filtered_jobs,
            )
            summary["stages"]["statistics"] = stats

        except Exception as e:
            logger.error(f"Pipeline failed: {e}", exc_info=True)
            summary["error"] = str(e)

        finally:
            # Clean up
            self.glm.close()

            end_time = datetime.now()
            duration = (end_time - start_time).total_seconds()

            summary["completed_at"] = end_time.isoformat()
            summary["duration_seconds"] = duration

            logger.info("=" * 50)
            logger.info(f"Pipeline complete in {duration:.1f} seconds")
            logger.info(f"Summary: {summary}")

        return summary


def run_pipeline() -> dict:
    """
    Run the complete pipeline.

    Returns:
        Summary dict
    """
    orchestrator = PipelineOrchestrator()
    return orchestrator.run_pipeline()
```

**Acceptance Criteria**:
- [ ] Orchestrator imports all new ETL modules
- [ ] `run_pipeline()` calls all stages in order
- [ ] Returns summary with all stage results
- [ ] Error handling for each stage

---

### TASK-4: Update Scheduler Script

**Description**: Update cron schedule to 13:00

**Detailed Steps**:

1. Update `scripts/run_pipeline.py`:

```python
#!/usr/bin/env python
"""
Pipeline Entry Point
Run the job scraping and analysis pipeline
"""
import logging
import sys
from datetime import datetime

from croniter import croniter

from src.job_scraper.orchestrator import run_pipeline

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
)
logger = logging.getLogger(__name__)


def main(scheduled: bool = False):
    """
    Main entry point.

    Args:
        scheduled: If True, run in scheduled mode (cron)
    """
    if scheduled:
        logger.info("Running in scheduled mode")

        # Cron expression: 1:00 PM daily
        cron_expr = "0 13 * * *"
        cron = croniter(cron_expr, datetime.now())

        while True:
            next_run = cron.get_next(datetime)
            now = datetime.now()

            if now >= next_run:
                logger.info(f"Running scheduled pipeline at {now}")
                try:
                    run_pipeline()
                except Exception as e:
                    logger.error(f"Pipeline failed: {e}")

                # Get next occurrence
                cron = croniter(cron_expr, datetime.now())

            # Sleep until next check
            sleep_time = min(60, (next_run - now).total_seconds())
            import time
            time.sleep(sleep_time)
    else:
        # Run once
        logger.info("Running pipeline once")
        result = run_pipeline()
        return result


if __name__ == "__main__":
    scheduled = "--cron" in sys.argv or "-c" in sys.argv
    main(scheduled=scheduled)
```

**Acceptance Criteria**:
- [ ] Schedule set to "0 13 * * *" (1:00 PM)
- [ ] Uses croniter for next run calculation
- [ ] --cron flag enables scheduled mode

---

## Git Workflow

### Commits
```bash
git add src/job_scraper/etl/statistics.py tests/test_statistics.py
git commit -m "feat(etl): add statistics collector"

git add src/job_scraper/db/mongo_client.py
git commit -m "feat(db): add multi-database support"

git add src/job_scraper/orchestrator.py
git commit -m "feat(orchestrator): integrate new ETL pipelines"

git add scripts/run_pipeline.py
git commit -m "feat(scheduler): update to 1:00 PM schedule with croniter"
```

---

## Completion Checklist
- [ ] StatisticsCollector implemented
- [ ] MongoDB client updated with new databases
- [ ] Orchestrator runs complete pipeline
- [ ] Scheduler updated to 13:00
- [ ] Tests pass
- [ ] Commits pushed

---
**Version**: 1.0
**Created**: 2026-02-10
