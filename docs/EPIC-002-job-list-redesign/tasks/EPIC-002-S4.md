# EPIC-002-S4: E2E Testing & Code Review

## Overview
**Epic**: EPIC-002
**Repository**: `modules/test`
**Feature Branch**: `feature/job-list-redesign`
**Agent**: Test

## Context
This story validates the implementation through E2E testing and code review. Ensure all user flows work correctly and code quality standards are met.

Refer to:
- PRD Section 3 (Non-Functional Requirements)
- Task Plan Section EPIC-002-S4

## Tasks

### TASK-4-1: Write E2E Tests

**Description**: Create Playwright E2E tests for the new job list flow.

**Detailed Steps**:
1. Open or create test file in `modules/test/e2e/`:
   ```typescript
   import { test, expect } from '@playwright/test';

   test.describe('Job List Redesign', () => {
     test.beforeEach(async ({ page }) => {
       await page.goto('/');
     });

     test('job list page loads', async ({ page }) => {
       await expect(page.locator('h1')).toContainText('Jobs');
       await expect(page.locator('[data-testid="job-list"]')).toBeVisible();
     });

     test('filter updates URL', async ({ page }) => {
       await page.fill('[data-testid="search-input"]', 'frontend');
       await expect(page).toHaveURL(/keyword=frontend/);
     });

     test('clicking job navigates to detail', async ({ page }) => {
       await page.click('[data-testid="job-card"]:first-child');
       await expect(page).toHaveURL(/\/jobs\/[a-f0-9]+$/);
     });

     test('detail page loads correct job', async ({ page }) => {
       const jobId = 'test-job-id';
       await page.goto(`/jobs/${jobId}`);
       await expect(page.locator('[data-testid="job-detail"]')).toBeVisible();
     });

     test('back button preserves filters', async ({ page }) => {
       // Set filter
       await page.fill('[data-testid="search-input"]', 'frontend');
       await expect(page).toHaveURL(/keyword=frontend/);

       // Navigate to detail
       await page.click('[data-testid="job-card"]:first-child');

       // Go back
       await page.click('[data-testid="back-button"]');
       await expect(page).toHaveURL(/keyword=frontend/);
     });

     test('mobile filter toggle', async ({ page }) => {
       await page.setViewportSize({ width: 375, height: 667 });
       const filterPanel = page.locator('[data-testid="filter-panel"]');

       // Hidden by default
       await expect(filterPanel).not.toBeVisible();

       // Click toggle
       await page.click('[data-testid="filter-toggle"]');
       await expect(filterPanel).toBeVisible();
     });
   });
   ```
2. Add necessary data-testid attributes to components (coordinate with Frontend Agent)
3. Test on desktop viewport (1920x1080)
4. Test on mobile viewport (375x667)

**Acceptance Criteria**:
- [ ] All tests written and pass
- [ ] Tests cover main user flows
- [ ] Tests run without flakiness
- [ ] Tests complete within 60 seconds

**Testing**:
- Run `npm run test:e2e` in modules/test
- Review test coverage
- Fix any failures

---

### TASK-4-2: Run E2E Tests

**Description**: Execute full E2E test suite and verify results.

**Detailed Steps**:
1. Navigate to `modules/test`
2. Start the frontend dev server (if not running)
3. Run E2E tests:
   ```bash
   npm run test:e2e
   ```
4. Review test results output
5. If failures occur:
   - Review failure details
   - Identify root cause
   - Fix issue (coordinate with Frontend Agent if needed)
   - Re-run tests
6. Continue until all tests pass

**Acceptance Criteria**:
- [ ] All E2E tests pass
- [ ] No test failures
- [ ] No flaky tests (intermittent failures)
- [ ] Test execution time reasonable

**Testing**:
- Run full test suite 3 times to check for flakiness
- Report any issues to Team Lead

---

### TASK-4-3: Code Review

**Description**: Perform code review using pragmatic-clean-code-reviewer.

**Detailed Steps**:
1. Navigate to `modules/frontend`
2. Run pragmatic-clean-code-reviewer skill with L3 severity:
   - Review all modified files
   - Focus on: SOLID violations, code duplication, complexity
   - Check for: proper error handling, type safety, accessibility
3. Review findings:
   - Categorize by severity (critical, major, minor)
   - Determine action required for each
4. Fix critical issues immediately
5. Document acceptable warnings (if any)
6. Re-run review until no critical issues remain

**Review Checklist**:
- [ ] Single Responsibility Principle followed
- [ ] No code duplication (DRY)
- [ ] Code is simple and readable (KISS)
- [ ] Proper error handling
- [ ] TypeScript types properly defined
- [ ] No console.log statements
- [ ] Accessibility (ARIA labels, keyboard navigation)
- [ ] Performance (no unnecessary re-renders)

**Acceptance Criteria**:
- [ ] Code review completed
- [ ] No critical issues remaining
- [ ] Code follows SOLID/KISS/DRY principles
- [ ] TypeScript strict mode satisfied
- [ ] Issues documented (if any acceptable warnings)

**Testing**:
- Run `npm run type-check`
- Run `npm run lint`
- Manual code review

## Git Workflow

### Development Principles
- **Thoroughness**: Test all user flows
- **Quality**: Enforce code quality standards
- **Communication**: Report issues clearly to Team Lead

### Commit Guidelines
```bash
# After TASK-4-1
git commit -m "test(e2e): add tests for job list redesign"

# After TASK-4-2 (if fixes needed)
git commit -m "test(e2e): fix test failures"

# After TASK-4-3 (if fixes needed)
git commit -m "refactor: address code review findings"

git push origin feature/job-list-redesign
```

### PR Creation
If Test Agent has own PR:
```bash
gh pr create \
  --title "test(e2e): add E2E tests for job list redesign" \
  --body "Completes EPIC-002-S4

## Changes
- Added E2E tests for job list flow
- Verified all tests passing
- Completed code review (L3)

## Test Results
- All tests passing
- No critical issues found"
```

## Completion Checklist
- [ ] All 3 tasks completed
- [ ] E2E tests passing (run 3+ times)
- [ ] Code review completed
- [ ] No critical code issues
- [ ] Documentation updated (if needed)
- [ ] Commits pushed to remote
- [ ] PR created/updated
- [ ] Notify Team Lead: "Tests passed for EPIC-002-S4"

---
**Version**: 1.0
**Created**: 2026-02-09
**Epic ID**: EPIC-002
**Story ID**: EPIC-002-S4
