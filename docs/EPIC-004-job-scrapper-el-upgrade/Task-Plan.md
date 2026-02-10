# Task Plan - Job Scraper ELT Upgrade

## Epic Overview

| Epic ID | Epic Name | Description | Priority |
|---------|-----------|-------------|----------|
| EPIC-004 | Job Scraper ELT Upgrade | Replace Ollama with GLM-4, add JD analysis and skill matching | P0 |

## Stories Overview

| Story ID | Story Name | Agent | Repo | Branch |
|----------|------------|-------|------|--------|
| EPIC-004-S1 | GLM Client & Config | Backend | modules/brightdata | feature/epic-004-job-scraper-elt-upgrade |
| EPIC-004-S2 | Normalization ETL | Backend | modules/brightdata | feature/epic-004-job-scraper-elt-upgrade |
| EPIC-004-S3 | JD Analysis | Backend | modules/brightdata | feature/epic-004-job-scraper-elt-upgrade |
| EPIC-004-S4 | Statistics & Orchestrator | Backend | modules/brightdata | feature/epic-004-job-scraper-elt-upgrade |
| EPIC-004-S5 | Testing & Documentation | Backend | modules/brightdata | feature/epic-004-job-scraper-elt-upgrade |

---

## EPIC-004-S1: GLM Client & Configuration

**Repository**: `modules/brightdata`
**Feature Branch**: `feature/epic-004-job-scraper-elt-upgrade`
**Agent Type**: Backend
**Required Skills**: `test-driven-development`, `code-quality-principles`

### Success Criteria
- [ ] GLM-4 client successfully connects to Zhipu AI API
- [ ] Chat and JSON modes working
- [ ] Resume data loaded from `config/resume.json`
- [ ] All config files updated with new keywords/locations
- [ ] Environment variables configured for GLM-4

### Execution Order
EPIC-004-S1-TASK-1 → EPIC-004-S1-TASK-2 → EPIC-004-S1-TASK-3 → EPIC-004-S1-TASK-4

---

### EPIC-004-S1-TASK-1: Create GLM Client

**Tags**: `#backend #llm #api`

**Description**: Implement GLM-4 API client to replace Ollama

**Steps**:
1. Create `src/job_scraper/llm/glm_client.py` with GLMClient class
2. Implement JWT token generation (Zhipu AI uses JWT)
3. Implement `chat()` method for text responses
4. Implement `chat_json()` method for JSON responses
5. Add retry logic with exponential backoff
6. Write unit tests in `tests/test_glm_client.py`

**Acceptance Criteria**:
- [ ] GLMClient class with `__init__(api_key, model)`
- [ ] `chat()` method returns string response
- [ ] `chat_json()` method returns parsed dict
- [ ] Retry logic (3 attempts, 2s delay)
- [ ] Timeout configured (120s default)
- [ ] Tests pass with mocked API responses

**Git Workflow**:
```bash
cd modules/brightdata
git checkout -b feature/epic-004-job-scraper-elt-upgrade
# Implement GLMClient
pytest tests/test_glm_client.py -v
git add src/job_scraper/llm/glm_client.py tests/test_glm_client.py
git commit -m "feat(llm): add GLM-4 API client"
```

**Estimated Time**: 45 minutes

---

### EPIC-004-S1-TASK-2: Update Environment Configuration

**Tags**: `#backend #config`

**Description**: Update .env files for GLM-4 configuration

**Steps**:
1. Update `.env.example` with GLM configuration
2. Create `.env` with actual API key
3. Update `src/job_scraper/config.py` to load GLM variables

**Acceptance Criteria**:
- [ ] `.env.example` has GLM_API_KEY, GLM_MODEL, GLM_BASE_URL, GLM_TIMEOUT
- [ ] `.env` created with actual API key
- [ ] `config.py` loads GLM_CONFIG dict
- [ ] OLLAMA_* variables removed

**Git Workflow**:
```bash
git add .env.example .env src/job_scraper/config.py
git commit -m "feat(config): add GLM-4 environment variables"
```

**Estimated Time**: 15 minutes

---

### EPIC-004-S1-TASK-3: Create Resume Configuration

**Tags**: `#backend #config`

**Description**: Create structured resume.json from PDF data

**Steps**:
1. Create `config/resume.json` with Eric's profile
2. Include: skills, experience, certifications, exclusion criteria
3. Update `.gitignore` to exclude resume.json
4. Create resume loader utility

**Acceptance Criteria**:
- [ ] `config/resume.json` with complete profile data
- [ ] Loader function in `src/job_scraper/config.py`
- [ ] `.gitignore` excludes `config/resume.json`

