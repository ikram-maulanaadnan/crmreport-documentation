# CMLabs CRM - API Documentation

Dokumentasi lengkap API CMLabs CRM Backend.

**Base URL:** `http://localhost:4000/api`  
**Version:** 1.0.0

---

## Table of Contents

1. [Authentication](#1-authentication)
2. [Google Authentication](#2-google-authentication)
3. [Leads](#3-leads)
4. [Activities](#4-activities)
5. [Invoices](#5-invoices)
6. [Team](#6-team)
7. [Team Management](#7-team-management)
8. [Dashboard](#8-dashboard)
9. [Notifications](#9-notifications)
10. [Reports](#10-reports)
11. [Profile](#11-profile)
12. [Custom Fields](#12-custom-fields)
13. [Superadmin](#13-superadmin)
14. [Subscriptions](#14-subscriptions)
15. [Webhooks](#15-webhooks)

---

## Response Format

All API responses follow this standard format:

```json
{
  "success": true|false,
  "message": "Human readable message",
  "data": { ... },
  "pagination": { ... } // Optional, for list endpoints
}
```

## Authentication

Most endpoints require authentication via Bearer token in the Authorization header:

```
Authorization: Bearer <accessToken>
```

---

## 1. Authentication

Base Path: `/api/auth`

### 1.1 Sign Up
Register a new user account.

**Endpoint:** `POST /api/auth/signup`

**Headers:**
```
Content-Type: application/json
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "password123",
  "fullName": "John Doe",
  "phone": "+6281234567890" // Optional
}
```

**Validation Rules:**
- `email`: Valid email format, required
- `password`: Min 8 characters, must contain letters and numbers, required
- `fullName`: 2-100 characters, required
- `phone`: Valid mobile phone format, optional

**Response 201 Created:**
```json
{
  "success": true,
  "message": "Registration successful",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
  }
}
```

**Response 409 Conflict:**
```json
{
  "success": false,
  "message": "Email already registered"
}
```

### 1.2 Sign In
Authenticate and get access tokens.

**Endpoint:** `POST /api/auth/signin`

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response 200 OK:**
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
  }
}
```

**Response 401 Unauthorized:**
```json
{
  "success": false,
  "message": "Invalid email or password"
}
```

**Response 423 Locked:**
```json
{
  "success": false,
  "message": "Account is locked. Try again in X minutes"
}
```

### 1.3 Refresh Token
Get new access token using refresh token.

**Endpoint:** `POST /api/auth/refresh-token`

**Request Body:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
  }
}
```

### 1.4 Forgot Password
Request password reset link.

**Endpoint:** `POST /api/auth/forgot-password`

**Request Body:**
```json
{
  "email": "user@example.com"
}
```

**Response 200 OK:**
```json
{
  "success": true,
  "message": "If the email exists, a reset link has been sent"
}
```

### 1.5 Reset Password
Reset password using token from email.

**Endpoint:** `POST /api/auth/reset-password`

**Request Body:**
```json
{
  "token": "reset-token-from-email",
  "password": "newpassword123",
  "confirmPassword": "newpassword123"
}
```

**Response 200 OK:**
```json
{
  "success": true,
  "message": "Password reset successful"
}
```

### 1.6 Logout
Invalidate current session.

**Endpoint:** `POST /api/auth/logout`

**Headers:**
```
Authorization: Bearer <accessToken>
```

**Response 200 OK:**
```json
{
  "success": true,
  "message": "Logout successful"
}
```

### 1.7 Get Current User
Get authenticated user profile.

**Endpoint:** `GET /api/auth/me`

**Headers:**
```
Authorization: Bearer <accessToken>
```

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "user@example.com",
    "fullName": "John Doe",
    "phone": "+6281234567890",
    "location": "Jakarta",
    "bio": "Sales Manager",
    "skills": ["Negotiation", "CRM"],
    "status": "ACTIVE",
    "joinedAt": "2024-01-01T00:00:00Z",
    "reportsTo": {
      "id": "uuid",
      "fullName": "Manager Name"
    },
    "emailDealUpdate": true,
    "emailActivityReminder": true,
    "emailMarketing": false,
    "pushDealUpdate": true,
    "pushActivityReminder": true,
    "teams": [
      {
        "id": "uuid",
        "name": "Sales Team",
        "slug": "sales-team",
        "logo": "https://...",
        "role": { ... },
        "isDefault": true,
        "isOwner": false
      }
    ],
    "currentTeam": { ... }
  }
}
```

---

## 2. Google Authentication

Base Path: `/api/google`

### 2.1 Get Google Auth URL
Get OAuth URL for Google authentication.

**Endpoint:** `GET /api/google/auth-url`

**Query Parameters:**
- `type`: `"login"` or `"link"` (optional, default: "login")

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "authUrl": "https://accounts.google.com/o/oauth2/v2/auth?..."
  }
}
```

### 2.2 Handle Google Callback
OAuth callback endpoint (handled by Google redirect).

**Endpoint:** `GET /api/google/callback`

**Query Parameters:**
- `code`: Authorization code from Google
- `state`: State parameter for CSRF protection

**Response:** Redirects to frontend with tokens or error.

### 2.3 Get Google Link Status
Check if Google account is linked.

**Endpoint:** `GET /api/google/status`

**Headers:**
```
Authorization: Bearer <accessToken>
```

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "isLinked": true,
    "email": "user@gmail.com"
  }
}
```

### 2.4 Unlink Google Account
Remove Google account link.

**Endpoint:** `DELETE /api/google/unlink`

**Headers:**
```
Authorization: Bearer <accessToken>
```

**Response 200 OK:**
```json
{
  "success": true,
  "message": "Google account unlinked successfully"
}
```

---

## 3. Leads

Base Path: `/api/leads`

All endpoints require: `Authorization: Bearer <token>` and `X-Team-Id: <teamId>`

### 3.1 Get All Leads
Get paginated list of leads with filters.

**Endpoint:** `GET /api/leads`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | number | Page number (default: 1) |
| `limit` | number | Items per page (default: 20) |
| `stage` | string | Filter by stage |
| `picId` | UUID | Filter by PIC |
| `source` | string | Filter by source |
| `label` | string | Filter by label |
| `priority` | string | Filter by priority |
| `search` | string | Search in title, company, contact |
| `startDate` | date | Filter from date |
| `endDate` | date | Filter to date |
| `isArchived` | boolean | Include archived leads |
| `sortBy` | string | Sort field (default: createdAt) |
| `sortOrder` | string | asc or desc (default: desc) |

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "leads": [
      {
        "id": "uuid",
        "title": "Lead Title",
        "companyName": "Company Inc",
        "value": 1000000,
        "currency": "IDR",
        "stage": "NEGOTIATION",
        "priority": "HIGH",
        "pic": {
          "id": "uuid",
          "fullName": "John Doe",
          "email": "john@example.com"
        },
        "_count": {
          "activities": 5,
          "notes": 3,
          "invoices": 2
        }
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 100,
      "totalPages": 5
    }
  }
}
```

### 3.2 Get Leads Kanban
Get leads grouped by stage for Kanban view.

**Endpoint:** `GET /api/leads/kanban`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `picId` | UUID | Filter by PIC |
| `source` | string | Filter by source |
| `priority` | string | Filter by priority |
| `label` | string | Filter by label |
| `search` | string | Search query |
| `year` | number | Filter by year |
| `month` | number | Filter by month (0-11) |

**Response 200 OK:**
```json
{
  "success": true,
  "data": [
    {
      "stage": "NEED_IDENTIFIED",
      "leads": [ ... ],
      "count": 10,
      "totalValue": 5000000
    },
    {
      "stage": "WON",
      "leads": [ ... ],
      "count": 5,
      "totalValue": 2500000
    }
  ]
}
```

### 3.3 Search Leads
Global search across leads, team, and invoices.

**Endpoint:** `GET /api/leads/search`

**Query Parameters:**
- `q`: Search query (min 2 characters)
- `limit`: Max results (default: 10)

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "leads": [ ... ],
    "team": [ ... ],
    "invoices": [ ... ],
    "query": "search term"
  }
}
```

### 3.4 Create Lead
Create a new lead.

**Endpoint:** `POST /api/leads`

**Request Body:**
```json
{
  "title": "New Lead",
  "value": 1000000,
  "currency": "IDR",
  "stage": "NEED_IDENTIFIED",
  "priority": "MEDIUM",
  "labels": ["hot", "enterprise"],
  "dueDate": "2024-12-31T00:00:00Z",
  "description": "Lead description",
  "companyName": "Company Inc",
  "contactPerson": "Contact Name",
  "contactEmail": "contact@company.com",
  "contactPhone": "+6281234567890",
  "clientType": "ENTERPRISE",
  "source": "WEBSITE",
  "origin": "manual",
  "channel": "WEB",
  "picId": "uuid",
  "customFields": [
    { "fieldId": "uuid", "value": "Custom Value" }
  ]
}
```

**Validation Rules:**
- `title`: Required, max 255 characters
- `value`: Optional, numeric
- `currency`: Optional, one of ["IDR", "USD", "EUR", "GBP", "SGD"]
- `stage`: Optional, one of ["NEED_IDENTIFIED", "CONTACT_MADE", "PROPOSAL_MADE", "NEGOTIATION", "CONTRACT_SENT", "WON", "LOST"]
- `priority`: Optional, one of ["LOW", "MEDIUM", "HIGH", "URGENT"]
- `contactEmail`: Optional, valid email format
- `dueDate`: Optional, ISO 8601 format

**Response 201 Created:**
```json
{
  "success": true,
  "message": "Lead created successfully",
  "data": {
    "id": "uuid",
    "title": "New Lead",
    ...
  }
}
```

### 3.5 Get Single Lead
Get detailed lead information.

**Endpoint:** `GET /api/leads/:id`

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Lead Title",
    "companyName": "Company Inc",
    "value": 1000000,
    "currency": "IDR",
    "stage": "NEGOTIATION",
    "priority": "HIGH",
    "pic": { ... },
    "customFieldValues": [
      {
        "id": "uuid",
        "value": "Custom Value",
        "customField": { ... }
      }
    ],
    "stageHistory": [ ... ],
    "_count": {
      "activities": 5,
      "notes": 3,
      "meetings": 2,
      "calls": 4,
      "emails": 1,
      "invoices": 2
    }
  }
}
```

### 3.6 Update Lead
Update lead information.

**Endpoint:** `PUT /api/leads/:id`

**Request Body:** (Same as create, all fields optional)

**Response 200 OK:**
```json
{
  "success": true,
  "message": "Lead updated successfully",
  "data": { ... }
}
```

### 3.7 Update Lead Stage
Quick update for drag-and-drop Kanban.

**Endpoint:** `PATCH /api/leads/:id/stage`

**Request Body:**
```json
{
  "stage": "WON"
}
```

**Response 200 OK:**
```json
{
  "success": true,
  "message": "Lead stage updated",
  "data": { ... }
}
```

### 3.8 Delete Lead
Delete a lead permanently.

**Endpoint:** `DELETE /api/leads/:id`

**Response 200 OK:**
```json
{
  "success": true,
  "message": "Lead deleted successfully"
}
```

### 3.9 Bulk Actions
Perform bulk operations on leads.

**Endpoint:** `POST /api/leads/bulk`

**Request Body:**
```json
{
  "ids": ["uuid1", "uuid2", "uuid3"],
  "action": "delete" // or "won", "lost", "archive", "unarchive"
}
```

**Response 200 OK:**
```json
{
  "success": true,
  "message": "Successfully delete 3 leads",
  "data": {
    "affected": 3
  }
}
```

---

## 4. Activities

Base Path: `/api/activities`

All endpoints require: `Authorization: Bearer <token>` and `X-Team-Id: <teamId>`

### 4.1 Get Activities by Lead
Get all activities for a lead.

**Endpoint:** `GET /api/activities/:leadId/activities`

**Query Parameters:**
- `type`: Filter by type (NOTE, MEETING, CALL, EMAIL)
- `page`: Page number
- `limit`: Items per page

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "activities": [
      {
        "id": "uuid",
        "type": "NOTE",
        "title": "Note added",
        "description": "Content preview",
        "user": { ... },
        "createdAt": "2024-01-01T00:00:00Z"
      }
    ],
    "pagination": { ... }
  }
}
```

### 4.2 Get Timeline
Get combined timeline of all activities.

**Endpoint:** `GET /api/activities/:leadId/timeline`

**Query Parameters:**
- `type`: Filter by type
- `startDate`: Filter from date
- `endDate`: Filter to date
- `page`: Page number
- `limit`: Items per page

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "timeline": [
      {
        "id": "uuid",
        "activityType": "MEETING",
        "title": "Meeting Title",
        "sortDate": "2024-01-01T10:00:00Z",
        "user": { ... }
      }
    ],
    "pagination": { ... }
  }
}
```

### 4.3 Notes

#### Get Notes
**Endpoint:** `GET /api/activities/:leadId/notes`

**Response 200 OK:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "content": "Note content",
      "user": { ... },
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ]
}
```

#### Create Note
**Endpoint:** `POST /api/activities/:leadId/notes`

**Request Body:**
```json
{
  "content": "Note content here"
}
```

**Response 201 Created:**
```json
{
  "success": true,
  "message": "Note created successfully",
  "data": { ... }
}
```

#### Update Note
**Endpoint:** `PUT /api/activities/notes/:noteId`

**Request Body:**
```json
{
  "content": "Updated content"
}
```

#### Delete Note
**Endpoint:** `DELETE /api/activities/notes/:noteId`

### 4.4 Meetings

#### Get Meetings
**Endpoint:** `GET /api/activities/:leadId/meetings`

**Query Parameters:**
- `upcoming`: Filter for upcoming meetings only (true/false)

#### Create Meeting
**Endpoint:** `POST /api/activities/:leadId/meetings`

**Request Body:**
```json
{
  "title": "Meeting Title",
  "description": "Meeting description",
  "startTime": "2024-01-01T10:00:00Z",
  "endTime": "2024-01-01T11:00:00Z",
  "timezone": "Asia/Jakarta",
  "location": "Conference Room A",
  "meetingLink": "https://meet.google.com/...",
  "attendees": ["email1@example.com", "email2@example.com"],
  "reminderMinutes": 15
}
```

**Validation Rules:**
- `title`: Required
- `startTime`: Required, ISO 8601 format
- `endTime`: Required, ISO 8601 format

#### Get Meeting by ID
**Endpoint:** `GET /api/activities/meetings/:meetingId`

#### Update Meeting
**Endpoint:** `PUT /api/activities/meetings/:meetingId`

#### Delete Meeting
**Endpoint:** `DELETE /api/activities/meetings/:meetingId`

### 4.5 Calls

#### Get Calls
**Endpoint:** `GET /api/activities/:leadId/calls`

**Query Parameters:**
- `status`: Filter by status (SCHEDULED, COMPLETED, MISSED)

#### Create Call
**Endpoint:** `POST /api/activities/:leadId/calls`

**Request Body:**
```json
{
  "direction": "OUTBOUND", // or "INBOUND"
  "scheduledAt": "2024-01-01T10:00:00Z",
  "duration": 30, // minutes
  "reminderMinutes": 15
}
```

**Validation Rules:**
- `direction`: Required, one of ["INBOUND", "OUTBOUND"]
- `scheduledAt`: Required, ISO 8601 format

#### Get Call by ID
**Endpoint:** `GET /api/activities/calls/:callId`

#### Update Call
**Endpoint:** `PUT /api/activities/calls/:callId`

#### Complete Call
**Endpoint:** `PUT /api/activities/calls/:callId/complete`

**Request Body:**
```json
{
  "result": "Interested",
  "notes": "Call notes"
}
```

#### Mark Call as Missed
**Endpoint:** `PUT /api/activities/calls/:callId/miss`

#### Delete Call
**Endpoint:** `DELETE /api/activities/calls/:callId`

### 4.6 Emails

#### Get Emails
**Endpoint:** `GET /api/activities/:leadId/emails`

#### Create Email
**Endpoint:** `POST /api/activities/:leadId/emails`

**Request Body:**
```json
{
  "to": ["recipient@example.com"],
  "cc": ["cc@example.com"],
  "bcc": ["bcc@example.com"],
  "subject": "Email Subject",
  "body": "Email body content",
  "scheduledAt": "2024-01-01T10:00:00Z" // Optional, for scheduled send
}
```

**Validation Rules:**
- `to`: Required, array of emails
- `subject`: Required
- `body`: Required

#### Get Email by ID
**Endpoint:** `GET /api/activities/emails/:emailId`

#### Send Email
**Endpoint:** `POST /api/activities/emails/:emailId/send`

Send a scheduled email immediately.

#### Delete Email
**Endpoint:** `DELETE /api/activities/emails/:emailId`

---

## 5. Invoices

Base Path: `/api/invoices`

All endpoints require: `Authorization: Bearer <token>` and `X-Team-Id: <teamId>`

### 5.1 Get All Invoices
Get paginated list of invoices.

**Endpoint:** `GET /api/invoices`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | number | Page number |
| `limit` | number | Items per page |
| `status` | string | Filter by status (DRAFT, SENT, PAID) |
| `startDate` | date | Filter from date |
| `endDate` | date | Filter to date |
| `overdue` | boolean | Show only overdue invoices |
| `year` | number | Filter by year |
| `month` | number | Filter by month (0-11) |

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "invoices": [
      {
        "id": "uuid",
        "invoiceNumber": "INV-202401-0001",
        "lead": {
          "id": "uuid",
          "title": "Lead Title",
          "companyName": "Company Inc"
        },
        "totalAmount": 1000000,
        "status": "PAID",
        "invoiceDate": "2024-01-01T00:00:00Z",
        "dueDate": "2024-01-15T00:00:00Z",
        "user": { ... },
        "_count": { "items": 3 }
      }
    ],
    "pagination": { ... }
  }
}
```

### 5.2 Get Invoice by ID
Get detailed invoice information.

**Endpoint:** `GET /api/invoices/:id`

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "invoiceNumber": "INV-202401-0001",
    "lead": { ... },
    "items": [
      {
        "id": "uuid",
        "itemName": "Service A",
        "quantity": 2,
        "unitPrice": 500000,
        "totalPrice": 1000000
      }
    ],
    "subtotal": 1000000,
    "taxRate": 10,
    "taxAmount": 100000,
    "totalAmount": 1100000,
    "status": "PAID",
    "notes": "Invoice notes",
    "user": { ... }
  }
}
```

### 5.3 Get Invoices by Lead
Get all invoices for a specific lead.

**Endpoint:** `GET /api/invoices/lead/:leadId`

**Query Parameters:**
- `status`: Filter by status

### 5.4 Create Invoice
Create a new invoice for a lead.

**Endpoint:** `POST /api/invoices/lead/:leadId`

**Request Body:**
```json
{
  "invoiceDate": "2024-01-01T00:00:00Z",
  "dueDate": "2024-01-15T00:00:00Z",
  "items": [
    {
      "itemName": "Service A",
      "quantity": 2,
      "unitPrice": 500000
    }
  ],
  "notes": "Invoice notes",
  "taxRate": 10
}
```

**Validation Rules:**
- `invoiceDate`: Required, ISO 8601 format
- `dueDate`: Required, ISO 8601 format, must be >= invoiceDate
- `items`: Required, array with at least 1 item
  - `itemName`: Required
  - `quantity`: Required, integer >= 1
  - `unitPrice`: Required, numeric

**Response 201 Created:**
```json
{
  "success": true,
  "message": "Invoice created successfully",
  "data": { ... }
}
```

### 5.5 Update Invoice
Update invoice details (only DRAFT invoices can be edited).

**Endpoint:** `PUT /api/invoices/:id`

**Request Body:** (Same as create)

### 5.6 Update Invoice Status
Change invoice status.

**Endpoint:** `PATCH /api/invoices/:id/status`

**Request Body:**
```json
{
  "status": "SENT" // or "PAID"
}
```

**Status Transitions:**
- DRAFT → SENT
- SENT → PAID
- PAID → (no further transitions)

### 5.7 Delete Invoice
Delete a draft invoice.

**Endpoint:** `DELETE /api/invoices/:id`

---

## 6. Team

Base Path: `/api/team`

All endpoints require: `Authorization: Bearer <token>` and `X-Team-Id: <teamId>`

### 6.1 Get Team Members
Get all team members.

**Endpoint:** `GET /api/team`

**Query Parameters:**
- `page`: Page number
- `limit`: Items per page
- `status`: Filter by status
- `search`: Search by name or email

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "members": [
      {
        "id": "uuid",
        "email": "member@example.com",
        "fullName": "Member Name",
        "phone": "+6281234567890",
        "location": "Jakarta",
        "bio": "Team member bio",
        "skills": ["Skill 1", "Skill 2"],
        "status": "ACTIVE",
        "joinedAt": "2024-01-01T00:00:00Z",
        "teamMemberships": [
          {
            "role": { ... }
          }
        ],
        "_count": { "leads": 10 }
      }
    ],
    "pagination": { ... }
  }
}
```

### 6.2 Get Assignable Members
Get members that can be assigned leads based on role hierarchy.

**Endpoint:** `GET /api/team/assignable`

**Response 200 OK:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "fullName": "Member Name",
      "email": "member@example.com",
      "teamMemberships": [
        {
          "role": { ... }
        }
      ]
    }
  ]
}
```

### 6.3 Get Team Member
Get detailed member information.

**Endpoint:** `GET /api/team/:id`

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "member@example.com",
    "fullName": "Member Name",
    "teamMemberships": [ ... ],
    "leads": [ ... ],
    "_count": { ... }
  }
}
```

