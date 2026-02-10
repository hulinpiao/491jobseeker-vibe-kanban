# EPIC-004-S1: GLM Client & Configuration

## Overview
**Epic**: EPIC-004
**Repository**: `modules/brightdata`
**Feature Branch**: `feature/epic-004-job-scraper-elt-upgrade`
**Agent**: Backend

## Context
This story replaces the Ollama LLM integration with Zhipu AI GLM-4 API. It creates the foundation for all subsequent LLM-based operations (normalization and JD analysis).

Reference:
- PRD: Section 2.1 (F1: GLM-4 Integration)
- Solution Architecture: Section 2.1 (GLM Client)

## Tasks

### TASK-1: Create GLM Client

**Description**: Implement GLM-4 API client with JWT authentication and retry logic

**Detailed Steps**:

1. Create file `src/job_scraper/llm/glm_client.py`:

```python
"""
GLM-4 API Client for Zhipu AI
"""
import time
import json
import hmac
import hashlib
import base64
from datetime import datetime, timedelta
from typing import Optional

import httpx


class GLMClientError(Exception):
    """Base exception for GLM client errors"""
    pass


class GLMClient:
    """Zhipu AI GLM-4 API client"""

    def __init__(
        self,
        api_key: str,
        model: str = "glm-4-plus",
        base_url: str = "https://open.bigmodel.cn/api/paas/v4",
        timeout: int = 120,
    ):
        """
        Initialize GLM-4 client.

        Args:
            api_key: Zhipu AI API key (format: id.secret)
            model: Model name (default: glm-4-plus)
            base_url: API base URL
            timeout: Request timeout in seconds
        """
        self.api_key = api_key
        self.model = model
        self.base_url = base_url.rstrip("/")
        self.timeout = timeout
        self._client: Optional[httpx.Client] = None

    def _generate_jwt(self) -> str:
        """
        Generate JWT token for API authentication.

        Zhipu AI uses a simple JWT format:
        - Header: {"alg": "HS256", "sign_type": "SIGN"}
        - Payload: {"api_key": id, "exp": timestamp, "timestamp": timestamp}
        """
        try:
            api_id, api_secret = self.api_key.split(".")
        except ValueError:
            raise GLMClientError("Invalid API key format. Expected: id.secret")

        now = datetime.now()
        exp = now + timedelta(hours=1)  # Token valid for 1 hour
        timestamp = int(now.timestamp())

        header = {"alg": "HS256", "sign_type": "SIGN"}
        payload = {
            "api_key": api_id,
            "exp": int(exp.timestamp()),
            "timestamp": timestamp,
        }

        def base64url_encode(data: str) -> str:
            return base64.urlsafe_b64encode(data.encode()).rstrip(b"=").decode()

        def base64url_encode_json(obj: dict) -> str:
            return base64url_encode(json.dumps(obj, separators=(",", ":")))

        header_encoded = base64url_encode_json(header)
        payload_encoded = base64url_encode_json(payload)

        message = f"{header_encoded}.{payload_encoded}"
        signature = hmac.new(
            api_secret.encode(), message.encode(), hashlib.sha256
        ).digest()
        signature_encoded = base64url_encode(signature)

        return f"{message}.{signature_encoded}"

    @property
    def client(self) -> httpx.Client:
        """Lazy initialization of HTTP client"""
        if self._client is None:
            self._client = httpx.Client(timeout=self.timeout)
        return self._client

    def _make_request(self, messages: list, json_mode: bool = False) -> dict:
        """
        Make API request with retry logic.

        Args:
            messages: Chat messages list
            json_mode: Enable JSON output format

        Returns:
            API response as dict

        Raises:
            GLMClientError: On API errors after retries
        """
        token = self._generate_jwt()
        headers = {
            "Authorization": f"Bearer {token}",
            "Content-Type": "application/json",
        }

        body = {
            "model": self.model,
            "messages": messages,
            "temperature": 0,
        }

        if json_mode:
            body["response_format"] = {"type": "json_object"}

        max_retries = 3
        retry_delay = 2

        for attempt in range(max_retries):
            try:
                response = self.client.post(
                    f"{self.base_url}/chat/completions",
                    headers=headers,
                    json=body,
                )
                response.raise_for_status()
                return response.json()

            except httpx.TimeoutException:
                if attempt < max_retries - 1:
                    time.sleep(retry_delay)
                    continue
                raise GLMClientError("Request timeout")

            except httpx.HTTPStatusError as e:
                if e.response.status_code in (429, 500, 502, 503, 504):
                    if attempt < max_retries - 1:
                        time.sleep(retry_delay)
                        continue
                raise GLMClientError(f"HTTP error: {e.response.status_code}")

            except (httpx.NetworkError, httpx.DecodeError) as e:
                if attempt < max_retries - 1:
                    time.sleep(retry_delay)
                    continue
                raise GLMClientError(f"Network error: {e}")

        raise GLMClientError("Max retries exceeded")

    def chat(self, system_prompt: str, user_prompt: str) -> str:
        """
        Send chat request and return text response.

        Args:
            system_prompt: System message content
            user_prompt: User message content

        Returns:
            Response text content
        """
        messages = [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt},
        ]

        response = self._make_request(messages, json_mode=False)

        try:
            return response["choices"][0]["message"]["content"]
        except (KeyError, IndexError) as e:
            raise GLMClientError(f"Invalid response format: {e}")

    def chat_json(self, system_prompt: str, user_prompt: str) -> dict:
        """
        Send chat request and return parsed JSON response.

        Args:
            system_prompt: System message content
            user_prompt: User message content

        Returns:
            Parsed JSON response as dict
        """
        messages = [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt},
        ]

        response = self._make_request(messages, json_mode=True)

        try:
            content = response["choices"][0]["message"]["content"]
            return json.loads(content)
        except (KeyError, IndexError, json.JSONDecodeError) as e:
            raise GLMClientError(f"Invalid JSON response: {e}")

    def close(self):
        """Close HTTP client"""
        if self._client is not None:
            self._client.close()
            self._client = None

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()
```