**Git Workflow**:
```bash
git add config/resume.json src/job_scraper/config.py .gitignore
git commit -m "feat(config): add resume profile data"
```

**Estimated Time**: 20 minutes

---

### EPIC-004-S1-TASK-4: Update Search Configurations

**Tags**: `#backend #config`

**Description**: Update YAML configs with new keywords and locations

**Steps**:
1. Update `config/linkedin_search.yaml` with new keywords/locations
2. Update `config/indeed_search.yaml` with new keywords/locations
3. Update `config/seek_search.yaml` with new keywords/locations
4. Update `config/pipeline.yaml` with 13:00 schedule

**Acceptance Criteria**:
- [ ] All configs have: DevOps Engineer, SRE, Cloud Engineer, Platform Engineer, Senior DevOps
- [ ] All configs have: Remote, Canberra, Brisbane, Sydney + regional areas
- [ ] Time filters set to 24 hours
- [ ] Schedule set to "0 13 * * *"

**Git Workflow**:
```bash
git add config/
git commit -m "feat(config): update search parameters and schedule"
```

**Estimated Time**: 20 minutes

---

## EPIC-004-S2: Normalization ETL

**Repository**: `modules/brightdata`
**Feature Branch**: `feature/epic-004-job-scraper-elt-upgrade`
**Agent Type**: Backend
**Required Skills**: `test-driven-development`, `code-quality-principles`

### Success Criteria
- [ ] Normalization pipeline extracts today's jobs from raw collections
- [ ] GLM-4 normalizes jobs with correct schema
- [ ] Deduplication works correctly
- [ ] Data loaded to `normalized_jobs` database

### Execution Order
EPIC-004-S2-TASK-1 → EPIC-004-S2-TASK-2 → EPIC-004-S2-TASK-3 → EPIC-004-S2-TASK-4

---

### EPIC-004-S2-TASK-1: Create Normalization ETL Module

**Tags**: `#backend #etl #llm`

**Description**: Implement normalization using GLM-4

**Steps**:
1. Create `src/job_scraper/etl/normalization_etl.py`
2. Implement `NormalizationETL` class
3. Implement `extract_today_jobs()` - query by `created_at` date
4. Implement `normalize_jobs()` - batch processing with GLM-4
5. Implement `deduplicate_jobs()` - by company+title+location
6. Implement `load_normalized_jobs()` - to `normalized_jobs.YYYY_MM_DD`

**Acceptance Criteria**:
- [ ] Class extracts jobs from all 3 platform collections
- [ ] Batch size 8 for GLM-4 calls
- [ ] Normalized schema: job_id, job_title, company_name, job_description, job_location, employment_type, work_arrangement, posted_date, sources
- [ ] Deduplication key: "company::title::city::state"
- [ ] Creates date-named collection in `normalized_jobs` DB

**Git Workflow**:
```bash
# Write tests first in tests/test_normalization_etl.py
# Implement NormalizationETL
pytest tests/test_normalization_etl.py -v
git add src/job_scraper/etl/normalization_etl.py tests/test_normalization_etl.py
git commit -m "feat(etl): add GLM-4 normalization pipeline"
```

**Estimated Time**: 60 minutes

---

### EPIC-004-S2-TASK-2: Create GLM Normalization Prompts

**Tags**: `#backend #llm #prompts`

**Description**: Design prompts for job normalization

**Steps**:
1. Create `src/job_scraper/llm/prompts.py`
2. Define `NORMALIZATION_SYSTEM_PROMPT`
3. Define `NORMALIZATION_USER_PROMPT_TEMPLATE`
4. Add validation for expected fields

**Acceptance Criteria**:
- [ ] System prompt defines role and expected output format
- [ ] User prompt template accepts job list
- [ ] Output includes: job_id, job_title, company_name, job_description, job_location, employment_type, work_arrangement, posted_date
- [ ] Enum values for employment_type and work_arrangement

**Git Workflow**:
```bash
git add src/job_scraper/llm/prompts.py
git commit -m "feat(llm): add normalization prompts"
```

**Estimated Time**: 20 minutes

---

### EPIC-004-S2-TASK-3: Delete Old Ollama Code

**Tags**: `#backend #cleanup`

**Description**: Remove Ollama-related code

**Steps**:
1. Delete `src/job_scraper/llm/ollama_client.py`
2. Delete `src/job_scraper/etl/unified_etl.py`
3. Update imports in other files
4. Remove OLLAMA_* from config