### 6.4 Get Team Member Performance
Get performance metrics for a team member.

**Endpoint:** `GET /api/team/:id/performance`

**Query Parameters:**
- `startDate`: Filter from date
- `endDate`: Filter to date

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "leads": {
      "total": 50,
      "won": 20,
      "lost": 10,
      "active": 20,
      "winRate": "40.0"
    },
    "activities": {
      "total": 100,
      "meetings": 30,
      "calls": 50,
      "invoices": 20
    },
    "revenue": 50000000
  }
}
```

### 6.5 Create Team Member
Create a new team member.

**Endpoint:** `POST /api/team`

**Request Body:**
```json
{
  "email": "newmember@example.com",
  "password": "password123",
  "fullName": "New Member",
  "phone": "+6281234567890",
  "location": "Jakarta",
  "bio": "Member bio",
  "skills": ["Skill 1"],
  "status": "ACTIVE"
}
```

**Validation Rules:**
- `email`: Valid email, required
- `password`: Min 8 characters, letters and numbers, required
- `fullName`: Required
- `status`: Optional, one of ["ACTIVE", "INACTIVE", "ONBOARDING", "ON_LEAVE"]

### 6.6 Update Team Member
Update member information.

**Endpoint:** `PUT /api/team/:id`

### 6.7 Toggle Member Status
Activate or deactivate a member.

**Endpoint:** `PATCH /api/team/:id/status`

**Request Body:**
```json
{
  "status": "ACTIVE" // or "INACTIVE"
}
```

### 6.8 Delete Team Member
Delete a team member.

**Endpoint:** `DELETE /api/team/:id`

**Note:** Cannot delete members with assigned leads.

---

## 7. Team Management

Base Path: `/api/teams`

All endpoints require: `Authorization: Bearer <token>`

### 7.1 Get User Teams
Get all teams the user is a member of.

**Endpoint:** `GET /api/teams`

**Response 200 OK:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Team Name",
      "slug": "team-name",
      "logo": "https://...",
      "owner": { ... },
      "role": { ... },
      "isDefault": true,
      "isOwner": false,
      "_count": { "members": 10, "leads": 50 }
    }
  ]
}
```

