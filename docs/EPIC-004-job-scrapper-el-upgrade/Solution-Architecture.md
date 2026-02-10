# Solution Architecture - Job Scraper ELT Upgrade

## 1. Architecture Overview

### 1.1 Design Principles
- **SOLID**: Single Responsibility for each module (scraping, normalization, analysis, statistics)
- **KISS**: Simple rule-based filters where possible, GLM-4 only for complex analysis
- **DRY**: Reusable GLM client for all LLM operations
- **YAGNI**: Only implement required features, no over-engineering

### 1.2 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           Scheduler (Cron)                              │
│                        Daily at 13:00 (1:00 PM)                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          Orchestrator                                   │
│                    coordinates all pipelines                            │
└─────────────────────────────────────────────────────────────────────────┘
        │                │                │                │
        ▼                ▼                ▼                ▼
   ┌────────┐      ┌────────┐      ┌────────┐      ┌────────┐
   │LinkedIn│      │ Indeed │      │  Seek  │      │ Config │
   │Scraper │      │Scraper │      │Scraper │      │ Loader │
   └────────┘      └────────┘      └────────┘      └────────┘
        │                │                │
        └────────────────┴────────────────┘
                         │
                         ▼
            ┌──────────────────────────────┐
            │    Raw Data Storage          │
            │    Database: job_scraper     │
            │    Collections:              │
            │    - indeed_jobs             │
            │    - seek_jobs               │
            │    - linkedin_jobs           │
            └──────────────────────────────┘
                         │
                         ▼
            ┌──────────────────────────────┐
            │  Normalization Pipeline      │
            │  (GLM-4 based)               │
            │  - Extract today's jobs      │
            │  - Normalize with GLM-4      │
            │  - Deduplicate              │
            └──────────────────────────────┘
                         │
                         ▼
            ┌──────────────────────────────┐
            │  Normalized Data Storage     │
            │  Database: normalized_jobs   │
            │  Collection: YYYY_MM_DD      │
            └──────────────────────────────┘
                         │
                         ▼
            ┌──────────────────────────────┐
            │    JD Analysis Pipeline      │
            │  1. Visa/PR filter           │
            │  2. Security clearance filter│
            │  3. Skill/experience match   │
            └──────────────────────────────┘
                         │
                         ▼
            ┌──────────────────────────────┐
            │   Filtered Jobs Storage      │
            │   Database: filtered_jobs    │
            │   Collection: YYYY_MM_DD     │
            └──────────────────────────────┘
                         │
                         ▼
            ┌──────────────────────────────┐
            │    Statistics Collection      │
            │    Database: job_statistics  │
            │    Collection: daily_stats   │
            └──────────────────────────────┘
```

## 2. Component Design

### 2.1 Backend Components

#### GLM Client
**File:** `src/job_scraper/llm/glm_client.py`

```python
class GLMClient:
    """Zhipu AI GLM-4 API client"""

    def __init__(self, api_key: str, model: str = "glm-4-plus"):
        self.api_key = api_key
        self.model = model
        self.base_url = "https://open.bigmodel.cn/api/paas/v4"
        self.timeout = 120

    def chat(self, system_prompt: str, user_prompt: str) -> str:
        """Send chat request and return text response"""

    def chat_json(self, system_prompt: str, user_prompt: str) -> dict:
        """Send chat request and return JSON response"""

    def batch_normalize(self, jobs: list[dict]) -> list[dict]:
        """Normalize a batch of jobs"""

    def analyze_jd(self, job_description: str, resume: dict) -> dict:
        """Analyze JD against resume profile"""
```

**API Endpoint:**
```
POST https://open.bigmodel.cn/api/paas/v4/chat/completions
Headers:
  Authorization: Bearer <api_key>
  Content-Type: application/json

Body:
{
  "model": "glm-4-plus",
  "messages": [
    {"role": "system", "content": "..."},
    {"role": "user", "content": "..."}
  ],
  "temperature": 0,
  "response_format": {"type": "json_object"}
}
```

#### Normalization ETL
**File:** `src/job_scraper/etl/normalization_etl.py`

```python
class NormalizationETL:
    """Normalize raw jobs from all platforms"""

    def __init__(self, mongo_client, glm_client):
        self.mongo = mongo_client
        self.glm = glm_client

    def extract_today_jobs(self) -> list[dict]:
        """Extract jobs created today from all platform collections"""

    def normalize_jobs(self, jobs: list[dict]) -> list[dict]:
        """Normalize jobs using GLM-4 in batches"""

    def deduplicate_jobs(self, jobs: list[dict]) -> list[dict]:
        """Remove duplicate jobs (same company, title, location)"""

    def load_normalized_jobs(self, jobs: list[dict], date: str):
        """Load normalized jobs to normalized_jobs database"""

    def run(self) -> dict:
        """Run full normalization pipeline"""
