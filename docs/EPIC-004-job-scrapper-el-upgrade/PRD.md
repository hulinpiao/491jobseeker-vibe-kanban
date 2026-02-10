# PRD - Job Scraper ELT Upgrade

## 1. Project Overview

### 1.1 Project Name
Job Scraper ELT Upgrade with GLM-4 Integration

### 1.2 Background
The current brightdata job scraper system uses Ollama (qwen3:8b) for LLM-based data normalization and deduplication. The system lacks:
- JD analysis for visa/security clearance requirements
- Skill matching against candidate profile
- Statistics tracking for filtering decisions

This upgrade will:
1. Replace Ollama with Zhipu AI GLM-4 API
2. Add JD analysis pipeline for 491 visa compatibility
3. Add skill/experience matching
4. Add comprehensive statistics tracking

### 1.3 Target Users
- **Primary**: Eric Piao (491 visa holder, Senior DevSecOps Engineer)
- **Secondary**: Future extension to other job seekers

## 2. Core Functional Requirements

### 2.1 Feature List

| ID | Name | Priority | Description |
|----|------|----------|-------------|
| F1 | GLM-4 Integration | P0 | Replace Ollama with Zhipu AI GLM-4 API |
| F2 | Config Update | P0 | Update search keywords and locations |
| F3 | Normalization ETL | P0 | Rewrite normalization using GLM-4 |
| F4 | JD Analysis | P0 | Visa, security clearance, skill matching |
| F5 | Statistics Tracking | P1 | Track scraping and filtering metrics |
| F6 | Scheduling Update | P2 | Change to daily 1:00 PM execution |

### 2.2 Feature Details

#### F1: GLM-4 Integration
- Remove Ollama client (`src/job_scraper/llm/ollama_client.py`)
- Create GLM-4 client (`src/job_scraper/llm/glm_client.py`)
- Support chat and JSON output modes
- Batch processing for normalization
- Retry logic with exponential backoff

#### F2: Config Update
**Search Keywords:**
- DevOps Engineer
- SRE
- Cloud Engineer
- Platform Engineer
- Senior DevOps

**Locations:**
- Remote
- Canberra, ACT
- Brisbane, QLD
- Sydney, NSW
- Regional areas (Wollongong, Newcastle, Gold Coast, Sunshine Coast)

**Time Filter:** 24 hours (1 day ago)

#### F3: Normalization ETL
- Extract today's raw jobs from platform collections
- Normalize using GLM-4 (glm-4-plus model)
- Output to `normalized_jobs` database with date-named collections
- Fields: job_id, job_title, company_name, job_description, job_location, employment_type, work_arrangement, posted_date, sources

#### F4: JD Analysis
**Three-stage filtering:**

1. **Visa/PR Filter (Rule-based)**
   - Check for citizenship/PR requirements
   - Keywords: "Citizenship", "PR", "Permanent Resident", "Australian Citizen"
   - Exclude: 491 visa holders cannot apply

2. **Security Clearance Filter (Rule-based)**
   - Check for clearance requirements
   - Keywords: "NV1", "NV2", "Baseline", "Security clearance", "Defence clearance"
   - Exclude: 491 visa holders unlikely to have these

3. **Skill/Experience Match (GLM-4 based)**
   - Compare JD with resume profile
   - Output: score (0-100), matched skills, missing skills, experience match
   - Threshold: 60 points minimum

**Output:** `filtered_jobs` database with date-named collections

#### F5: Statistics Tracking
Track per day:
- Raw job counts by platform (indeed, seek, linkedin)
- Normalized job count
- Filter results: visa incompatible, security clearance required, skill mismatch, passed
- Skill match score distribution (min, max, avg)

**Output:** `job_statistics` database, `daily_stats` collection

#### F6: Scheduling Update
- Change from 8:00 AM to 1:00 PM daily
- Update `config/pipeline.yaml`

## 3. Non-Functional Requirements

### 3.1 Performance
- Normalization: Batch size 8 jobs per GLM-4 call
- JD Analysis: Parallel processing with configurable workers
- Timeout: 120 seconds per GLM-4 call

### 3.2 Compatibility
- Python 3.12+
- MongoDB 7.0+
- GLM-4 API (Zhipu AI)

### 3.3 Code Quality
- **SOLID**: Single Responsibility, Dependency Inversion
- **KISS**: Simple, readable implementations
- **DRY**: Reusable components
- **TDD**: Tests before implementation

## 4. Data Sources

### 4.1 Database/API

| Source | Type | Purpose |
|--------|------|---------|
| Bright Data API | External | Scraping LinkedIn/Indeed jobs |
| Seek DCA API | External | Scraping Seek jobs |
| MongoDB | Internal | Data storage |
| GLM-4 API | External | LLM processing |

### 4.2 Data Structure

**Resume Data:** `config/resume.json`
- Profile info (name, title, visa status)
- Technical skills (categorized)
- Work experience
- Certifications
- Target keywords/locations
- Exclusion criteria

**Database Schema:** (See Solution Architecture)

## 5. Success Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| GLM-4 API Success Rate | > 95% | API response tracking |
| Normalization Accuracy | > 90% | Manual spot check |
| Skill Match Precision | > 80% | User feedback |
| Pipeline Runtime | < 30 min | Logging |
| Daily Job Delivery | Before 2 PM | Scheduled execution |

## 6. Tech Stack

### 6.1 Backend
- Python 3.12+
- PyMongo (MongoDB client)
- HTTPX (HTTP client for GLM-4 API)

### 6.2 Frontend
- None (CLI-based pipeline)

### 6.3 Testing
- Pytest
- pytest-mock
- pytest-cov

## 7. Development Method

- **TDD**: Write tests first, then implement
- **Agent Teams**: Single backend agent for this epic
- **Git Workflow**: Feature branch `feature/epic-004-job-scraper-elt-upgrade`

---
**Version**: 1.0
**Created**: 2026-02-10
**Epic ID**: EPIC-004