### 7.2 Create Team
Create a new team.

**Endpoint:** `POST /api/teams`

**Request Body:**
```json
{
  "name": "New Team",
  "description": "Team description",
  "logo": "https://..."
}
```

**Response 201 Created:**
```json
{
  "success": true,
  "message": "Team created successfully",
  "data": {
    "id": "uuid",
    "name": "New Team",
    "slug": "new-team",
    ...
  }
}
```

### 7.3 Get Team Details
Get detailed team information.

**Endpoint:** `GET /api/teams/:teamId`

### 7.4 Update Team
Update team information.

**Endpoint:** `PUT /api/teams/:teamId`

### 7.5 Delete Team
Delete a team.

**Endpoint:** `DELETE /api/teams/:teamId`

**Note:** Only team owner can delete.

### 7.6 Set Default Team
Set a team as default for the user.

**Endpoint:** `POST /api/teams/:teamId/set-default`

### 7.7 Get Team Roles
Get all roles in a team.

**Endpoint:** `GET /api/teams/:teamId/roles`

**Response 200 OK:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Admin",
      "description": "Full access",
      "color": "#6366f1",
      "permissions": {
        "leads": { "create": true, "read": true, "update": true, "delete": true },
        "invoices": { "create": true, "read": true, "update": true, "delete": true },
        "team": { "create": true, "read": true, "update": true, "delete": true, "invite": true },
        "roles": { "create": true, "read": true, "update": true, "delete": true },
        "settings": { "read": true, "update": true }
      },
      "reportToRole": { ... },
      "subordinateRoles": [ ... ],
      "_count": { "members": 5 }
    }
  ]
}
```

### 7.8 Create Team Role
Create a new role.

**Endpoint:** `POST /api/teams/:teamId/roles`

**Request Body:**
```json
{
  "name": "Manager",
  "description": "Manager role",
  "color": "#10b981",
  "permissions": {
    "leads": { "create": true, "read": true, "update": true, "delete": false },
    "invoices": { "create": true, "read": true, "update": false, "delete": false },
    "team": { "create": false, "read": true, "update": false, "delete": false, "invite": true },
    "roles": { "create": false, "read": true, "update": false, "delete": false },
    "settings": { "read": true, "update": false }
  },
  "isDefault": false,
  "reportToRoleId": "uuid" // Optional, for role hierarchy
}
```

### 7.9 Update Team Role
Update role information.

**Endpoint:** `PUT /api/teams/:teamId/roles/:roleId`

### 7.10 Delete Team Role
Delete a role.

**Endpoint:** `DELETE /api/teams/:teamId/roles/:roleId`

**Note:** Cannot delete role with assigned members.

### 7.11 Get Team Members
Get all members of a team.

**Endpoint:** `GET /api/teams/:teamId/members`

### 7.12 Update Member Role
Change a member's role.

**Endpoint:** `PUT /api/teams/:teamId/members/:memberId`

**Request Body:**
```json
{
  "roleId": "uuid"
}
```

### 7.13 Remove Member
Remove a member from the team.

**Endpoint:** `DELETE /api/teams/:teamId/members/:memberId`

### 7.14 Invite Member
Send invitation to join team.

**Endpoint:** `POST /api/teams/:teamId/invitations`

**Request Body:**
```json
{
  "email": "invite@example.com",
  "roleId": "uuid"
}
```

### 7.15 Get Invitations
Get pending invitations.

**Endpoint:** `GET /api/teams/:teamId/invitations`

### 7.16 Cancel Invitation
Cancel a pending invitation.

**Endpoint:** `DELETE /api/teams/:teamId/invitations/:invitationId`

### 7.17 Accept Invitation
Accept invitation to join team.

**Endpoint:** `POST /api/teams/invitations/:token/accept`

### 7.18 Get SMTP Config
Get team's SMTP configuration.

**Endpoint:** `GET /api/teams/:teamId/smtp`

**Note:** Team owner only.

### 7.19 Save SMTP Config
Save or update SMTP configuration.

**Endpoint:** `POST /api/teams/:teamId/smtp`

**Request Body:**
```json
{
  "host": "smtp.gmail.com",
  "port": 587,
  "secure": false,
  "username": "email@gmail.com",
  "password": "app-password",
  "fromEmail": "noreply@company.com",
  "fromName": "Company Name"
}
```

### 7.20 Verify SMTP Config
Test SMTP configuration.

**Endpoint:** `POST /api/teams/:teamId/smtp/verify`

**Request Body:**
```json
{
  "testEmail": "test@example.com" // Optional, defaults to user's email
}
```

---

## 8. Dashboard

Base Path: `/api/dashboard`

All endpoints require: `Authorization: Bearer <token>` and `X-Team-Id: <teamId>`

### 8.1 Get Dashboard Analytics
Get KPI cards and summary data.

**Endpoint:** `GET /api/dashboard/analytics`

**Query Parameters:**
- `picId`: Filter by PIC
- `source`: Filter by source
- `stage`: Filter by stage
- `period`: daily, weekly, monthly, quarterly, yearly (default: monthly)
- `year`: Specific year
- `month`: Specific month (0-11)

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "kpiCards": {
      "totalPipelineValue": 50000000,
      "activeDeals": 25,
      "averageDealSize": 2000000,
      "totalWon": 10,
      "wonValue": 20000000,
      "totalLost": 5,
      "totalLeads": 40,
      "activeLeads": 25
    },
    "period": "monthly",
    "filters": { ... }
  }
}
```

