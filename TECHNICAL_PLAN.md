# Technical Plan: Helping Hand Charity Platform

This document outlines the technical implementation plan for the Helping Hand Charity Management Platform, based on the requirements from `PROJECT_SPEC.md` and adhering to the standards defined in `CONSTITUTION.md`.

## 1. Database Schema (Prisma for PostgreSQL)

Below is the Prisma schema for the PostgreSQL database, designed to meet the requirements of the Helping Hand Charity Platform.

```prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// User Management Tables

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  password      String?
  name          String?
  avatarUrl     String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  deletedAt     DateTime?
  updatedBy     String?   // User ID of the admin who last updated this record

  roles         UserRole[]
  donations     Donation[]
  volunteerProfile Volunteer?
  beneficiaryProfile Beneficiary?
  posts         BlogPost[]
}

model Role {
  id            String    @id @default(cuid())
  name          String    @unique
  permissions   RolePermission[]
  users         UserRole[]
}

model Permission {
  id            String    @id @default(cuid())
  name          String    @unique
  roles         RolePermission[]
}

model RolePermission {
  roleId        String
  permissionId  String
  role          Role      @relation(fields: [roleId], references: [id])
  permission    Permission @relation(fields: [permissionId], references: [id])

  @@id([roleId, permissionId])
}

model UserRole {
  userId        String
  roleId        String
  user          User      @relation(fields: [userId], references: [id])
  role          Role      @relation(fields: [roleId], references: [id])

  @@id([userId, roleId])
}

// Donation System Tables

model Campaign {
  id            String    @id @default(cuid())
  title         String
  description   String
  goalAmount    Float
  currentAmount Float     @default(0)
  startDate     DateTime
  endDate       DateTime
  donations     Donation[]
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

model Donation {
  id            String    @id @default(cuid())
  amount        Float
  userId        String
  campaignId    String
  paymentMethodId String
  createdAt     DateTime  @default(now())
  deletedAt     DateTime?
  updatedBy     String?

  user          User      @relation(fields: [userId], references: [id])
  campaign      Campaign  @relation(fields: [campaignId], references: [id])
  paymentMethod PaymentMethod @relation(fields: [paymentMethodId], references: [id])
}

model PaymentMethod {
  id            String @id @default(cuid())
  name          String // e.g., "Bank Transfer", "Credit Card"
  details       Json   // For bank transfer, this could store account info
  donations     Donation[]
}

// Volunteer Management Tables

model Volunteer {
  id            String    @id @default(cuid())
  userId        String    @unique
  skills        String[]
  availability  String
  user          User      @relation(fields: [userId], references: [id])
  assignments   Assignment[]
}

model Event {
  id            String    @id @default(cuid())
  name          String
  description   String
  date          DateTime
  location      String
  assignments   Assignment[]
}

model Assignment {
  id            String    @id @default(cuid())
  volunteerId   String
  eventId       String
  role          String
  volunteer     Volunteer @relation(fields: [volunteerId], references: [id])
  event         Event     @relation(fields: [eventId], references: [id])
  tasks         Task[]
}

model Task {
  id            String    @id @default(cuid())
  assignmentId  String
  description   String
  completed     Boolean   @default(false)
  assignment    Assignment @relation(fields: [assignmentId], references: [id])
}

// Beneficiary System Tables

model Beneficiary {
  id            String    @id @default(cuid())
  userId        String    @unique
  address       String
  familyInfo    Json
  incomeStatus  String
  user          User      @relation(fields: [userId], references: [id])
  requests      Request[]
  deletedAt     DateTime?
  updatedBy     String?
}

model Request {
  id            String    @id @default(cuid())
  beneficiaryId String
  category      String
  description   String
  status        String    @default("Submitted") // Submitted, Under Review, Approved, Rejected
  beneficiary   Beneficiary @relation(fields: [beneficiaryId], references: [id])
  applications  Application[]
}

model Application {
  id            String    @id @default(cuid())
  requestId     String
  documents     String[]
  request       Request   @relation(fields: [requestId], references: [id])
  distributions Distribution[]
}

model Distribution {
  id            String    @id @default(cuid())
  applicationId String
  items         Json
  distributedAt DateTime  @default(now())
  application   Application @relation(fields: [applicationId], references: [id])
  deletedAt     DateTime?
  updatedBy     String?
}

// Content Management Tables

model BlogPost {
  id            String    @id @default(cuid())
  title         String
  content       String
  authorId      String
  published     Boolean   @default(false)
  author        User      @relation(fields: [authorId], references: [id])
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

model Media {
  id            String    @id @default(cuid())
  url           String
  altText       String
  uploadedAt    DateTime  @default(now())
}

model Page {
  id            String    @id @default(cuid())
  slug          String    @unique
  title         String
  content       String
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

// Audit and Logging Tables

model AuditLog {
  id            String    @id @default(cuid())
  action        String
  timestamp     DateTime  @default(now())
  userId        String?
  details       Json
}
```

