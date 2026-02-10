# PRD - Landing Page & Login/Registration

**Status**: ✅ **COMPLETED** (2026-02-10)

## Completion Summary
- Backend PR #2: Merged
- Frontend PR #3: Merged
- All stories (S1-S6) completed
- Integration tested and working

## 1. Project Overview

### 1.1 Project Name
**Landing Page & User Authentication System**

### 1.2 Background
491JobSeeker is a job platform designed for regional visa holders in Australia. Currently, the platform only has a job listing page. To acquire new users, we need a landing page that:

1. Tells the product story and explains the value proposition
2. Builds trust with the target audience (491/494 visa holders)
3. Provides clear user onboarding through login/registration

### 1.3 Target Users
- **Primary**: Holders of Australian regional visas (491, 494, etc.)
- **Locations**: Canberra, Adelaide, Perth, and other regional areas
- **Professions**: IT/Tech professionals, skilled migrants
- **Pain Points**:
  - Jobs are scattered across multiple platforms
  - Visa restrictions are unclear in job postings
  - Risk of applying to non-compliant roles
  - Time-consuming manual filtering

## 2. Core Functional Requirements

### 2.1 Feature List

| ID | Feature Name | Priority | Description |
|----|--------------|----------|-------------|
| F1 | Landing Page Hero Section | P0 | Main visual section with value proposition and CTAs |
| F2 | How It Works Section | P0 | 3-step process explanation |
| F3 | Why Choose Us Section | P0 | 4 core differentiators |
| F4 | Login Page | P0 | Email/password authentication |
| F5 | Registration Page | P0 | Multi-step registration with email verification |
| F6 | User Profile Page | P1 | Basic profile display and edit |
| F7 | Navigation Header | P0 | Site-wide navigation with auth state |
| F8 | Email Verification Service | P0 | Send and verify email confirmation codes |

### 2.2 Feature Details

#### F1: Landing Page Hero Section
- **Headline**: "Find Jobs That Accept Your 491 Visa"
- **Subheadline**: "Australia's first job platform built for regional visa holders"
- **Primary CTA**: "Get Started Free" → `/register`
- **Secondary CTA**: "Browse Jobs" → `/jobs`
- **Visual**: Professional photo of regional Australian professionals working
- **Design**: Match existing job listing page design system (Tailwind CSS)

#### F2: How It Works Section
Three-step process with icons:
1. **Tell us your visa type and skills** — User creates profile
2. **We match you with compliant regional jobs** — AI-powered filtering
3. **Apply with AI-optimized resumes** — One-click applications

#### F3: Why Choose Us Section
Four key differentiators:
- ✓ Visa-compliant jobs only
- ✓ Regional + Remote opportunities
- ✓ AI-powered resume matching
- ✓ Save time, reduce risk

#### F4: Login Page
- Email input field
- Password input field (masked)
- "Forgot password" link (future feature)
- "Don't have an account? Sign up" link
- Submit button
- Form validation (email format, password required)
- Error handling for invalid credentials
- On success: Redirect to `/profile`
- Design match: Use existing UI components (Input, Button, Card)

#### F5: Registration Page
**Step 1: Account Creation**
- Email input
- Password input (min 8 characters)
- Confirm password input
- "Send verification code" button
- Email verification code input (6-digit)
- Timer for resend code (60 seconds)
- Validation: Email format, password match

**Step 2: Personal Information** (after email verified)
- Full name (required)
- Visa type (dropdown: 491, 494, Other)
- Visa expiry date (date picker)
- LinkedIn URL (optional)
- Preferred job location (optional, dropdown: Canberra, Adelaide, Perth, Other)

**Flow:**
1. User submits email → Send verification email
2. User enters code → Verify code
3. If valid → Show Step 2 form
4. User completes Step 2 → Create account, auto-login, redirect to `/profile`

#### F6: User Profile Page
Display user information:
- Name
- Email
- Visa type and expiry
- LinkedIn URL (if provided)
- Preferred location
- "Edit Profile" button (future feature)

#### F7: Navigation Header
- Logo/Brand name (491JobSeeker)
- Links: Home, Browse Jobs
- Auth state aware:
  - Not logged in: "Log In", "Sign Up" buttons
  - Logged in: User avatar/name dropdown → Profile, Logout

#### F8: Email Verification Service
- Generate 6-digit verification code
- Send email with code
- Code expires in 15 minutes
- Store code securely (hashed)
- Verify code API endpoint

## 3. Non-Functional Requirements

### 3.1 Performance
- Landing page load time: < 2 seconds
- Email verification send: < 5 seconds
- Form validation: Instant feedback
- Responsive: Mobile, tablet, desktop

### 3.2 Compatibility
- Modern browsers (Chrome, Firefox, Safari, Edge)
- Mobile devices (iOS Safari, Chrome Mobile)
- Screen sizes: 320px - 1920px+

### 3.3 Code Quality
- Follow SOLID principles
- KISS: Keep forms simple and straightforward
- DRY: Reuse existing UI components
- TypeScript strict mode
- Unit tests for all business logic
- E2E tests for critical user flows

### 3.4 Security
- Passwords hashed (bcrypt or similar)
- JWT tokens for authentication
- HTTPS only for production
- Rate limiting on email verification
- Input sanitization
- XSS protection

## 4. Data Sources

### 4.1 Database Schema

**Users Collection**
```javascript
{
  _id: ObjectId,
  email: String (unique, indexed),
  passwordHash: String,
  emailVerified: Boolean (default: false),
  verificationCode: String,
  verificationCodeExpires: Date,
  profile: {
    name: String,
    visaType: String, // '491', '494', 'other'
    visaExpiry: Date,
    linkedInUrl: String,
    preferredLocation: String
  },
  createdAt: Date,
  updatedAt: Date
}
```

### 4.2 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/auth/register | Create account, send verification email |
| POST | /api/auth/verify-email | Verify email code |
| POST | /api/auth/login | Authenticate user |
| GET | /api/auth/me | Get current user profile |
| POST | /api/auth/logout | Logout user |
| GET | /api/auth/verify-token | Verify JWT token validity |

## 5. Success Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Landing page conversion rate | > 5% | (Sign-ups / Landing page visitors) × 100% |
| Email verification completion | > 80% | (Verified emails / Sent emails) × 100% |
| Registration completion | > 60% | (Completed registrations / Started registrations) × 100% |
| Time to register | < 3 minutes | Average time from start to successful registration |
| Page load time | < 2s | Lighthouse / Web Vitals |

## 6. Tech Stack

### 6.1 Frontend
- **Framework**: React 19 + TypeScript
- **Styling**: Tailwind CSS (existing design system)
- **Routing**: React Router DOM v6
- **State**: React Query for server state
- **Components**: Existing UI component library (Button, Input, Card)
- **Forms**: React Hook Form + Zod validation
- **Icons**: Lucide React

### 6.2 Backend
- **Framework**: Node.js + Express (or existing backend)
- **Database**: MongoDB
- **Authentication**: JWT (jsonwebtoken)
- **Email**: Nodemailer or SendGrid
- **Password Hashing**: bcrypt
- **Validation**: Zod (shared types with frontend)

### 6.3 Testing
- **Frontend**: Vitest + Testing Library
- **E2E**: Playwright
- **API**: Supertest (if backend testing in this epic)

## 7. Development Method

- **TDD**: Write tests before implementation
- **Agent Teams**: Backend, Frontend, Test agents working in parallel
- **Git Workflow**: Feature branches with pull requests
- **Code Review**: Pragmatic Clean Code Reviewer skill

---

**Version**: 1.0
**Created**: 2026-02-10
**Epic ID**: EPIC-003