**Acceptance Criteria**:
- [ ] `ollama_client.py` deleted
- [ ] `unified_etl.py` deleted
- [ ] No broken imports
- [ ] Tests still pass

**Git Workflow**:
```bash
git rm src/job_scraper/llm/ollama_client.py
git rm src/job_scraper/etl/unified_etl.py
# Update imports
pytest tests/ -v
git add -u
git commit -m "refactor: remove Ollama and old unified ETL"
```

**Estimated Time**: 15 minutes

---

### EPIC-004-S2-TASK-4: Update Platform ETLs

**Tags**: `#backend #etl`

**Description**: Update platform ETLs to preserve today's jobs with created_at timestamp

**Steps**:
1. Review `src/job_scraper/etl/linkedin_etl.py`
2. Review `src/job_scraper/etl/indeed_etl.py`
3. Review `src/job_scraper/etl/seek_etl.py`
4. Ensure `created_at` and `scraped_at` are set to current time

**Acceptance Criteria**:
- [ ] All ETLs set `created_at` to `datetime.utcnow()`
- [ ] All ETLs set `scraped_at` to `datetime.utcnow()`
- [ ] Existing tests pass

**Git Workflow**:
```bash
pytest tests/test_*_etl.py -v
git add src/job_scraper/etl/
git commit -m "fix(etl): ensure created_at timestamp is set"
```

**Estimated Time**: 15 minutes

---

## EPIC-004-S3: JD Analysis

**Repository**: `modules/brightdata`
**Feature Branch**: `feature/epic-004-job-scraper-elt-upgrade`
**Agent Type**: Backend
**Required Skills**: `test-driven-development`, `code-quality-principles`

### Success Criteria
- [ ] Visa/PR filter correctly excludes citizenship requirements
- [ ] Security clearance filter correctly identifies clearance requirements
- [ ] Skill matching returns scores 0-100
- [ ] Filtered jobs saved to `filtered_jobs` database

### Execution Order
EPIC-004-S3-TASK-1 → EPIC-004-S3-TASK-2 → EPIC-004-S3-TASK-3 → EPIC-004-S3-TASK-4

---

### EPIC-004-S3-TASK-1: Create JD Analyzer Module

**Tags**: `#backend #analysis #llm`

**Description**: Implement JD analysis with three-stage filtering

**Steps**:
1. Create `src/job_scraper/etl/jd_analyzer.py`
2. Implement `JDAnalyzer` class
3. Implement `check_visa_requirements()` - rule-based keyword matching
4. Implement `check_security_clearance()` - rule-based keyword matching
5. Implement `match_skills()` - GLM-4 based matching
6. Implement `analyze_job()` - combine all checks
7. Implement `filter_jobs()` - return (passed, failed)
8. Implement `load_filtered_jobs()` - to `filtered_jobs.YYYY_MM_DD`

**Acceptance Criteria**:
- [ ] Visa filter checks for: Citizenship, PR, Permanent Resident, Australian Citizen
- [ ] Security filter checks for: NV1, NV2, Baseline, Security clearance, Defence clearance
- [ ] Skill matching uses GLM-4 with resume profile
- [ ] Returns score 0-100, matched/missing skills, experience match, reason
- [ ] Filter threshold: 60 points minimum
- [ ] Creates date-named collection in `filtered_jobs` DB

**Git Workflow**:
```bash
# Write tests first in tests/test_jd_analyzer.py
# Implement JDAnalyzer
pytest tests/test_jd_analyzer.py -v
git add src/job_scraper/etl/jd_analyzer.py tests/test_jd_analyzer.py
git commit -m "feat(etl): add JD analysis pipeline"
```

**Estimated Time**: 75 minutes

---

### EPIC-004-S3-TASK-2: Create JD Analysis Prompts

**Tags**: `#backend #llm #prompts`

**Description**: Design prompts for skill matching

**Steps**:
1. Add to `src/job_scraper/llm/prompts.py`
2. Define `JD_ANALYSIS_SYSTEM_PROMPT`
3. Define `JD_ANALYSIS_USER_PROMPT_TEMPLATE`

**Acceptance Criteria**:
- [ ] System prompt defines role as expert job matcher
- [ ] User prompt template accepts JD and resume profile
- [ ] Output includes: visa_compatible, security_clearance_required, skill_match_score, matched_skills, missing_skills, experience_match, reason

**Git Workflow**:
```bash
git add src/job_scraper/llm/prompts.py
git commit -m "feat(llm): add JD analysis prompts"
```

**Estimated Time**: 20 minutes

---

### EPIC-004-S3-TASK-3: Create Exclusion Criteria Config

**Tags**: `#backend #config`

