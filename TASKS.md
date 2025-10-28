# Developer Tasks: Helping Hand Charity Platform

This document breaks down the development of the Helping Hand Charity Platform into small, actionable tasks. Each task is designed to be implementable in 1-3 days.

## Foundation Tasks

### Task 1: Project Setup
- **Deliverables**: A new Next.js 14 project with TypeScript, Tailwind CSS, ESLint, Prettier, and Husky configured according to `CONSTITUTION.md`.
- **Acceptance Criteria**: The project can be started locally (`npm run dev`). Linting and formatting rules are enforced on pre-commit hooks.
- **Dependencies**: None
- **Complexity**: Simple
- **Files to create/modify**: `package.json`, `.eslintrc.json`, `prettier.config.js`, `next.config.js`, `.husky/pre-commit`, `tsconfig.json`, `src/styles/globals.css`.

### Task 2: Database Setup & Initial Migration
- **Deliverables**: A running PostgreSQL database with the schema from `TECHNICAL_PLAN.md` applied.
- **Acceptance Criteria**: The Prisma client can connect to the database. The initial migration is successfully applied, creating all tables.
- **Dependencies**: Task 1
- **Complexity**: Simple
- **Files to create/modify**: `prisma/schema.prisma`, `.env`, `src/lib/prisma.ts`.
- **Database migrations**: The initial migration to create all tables.

### Task 3: Authentication Setup (Email/Password & Google OAuth)
- **Deliverables**: A fully functional authentication system using `next-auth`.
- **Acceptance Criteria**: Users can register, log in, and log out using both email/password and Google OAuth. A valid session is created and can be retrieved. Protected routes are inaccessible to unauthenticated users.
- **Dependencies**: Task 2
- **Complexity**: Complex
- **Files to create/modify**:
  - `src/app/(auth)/login/page.tsx`
  - `src/app/(auth)/register/page.tsx`
  - `src/app/api/auth/[...nextauth]/route.ts`
  - `src/lib/auth.ts`
  - `src/middleware.ts`
- **Components to build**:
  - `src/components/organisms/LoginForm.tsx`
  - `src/components/organisms/RegisterForm.tsx`
  - `src/components/molecules/GoogleSignInButton.tsx`
- **API endpoints to implement**:
  - `POST /api/v1/auth/register`
  - `POST /api/v1/auth/login`
  - `POST /api/v1/auth/logout`
  - `GET /api/v1/auth/session`
- **Tests to write**: Unit tests for authentication logic and API endpoints.


## Core Feature Tasks

### Task 4: User Profile Management
- **Deliverables**: A user profile page where users can view and update their name and avatar.
- **Acceptance Criteria**: Authenticated users can access their profile page. They can update their name and avatar, and the changes are reflected in the database.
- **Dependencies**: Task 3
- **Complexity**: Medium
- **Files to create/modify**:
  - `src/app/(dashboard)/profile/page.tsx`
- **Components to build**:
  - `src/components/organisms/ProfileForm.tsx`
- **API endpoints to implement**:
  - `GET /api/v1/users/me`
  - `PUT /api/v1/users/me`
- **Tests to write**: Unit tests for the profile update API endpoint.

### Task 5: Donation System (Guest Checkout)
- **Deliverables**: A public donation page where guest users can make a one-time donation.
- **Acceptance Criteria**: Users can select a campaign, enter a donation amount, and see bank transfer details. A donation record is created in the database with a `null` user ID.
- **Dependencies**: Task 2
- **Complexity**: Medium
- **Files to create/modify**:
  - `src/app/donate/page.tsx`
- **Components to build**:
  - `src/components/organisms/DonationForm.tsx`
- **API endpoints to implement**:
  - `POST /api/v1/donations`
- **Tests to write**: Unit tests for the donation creation API endpoint.

### Task 6: Beneficiary Application System
- **Deliverables**: A system for beneficiaries to register and submit assistance requests.
- **Acceptance Criteria**: Beneficiaries can create a profile and submit an application with all required details and documents. The application is saved in the database with a "Submitted" status.
- **Dependencies**: Task 3
- **Complexity**: Complex
- **Files to create/modify**:
  - `src/app/(dashboard)/beneficiary/apply/page.tsx`
  - `src/app/(dashboard)/beneficiary/requests/page.tsx`
- **Components to build**:
  - `src/components/organisms/BeneficiaryApplicationForm.tsx`
  - `src/components/organisms/RequestList.tsx`
- **API endpoints to implement**:
  - `POST /api/v1/beneficiaries/:id/requests`
  - `GET /api/v1/beneficiaries/:id/requests`
- **Tests to write**: Unit tests for the application submission and retrieval API endpoints.


## Dashboard Tasks

### Task 7: Admin Dashboard - User Management
- **Deliverables**: A user management interface within the admin dashboard.
- **Acceptance Criteria**: Admins can view a list of all users, update user roles, and soft-delete users.
- **Dependencies**: Task 4
- **Complexity**: Medium
- **Files to create/modify**:
  - `src/app/(dashboard)/admin/users/page.tsx`