### 8.2 Get Leads by Month
Get lead creation data for charts.

**Endpoint:** `GET /api/dashboard/leads-by-month`

**Query Parameters:**
- `year`: Year to display (default: current year)
- `month`: Specific month (optional)

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "year": 2024,
    "months": ["Jan", "Feb", "Mar", ...],
    "values": [10, 15, 20, ...]
  }
}
```

### 8.3 Get Revenue by Month
Get revenue data for charts.

**Endpoint:** `GET /api/dashboard/revenue-by-month`

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "year": 2024,
    "months": ["Jan", "Feb", "Mar", ...],
    "estimation": [1000000, 1500000, ...],
    "realisation": [800000, 1200000, ...]
  }
}
```

### 8.4 Get Lead Source Breakdown
Get leads grouped by source.

**Endpoint:** `GET /api/dashboard/lead-sources`

### 8.5 Get Pipeline Overview
Get leads grouped by stage.

**Endpoint:** `GET /api/dashboard/pipeline-overview`

### 8.6 Get Recent Deals
Get recently updated deals.

**Endpoint:** `GET /api/dashboard/recent-deals`

**Query Parameters:**
- `limit`: Number of deals (default: 10)

### 8.7 Get Upcoming Activities
Get upcoming meetings, calls, and tasks.

