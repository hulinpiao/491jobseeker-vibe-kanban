# Solution Architecture - Job List Page Redesign

## 1. Architecture Overview

### 1.1 Design Principles
- **SOLID:** Single Responsibility for components, Dependency Inversion for routing
- **KISS:** Leverage existing react-router, avoid custom state management
- **DRY:** Reuse existing UI components (Card, Badge, Button)
- **YAGNI:** Only implement required features, defer button actions

### 1.2 High-Level Architecture

**Before:**
```
┌─────────────────────────────────────────────────────────┐
│                      App (Single Page)                   │
│  ┌─────────┬──────────────┬───────────────────────────┐  │
│  │ Filter  │   JobList    │       JobDetail           │  │
│  │ (3 col) │   (4 col)    │        (5 col)            │  │
│  └─────────┴──────────────┴───────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

**After:**
```
┌─────────────────────────────────────────────────────────┐
│                     BrowserRouter                        │
│  ┌───────────────────────────────────────────────────┐  │
│  │                   Routes                          │  │
│  │  ┌────────────────┐    ┌──────────────────────┐  │  │
│  │  │   JobListPage  │    │  JobDetailPage       │  │  │
│  │  │   (/)          │    │  (/jobs/:id)         │  │  │
│  │  │  - SearchBar   │    │  - BackButton        │  │  │
│  │  │  - FilterPanel │    │  - JobInfo           │  │  │
│  │  │  - JobList     │    │  - ActionButtons     │  │  │
│  │  │                │    │  - FullDescription   │  │  │
│  │  └────────────────┘    └──────────────────────┘  │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## 2. Component Design

### 2.1 Frontend Components

#### New Components

| Component | File | Responsibility |
|-----------|------|----------------|
| `JobListPage` | `src/pages/JobListPage.tsx` | Container for list page layout |
| `JobDetailPage` | `src/pages/JobDetailPage.tsx` | Container for detail page |
| `FilterToggle` | `src/components/FilterToggle.tsx` | Mobile filter toggle button |

#### Modified Components

| Component | Changes |
|-----------|---------|
| `App.tsx` | Add BrowserRouter, Routes, Route configuration |
| `JobCard.tsx` | Replace onClick with Link component |
| `JobList.tsx` | Remove col-span, remove selectedJobId logic |
| `JobDetail.tsx` | Convert to page component, use useParams |
| `FilterPanel.tsx` | Remove col-span, add mobile responsive logic |

#### Unchanged Components

| Component | Notes |
|-----------|-------|
| `SearchBar.tsx` | No changes required |
| `Pagination.tsx` | No changes required |
| `ui/*.tsx` | Reuse existing Card, Badge, Button, Input |

### 2.2 Database Schema

No changes required. Existing Job schema is sufficient.

## 3. API Design

### 3.1 Endpoints

No new endpoints required. Use existing:

| Method | Endpoint | Usage |
|--------|----------|-------|
| GET | `/api/jobs` | Job list with filters (existing) |
| GET | `/api/jobs/:id` | Single job details (existing) |

### 3.2 Request/Response Format

No changes to existing API contracts.

### 3.3 Error Handling

| Scenario | Handling |
|----------|----------|
| Job not found (404) | Display 404 page with "Back to List" button |
| Network error | Display error message with retry button |
| Loading state | Display skeleton/spinner |

## 4. Data Flow

### 4.1 User Flow Diagram

```
User enters site
     ↓
JobListPage (/)
     ↓
[User enters keyword, applies filters]
     ↓
URL updates: /jobs?keyword=xxx&location=yyy
     ↓
[User clicks job card]
     ↓
Navigate to JobDetailPage (/jobs/:id?return=/jobs?keyword=xxx)
     ↓
[User clicks back button]
     ↓
Navigate back to JobListPage with preserved filters
```

### 4.2 State Flow Diagram

```
┌─────────────────────────────────────────────────────────┐
│                     URL State                           │
│  ┌───────────────────────────────────────────────────┐  │
│  │ ?keyword=frontend&location=shanghai&page=2&type=  │  │
│  └───────────────────────────────────────────────────┘  │
│                          ↓                               │
│  ┌───────────────────────────────────────────────────┐  │
│  │         useSearchParams() hook                    │  │
│  └───────────────────────────────────────────────────┘  │
│                          ↓                               │
│  ┌───────────────────────────────────────────────────┐  │
│  │    JobListPage Component State                    │  │
│  │  - filters (derived from URL)                     │  │
│  │  - pagination (derived from URL)                  │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## 5. Technology Decisions

### 5.1 Frontend Tech Stack

| Technology | Purpose | Rationale |
|------------|---------|-----------|
| React Router v6 | Client-side routing | Industry standard, excellent TypeScript support |
| URL Search Params | State preservation | Shareable links, browser history support |
| React Query | Data fetching | Already in use, handles caching |

### 5.2 Libraries and Frameworks

**New Dependency:**
```json
{
  "react-router-dom": "^6.22.0"
}
```

**Existing (unchanged):**
- React 18
- TypeScript 5
- Tailwind CSS 3
- React Query (TanStack Query)

## 6. Security Considerations

- **XSS Prevention:** React's built-in escaping protects against XSS
- **URL Injection:** Validate job ID format before API call
- **CSRF:** Not applicable (GET requests only)

## 7. Scalability Considerations

### 7.1 Performance
- Client-side routing eliminates full page reload
- React Query caching reduces API calls
- Code splitting possible for lazy loading pages

### 7.2 Future Extensibility
- Route structure supports additional pages (e.g., `/jobs/:id/apply`)
- Filter state preservation pattern applies to other list pages
- Mobile-first design prepares for PWA

---
**Version**: 1.0
**Created**: 2026-02-09
**Epic ID**: EPIC-002