2. Create test file `tests/test_glm_client.py`:

```python
"""
Tests for GLM-4 Client
"""
import json
from unittest.mock import Mock, patch

import pytest

from src.job_scraper.llm.glm_client import GLMClient, GLMClientError


class TestGLMClientInit:
    """Test GLMClient initialization"""

    def test_init_with_default_params(self):
        client = GLMClient(api_key="test.id.secret")
        assert client.api_key == "test.id.secret"
        assert client.model == "glm-4-plus"
        assert client.base_url == "https://open.bigmodel.cn/api/paas/v4"
        assert client.timeout == 120

    def test_init_with_custom_params(self):
        client = GLMClient(
            api_key="test.id.secret",
            model="glm-4-air",
            base_url="https://custom.api",
            timeout=60,
        )
        assert client.model == "glm-4-air"
        assert client.base_url == "https://custom.api"
        assert client.timeout == 60


class TestJWTGeneration:
    """Test JWT token generation"""

    def test_generate_valid_jwt(self):
        client = GLMClient(api_key="test123.abc456")
        token = client._generate_jwt()

        # JWT should have 3 parts separated by dots
        parts = token.split(".")
        assert len(parts) == 3

        # Token should be decodable
        header = json.loads(base64url_decode(parts[0]))
        payload = json.loads(base64url_decode(parts[1]))

        assert header["alg"] == "HS256"
        assert header["sign_type"] == "SIGN"
        assert payload["api_key"] == "test123"
        assert "exp" in payload
        assert "timestamp" in payload

    def test_invalid_api_key_format(self):
        client = GLMClient(api_key="invalid_key")
        with pytest.raises(GLMClientError, match="Invalid API key format"):
            client._generate_jwt()


class TestChatMethods:
    """Test chat methods"""

    @patch("src.job_scraper.llm.glm_client.httpx.Client")
    def test_chat_returns_text(self, mock_client_class):
        mock_response = Mock()
        mock_response.raise_for_status = Mock()
        mock_response.json = Mock(return_value={
            "choices": [{"message": {"content": "Test response"}}]
        })
        mock_client = Mock()
        mock_client.post = Mock(return_value=mock_response)
        mock_client_class.return_value = mock_client

        client = GLMClient(api_key="test.id.secret")
        result = client.chat("System prompt", "User prompt")

        assert result == "Test response"

    @patch("src.job_scraper.llm.glm_client.httpx.Client")
    def test_chat_json_returns_dict(self, mock_client_class):
        mock_response = Mock()
        mock_response.raise_for_status = Mock()
        mock_response.json = Mock(return_value={
            "choices": [{"message": {"content": '{"key": "value"}'}}]
        })
        mock_client = Mock()
        mock_client.post = Mock(return_value=mock_response)
        mock_client_class.return_value = mock_client

        client = GLMClient(api_key="test.id.secret")
        result = client.chat_json("System prompt", "User prompt")

        assert result == {"key": "value"}

    @patch("src.job_scraper.llm.glm_client.httpx.Client")
    def test_retry_on_timeout(self, mock_client_class):
        import httpx

        mock_client = Mock()
        mock_client.post.side_effect = [
            httpx.TimeoutException("Timeout"),
            httpx.TimeoutException("Timeout"),
            Mock(
                raise_for_status=Mock(),
                json=Mock(return_value={
                    "choices": [{"message": {"content": "Success"}}]
                })
            ),
        ]
        mock_client_class.return_value = mock_client

        client = GLMClient(api_key="test.id.secret")
        result = client.chat("System", "User")

        assert result == "Success"
        assert mock_client.post.call_count == 3


def base64url_decode(data: str) -> str:
    """Helper to decode base64url"""
    # Add padding
    padding = 4 - len(data) % 4
    if padding != 4:
        data += "=" * padding
    import base64
    return base64.urlsafe_b64decode(data).decode()
```