**Endpoint:** `GET /api/dashboard/upcoming-activities`

**Query Parameters:**
- `type`: Filter by type (meeting, call, email, invoice)
- `limit`: Number of items (default: 10)

### 8.8 Get Quarter Summary
Get quarterly performance summary.

**Endpoint:** `GET /api/dashboard/quarter-summary`

**Query Parameters:**
- `year`: Year (default: current)
- `quarter`: Quarter 1-4 (default: current)

---

## 9. Notifications

Base Path: `/api/notifications`

All endpoints require: `Authorization: Bearer <token>` and `X-Team-Id: <teamId>`

### 9.1 Get Notifications
Get user notifications.

**Endpoint:** `GET /api/notifications`

**Query Parameters:**
- `page`: Page number
- `limit`: Items per page
- `unreadOnly`: Show only unread (true/false)

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "notifications": [
      {
        "id": "uuid",
        "title": "New Lead Assigned",
        "message": "You have been assigned to lead: ...",
        "type": "deal_update",
        "isRead": false,
        "metadata": { "leadId": "uuid" },
        "createdAt": "2024-01-01T00:00:00Z"
      }
    ],
    "unreadCount": 5,
    "pagination": { ... }
  }
}
```

### 9.2 Mark as Read
Mark a notification as read.

**Endpoint:** `PATCH /api/notifications/:id/read`

### 9.3 Mark All as Read
Mark all notifications as read.

**Endpoint:** `PATCH /api/notifications/read-all`

### 9.4 Delete Notification
Delete a notification.

**Endpoint:** `DELETE /api/notifications/:id`

### 9.5 Get Preferences
Get notification preferences.

**Endpoint:** `GET /api/notifications/preferences`

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "emailDealUpdate": true,
    "emailActivityReminder": true,
    "emailMarketing": false,
    "pushDealUpdate": true,
    "pushActivityReminder": true
  }
}
```

