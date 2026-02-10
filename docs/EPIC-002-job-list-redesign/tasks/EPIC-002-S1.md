# EPIC-002-S1: Routing Setup & Layout Refactor

## Overview
**Epic**: EPIC-002
**Repository**: `modules/frontend`
**Feature Branch**: `feature/job-list-redesign`
**Agent**: Frontend

## Context
This story establishes the foundation for client-side routing in the application. We need to install react-router-dom, configure the routing structure, and migrate existing components to a page-based architecture.

Refer to:
- PRD Section 2.1 (Feature F1, F2, F3)
- Solution Architecture Section 1.2, 2.1

## Tasks

### TASK-1-1: Install Dependencies

**Description**: Install react-router-dom and configure TypeScript types.

**Detailed Steps**:
1. Navigate to modules/frontend directory
2. Run `npm install react-router-dom@^6.22.0`
3. Verify package.json includes the dependency
4. Run `npm run type-check` to ensure no TypeScript errors

**Acceptance Criteria**:
- [ ] react-router-dom@^6.22.0 in package.json dependencies
- [ ] npm install completed without errors
- [ ] TypeScript recognizes react-router-dom types
- [ ] `npm run type-check` passes

**Testing**:
- Verify import: `import { BrowserRouter } from 'react-router-dom'` works

---

### TASK-1-2: Configure BrowserRouter

**Description**: Wrap App with BrowserRouter in main.tsx.

**Detailed Steps**:
1. Open `src/main.tsx`
2. Import BrowserRouter: `import { BrowserRouter } from 'react-router-dom'`
3. Wrap the existing `<App />` with `<BrowserRouter>`
4. Start dev server and verify app renders

**Acceptance Criteria**:
- [ ] BrowserRouter wraps App component
- [ ] No console errors on startup
- [ ] Dev server runs successfully on localhost:5173
- [ ] Existing functionality still works

**Testing**:
- Manual: Start dev server, verify app loads
- Check browser console for errors

---

### TASK-1-3: Create Route Configuration

**Description**: Add Routes and Route components to App.tsx.

**Detailed Steps**:
1. Create `src/pages/JobListPage.tsx`:
   ```tsx
   export function JobListPage() {
     return <div>Job List Page - Placeholder</div>;
   }
   ```
2. Create `src/pages/JobDetailPage.tsx`:
   ```tsx
   export function JobDetailPage() {
     return <div>Job Detail Page - Placeholder</div>;
   }
   ```
3. Update `src/App.tsx`:
   - Import Routes, Route, Navigate from react-router-dom
   - Import JobListPage and JobDetailPage
   - Add route configuration:
     ```tsx
     <Routes>
       <Route path="/" element={<JobListPage />} />
       <Route path="/jobs/:jobId" element={<JobDetailPage />} />
     </Routes>
     ```
4. Test navigation manually

**Acceptance Criteria**:
- [ ] Routes configured in App.tsx
- [ ] Navigating to / shows "Job List Page - Placeholder"
- [ ] Navigating to /jobs/test-job-id shows "Job Detail Page - Placeholder"
- [ ] No console errors
- [ ] Existing SearchBar, FilterPanel, JobList removed from App.tsx

**Testing**:
- Manual: Visit http://localhost:5173/
- Manual: Visit http://localhost:5173/jobs/123
- Browser console: No errors

---

### TASK-1-4: Migrate Existing Components

**Description**: Move existing components to new page structure.

**Detailed Steps**:
1. Copy current App.tsx layout content to JobListPage.tsx:
   - Import SearchBar, FilterPanel, JobList, Pagination
   - Keep the existing structure but remove JobDetail
   - Use flex-col layout instead of grid
2. Copy JobDetail component content to JobDetailPage.tsx:
   - Import the JobDetail component
   - Pass mock data for now (will be replaced in S3)
3. Remove the old three-column layout from App.tsx
4. Test basic navigation between pages

**Acceptance Criteria**:
- [ ] JobListPage shows SearchBar, FilterPanel, JobList in vertical stack
- [ ] JobListPage does NOT show JobDetail side panel
- [ ] JobDetailPage shows job detail content (placeholder data)
- [ ] Manual navigation between pages works
- [ ] No TypeScript errors
- [ ] No console errors

**Testing**:
- Manual: Navigate to /, verify list page renders
- Manual: Navigate to /jobs/123, verify detail page renders
- Browser console: No errors

## Git Workflow

### Branch Creation
```bash
cd modules/frontend
git checkout main
git pull origin main
git checkout -b feature/job-list-redesign
```

### Development Principles
- **SOLID**: Single Responsibility for each page component
- **KISS**: Simple routing configuration, no over-engineering
- **DRY**: Reuse existing components
- **TDD**: Write tests before implementation (when applicable)

### Commit Guidelines
```bash
# Format: <type>(<scope>): <description>

git add .
git commit -m "feat(deps): add react-router-dom for client-side routing"
git push origin feature/job-list-redesign
```

### PR Creation
```bash
gh pr create \
  --title "feat: implement routing setup and layout refactor" \
  --body "Completes EPIC-002-S1

## Changes
- Installed react-router-dom
- Configured BrowserRouter
- Created JobListPage and JobDetailPage
- Migrated components to page structure

## Checklist
- [ ] All tasks completed
- [ ] No TypeScript errors
- [ ] No console errors
- [ ] Manual testing passed"
```

## Completion Checklist
- [ ] All 4 tasks completed
- [ ] TypeScript compilation passing
- [ ] No console errors
- [ ] Manual navigation works
- [ ] Commits pushed to remote
- [ ] PR created
- [ ] Notify Team Lead: "I'm done with EPIC-002-S1"

---
**Version**: 1.0
**Created**: 2026-02-09
**Epic ID**: EPIC-002
**Story ID**: EPIC-002-S1
