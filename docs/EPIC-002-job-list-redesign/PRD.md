# PRD - Job List Page Redesign

## 1. Project Overview

### 1.1 Project Name
Job List Page Redesign - Navigation Layout Refactor

### 1.2 Background
The current job list page uses a three-column layout (Filter | JobList | JobDetail) where clicking a job displays details in a side panel. This design limits the job list width and doesn't provide a dedicated job detail page.

### 1.3 Target Users
- Job seekers browsing and filtering job listings
- Users who want to view full job details in a dedicated page
- Mobile users who need a better browsing experience

## 2. Core Functional Requirements

### 2.1 Feature List

| ID | Feature Name | Priority | Description |
|----|--------------|----------|-------------|
| F1 | Vertical Layout Refactor | P0 | Change from horizontal 3-column to vertical layout (Search → Filter → JobList) |
| F2 | Job List Full Width | P0 | JobList occupies full width for better content display |
| F3 | Separate Job Detail Page | P0 | Navigate to dedicated `/jobs/:id` page when clicking a job |
| F4 | Filter State Preservation | P1 | Preserve search/filter state when navigating between list and detail |
| F5 | Job Card Content Redesign | P1 | Display job title, company, location, work type, and JD preview |
| F6 | Job Detail Page UI | P1 | Display full job info with 3 action buttons (Apply, Customize Resume, Generate CV) |
| F7 | Mobile Filter Toggle | P2 | Collapsible filter panel for mobile devices |

### 2.2 Feature Details

#### F1: Vertical Layout Refactor
- Remove side panel JobDetail component from list page
- Reorder components: SearchBar → FilterPanel → JobList (top to bottom)
- FilterPanel spans full width

#### F2: Job List Full Width
- Remove column constraints (no more col-span-4)
- JobList cards utilize available horizontal space
- Responsive design maintained

#### F3: Separate Job Detail Page
- Implement client-side routing with react-router-dom
- Route: `/jobs/:jobId`
- Back button returns to list with preserved state

#### F4: Filter State Preservation
- Store search keyword, filters, and pagination in URL query strings
- Example: `/jobs?page=2&keyword=frontend&location=shanghai`
- State restored when navigating back from detail page

#### F5: Job Card Content Redesign
Content structure (top to bottom):
1. **Job Title** - Large, bold, prominent
2. **Company Info Bar** - Company name, location, work arrangement (badges)
3. **JD Preview** - First 3-4 lines of job description, truncated with line-clamp
4. **Navigation Hint** - "View Details" arrow/link

#### F6: Job Detail Page UI
- Full job information display
- Three action buttons in top-right corner:
  - Apply (disabled with "Coming Soon" tooltip)
  - Customize Resume based on JD (disabled)
  - Generate CV (disabled)
- Back button to return to list

#### F7: Mobile Filter Toggle
- Filter panel hidden by default on mobile
- Toggle button to show/hide filters
- Smooth transition animation

## 3. Non-Functional Requirements

### 3.1 Performance
- Page transition should be instant (client-side routing)
- Job detail page should load within 500ms
- Filter state restoration should be seamless

### 3.2 Compatibility
- Desktop: Chrome, Firefox, Safari, Edge (latest 2 versions)
- Mobile: iOS Safari, Chrome Mobile (latest version)
- Screen sizes: 320px - 1920px width

### 3.3 Code Quality
- Follow SOLID principles
- KISS: Keep components simple
- DRY: Reuse existing UI components
- TypeScript strict mode enabled

## 4. Data Sources

### 4.1 Database/API
- Existing MongoDB jobs collection
- API endpoints:
  - `GET /api/jobs` - Job list with pagination and filters
  - `GET /api/jobs/:id` - Single job details

### 4.2 Data Structure
Existing Job type (no changes required):
```typescript
interface Job {
  id: string
  title: string
  company: string
  location: string
  city: string
  state: string
  country: string
  description: string
  employmentType: EmploymentType
  workArrangement: WorkArrangement
  applyLink: string
  sources: Array<{ platform, jobId, url }>
  createdAt: string
  updatedAt: string
}
```

## 5. Success Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Page Transition Speed | < 100ms | React DevTools Profiler |
| Job List Width Increase | +125% | Before/After screenshot comparison |
| Mobile Usability Score | Pass | Manual testing on mobile devices |
| Zero Console Errors | 100% | Browser console inspection |
| State Preservation Accuracy | 100% | Navigate back and verify filters |

## 6. Tech Stack

### 6.1 Backend
- No changes required (existing Express + MongoDB)

### 6.2 Frontend
- **New:** `react-router-dom` - Client-side routing
- Existing: React, TypeScript, Tailwind CSS, React Query

### 6.3 Testing
- Unit tests: Vitest
- Component tests: React Testing Library
- E2E tests: Playwright (in modules/test)

## 7. Development Method

- **TDD:** Write tests before implementation
- **Agent Teams:** Frontend Agent for UI changes, Test Agent for validation
- **Git Workflow:** Feature branch `feature/job-list-redesign`, PR review required

---
**Version**: 1.0
**Created**: 2026-02-09
**Epic ID**: EPIC-002