### 9.6 Update Preferences
Update notification preferences.

**Endpoint:** `PUT /api/notifications/preferences`

**Request Body:**
```json
{
  "emailDealUpdate": true,
  "emailActivityReminder": true,
  "emailMarketing": false,
  "pushDealUpdate": true,
  "pushActivityReminder": true
}
```

---

## 10. Reports

Base Path: `/api/reports`

All endpoints require: `Authorization: Bearer <token>`

### 10.1 Get Leads Report
Get detailed leads report.

**Endpoint:** `GET /api/reports/leads`

**Query Parameters:**
- `stage`: Filter by stage
- `picId`: Filter by PIC
- `source`: Filter by source
- `startDate`: Filter from date
- `endDate`: Filter to date

### 10.2 Get Deals Report
Get closed deals report.

**Endpoint:** `GET /api/reports/deals`

**Query Parameters:**
- `startDate`: Filter from date
- `endDate`: Filter to date
- `picId`: Filter by PIC

### 10.3 Get Activity Report
Get activity summary report.

**Endpoint:** `GET /api/reports/activities`

**Query Parameters:**
- `userId`: Filter by user
- `leadId`: Filter by lead
- `startDate`: Filter from date
- `endDate`: Filter to date

### 10.4 Get Invoice Report
Get invoice summary report.

**Endpoint:** `GET /api/reports/invoices`

**Query Parameters:**
- `status`: Filter by status
- `startDate`: Filter from date
- `endDate`: Filter to date
- `overdue`: Show only overdue

### 10.5 Export to CSV
Export report data to CSV.

