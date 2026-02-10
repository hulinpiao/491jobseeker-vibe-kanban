# Task Plan - Job List Page Redesign

## Epic Overview

| Epic ID | Epic Name | Description | Priority |
|---------|-----------|-------------|----------|
| EPIC-002 | Job List Page Redesign | Refactor job list from side-panel layout to separate pages with vertical layout | P0 |

## Stories Overview

| Story ID | Story Name | Agent | Repo | Branch |
|----------|------------|-------|------|--------|
| EPIC-002-S1 | Routing Setup & Layout Refactor | Frontend | frontend | feature/job-list-redesign |
| EPIC-002-S2 | Job List Page Implementation | Frontend | frontend | feature/job-list-redesign |
| EPIC-002-S3 | Job Detail Page Implementation | Frontend | frontend | feature/job-list-redesign |
| EPIC-002-S4 | E2E Testing & Code Review | Test | test | feature/job-list-redesign |

---

## EPIC-002-S1: Routing Setup & Layout Refactor

**Repository**: `modules/frontend`
**Feature Branch**: `feature/job-list-redesign`
**Agent Type**: Frontend
**Required Skills**: `code-quality-principles`, `test-driven-development`

### Success Criteria
- [ ] react-router-dom installed
- [ ] BrowserRouter configured in main.tsx
- [ ] Routes configured with JobListPage and JobDetailPage
- [ ] JobListPage and JobDetailPage placeholder components created
- [ ] App compiles without errors

### Execution Order
TASK-1-1 → TASK-1-2 → TASK-1-3 → TASK-1-4

---

### TASK-1-1: Install Dependencies

**Tags**: `#frontend #setup`

**Description**: Install react-router-dom and configure TypeScript types.

**Steps**:
1. Run `npm install react-router-dom` in modules/frontend
2. Verify installation in package.json
3. Run type check to ensure no errors

**Acceptance Criteria**:
- [ ] react-router-dom@^6.22.0 in package.json
- [ ] TypeScript types available
- [ ] `npm run type-check` passes

**Git Workflow**:
- Commit: `feat(deps): add react-router-dom for client-side routing`

**Estimated Time**: 5 minutes

---

### TASK-1-2: Configure BrowserRouter

**Tags**: `#frontend #routing`

**Description**: Wrap App with BrowserRouter in main.tsx.

**Steps**:
1. Import BrowserRouter from react-router-dom
2. Wrap existing App component
3. Verify app renders without errors

**Acceptance Criteria**:
- [ ] BrowserRouter wrapping App
- [ ] No console errors
- [ ] Dev server runs successfully

**Git Workflow**:
- Commit: `feat(routing): configure BrowserRouter in main.tsx`

**Estimated Time**: 5 minutes

---

### TASK-1-3: Create Route Configuration

**Tags**: `#frontend #routing`

**Description**: Add Routes and Route components to App.tsx.

**Steps**:
1. Create src/pages/JobListPage.tsx (placeholder)
2. Create src/pages/JobDetailPage.tsx (placeholder)
3. Update App.tsx with Routes configuration:
   - Route path="/" element={<JobListPage />}
   - Route path="/jobs/:jobId" element={<JobDetailPage />}

**Acceptance Criteria**:
- [ ] Routes configured in App.tsx
- [ ] Navigating to / shows JobListPage
- [ ] Navigating to /jobs/123 shows JobDetailPage
- [ ] No console errors

**Git Workflow**:
- Commit: `feat(routing): add route configuration for job pages`

**Estimated Time**: 10 minutes

---

### TASK-1-4: Migrate Existing Components

**Tags**: `#frontend #refactor`

**Description**: Move existing components to new page structure.

**Steps**:
1. Copy current App.tsx content to JobListPage.tsx
2. Remove JobDetail from the layout (keep only Filter + JobList)
3. Copy JobDetail component content to JobDetailPage.tsx
4. Test basic navigation

**Acceptance Criteria**:
- [ ] JobListPage shows SearchBar + Filter + JobList
- [ ] JobDetailPage shows job detail placeholder
- [ ] Manual navigation between pages works

**Git Workflow**:
- Commit: `refactor(pages): migrate components to page structure`

**Estimated Time**: 15 minutes

---

## EPIC-002-S2: Job List Page Implementation

**Repository**: `modules/frontend`
**Feature Branch**: `feature/job-list-redesign`
**Agent Type**: Frontend
**Required Skills**: `code-quality-principles`, `test-driven-development`

### Success Criteria
- [ ] Vertical layout: SearchBar → Filter → JobList
- [ ] Filter and JobList span full width
- [ ] JobCard displays: Title, Company, Location, Work Type, JD preview
- [ ] JobCard clickable, navigates to detail page
- [ ] URL query params sync with filters
- [ ] Mobile filter toggle working

### Execution Order
TASK-2-1 → TASK-2-2 → TASK-2-3 → TASK-2-4

---