```

#### JD Analyzer
**File:** `src/job_scraper/etl/jd_analyzer.py`

```python
class JDAnalyzer:
    """Analyze JDs for visa, clearance, and skill compatibility"""

    def __init__(self, glm_client, resume_path: str):
        self.glm = glm_client
        self.resume = self._load_resume(resume_path)

    def check_visa_requirements(self, job_description: str) -> dict:
        """Rule-based check for citizenship/PR requirements"""

    def check_security_clearance(self, job_description: str) -> dict:
        """Rule-based check for security clearance requirements"""

    def match_skills(self, job: dict) -> dict:
        """GLM-4 based skill and experience matching"""

    def analyze_job(self, job: dict) -> dict:
        """Run full analysis on a job"""

    def filter_jobs(self, jobs: list[dict]) -> tuple[list[dict], list[dict]]:
        """Filter jobs, return (passed, failed)"""

    def load_filtered_jobs(self, jobs: list[dict], date: str):
        """Load filtered jobs to filtered_jobs database"""

    def run(self, jobs: list[dict]) -> dict:
        """Run full JD analysis pipeline"""
```

#### Statistics Collector
**File:** `src/job_scraper/etl/statistics.py`

```python
class StatisticsCollector:
    """Collect and store pipeline statistics"""

    def __init__(self, mongo_client):
        self.mongo = mongo_client

    def collect_raw_counts(self) -> dict:
        """Count raw jobs by platform for today"""

    def collect_filter_stats(self, analysis_results: list[dict]) -> dict:
        """Aggregate filter results"""

    def collect_skill_scores(self, filtered_jobs: list[dict]) -> dict:
        """Calculate skill score statistics"""

    def save_daily_stats(self, date: str, stats: dict):
        """Save daily statistics to job_statistics.daily_stats"""

    def run(self, pipeline_stats: dict) -> dict:
        """Run full statistics collection"""
```

### 2.2 Frontend Components
None - CLI-based pipeline

### 2.3 Database Schema

#### Database: `normalized_jobs`
**Collection:** `YYYY_MM_DD` (e.g., `2026_02_10`)

```javascript
{
  _id: ObjectId,
  job_id: string,              // Unique identifier
  job_title: string,
  company_name: string,
  job_description: string,
  job_location: string,        // "City, State" format
  employment_type: string,     // full_time, part_time, contract, casual, internship
  work_arrangement: string,    // onsite, remote, hybrid
  posted_date: ISODate,
  sources: [{                  // Source platforms
    platform: string,          // linkedin, indeed, seek
    job_posting_id: string,
    url: string
  }],
  created_at: ISODate
}

Indexes:
  - job_id (unique)
  - posted_date
  - job_location
```

#### Database: `filtered_jobs`
**Collection:** `YYYY_MM_DD`

```javascript
{
  // ... all normalized fields ...
  analysis_result: {
    visa_compatible: boolean,
    security_clearance_required: boolean,
    skill_match_score: number,      // 0-100
    skill_match_details: {
      matched_skills: [string],
      missing_skills: [string],
      experience_match: boolean,
      reason: string
    }
  },
  filtered_at: ISODate
}

Indexes:
  - job_id (unique)
  - filtered_at
  - "analysis_result.skill_match_score"
```

#### Database: `job_statistics`
**Collection:** `daily_stats`

```javascript
{
  _id: ObjectId,
  date: string,                // YYYY_MM_DD (unique)
  scraped_at: ISODate,
  pipeline_run_id: string,
  raw_counts: {
    indeed: number,
    seek: number,
    linkedin: number,
    total: number
  },
  normalized_count: number,
  filter_results: {
    total_analyzed: number,
    visa_incompatible: number,
    security_clearance_required: number,
    skill_mismatch: number,
    passed: number
  },
  skill_match_scores: {
    min: number,
    max: number,
    avg: number
  }
}

Indexes:
  - date (unique)
  - scraped_at