**Description**: Centralize exclusion criteria keywords

**Steps**:
1. Add exclusion criteria to `config/resume.json`
2. Create utility to load exclusion criteria

**Acceptance Criteria**:
- [ ] `exclusion_criteria.visa_requirements` array
- [ ] `exclusion_criteria.security_clearance` array
- [ ] JDAnalyzer loads from resume config

**Git Workflow**:
```bash
git add config/resume.json src/job_scraper/etl/jd_analyzer.py
git commit -m "feat(config): add exclusion criteria"
```

**Estimated Time**: 10 minutes

---

### EPIC-004-S3-TASK-4: Implement Filter Statistics

**Tags**: `#backend #statistics`

**Description**: Track filter results for statistics

**Steps**:
1. Add statistics tracking to `JDAnalyzer`
2. Count: visa_incompatible, security_clearance_required, skill_mismatch, passed
3. Return stats from `run()` method

**Acceptance Criteria**:
- [ ] `filter_jobs()` returns (passed, failed, stats)
- [ ] Stats dict includes counts for each filter category

**Git Workflow**:
```bash
pytest tests/test_jd_analyzer.py -v
git add src/job_scraper/etl/jd_analyzer.py tests/test_jd_analyzer.py
git commit -m "feat(etl): add filter statistics tracking"
```

**Estimated Time**: 15 minutes

---

## EPIC-004-S4: Statistics & Orchestrator

**Repository**: `modules/brightdata`
**Feature Branch**: `feature/epic-004-job-scraper-elt-upgrade`
**Agent Type**: Backend
**Required Skills**: `test-driven-development`, `code-quality-principles`

### Success Criteria
- [ ] Statistics module collects all pipeline metrics
- [ ] Orchestrator runs complete pipeline
- [ ] Schedule updated to 13:00 daily

### Execution Order
EPIC-004-S4-TASK-1 → EPIC-004-S4-TASK-2 → EPIC-004-S4-TASK-3 → EPIC-004-S4-TASK-4

---

### EPIC-004-S4-TASK-1: Create Statistics Module

**Tags**: `#backend #statistics`

**Description**: Implement statistics collection and storage

**Steps**:
1. Create `src/job_scraper/etl/statistics.py`
2. Implement `StatisticsCollector` class
3. Implement `collect_raw_counts()` - count by platform
4. Implement `collect_filter_stats()` - aggregate from analyzer
5. Implement `collect_skill_scores()` - min, max, avg
6. Implement `save_daily_stats()` - to `job_statistics.daily_stats`
7. Implement `run()` - full collection

**Acceptance Criteria**:
- [ ] Counts raw jobs by platform for today
- [ ] Aggregates filter results
- [ ] Calculates score statistics
- [ ] Upserts to `job_statistics.daily_stats` with date as unique key

**Git Workflow**:
```bash
# Write tests first in tests/test_statistics.py
# Implement StatisticsCollector
pytest tests/test_statistics.py -v
git add src/job_scraper/etl/statistics.py tests/test_statistics.py
git commit -m "feat(etl): add statistics collector"
```

**Estimated Time**: 45 minutes

---

### EPIC-004-S4-TASK-2: Update Orchestrator

**Tags**: `#backend #orchestrator`

**Description**: Integrate new ETL pipelines into orchestrator

**Steps**:
1. Review `src/job_scraper/orchestrator.py`
2. Add import for NormalizationETL, JDAnalyzer, StatisticsCollector
3. Replace old unified_etl with new pipelines
4. Update `run_pipeline()` function
5. Add error handling for each stage

**Acceptance Criteria**:
- [ ] Pipeline runs: Scraping → Normalization → JD Analysis → Statistics
- [ ] Each stage logs progress
- [ ] Failure in one stage doesn't prevent statistics
- [ ] Returns summary dict

**Git Workflow**:
```bash
# Update orchestrator
pytest tests/test_orchestrator.py -v
git add src/job_scraper/orchestrator.py
git commit -m "feat(orchestrator): integrate new ETL pipelines"
```

**Estimated Time**: 45 minutes

---

### EPIC-004-S4-TASK-3: Update Scheduler Script

**Tags**: `#backend #scheduler`

**Description**: Update cron schedule and add croniter support

**Steps**:
1. Review `scripts/run_pipeline.py`
2. Add croniter import
2. Update default cron to "0 13 * * *"
3. Improve scheduling logic with croniter

**Acceptance Criteria**:
- [ ] Default schedule: 13:00 (1:00 PM)
- [ ] Uses croniter for next run calculation
- [ ] Logs next run time

