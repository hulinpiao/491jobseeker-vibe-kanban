# EPIC-002-S3: Job Detail Page Implementation

## Overview
**Epic**: EPIC-002
**Repository**: `modules/frontend`
**Feature Branch**: `feature/job-list-redesign`
**Agent**: Frontend

## Context
This story implements the Job Detail Page with data fetching, proper layout, action buttons, and back navigation with state preservation.

Refer to:
- PRD Section 2.2 (Feature F3, F6)
- Solution Architecture Section 3 (API Design)

## Tasks

### TASK-3-1: Implement Job Data Fetching

**Description**: Fetch job data based on URL parameter using React Query.

**Detailed Steps**:
1. Update `src/pages/JobDetailPage.tsx`:
   ```tsx
   import { useParams } from 'react-router-dom';
   import { useQuery } from '@tanstack/react-query';
   import { fetchJobById } from '@/services/api';

   export function JobDetailPage() {
     const { jobId } = useParams<{ jobId: string }>();

     const { data: job, isLoading, error, isError } = useQuery({
       queryKey: ['job', jobId],
       queryFn: () => fetchJobById(jobId!),
       enabled: !!jobId,
     });

     if (isLoading) return <JobDetailSkeleton />;
     if (isError) return <JobDetailError error={error} />;
     if (!job) return <JobNotFound />;

     return <JobDetailContent job={job} />;
   }
   ```
2. Create `JobDetailSkeleton` component
3. Create `JobDetailError` component with retry button
4. Create `JobNotFound` component with back button
5. Test all states

**Acceptance Criteria**:
- [ ] JobId extracted from URL parameter
- [ ] Job data fetched using fetchJobById API
- [ ] Loading state shows skeleton/spinner
- [ ] Error state shows message with retry button
- [ ] 404 state shows "Job not found" with back button
- [ ] Data displays correctly when loaded

**Testing**:
- Manual: Visit /jobs/valid-id, verify job loads
- Manual: Visit /jobs/invalid-id, verify 404 state
- Manual: Disconnect network, verify error state

---

### TASK-3-2: Design Job Detail Layout

**Description**: Create layout for job detail page content.

**Detailed Steps**:
1. Create `JobDetailContent` component:
   ```tsx
   export function JobDetailContent({ job }) {
     return (
       <div className="max-w-4xl mx-auto p-6">
         {/* Back Button */}
         <BackButton />

         {/* Job Header */}
         <Card className="mb-6">
           <h1 className="text-3xl font-bold">{job.title}</h1>
           <div className="flex flex-wrap gap-2 mt-4">
             <Badge>{job.company}</Badge>
             <Badge>{job.location}</Badge>
             <Badge variant="secondary">{job.workArrangement}</Badge>
             <Badge variant="outline">{job.employmentType}</Badge>
           </div>
         </Card>

         {/* Action Buttons - Right aligned */}
         <div className="flex justify-end gap-2 mb-6">
           {/* Buttons added in TASK-3-3 */}
         </div>

         {/* Full Description */}
         <Card>
           <h2 className="text-xl font-semibold mb-4">Job Description</h2>
           <div className="prose max-w-none">
             {job.description}
           </div>
         </Card>
       </div>
     );
   }
   ```
2. Add proper spacing and typography
3. Ensure responsive design

**Acceptance Criteria**:
- [ ] Back button visible (top left)
- [ ] Job title prominent (text-3xl, bold)
- [ ] Job info displayed with badges
- [ ] Full description readable with proper formatting
- [ ] Layout responsive (mobile and desktop)
- [ ] Max width constraint for readability

**Testing**:
- Visual: Check layout at 1920px width
- Visual: Check layout at 375px width
- Manual: Verify all job info displays correctly

---

### TASK-3-3: Add Action Buttons

**Description**: Add three action buttons to detail page (disabled state).

**Detailed Steps**:
1. Create action buttons in `JobDetailContent`:
   ```tsx
   <div className="flex justify-end gap-2 mb-6">
     <Button
       disabled
       title="Apply functionality coming soon"
     >
       Apply
     </Button>
     <Button
       disabled
       title="Resume customization coming soon"
       variant="secondary"
     >
       Customize Resume
     </Button>
     <Button
       disabled
       title="CV generation coming soon"
       variant="outline"
     >
       Generate CV
     </Button>
   </div>
   ```
2. Style buttons consistently with design system
3. Ensure disabled state is visually clear
4. Test hover tooltips

**Acceptance Criteria**:
- [ ] Three buttons visible (Apply, Customize Resume, Generate CV)
- [ ] All buttons in disabled state
- [ ] Hover shows "Coming Soon" tooltip
- [ ] Buttons aligned to right
- [ ] No console warnings

**Testing**:
- Visual: Verify all three buttons render
- Manual: Hover over each button, verify tooltip
- Browser Console: Check for React warnings

---

### TASK-3-4: Implement Back Navigation

**Description**: Handle back button navigation with state preservation.

**Detailed Steps**:
1. Create `BackButton` component:
   ```tsx
   import { useNavigate } from 'react-router-dom';

   export function BackButton() {
     const navigate = useNavigate();

     const handleBack = () => {
       // Navigate back to jobs list
       navigate('/jobs');
     };

     return (
       <button
         onClick={handleBack}
         className="flex items-center gap-2 text-gray-600 hover:text-gray-900"
       >
         ← Back to Jobs
       </button>
     );
   }
   ```
2. Alternatively, use browser back button:
   ```tsx
   import { useNavigate } from 'react-router-dom';

   const navigate = useNavigate();
   const handleBack = () => navigate(-1);
   ```
3. Test navigation preserves filter state
4. Test both in-page back button and browser back button

**Acceptance Criteria**:
- [ ] Back button returns to /jobs
- [ ] Filter state preserved when returning
- [ ] Works from in-page back button
- [ ] Works from browser back button
- [ ] Page scroll position reset on return

**Testing**:
- Manual: Set filters, click job, click back, verify filters persist
- Manual: Use browser back button, verify same behavior
- Manual: Check URL parameters are preserved

## Git Workflow

### Development Principles
- **SOLID**: Single Responsibility for each sub-component
- **KISS**: Simple navigation using react-router hooks
- **DRY**: Reuse existing Card, Badge, Button components
- **TDD**: Write tests for data fetching logic

### Commit Guidelines
```bash
# After TASK-3-1
git commit -m "feat(detail-page): implement job data fetching"

# After TASK-3-2
git commit -m "feat(detail-page): design job detail layout"

# After TASK-3-3
git commit -m "feat(detail-page): add action buttons (disabled)"

# After TASK-3-4
git commit -m "feat(detail-page): implement back navigation with state preservation"

git push origin feature/job-list-redesign
```

### PR Creation
Update existing PR with new commits.

## Completion Checklist
- [ ] All 4 tasks completed
- [ ] TypeScript compilation passing
- [ ] No console errors
- [ ] Job detail page loads with valid job ID
- [ ] 404 state works with invalid job ID
- [ ] Back navigation preserves filter state
- [ ] All three buttons displayed (disabled)
- [ ] Commits pushed to remote
- [ ] PR updated
- [ ] Notify Team Lead: "I'm done with EPIC-002-S3"

---
**Version**: 1.0
**Created**: 2026-02-09
**Epic ID**: EPIC-002
**Story ID**: EPIC-002-S3