**Acceptance Criteria**:
- [ ] GLMClient class created with JWT generation
- [ ] `chat()` method returns string
- [ ] `chat_json()` method returns dict
- [ ] Retry logic (3 attempts, 2s delay)
- [ ] Tests pass

**Testing**:
```bash
cd modules/brightdata
pytest tests/test_glm_client.py -v
```

---

### TASK-2: Update Environment Configuration

**Description**: Update .env files for GLM-4 configuration

**Detailed Steps**:

1. Update `.env.example` (already done in previous steps)

2. Update `src/job_scraper/config.py`:

```python
"""
Configuration management for job scraper
"""
import os
from pathlib import Path
from typing import Dict, Any

from dotenv import load_dotenv

# Load environment variables
load_dotenv()


class Config:
    """Application configuration"""

    # Bright Data
    BRIGHTDATA_API_TOKEN = os.getenv("BRIGHTDATA_API_TOKEN")
    BRIGHTDATA_LINKEDIN_DATASET_ID = os.getenv("BRIGHTDATA_LINKEDIN_DATASET_ID")
    BRIGHTDATA_INDEED_DATASET_ID = os.getenv("BRIGHTDATA_INDEED_DATASET_ID")
    BRIGHTDATA_SEEK_COLLECTOR_ID = os.getenv("BRIGHTDATA_SEEK_COLLECTOR_ID")

    # MongoDB
    MONGODB_URI = os.getenv("MONGODB_URI", "mongodb://localhost:27017")
    MONGODB_DATABASE = os.getenv("MONGODB_DATABASE", "job_scraper")

    # GLM-4 Configuration
    GLM_API_KEY = os.getenv("GLM_API_KEY")
    GLM_MODEL = os.getenv("GLM_MODEL", "glm-4-plus")
    GLM_BASE_URL = os.getenv("GLM_BASE_URL", "https://open.bigmodel.cn/api/paas/v4")
    GLM_TIMEOUT = int(os.getenv("GLM_TIMEOUT", "120"))

    # Pipeline defaults
    DEFAULT_TIME_RANGE = os.getenv("DEFAULT_TIME_RANGE", "24h")
    DEFAULT_LOCATION = os.getenv("DEFAULT_LOCATION", "Remote")
    DEFAULT_RECORD_LIMIT = int(os.getenv("DEFAULT_RECORD_LIMIT", "100"))

    @classmethod
    def get_glm_config(cls) -> Dict[str, Any]:
        """Get GLM client configuration"""
        if not cls.GLM_API_KEY:
            raise ValueError("GLM_API_KEY not set in environment")
        return {
            "api_key": cls.GLM_API_KEY,
            "model": cls.GLM_MODEL,
            "base_url": cls.GLM_BASE_URL,
            "timeout": cls.GLM_TIMEOUT,
        }

    @classmethod
    def validate(cls) -> None:
        """Validate required configuration"""
        required = [
            ("BRIGHTDATA_API_TOKEN", cls.BRIGHTDATA_API_TOKEN),
            ("GLM_API_KEY", cls.GLM_API_KEY),
            ("MONGODB_URI", cls.MONGODB_URI),
        ]
        missing = [name for name, value in required if not value]
        if missing:
            raise ValueError(f"Missing required config: {', '.join(missing)}")


def get_resume_path() -> Path:
    """Get path to resume configuration file"""
    base_dir = Path(__file__).parent.parent
    return base_dir / "config" / "resume.json"
```

**Acceptance Criteria**:
- [ ] Config class has GLM_* attributes
- [ ] `get_glm_config()` returns dict
- [ ] `validate()` checks GLM_API_KEY

**Testing**:
```bash
pytest tests/test_config.py -v
```

---

### TASK-3: Create Resume Configuration