### TASK-2-1: Refactor Layout to Vertical

**Tags**: `#frontend #layout`

**Description**: Change JobListPage from horizontal grid to vertical stack.

**Steps**:
1. Remove grid-cols-12 from JobListPage
2. Remove col-span classes from FilterPanel and JobList
3. Use flex-col with gap-4 for vertical spacing
4. Test on desktop and mobile

**Acceptance Criteria**:
- [ ] Components stack vertically
- [ ] FilterPanel spans full width
- [ ] JobList spans full width
- [ ] Responsive design maintained

**Git Workflow**:
- Commit: `refactor(layout): change to vertical layout`

**Estimated Time**: 10 minutes

---

### TASK-2-2: Redesign JobCard Content

**Tags**: `#frontend #ui`

**Description**: Update JobCard to show new content structure.

**Steps**:
1. Update JobCard.tsx layout:
   - Job title (large, bold)
   - Company, location, work type (badge row)
   - JD preview (line-clamp-3)
2. Wrap card content with Link to `/jobs/${job.id}`
3. Add hover effect for interactivity
4. Test visual appearance

**Acceptance Criteria**:
- [ ] Job title prominent
- [ ] Company info as badges
- [ ] JD truncated to 3 lines
- [ ] Click navigates to detail page
- [ ] Hover state indicates clickable

**Git Workflow**:
- Commit: `feat(job-card): redesign content structure with navigation`

**Estimated Time**: 20 minutes

---

### TASK-2-3: Implement Filter State in URL

**Tags**: `#frontend #state`

**Description**: Sync filter state with URL query parameters.

**Steps**:
1. Add useSearchParams hook from react-router-dom
2. Update FilterPanel to read/write to URL params
3. Update SearchBar to read/write keyword param
4. Update Pagination to read/write page param
5. Test that URL updates and state persists on refresh

**Acceptance Criteria**:
- [ ] Filter changes update URL
- [ ] URL params populate filters on load
- [ ] Page refresh preserves filter state
- [ ] Back/forward browser buttons work

**Git Workflow**:
- Commit: `feat(filters): sync filter state with URL params`

**Estimated Time**: 30 minutes

---

### TASK-2-4: Implement Mobile Filter Toggle

**Tags**: `#frontend #mobile`

**Description**: Add collapsible filter panel for mobile.

**Steps**:
1. Create FilterToggle component (button with show/hide logic)
2. Add state to control filter visibility
3. Hide FilterPanel by default on mobile (< 768px)
4. Add transition animation for smooth toggle
5. Test on mobile viewport

**Acceptance Criteria**:
- [ ] Filter toggle button visible on mobile
- [ ] FilterPanel hidden by default on mobile
- [ ] Toggle shows/hides FilterPanel
- [ ] Smooth animation
- [ ] Desktop unaffected (always visible)

**Git Workflow**:
- Commit: `feat(mobile): add collapsible filter panel toggle`

**Estimated Time**: 20 minutes

---

## EPIC-002-S3: Job Detail Page Implementation

**Repository**: `modules/frontend`
**Feature Branch**: `feature/job-list-redesign`
**Agent Type**: Frontend
**Required Skills**: `code-quality-principles`, `test-driven-development`

### Success Criteria
- [ ] Job detail page loads data from URL param
- [ ] Displays full job information
- [ ] Three action buttons visible (disabled state)
- [ ] Back button returns to list with preserved state
- [ ] Loading and error states handled

### Execution Order
TASK-3-1 → TASK-3-2 → TASK-3-3 → TASK-3-4

---

### TASK-3-1: Implement Job Data Fetching

**Tags**: `#frontend #data`

**Description**: Fetch job data based on URL parameter.

**Steps**:
1. Use useParams hook to get jobId from URL
2. Call fetchJobById API using React Query
3. Add loading skeleton
4. Add error state with retry button
5. Add 404 state for missing jobs

**Acceptance Criteria**:
- [ ] JobId extracted from URL
- [ ] Job data fetched and displayed
- [ ] Loading state shows skeleton
- [ ] Error state shows message
- [ ] 404 shows "Job not found" with back button

**Git Workflow**:
- Commit: `feat(detail-page): implement job data fetching`

**Estimated Time**: 20 minutes

---

### TASK-3-2: Design Job Detail Layout

**Tags**: `#frontend #ui`

**Description**: Create layout for job detail page content.

**Steps**:
1. Add back button (top left)
2. Add job info section (title, company, location, etc.)
3. Add full job description
4. Use existing Card component for container
5. Apply proper spacing and typography

**Acceptance Criteria**:
- [ ] Back button visible and functional
- [ ] Job info clearly displayed
- [ ] Full description readable
- [ ] Layout responsive
- [ ] Visual hierarchy clear

**Git Workflow**:
- Commit: `feat(detail-page): design job detail layout`

**Estimated Time**: 20 minutes

---

### TASK-3-3: Add Action Buttons