**Endpoint:** `GET /api/reports/export/:type/csv`

**Path Parameters:**
- `type`: leads, deals, or invoices

**Query Parameters:** Same as respective report endpoint.

**Response:** CSV file download

### 10.6 Export to PDF
Export report data to PDF.

**Endpoint:** `GET /api/reports/export/:type/pdf`

**Response:** PDF file download

---

## 11. Profile

Base Path: `/api/profile`

All endpoints require: `Authorization: Bearer <token>`

### 11.1 Get Profile
Get current user profile.

**Endpoint:** `GET /api/profile`

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "user@example.com",
    "fullName": "John Doe",
    "phone": "+6281234567890",
    "location": "Jakarta",
    "bio": "Sales Manager",
    "skills": ["Negotiation", "CRM"],
    "status": "ACTIVE",
    "joinedAt": "2024-01-01T00:00:00Z",
    "teams": [ ... ],
    "currentTeam": { ... }
  }
}
```

### 11.2 Update Profile
Update user profile.

**Endpoint:** `PUT /api/profile`

**Request Body:**
```json
{
  "fullName": "John Doe",
  "phone": "+6281234567890",
  "location": "Jakarta",
  "bio": "Updated bio",
  "skills": ["Skill 1", "Skill 2"]
}
```

### 11.3 Change Password
Change user password.

**Endpoint:** `POST /api/profile/change-password`

**Request Body:**
```json
{
  "currentPassword": "oldpassword",
  "newPassword": "newpassword123",
  "confirmPassword": "newpassword123"
}
```

**Validation Rules:**
- `currentPassword`: Required
- `newPassword`: Min 8 characters, letters and numbers
- `confirmPassword`: Must match newPassword

---

## 12. Custom Fields

Base Path: `/api/custom-fields`

All endpoints require: `Authorization: Bearer <token>`

### 12.1 Get Custom Fields
Get all active custom fields.

**Endpoint:** `GET /api/custom-fields`

**Response 200 OK:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Industry",
      "fieldType": "select",
      "options": ["Tech", "Finance", "Healthcare"],
      "section": "default",
      "sortOrder": 1,
      "isRequired": false
    }
  ]
}
```

### 12.2 Create Custom Field
Create a new custom field.

**Endpoint:** `POST /api/custom-fields`

**Request Body:**
```json
{
  "name": "Industry",
  "fieldType": "select", // text, number, date, select, multiselect
  "options": ["Tech", "Finance", "Healthcare"],
  "section": "default",
  "sortOrder": 1,
  "isRequired": false
}
```

**Validation Rules:**
- `name`: Required
- `fieldType`: Required, one of ["text", "number", "date", "select", "multiselect"]

**Note:** Team owner only.

### 12.3 Update Custom Field
Update custom field.

**Endpoint:** `PUT /api/custom-fields/:id`

### 12.4 Delete Custom Field
Soft delete custom field.

**Endpoint:** `DELETE /api/custom-fields/:id`

### 12.5 Reorder Custom Fields
Update sort order of fields.

**Endpoint:** `POST /api/custom-fields/reorder`

**Request Body:**
```json
{
  "fields": [
    { "id": "uuid1", "sortOrder": 1 },
    { "id": "uuid2", "sortOrder": 2 }
  ]
}
```

### 12.6 Update Lead Custom Fields
Update custom field values for a lead.

**Endpoint:** `PUT /api/custom-fields/lead/:leadId`

**Request Body:**
```json
{
  "fields": [
    { "fieldId": "uuid", "value": "Tech" }
  ]
}
```

---

## 13. Superadmin

Base Path: `/api/superadmin`

Superadmin endpoints use separate authentication.

### 13.1 Sign In
Superadmin login.

**Endpoint:** `POST /api/superadmin/auth/signin`

**Request Body:**
```json
{
  "email": "admin@example.com",
  "password": "password"
}
```

### 13.2 Refresh Token
**Endpoint:** `POST /api/superadmin/auth/refresh-token`

### 13.3 Logout
**Endpoint:** `POST /api/superadmin/auth/logout`

### 13.4 Get Profile
**Endpoint:** `GET /api/superadmin/profile`

### 13.5 Update Profile
**Endpoint:** `PUT /api/superadmin/profile`

### 13.6 Change Password
**Endpoint:** `PUT /api/superadmin/profile/password`

### 13.7 Get Dashboard Stats
Get platform-wide statistics.