- **Components to build**:
  - `src/components/organisms/UserManagementTable.tsx`
- **API endpoints to implement**:
  - `GET /api/v1/users`
  - `PUT /api/v1/users/:id`
  - `DELETE /api/v1/users/:id`
  - `PUT /api/v1/users/:id/role`
- **Tests to write**: Unit tests for all user management API endpoints.

### Task 8: Admin Dashboard - Beneficiary Request Management
- **Deliverables**: An interface for admins to review and manage beneficiary requests.
- **Acceptance Criteria**: Admins can view a queue of submitted requests, see application details, and update the status of a request (e.g., "Approved", "Rejected").
- **Dependencies**: Task 6
- **Complexity**: Complex
- **Files to create/modify**:
  - `src/app/(dashboard)/admin/requests/page.tsx`
  - `src/app/(dashboard)/admin/requests/[id]/page.tsx`
- **Components to build**:
  - `src/components/organisms/RequestQueue.tsx`
  - `src/components/organisms/RequestDetails.tsx`
- **API endpoints to implement**:
  - `GET /api/v1/requests`
  - `GET /api/v1/requests/:id`
  - `PUT /api/v1/requests/:id/status`
- **Tests to write**: Unit tests for the request management API endpoints.

### Task 9: Donor Dashboard - Donation History
- **Deliverables**: A page for registered donors to view their donation history.
- **Acceptance Criteria**: Logged-in users with a donor role can see a list of their past donations, including amount, date, and campaign.
- **Dependencies**: Task 3, Task 5
- **Complexity**: Simple
- **Files to create/modify**:
  - `src/app/(dashboard)/donor/history/page.tsx`
- **Components to build**:
  - `src/components/organisms/DonationHistoryTable.tsx`
- **API endpoints to implement**:
  - `GET /api/v1/users/:userId/donations`
- **Tests to write**: Unit tests for the donation history API endpoint.

### Task 10: Beneficiary Dashboard - Application Status Tracking
- **Deliverables**: A page for beneficiaries to track the status of their applications.
- **Acceptance Criteria**: Beneficiaries can view the real-time status of their submitted requests (e.g., "Submitted," "Under Review," "Approved").
- **Dependencies**: Task 6
- **Complexity**: Simple
- **Files to create/modify**:
  - `src/app/(dashboard)/beneficiary/requests/page.tsx` (Enhance this page)
- **Components to build**:
  - `src/components/molecules/RequestStatusTimeline.tsx`
- **API endpoints to implement**:
  - The `GET /api/v1/beneficiaries/:id/requests` endpoint will be consumed by the new component.
- **Tests to write**: Component tests for the status timeline.


## Integration Tasks

### Task 11: Email Service Integration
- **Deliverables**: An email service integration (e.g., using Resend or SendGrid) for sending transactional emails.
- **Acceptance Criteria**: The system sends a welcome email upon registration and a confirmation email after a donation is made.
- **Dependencies**: Task 3, Task 5
- **Complexity**: Medium
- **Files to create/modify**:
  - `src/lib/email.ts`
  - Logic within the registration and donation API endpoints to trigger emails.
- **Tests to write**: Unit tests that mock the email service and verify that it is called with the correct parameters.

### Task 12: Analytics Integration
- **Deliverables**: Integration with a web analytics service (e.g., Google Analytics or Vercel Analytics).
- **Acceptance Criteria**: Page views and key events (e.g., donations, registrations) are tracked in the analytics platform.
- **Dependencies**: Task 1
- **Complexity**: Simple
- **Files to create/modify**:
  - `src/app/layout.tsx`
  - `src/lib/analytics.ts`
- **Tests to write**: N/A


## Polish Tasks

### Task 13: Comprehensive Unit & Integration Testing
- **Deliverables**: A test suite with at least 80% coverage for all business logic and API endpoints.
- **Acceptance Criteria**: All tests pass in a CI/CD environment.
- **Dependencies**: All previous tasks
- **Complexity**: Complex
- **Files to create/modify**:
  - `*.test.ts` files throughout the codebase.
- **Tests to write**: As described in the deliverables.

### Task 14: Frontend & Backend Performance Optimization
- **Deliverables**: An optimized application that meets the performance requirements in `CONSTITUTION.md`.
- **Acceptance Criteria**: Page load times are under 3 seconds on a 3G network. Core Web Vitals scores are "good." N+1 queries are identified and removed.
- **Dependencies**: All previous tasks
- **Complexity**: Medium
- **Files to create/modify**:
  - `next.config.js` (for bundle analysis)
  - Various components and API endpoints to implement caching and other optimizations.
- **Tests to write**: N/A (performance will be measured with tools like Lighthouse and Vercel Analytics).

### Task 15: Deployment to Production
- **Deliverables**: A production-ready deployment of the application on Vercel.
- **Acceptance Criteria**: The application is live and accessible to the public. HTTPS is enforced. Environment variables are configured correctly for production.
- **Dependencies**: All previous tasks
- **Complexity**: Medium
- **Files to create/modify**:
  - `vercel.json` (if needed)
- **Tests to write**: N/A