**Tags**: `#frontend #ui`

**Description**: Add three action buttons to detail page.

**Steps**:
1. Add button container (top right)
2. Create "Apply" button (disabled, title="Coming Soon")
3. Create "Customize Resume" button (disabled, title="Coming Soon")
4. Create "Generate CV" button (disabled, title="Coming Soon")
5. Style consistently with design system

**Acceptance Criteria**:
- [ ] Three buttons visible
- [ ] All buttons disabled
- [ ] Hover shows "Coming Soon" tooltip
- [ ] Buttons aligned properly
- [ ] No console warnings

**Git Workflow**:
- Commit: `feat(detail-page): add action buttons (disabled)`

**Estimated Time**: 10 minutes

---

### TASK-3-4: Implement Back Navigation

**Tags**: `#frontend #routing`

**Description**: Handle back button navigation with state preservation.

**Steps**:
1. Implement back button click handler
2. Navigate to /jobs with current filter params
3. Test navigation from detail back to list
4. Verify filters are preserved

**Acceptance Criteria**:
- [ ] Back button returns to list page
- [ ] Filter state preserved
- [ ] Works from browser back button
- [ ] Works from in-page back button

**Git Workflow**:
- Commit: `feat(detail-page): implement back navigation with state preservation`

**Estimated Time**: 15 minutes

---

## EPIC-002-S4: E2E Testing & Code Review

**Repository**: `modules/test`
**Feature Branch**: `feature/job-list-redesign`
**Agent Type**: Test
**Required Skills**: `subagent-testing`, `pragmatic-clean-code-reviewer`

### Success Criteria
- [ ] All E2E tests passing
- [ ] Code review completed (L3 severity)
- [ ] No critical issues found
- [ ] Documentation updated if needed

### Execution Order
TASK-4-1 → TASK-4-2 → TASK-4-3

---

### TASK-4-1: Write E2E Tests

**Tags**: `#testing #e2e`

**Description**: Create Playwright E2E tests for the new flow.

**Steps**:
1. Test job list page loads
2. Test filter updates URL
3. Test clicking job navigates to detail
4. Test detail page loads correct job
5. Test back button preserves filters
6. Test mobile filter toggle

**Acceptance Criteria**:
- [ ] All tests pass
- [ ] Tests cover main user flows
- [ ] No flaky tests

**Git Workflow**:
- Commit: `test(e2e): add tests for job list redesign`

**Estimated Time**: 30 minutes

---

### TASK-4-2: Run E2E Tests

**Tags**: `#testing #e2e`

**Description**: Execute full E2E test suite.

**Steps**:
1. Run `npm run test:e2e` in modules/test
2. Review test results
3. Fix any failures
4. Re-run until all pass

**Acceptance Criteria**:
- [ ] All E2E tests pass
- [ ] No failures or flakes

**Git Workflow**:
- Commit if fixes needed: `test(e2e): fix test failures`

**Estimated Time**: 15 minutes

---

### TASK-4-3: Code Review

**Tags**: `#review #quality`

**Description**: Perform code review using pragmatic-clean-code-reviewer.

**Steps**:
1. Run `pragmatic-clean-code-reviewer` skill (L3 severity)
2. Review findings
3. Fix any critical issues
4. Document any acceptable warnings

**Acceptance Criteria**:
- [ ] Code review completed
- [ ] No critical issues remaining
- [ ] Code follows SOLID/KISS/DRY

**Git Workflow**:
- Commit if fixes needed: `refactor: address code review findings`

**Estimated Time**: 20 minutes

---

## Team Lead Prompt

### Agent Configuration

Create the following agents for this Epic:

1. **Team Lead Agent** (Orchestrator)
   - Role: Coordinate all agents, monitor Kanban, assign tasks
   - Skills: `using-superpowers`, `code-quality-principles`

2. **Frontend Agent**
   - Work Repo: `modules/frontend`
   - Skills: `test-driven-development`, `code-quality-principles`

3. **Test Agent**
   - Work Repo: `modules/test`
   - Skills: `subagent-testing`, `pragmatic-clean-code-reviewer`

### Execution Order

1. **Frontend Agent**: S1 (Routing Setup) → S2 (Job List Page) → S3 (Job Detail Page)
2. **Frontend Agent**: Run self-tests after each story
3. **Test Agent**: S4 (E2E Testing & Code Review)

### Git Workflow

Each agent:
1. Creates feature branch: `feature/job-list-redesign`
2. Implements tasks following TDD
3. Runs self-tests
4. Commits with conventional commit messages
5. Pushes to remote
6. Creates PR

### Communication Protocol

- Agent → Team Lead: "I'm done" when story complete
- Test Agent → Team Lead: "Tests passed" or "Tests failed: [details]"
- Team Lead → Agent: Task assignments and fixes

---
**Version**: 1.0
**Created**: 2026-02-09
**Epic ID**: EPIC-002
