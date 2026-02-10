# EPIC-002-S2: Job List Page Implementation

## Overview
**Epic**: EPIC-002
**Repository**: `modules/frontend`
**Feature Branch**: `feature/job-list-redesign`
**Agent**: Frontend

## Context
This story implements the new Job List Page with vertical layout, redesigned JobCard component, URL-based filter state, and mobile filter toggle.

Refer to:
- PRD Section 2.2 (Feature F1, F2, F4, F5, F7)
- Solution Architecture Section 4 (Data Flow)

## Tasks

### TASK-2-1: Refactor Layout to Vertical

**Description**: Change JobListPage from horizontal grid to vertical stack.

**Detailed Steps**:
1. Open `src/pages/JobListPage.tsx`
2. Remove any `grid`, `grid-cols-12` classes
3. Change to flex-col layout with gap:
   ```tsx
   <div className="flex flex-col gap-4">
     <SearchBar />
     <FilterPanel />
     <JobList />
   </div>
   ```
4. Remove col-span classes from FilterPanel and JobList components
5. Test on desktop (1920x1080) and mobile (375x667)

**Acceptance Criteria**:
- [ ] Components stack vertically (Search → Filter → List)
- [ ] FilterPanel spans full width
- [ ] JobList spans full width
- [ ] Responsive design maintained (mobile still usable)
- [ ] No horizontal scroll on any screen size

**Testing**:
- Visual: Check desktop view, verify vertical stack
- Visual: Check mobile view (375px width)
- Browser DevTools: Verify no overflow-x

---

### TASK-2-2: Redesign JobCard Content

**Description**: Update JobCard to show new content structure.

**Detailed Steps**:
1. Open `src/components/JobCard.tsx`
2. Restructure the card content:
   ```tsx
   <Card className="hover:shadow-lg transition-shadow cursor-pointer">
     <Link to={`/jobs/${job.id}`} className="block">
       {/* Job Title */}
       <h3 className="text-xl font-bold">{job.title}</h3>

       {/* Company Info Bar */}
       <div className="flex flex-wrap gap-2 mt-2">
         <Badge>{job.company}</Badge>
         <Badge>{job.location}</Badge>
         <Badge variant="secondary">{job.workArrangement}</Badge>
       </div>

       {/* JD Preview */}
       <p className="mt-3 text-gray-600 line-clamp-3">
         {job.description}
       </p>
     </Link>
   </Card>
   ```
3. Import Link from react-router-dom
4. Add hover effects for better UX
5. Test visual appearance

**Acceptance Criteria**:
- [ ] Job title is large (text-xl) and bold
- [ ] Company, location, work type shown as badges
- [ ] Job description truncated to 3 lines (line-clamp-3)
- [ ] Entire card is clickable
- [ ] Hover effect shows interactivity
- [ ] Clicking navigates to /jobs/:id

**Testing**:
- Visual: Check card appearance matches design
- Manual: Click card, verify navigation
- Browser DevTools: Check Link href is correct

---

### TASK-2-3: Implement Filter State in URL

**Description**: Sync filter state with URL query parameters.

**Detailed Steps**:
1. Update `src/pages/JobListPage.tsx`:
   ```tsx
   const [searchParams] = useSearchParams();
   const keyword = searchParams.get('keyword') || '';
   const location = searchParams.get('location') || '';
   const page = parseInt(searchParams.get('page') || '1');
   ```
2. Update `SearchBar` component:
   - Accept `value` and `onChange` props
   - Call `onChange` with new keyword
   - Parent updates URL: `setSearchParams({ keyword: newValue })`
3. Update `FilterPanel` component:
   - Read initial values from URL params
   - Update URL when filters change
4. Update `Pagination` component:
   - Read page from URL params
   - Update URL when page changes
5. Test that URL updates and state persists on refresh

**Acceptance Criteria**:
- [ ] Filter changes update URL query parameters
- [ ] URL params populate filters on page load
- [ ] Page refresh preserves all filter state
- [ ] Browser back/forward buttons work with filter history
- [ ] Direct URL visit (e.g., /jobs?keyword=frontend) works

**Testing**:
- Manual: Change filter, verify URL updates
- Manual: Refresh page, verify filters persist
- Manual: Use browser back button, verify state changes

---

### TASK-2-4: Implement Mobile Filter Toggle

**Description**: Add collapsible filter panel for mobile devices.

**Detailed Steps**:
1. Create `src/components/FilterToggle.tsx`:
   ```tsx
   export function FilterToggle({ show, onToggle }) {
     return (
       <button
         onClick={onToggle}
         className="md:hidden w-full py-2 px-4 bg-gray-100"
       >
         Filter {show ? '▲' : '▼'}
       </button>
     );
   }
   ```
2. Update `JobListPage.tsx`:
   - Add state: `const [showFilter, setShowFilter] = useState(false)`
   - Add conditional rendering: `showFilter || window.innerWidth >= 768`
   - Wrap FilterPanel in responsive container
3. Add CSS transition for smooth animation
4. Test on mobile viewport (375px width)

**Acceptance Criteria**:
- [ ] Filter toggle button visible only on mobile (< 768px)
- [ ] FilterPanel hidden by default on mobile
- [ ] Toggle button shows/hides FilterPanel
- [ ] Smooth transition animation
- [ ] Desktop view unaffected (FilterPanel always visible)

**Testing**:
- Mobile (375px): Verify toggle button appears
- Mobile: Verify FilterPanel hidden by default
- Mobile: Click toggle, verify FilterPanel appears
- Desktop (1024px): Verify toggle button hidden
- Desktop: Verify FilterPanel always visible

## Git Workflow

### Development Principles
- **SOLID**: Single Responsibility for URL state management
- **KISS**: Use native URLSearchParams, no additional state management
- **DRY**: Reuse existing Badge, Card components
- **TDD**: Write component tests before implementation

### Commit Guidelines
```bash
# After TASK-2-1
git commit -m "refactor(layout): change job list to vertical layout"

# After TASK-2-2
git commit -m "feat(job-card): redesign with new content structure"

# After TASK-2-3
git commit -m "feat(filters): sync filter state with URL params"

# After TASK-2-4
git commit -m "feat(mobile): add collapsible filter panel toggle"

git push origin feature/job-list-redesign
```

### PR Creation
Update existing PR with new commits.

## Completion Checklist
- [ ] All 4 tasks completed
- [ ] TypeScript compilation passing
- [ ] No console errors
- [ ] Layout responsive on all screen sizes
- [ ] URL state sync working
- [ ] Mobile toggle working
- [ ] Commits pushed to remote
- [ ] PR updated
- [ ] Notify Team Lead: "I'm done with EPIC-002-S2"

---
**Version**: 1.0
**Created**: 2026-02-09
**Epic ID**: EPIC-002
**Story ID**: EPIC-002-S2