**Endpoint:** `GET /api/superadmin/dashboard/stats`

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "users": { "total": 100, "active": 80 },
    "teams": { "total": 50 },
    "leads": { "total": 500 },
    "subscriptions": { "active": 30 },
    "revenue": { "thisMonth": 5000000, "currency": "IDR" },
    "recentPayments": [ ... ]
  }
}
```

### 13.8 Get Users
Get all users with pagination.

**Endpoint:** `GET /api/superadmin/users`

**Query Parameters:**
- `page`: Page number
- `limit`: Items per page
- `search`: Search by name or email
- `status`: Filter by status

### 13.9 Get User by ID
**Endpoint:** `GET /api/superadmin/users/:id`

### 13.10 Update User Status
**Endpoint:** `PUT /api/superadmin/users/:id/status`

**Request Body:**
```json
{
  "status": "ACTIVE" // ACTIVE, INACTIVE, ONBOARDING, ON_LEAVE
}
```

### 13.11 Suspend User
**Endpoint:** `POST /api/superadmin/users/:id/suspend`

**Request Body:**
```json
{
  "reason": "Violation of terms",
  "duration": 7 // days, optional
}
```

### 13.12 Delete User
**Endpoint:** `DELETE /api/superadmin/users/:id`

### 13.13 Get Teams
**Endpoint:** `GET /api/superadmin/teams`

### 13.14 Get Team by ID
**Endpoint:** `GET /api/superadmin/teams/:id`

### 13.15 Archive Team
**Endpoint:** `POST /api/superadmin/teams/:id/archive`

### 13.16 Get Plans
Get all subscription plans.

**Endpoint:** `GET /api/superadmin/plans`

### 13.17 Create Plan
**Endpoint:** `POST /api/superadmin/plans`

**Request Body:**
```json
{
  "name": "pro",
  "displayName": "Pro Plan",
  "description": "For growing teams",
  "maxUsers": 10,
  "maxTeams": 5,
  "maxLeads": 1000,
  "priceMonthly": 299000,
  "priceYearly": 2990000,
  "features": ["Feature 1", "Feature 2"],
  "sortOrder": 1,
  "isActive": true
}
```

### 13.18 Update Plan
**Endpoint:** `PUT /api/superadmin/plans/:id`

### 13.19 Delete Plan
**Endpoint:** `DELETE /api/superadmin/plans/:id`

**Note:** Cannot delete plan with active subscriptions.

### 13.20 Get Billing Overview
Get platform billing statistics.

**Endpoint:** `GET /api/superadmin/billing`

---

## 14. Subscriptions

Base Path: `/api/subscriptions`

All endpoints require: `Authorization: Bearer <token>`

### 14.1 Get Plans
Get all active subscription plans.

**Endpoint:** `GET /api/subscriptions/plans`

### 14.2 Get Plan by ID
**Endpoint:** `GET /api/subscriptions/plans/:id`

### 14.3 Get Current Subscription
Get user's current subscription.

**Endpoint:** `GET /api/subscriptions/current`

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "hasSubscription": true,
    "subscription": {
      "id": "uuid",
      "status": "ACTIVE",
      "currentPeriodStart": "2024-01-01T00:00:00Z",
      "currentPeriodEnd": "2024-02-01T00:00:00Z",
      "cancelAtPeriodEnd": false
    },
    "plan": { ... },
    "usage": {
      "users": 5,
      "teams": 2,
      "leads": 100,
      "maxUsers": 10,
      "maxTeams": 5,
      "maxLeads": 1000
    }
  }
}
```

### 14.4 Get Usage
Get current usage statistics.

**Endpoint:** `GET /api/subscriptions/usage`

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "users": { "current": 5, "limit": 10, "percentage": 50 },
    "teams": { "current": 2, "limit": 5, "percentage": 40 },
    "leads": { "current": 100, "limit": 1000, "percentage": 10 }
  }
}
```

### 14.5 Create Checkout
Create payment checkout session.

**Endpoint:** `POST /api/subscriptions/create-checkout`

**Request Body:**
```json
{
  "planId": "uuid"
}
```

**Response 200 OK:**
```json
{
  "success": true,
  "data": {
    "checkoutUrl": "https://checkout.xendit.co/..."
  }
}
```

### 14.6 Cancel Subscription
Cancel subscription at period end.

**Endpoint:** `POST /api/subscriptions/cancel`

### 14.7 Reactivate Subscription
Reactivate cancelled subscription.

**Endpoint:** `POST /api/subscriptions/reactivate`

### 14.8 Get Billing History
Get payment history.

**Endpoint:** `GET /api/subscriptions/billing/history`

### 14.9 Get Invoices
Get subscription invoices.

**Endpoint:** `GET /api/subscriptions/invoices`

### 14.10 Get Invoice by ID
**Endpoint:** `GET /api/subscriptions/invoices/:id`

---

## 15. Webhooks

Base Path: `/api/webhooks`

### 15.1 Xendit Webhook
Handle Xendit payment notifications.

**Endpoint:** `POST /api/webhooks/xendit`

**Headers:**
```
Xendit-Callback-Token: <webhook-secret>
```

**Note:** This endpoint is public but verifies signature.

### 15.2 Verify Payment
Manually verify payment status.

**Endpoint:** `GET /api/webhooks/verify/:invoiceId`

### 15.3 Get Payment Status
Get payment status by invoice ID.

**Endpoint:** `GET /api/webhooks/status/:invoiceId`

---

## Error Codes

| Status Code | Description |
|-------------|-------------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request - Validation error |
| 401 | Unauthorized - Invalid or missing token |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource doesn't exist |
| 409 | Conflict - Resource already exists |
| 423 | Locked - Account is locked |
| 500 | Internal Server Error |

## Common Error Response

```json
{
  "success": false,
  "message": "Error description",
  "errors": [ // Validation errors only
    {
      "field": "email",
      "message": "Please provide a valid email"
    }
  ]
}
```

---

## Enums

### Lead Stages
- `NEED_IDENTIFIED`
- `CONTACT_MADE`
- `PROPOSAL_MADE`
- `NEGOTIATION`
- `CONTRACT_SENT`
- `WON`
- `LOST`

### Lead Priority
- `LOW`
- `MEDIUM`
- `HIGH`
- `URGENT`

### Lead Source
- `WEBSITE`
- `REFERRAL`
- `SOCIAL_MEDIA`
- `EMAIL`
- `PHONE`
- `EVENT`
- `PARTNER`
- `OTHER`

### User Status
- `ACTIVE`
- `INACTIVE`
- `ONBOARDING`
- `ON_LEAVE`
- `SUSPENDED`

### Invoice Status
- `DRAFT`
- `SENT`
- `PAID`

### Subscription Status
- `ACTIVE`
- `CANCELLED`
- `PAST_DUE`
- `TRIALING`

### Call Direction
- `INBOUND`
- `OUTBOUND`

### Call Status
- `SCHEDULED`
- `COMPLETED`
- `MISSED`

### Custom Field Types
- `text`
- `number`
- `date`
- `select`
- `multiselect`