**Description**: Already created in `/modules/brightdata/config/resume.json`

**Verify**:
```bash
cat modules/brightdata/config/resume.json | head -20
```

---

### TASK-4: Update Search Configurations

**Description**: Update YAML configs with new parameters

**Detailed Steps**:

1. Update `config/linkedin_search.yaml`:

```yaml
# LinkedIn Search Configuration

keywords:
  - "DevOps Engineer"
  - "SRE"
  - "Cloud Engineer"
  - "Platform Engineer"
  - "Senior DevOps"

locations:
  - "Remote"
  - "Canberra, Australian Capital Territory"
  - "Brisbane, Queensland"
  - "Sydney, New South Wales"
  - "Wollongong, New South Wales"
  - "Newcastle, New South Wales"
  - "Gold Coast, Queensland"
  - "Sunshine Coast, Queensland"

country: "AU"
time_range: "Past 24 hours"
selective_search: true
limit_per_input: 50
record_limit: 200
```

2. Update `config/indeed_search.yaml`:

```yaml
# Indeed Search Configuration

keyword_search:
  - "DevOps Engineer"
  - "SRE"
  - "Cloud Engineer"
  - "Platform Engineer"
  - "Senior DevOps"

locations:
  - "Remote"
  - "Canberra ACT"
  - "Brisbane QLD"
  - "Sydney NSW"
  - "Wollongong NSW"
  - "Newcastle NSW"
  - "Gold Coast QLD"

country: "AU"
domain: "au.indeed.com"
date_posted: "Last 24 hours"
limit_per_input: 50
record_limit: 200
```

3. Update `config/seek_search.yaml`:

```yaml
# Seek Search Configuration

keywords:
  - "devops-engineer"
  - "sre"
  - "cloud-engineer"
  - "platform-engineer"
  - "senior-devops"

locations:
  - "remote"
  - "canberra"
  - "brisbane"
  - "sydney"
  - "wollongong"
  - "newcastle"
  - "gold-coast"
  - "sunshine-coast"

daterange: 1
worktype: 242
record_limit: 200
```

4. Update `config/pipeline.yaml`:

```yaml
# Pipeline Configuration

schedule:
  enabled: true
  cron: "0 13 * * *"  # Daily at 1:00 PM

platforms:
  linkedin:
    enabled: true
  indeed:
    enabled: true
  seek:
    enabled: true

glm:
  timeout: 120
  batch_size: 8

jd_analysis:
  enabled: true
  skill_match_threshold: 60
  resume_path: "config/resume.json"
```

**Acceptance Criteria**:
- [ ] All configs have 5 keywords
- [ ] All configs have target locations
- [ ] Time filters: 24 hours
- [ ] Schedule: 13:00

---

## Git Workflow

### Branch Creation
```bash
cd modules/brightdata
git checkout main
git pull origin main
git checkout -b feature/epic-004-job-scraper-elt-upgrade
```

### Development Principles
- **SOLID**: Single Responsibility - each class has one job
- **KISS**: Keep methods simple and readable
- **DRY**: Reuse JWT generation and request logic
- **TDD**: Tests written before implementation

### Commit Guidelines
```bash
# After each task
git add src/job_scraper/llm/glm_client.py tests/test_glm_client.py
git commit -m "feat(llm): add GLM-4 API client with JWT auth"

git add .env.example src/job_scraper/config.py
git commit -m "feat(config): add GLM-4 environment configuration"

git add config/linkedin_search.yaml config/indeed_search.yaml config/seek_search.yaml config/pipeline.yaml
git commit -m "feat(config): update search parameters and schedule to 1:00 PM"
```

### PR Creation
```bash
git push origin feature/epic-004-job-scraper-elt-upgrade
gh pr create \
  --title "feat(EPIC-004-S1): GLM-4 client and configuration" \
  --body "Implements GLM-4 API client, configuration, and search parameters.

- Added GLMClient with JWT authentication
- Updated environment configuration
- Updated search configs with new keywords/locations
- Changed schedule to 13:00 daily"
```

---

## Completion Checklist
- [ ] GLMClient implemented with JWT auth
- [ ] chat() and chat_json() methods working
- [ ] Retry logic (3 attempts, 2s delay)
- [ ] Environment configuration updated
- [ ] Resume data loaded from config
- [ ] Search configs updated with 5 keywords and 8 locations
- [ ] Schedule updated to 13:00
- [ ] All tests passing
- [ ] Commits pushed to remote
- [ ] PR created

---
**Version**: 1.0
**Created**: 2026-02-10