```

## 3. API Design

### 3.1 Endpoints
None - internal pipeline, no external API

### 3.2 Request/Response Format

**GLM-4 Normalize Request:**
```json
{
  "model": "glm-4-plus",
  "messages": [
    {
      "role": "system",
      "content": "You are a job data normalizer..."
    },
    {
      "role": "user",
      "content": "Jobs to normalize: [...]"
    }
  ],
  "temperature": 0,
  "response_format": {"type": "json_object"}
}
```

**GLM-4 Normalize Response:**
```json
{
  "jobs": [
    {
      "index": 0,
      "job_title": "Senior DevOps Engineer",
      "company_name": "Company ABC",
      "job_location": "Canberra, ACT",
      "employment_type": "full_time",
      "work_arrangement": "remote"
    }
  ]
}
```

**GLM-4 JD Analysis Request:**
```json
{
  "model": "glm-4-plus",
  "messages": [
    {
      "role": "system",
      "content": "You are an expert job matcher..."
    },
    {
      "role": "user",
      "content": "Job Description: ...\nCandidate Profile: ..."
    }
  ],
  "temperature": 0,
  "response_format": {"type": "json_object"}
}
```

**GLM-4 JD Analysis Response:**
```json
{
  "visa_compatible": true,
  "security_clearance_required": false,
  "skill_match_score": 85,
  "matched_skills": ["Docker", "Kubernetes", "AWS", "Terraform"],
  "missing_skills": ["Ansible"],
  "experience_match": true,
  "reason": "Strong match on core DevOps skills and experience"
}
```

### 3.3 Error Handling

| Error Type | Handling Strategy |
|------------|-------------------|
| GLM-4 API Timeout | Retry 3x with 2s delay, then fail |
| GLM-4 API Error | Log error, skip job, continue pipeline |
| MongoDB Error | Log error, raise exception (stop pipeline) |
| Invalid Resume | Fail fast at startup |
| JSON Parse Error | Use fallback normalization |

## 4. Data Flow

### 4.1 Pipeline Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│ 1. SCRAPING (Existing)                                      │
│    - LinkedIn scraper -> job_scraper.linkedin_jobs          │
│    - Indeed scraper -> job_scraper.indeed_jobs              │
│    - Seek scraper -> job_scraper.seek_jobs                  │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. NORMALIZATION (New)                                      │
│    - Extract today's jobs from all collections              │
│    - GLM-4 normalize (batch size 8)                         │
│    - Deduplicate by company+title+location                  │
│    - Load to normalized_jobs.YYYY_MM_DD                     │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. JD ANALYSIS (New)                                        │
│    - Visa/PR filter (rule-based)                           │
│    - Security clearance filter (rule-based)                 │
│    - Skill/experience match (GLM-4)                         │
│    - Load passed jobs to filtered_jobs.YYYY_MM_DD          │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. STATISTICS (New)                                         │
│    - Count raw jobs by platform                            │
│    - Aggregate filter results                              │
│    - Calculate skill score statistics                      │
│    - Save to job_statistics.daily_stats                     │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 State Machine

```
┌─────────┐    ┌───────────────┐    ┌─────────────┐
│  Raw    │ -> │ Normalized   │ -> │  Analyzed   │
│  Jobs   │    │    Jobs      │    │    Jobs     │
└─────────┘    └───────────────┘    └─────────────┘
                    │                      │
                    ▼                      ▼
              ┌─────────┐           ┌─────────┐
              │Filtered │           │Rejected │
              │  Jobs   │           │  Jobs   │
              └─────────┘           └─────────┘
```

## 5. Technology Decisions

### 5.1 Backend Tech Stack

| Component | Technology | Rationale |
|-----------|-----------|-----------|
| LLM API | Zhipu AI GLM-4 | User preference, Chinese language support |
| HTTP Client | HTTPX | Async support, modern API |
| Database | MongoDB | Existing infrastructure |
| Scheduling | croniter | Lightweight, flexible cron expressions |

### 5.2 Libraries

```toml
[dependencies]
pymongo = "^4.10"
httpx = "^0.27"
croniter = "^3.0"
pydantic = "^2.9"
python-dotenv = "^1.0"

[dev-dependencies]
pytest = "^8.0"
pytest-mock = "^3.14"
pytest-cov = "^6.0"
```

## 6. Security Considerations

### 6.1 API Key Management
- Store GLM API key in `.env` file
- `.env` in `.gitignore`
- Never log API keys
- Use environment variables in production

### 6.2 Data Privacy
- Resume data contains personal information
- `config/resume.json` in `.gitignore`
- No PII in logs

### 6.3 Rate Limiting
- Implement retry with exponential backoff
- Respect GLM-4 API rate limits
- Add circuit breaker if needed

## 7. Scalability Considerations

### 7.1 Batch Processing
- Normalization: 8 jobs per batch
- JD Analysis: Configurable parallel workers (default: 4)

### 7.2 Database Indexing
- All collections have appropriate indexes
- Date-named collections for easy pruning

### 7.3 Future Enhancements
- Add vector embeddings for semantic search
- Add skill taxonomy
- Support multiple user profiles

---
**Version**: 1.0
**Created**: 2026-02-10
**Epic ID**: EPIC-004
