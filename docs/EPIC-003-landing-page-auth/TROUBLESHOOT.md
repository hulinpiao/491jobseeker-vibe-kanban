# EPIC-003 Integration Testing Issues & Solutions

## Overview
This document captures integration issues discovered during EPIC-003 testing and their solutions.

---

## Issue #1: API Response Format Mismatch

### Discovery Date
2026-02-10

### Severity
High - Blocks user registration flow

### Description

Frontend and Backend had different expectations for API response formats.

**Backend Response Format:**
```json
{
  "success": true,
  "data": {
    "user": { ... },
    "message": "Registration successful..."
  }
}
```

**Frontend Expected Format:**
```json
{
  "user": { ... },
  "token": "..."  // Expected token, but backend doesn't return it on registration
}
```

### Root Cause

1. **Different Implementation Approaches**: Frontend and Backend were developed in separate repositories with different PRs
2. **No Shared API Contract**: No OpenAPI/Swagger spec was shared between teams
3. **Incomplete Integration Testing**: Backend tests only verified API contracts via Jest/Supertest, but frontend integration was not tested until runtime
4. **Auth Flow Misunderstanding**: Frontend expected token immediately after registration, but backend correctly requires email verification first (no token until login)

### Why Tests Didn't Catch This

| Test Type | What Was Tested | What Was Missed |
|-----------|----------------|------------------|
| Backend Unit Tests | Service logic, validation, database operations | API response format consumed by frontend |
| Backend Integration Tests | API endpoints with Supertest | Frontend's expectation of response structure |
| Frontend Component Tests | UI rendering, form validation | Actual API calls to backend |
| **End-to-End Tests** | **NOT RUN** | **Full user flow** |

### Solution

**File**: `modules/frontend/src/services/authService.ts`

1. Added `ApiResponse<T>` type wrapper to match backend response format
2. Updated `register()` to return `{ user }` instead of `{ user, token }`
3. Updated all API calls to handle `success` and `error` wrapper
4. Added proper error message extraction from backend error format

```typescript
// Before
export async function register(data: RegisterRequest): Promise<AuthResponse> {
  const response = await fetchApi<AuthResponse>('/api/auth/register', {
    method: 'POST',
    body: JSON.stringify(data),
  })
  setAuthToken(response.token)  // This failed - no token in response!
  return response
}

// After
export async function register(data: RegisterRequest): Promise<{ user: User }> {
  const response = await fetchApi<ApiResponse<RegisterResponseData>>('/api/auth/register', {
    method: 'POST',
    body: JSON.stringify({ email: data.email, password: data.password }),
  })
  if (!response.success || !response.data) {
    throw new Error(response.error?.message || 'Registration failed')
  }
  return { user: response.data.user }  // No token - correct!
}
```

### Prevention Strategies

#### 1. Shared API Contract (Recommended)

```bash
# Create OpenAPI spec in shared location
/docs/api-spec/openapi.yaml
```

Backend and Frontend both generate from this spec:
- Backend: Use `swagger-jsdoc` to generate docs from code
- Frontend: Use `openapi-typescript` to generate types from spec

#### 2. Contract Testing

Add Pact or similar contract testing:
```typescript
// Backend test - defines contract
describe('POST /api/auth/register', () => {
  it('should conform to API contract', async () => {
    const response = await request(app)
      .post('/api/auth/register')
      .send({ email: 'test@example.com', password: 'Password123' })

    // Contract assertion
    expect(response.body).toMatchSchema({
      success: true,
      data: {
        user: { id: 'string', email: 'string', emailVerified: 'boolean' },
        message: 'string'
      }
    })
  })
})
```

#### 3. End-to-End Testing with Playwright

```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Registration Flow', () => {
  test('should complete full registration', async ({ page }) => {
    await page.goto('/register')
    await page.fill('[name="email"]', 'test@example.com')
    await page.fill('[name="password"]', 'Password123')
    await page.click('button[type="submit"]')

    // Verify email step
    await page.waitForSelector('[name="verificationCode"]')

    // Get code from backend (in test environment)
    const code = await getVerificationCodeFromBackend()
    await page.fill('[name="verificationCode"]', code)
    await page.click('button:has-text("Verify")')

    // Should redirect to profile step
    await page.waitForURL('/profile')
  })
})
```

#### 4. CI/CD Integration Test Stage

```yaml
# .github/workflows/integration-test.yml
name: Integration Tests

on: [pull_request]

jobs:
  integration-test:
    runs-on: ubuntu-latest
    services:
      mongodb:
        image: mongo:7.0
        ports:
          - 27017:27017
      backend:
        image: backend:test
        ports:
          - 3000:3000
    steps:
      - name: Run E2E tests
        run: npm run test:e2e
```

#### 5. Feature Branch Testing Before PR

```bash
# Before creating PR
./scripts/integration-test.sh feature/landing-page-auth

# Script should:
# 1. Start MongoDB
# 2. Start backend
# 3. Start frontend
# 4. Run Playwright tests
# 5. Report results
```

### Lessons Learned

1. **API Contracts First**: Define OpenAPI spec before implementing
2. **Test Real Integration**: Unit tests are not enough - need E2E tests
3. **Share Types**: Use code generation to keep frontend/backend types in sync
4. **Mock Carefully**: Frontend mocks should match real backend responses
5. **Test Early**: Run integration tests before PR creation, not after deployment

### Related Documents

- [EPIC-003 Task Lists](./tasks/)
- [API Documentation](https://github.com/hulinpiao/491jobseeker-backend/pull/2)
- [Frontend PR](https://github.com/hulinpiao/491jobseeker-frontend/pull/3)

---

**Last Updated**: 2026-02-10
**Author**: EPIC-003 Team