## 2. Next.js 14 App Directory Structure

This structure uses the Next.js 14 app router and follows atomic design principles for component organization.

```
/
├── public/
│   ├── images/
│   ├── fonts/
│   └── svgs/
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   ├── register/
│   │   │   │   └── page.tsx
│   │   │   └── layout.tsx
│   │   ├── (dashboard)/
│   │   │   ├── admin/
│   │   │   ├── donor/
│   │   │   ├── volunteer/
│   │   │   └── beneficiary/
│   │   ├── api/
│   │   │   └── v1/
│   │   │       ├── auth/
│   │   │       │   └── [..nextauth]/
│   │   │       │       └── route.ts
│   │   │       ├── donations/
│   │   │       │   └── route.ts
│   │   │       └── ... (other API routes)
│   │   ├── about/
│   │   │   └── page.tsx
│   │   ├── blog/
│   │   │   ├── [slug]/
│   │   │   │   └── page.tsx
│   │   │   └── page.tsx
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/
│   │   ├── atoms/
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   └── ...
│   │   ├── molecules/
│   │   │   ├── FormField.tsx
│   │   │   ├── Card.tsx
│   │   │   └── ...
│   │   ├── organisms/
│   │   │   ├── Header.tsx
│   │   │   ├── Footer.tsx
│   │   │   ├── DonationForm.tsx
│   │   │   └── ...
│   │   └── templates/
│   │       ├── PageLayout.tsx
│   │       └── ...
│   ├── lib/
│   │   ├── prisma.ts
│   │   ├── auth.ts
│   │   └── ...
│   ├── styles/
│   │   └── globals.css
│   ├── types/
│   │   └── index.ts
│   └── middleware.ts
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── .env.local
├── .eslintrc.json
├── next.config.js
├── package.json
└── tsconfig.json
```

## 3. Key API Endpoints

All API endpoints will be versioned under `/api/v1/`.

### Authentication Endpoints (`/api/v1/auth`)
- `POST /api/v1/auth/register`: Register a new user with email and password.
- `POST /api/v1/auth/login`: Log in a user with email and password.
- `POST /api/v1/auth/logout`: Log out the current user.
- `GET /api/v1/auth/session`: Get the current user's session.
- `GET /api/v1/auth/oauth/google`: Initiate Google OAuth2 login.
- `GET /api/v1/auth/oauth/google/callback`: Callback for Google OAuth2.

### Donation Processing (`/api/v1/donations`)
- `POST /api/v1/donations`: Create a new donation.
- `GET /api/v1/donations`: Get a list of all donations (admin only).
- `GET /api/v1/donations/:id`: Get a specific donation.
- `GET /api/v1/users/:userId/donations`: Get all donations for a specific user.

### User Management (`/api/v1/users`)
- `GET /api/v1/users`: Get a list of all users (admin only).
- `GET /api/v1/users/:id`: Get a specific user.
- `PUT /api/v1/users/:id`: Update a user's profile.
- `DELETE /api/v1/users/:id`: Soft delete a user.
- `PUT /api/v1/users/:id/role`: Assign a role to a user (admin only).

### Beneficiary Management (`/api/v1/beneficiaries`)
- `POST /api/v1/beneficiaries/:id/requests`: Create a new request for a beneficiary.
- `GET /api/v1/beneficiaries/:id/requests`: Get all requests for a beneficiary.
- `GET /api/v1/requests`: Get all requests (admin only).
- `GET /api/v1/requests/:id`: Get a specific request.
- `PUT /api/v1/requests/:id/status`: Update the status of a request (admin only).