**Git Workflow**:
```bash
git add scripts/run_pipeline.py
git commit -m "feat(scheduler): update to 1:00 PM schedule"
```

**Estimated Time**: 20 minutes

---

### EPIC-004-S4-TASK-4: Update MongoDB Client

**Tags**: `#backend #database`

**Description**: Add support for multiple databases

**Steps**:
1. Review `src/job_scraper/db/mongo_client.py`
2. Add methods to access `normalized_jobs`, `filtered_jobs`, `job_statistics`

**Acceptance Criteria**:
- [ ] `get_normalized_jobs_db()` returns client for normalized_jobs
- [ ] `get_filtered_jobs_db()` returns client for filtered_jobs
- [ ] `get_statistics_db()` returns client for job_statistics

**Git Workflow**:
```bash
git add src/job_scraper/db/mongo_client.py
git commit -m "feat(db): add database helpers"
```

**Estimated Time**: 15 minutes

---

## EPIC-004-S5: Testing & Documentation

**Repository**: `modules/brightdata`
**Feature Branch**: `feature/epic-004-job-scraper-elt-upgrade`
**Agent Type**: Backend
**Required Skills**: `test-driven-development`, `pragmatic-clean-code-reviewer`

### Success Criteria
- [ ] All new modules have >80% test coverage
- [ ] E2E test validates complete pipeline
- [ ] Documentation updated

### Execution Order
EPIC-004-S5-TASK-1 → EPIC-004-S5-TASK-2 → EPIC-004-S5-TASK-3

---

### EPIC-004-S5-TASK-1: Write E2E Test

**Tags**: `#backend #testing`

**Description**: Create end-to-end test for full pipeline

**Steps**:
1. Create `tests/test_e2e_pipeline.py`
2. Mock all external APIs (Bright Data, GLM-4, MongoDB)
3. Test complete flow: Scraping → Normalization → JD Analysis → Statistics
4. Verify data in all databases

**Acceptance Criteria**:
- [ ] E2E test runs successfully
- [ ] All stages validated
- [ ] Data verified in correct databases/collections

**Git Workflow**:
```bash
pytest tests/test_e2e_pipeline.py -v
git add tests/test_e2e_pipeline.py
git commit -m "test(e2e): add end-to-end pipeline test"
```

**Estimated Time**: 45 minutes

---

### EPIC-004-S5-TASK-2: Update Documentation

**Tags**: `#docs`

**Description**: Update project documentation

**Steps**:
1. Update `README.md` with new architecture
2. Update API documentation
3. Add GLM-4 setup instructions

**Acceptance Criteria**:
- [ ] README reflects new architecture
- [ ] GLM-4 API key setup documented
- [ ] Resume config documented

**Git Workflow**:
```bash
git add README.md docs/
git commit -m "docs: update for GLM-4 integration"
```

**Estimated Time**: 30 minutes

---

### EPIC-004-S5-TASK-3: Code Review

**Tags**: `#review`

**Description**: Run pragmatic code review

**Steps**:
1. Run `pragmatic-clean-code-reviewer` skill
2. Address any L3+ issues
3. Ensure SOLID, KISS, DRY compliance

**Acceptance Criteria**:
- [ ] No L3+ code smells
- [ ] All tests passing
- [ ] Coverage >80%

**Git Workflow**:
```bash
# Fix any issues
pytest tests/ --cov
git add -u
git commit -m "refactor: address code review feedback"
```

**Estimated Time**: 30 minutes

---

## Team Lead Prompt

### Agent Configuration

Create the following agent for this Epic:

**Backend Agent** (Single agent for all stories)
- Work Repo: `modules/brightdata`
- Skills: `test-driven-development`, `code-quality-principles`
- Required Python packages: pymonogo, httpx, croniter, pydantic, python-dotenv

### Execution Order

1. **EPIC-004-S1**: GLM Client & Config
2. **EPIC-004-S2**: Normalization ETL
3. **EPIC-004-S3**: JD Analysis
4. **EPIC-004-S4**: Statistics & Orchestrator
5. **EPIC-004-S5**: Testing & Documentation

### Git Workflow

Each story:
1. Ensure on `feature/epic-004-job-scraper-elt-upgrade` branch
2. Implement tasks following TDD
3. Run all tests: `pytest tests/ -v`
4. Commit with conventional commit message
5. Push after each story

### Communication Protocol

- Agent → Team Lead: "I'm done with [STORY-ID]" when complete
- Report test results and any issues

---
**Version**: 1.0
**Created**: 2026-02-10
**Epic ID**: EPIC-004