### Volunteer Management (`/api/v1/volunteers`)
- `GET /api/v1/volunteers`: Get a list of all volunteers.
- `GET /api/v1/volunteers/:id`: Get a specific volunteer's profile.
- `PUT /api/v1/volunteers/:id`: Update a volunteer's profile.
- `GET /api/v1/events`: Get a list of all events.
- `POST /api/v1/events/:id/assign`: Assign a volunteer to an event (admin only).

### Content Management (`/api/v1/content`)
- `POST /api/v1/content/posts`: Create a new blog post.
- `GET /api/v1/content/posts`: Get all blog posts.
- `GET /api/v1/content/posts/:slug`: Get a specific blog post.
- `PUT /api/v1/content/posts/:slug`: Update a blog post.
- `DELETE /api/v1/content/posts/:slug`: Delete a blog post.

### Analytics and Reporting (`/api/v1/analytics`)
- `GET /api/v1/analytics/donations`: Get donation statistics.
- `GET /api/v1/analytics/volunteers`: Get volunteer engagement statistics.
- `GET /api/v1/analytics/beneficiaries`: Get beneficiary impact statistics.

## 4. Component Architecture

The component architecture will follow the atomic design methodology, organized into atoms, molecules, and organisms.

### Shared Components
- **Header (`/src/components/organisms/Header.tsx`)**: Will include navigation, logo, and user authentication status.
- **Footer (`/src/components/organisms/Footer.tsx`)**: Will contain links to important pages, social media, and contact information.
- **Forms (`/src/components/molecules/FormField.tsx`, etc.)**: A set of reusable form components (input, select, textarea) with built-in validation using a library like `react-hook-form` and `zod` for schema validation.

### Page-Specific Components
- Components used on a single page will be co-located with the page in the `src/app` directory. For example, a component specific to the "About Us" page would be in `src/app/about/components/`.

### Dashboard Components
- **Admin Dashboard**: A set of components for managing users, donations, and content. These will be located in `src/app/(dashboard)/admin/components/`.
- **Donor Dashboard**: Components for viewing donation history and managing recurring donations, located in `src/app/(dashboard)/donor/components/`.
- **Volunteer Dashboard**: Components for viewing assignments and logging hours, located in `src/app/(dashboard)/volunteer/components/`.
- **Beneficiary Dashboard**: Components for submitting and tracking requests, located in `src/app/(dashboard)/beneficiary/components/`.

### Form Components with Validation
- We will use `react-hook-form` for managing form state and validation.
- `zod` will be used to define validation schemas.
- Reusable form field components will be created in `/src/components/molecules/` to ensure consistent styling and error handling.

## 5. Authentication & Authorization Flow

We will use `next-auth` to handle both OAuth and traditional email/password authentication.

### Google OAuth2 Implementation
1. The user clicks the "Sign in with Google" button.
2. They are redirected to Google's authentication page.
3. After successful authentication, they are redirected back to the application.
4. `next-auth` handles the callback, creates a user in the database if they don't exist, and creates a session.

### Email/Password Authentication
1. The user enters their email and password in the login form.
2. The credentials are sent to the `/api/v1/auth/login` endpoint.
3. The server validates the credentials, comparing the provided password with the hashed password in the database (using a library like `bcrypt`).
4. If the credentials are valid, a session is created.

### Role-Based Access Control (RBAC)
- RBAC will be implemented using the `User`, `Role`, and `Permission` models in the database.
- The `middleware.ts` file will check the user's role and permissions for protected routes.
- API endpoints will also have checks to ensure the user has the necessary permissions to perform the requested action.

### Protected Routes Structure
- The `middleware.ts` file will be used to protect routes.
- It will check for a valid session and the user's role.
- If the user is not authenticated or does not have the required role, they will be redirected to the login page.

### Session Management
- `next-auth` will manage sessions using JWTs (JSON Web Tokens) stored in cookies.
- The session will contain the user's ID, name, email, and role.
- The session will be automatically refreshed and managed by `next-auth`.
