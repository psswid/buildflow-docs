# BuildFlow
## Construction Business Management System
### Business Requirements & Product Specification

**Version:** 1.0  
**Date:** November 12, 2025  
**Status:** Draft  
**Author:** Piotr Świderski

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Product Vision & Goals](#2-product-vision--goals)
3. [Target Market & User Personas](#3-target-market--user-personas)
4. [Business Model & Monetization](#4-business-model--monetization)
5. [Core Features & Functional Requirements](#5-core-features--functional-requirements)
6. [User Roles & Permissions](#6-user-roles--permissions)
7. [User Stories & Use Cases](#7-user-stories--use-cases)
8. [Data Model & Business Entities](#8-data-model--business-entities)
9. [Business Rules & Workflows](#9-business-rules--workflows)
10. [Integration Requirements](#10-integration-requirements)
11. [Non-Functional Requirements](#11-non-functional-requirements)
12. [Success Metrics & KPIs](#12-success-metrics--kpis)
13. [Roadmap & Phasing](#13-roadmap--phasing)
14. [Appendix](#14-appendix)

---

## 1. Executive Summary

### 1.1 Project Overview

BuildFlow is an open-source business management system designed specifically for small to medium-sized construction, renovation, and contracting businesses. The system addresses the administrative chaos that trades professionals face daily by providing an integrated platform for managing clients, quotes, projects, invoices, and client communication.

### 1.2 Problem Statement

Construction and renovation businesses typically struggle with:

- Scattered information across Excel, WhatsApp, email, and paper notes
- Unprofessional quote and invoice presentation
- Lost revenue from forgotten invoices and unbilled work
- Time wasted on administrative tasks instead of billable work
- Poor client communication leading to constant status update requests
- Lack of project documentation and historical data
- Difficulty scaling operations due to manual processes

### 1.3 Solution

BuildFlow provides a centralized, user-friendly platform that eliminates administrative overhead while projecting professionalism to clients. The system is designed for mobile-first usage, recognizing that contractors spend most of their time on job sites, not in offices.

### 1.4 Key Differentiators

- Built specifically for construction trades (not a generic CRM)
- Generous forever-free tier enabling immediate adoption
- Client portal that sets businesses apart from competitors
- Mobile-first design for on-site usage
- Technology-agnostic architecture allowing multiple implementations
- Open-source foundation enabling customization and transparency

---

## 2. Product Vision & Goals

### 2.1 Vision Statement

To become the de facto business management solution for small construction businesses worldwide, empowering trades professionals to operate with the same level of organization and professionalism as large enterprises, while remaining accessible and affordable.

### 2.2 Primary Goals

- Replace manual/spreadsheet-based workflows with integrated digital processes
- Reduce administrative time by 50% for typical users
- Increase professional perception and client satisfaction
- Reduce lost revenue from missed invoices and forgotten follow-ups
- Enable business scaling without proportional administrative overhead
- Build a sustainable open-source project with active community

### 2.3 Success Criteria

- 10,000+ active free tier users within 12 months
- 5% conversion rate from free to paid tiers
- Average user rating of 4.5+ stars
- 50+ community contributors
- Implementation in 3+ technology stacks (Next.js, Laravel, Symfony)

---

## 3. Target Market & User Personas

### 3.1 Target Market

**Primary Market:**
- Small construction companies (1-10 employees)
- Independent contractors and tradespeople
- Renovation and remodeling businesses
- Specialty contractors (electrical, plumbing, HVAC, etc.)
- Property maintenance companies

**Geographic Focus:**
- Global market with initial focus on English-speaking countries
- Country-agnostic core features (localization in future phases)
- No region-specific compliance in MVP

### 3.2 User Persona 1: Solo Contractor Sam

**Demographics:**
- Age: 35-50
- Role: Owner-operator of small renovation business
- Team: Works alone or with 1-2 helpers
- Annual projects: 20-50
- Tech comfort: Moderate (uses smartphone, email, basic apps)

**Pain Points:**
- Loses quotes and client information in email/WhatsApp chaos
- Spends 2-3 hours weekly on admin work
- Forgets to follow up on quotes and unpaid invoices
- Looks unprofessional with Word document quotes
- Can't easily show clients project progress

**Goals:**
- Spend more time on paid work, less on admin
- Look more professional to win better clients
- Never miss a payment or follow-up
- Keep all client communication in one place

### 3.3 User Persona 2: Growing Business Grace

**Demographics:**
- Age: 30-45
- Role: Owner of expanding construction company
- Team: 3-8 employees including project managers
- Annual projects: 50-200
- Tech comfort: High (early adopter, uses multiple SaaS tools)

**Pain Points:**
- Current spreadsheet system doesn't scale with team
- Team members lack visibility into project status
- Clients expect modern communication and transparency
- Difficult to track profitability across projects
- Can't efficiently manage multiple concurrent projects

**Goals:**
- Enable team collaboration without constant meetings
- Provide clients with self-service project information
- Scale operations without hiring admin staff
- Make data-driven decisions about business performance

---

## 4. Business Model & Monetization

### 4.1 Pricing Strategy

BuildFlow employs a freemium model with a generous forever-free tier and usage-based upgrade triggers. This approach maximizes adoption while creating natural conversion opportunities as businesses grow.

### 4.2 Pricing Tiers

#### Starter Plan (Free Forever)

**Price:** $0/month

**Limits:**
- 3 active projects simultaneously
- 10 total clients in database
- 5 quotes per month
- 3 invoices per month
- 100 MB file storage
- 1 user account

**Features:**
- Full client management (within limits)
- Professional quote builder (with watermark)
- Invoice generator (with watermark)
- Project tracking and timeline
- Document and photo upload
- Basic dashboard
- 10 email notifications/month
- Mobile-responsive interface
- Community support

#### Professional Plan

**Price:** $19-29/month

**Limits:**
- 20 active projects
- 100 total clients
- 50 quotes per month
- 30 invoices per month
- 10 GB file storage
- 3 user accounts

**Additional Features:**
- Client portal access (5 active clients)
- Branded PDFs (no watermark)
- Advanced analytics and reporting
- Calendar and scheduling views
- Unlimited email notifications
- Data export (CSV/Excel)
- Custom quote and invoice templates
- Email support (48-hour response)

#### Business Plan

**Price:** $49-79/month

**Limits:**
- Unlimited projects
- Unlimited clients
- Unlimited quotes and invoices
- 100 GB file storage
- 10 user accounts

**Additional Features:**
- Unlimited client portal access
- Team collaboration tools
- SMS notifications
- API access
- Webhook integrations
- White-label options
- Priority support (24-hour response)
- Custom integrations consulting

### 4.3 Revenue Streams

- Subscription revenue (primary)
- Add-on features and modules (future)
- Premium support packages (future)
- Partner integration revenue share (future)
- Self-hosted license fees for enterprises (future)

### 4.4 Conversion Strategy

**Natural upgrade triggers:**
- Project limit reached (most common trigger)
- Client requests portal access
- Professional branding needs (watermark removal)
- Storage limit exceeded
- Team member addition required

---

## 5. Core Features & Functional Requirements

### 5.1 Module 1: Client Management

#### Purpose
Centralized repository for all client information, replacing scattered contacts across phone, email, and notes.

#### Key Features

- Create, read, update, delete client records
- Store: name, company, address, phone, email, notes
- Tag/categorize clients (residential, commercial, lead, active, past)
- View all associated quotes, projects, and invoices
- Search and filter clients
- Import contacts from CSV
- Track client acquisition source
- Client communication history timeline

#### User Stories

- As a contractor, I want to quickly find a client's phone number, so I can call them back without searching through old messages
- As a business owner, I want to see all projects for a specific client, so I understand our relationship history
- As a contractor, I want to tag leads vs active clients, so I can prioritize follow-ups

### 5.2 Module 2: Quote Management

#### Purpose
Professional quote creation and tracking system that replaces Word documents and manual calculations.

#### Key Features

- Create quotes with line items (description, quantity, unit price)
- Categorize line items: Labor, Materials, Equipment, Other
- Automatic calculation of subtotals, taxes, discounts, totals
- Quote templates for common job types
- Quote versioning (revision tracking)
- Status tracking: Draft, Sent, Accepted, Rejected, Expired
- Generate professional PDF with company branding (Pro tier)
- Email quotes directly to clients
- Client-side quote acceptance (digital signature)
- Convert accepted quote to project (one-click)
- Quote validity period and expiration dates
- Duplicate quotes for similar projects

#### User Stories

- As a contractor, I want to create a quote in under 5 minutes, so I can respond quickly to client requests
- As a contractor, I want my quotes to look professional, so clients take me seriously
- As a business owner, I want to track which quotes are accepted vs rejected, so I can improve my pricing strategy
- As a contractor, I want to reuse quotes for similar jobs, so I don't reinvent the wheel each time

#### Quote Workflow

1. **Draft** → Edit line items, pricing, terms
2. **Send** → Email PDF to client, mark as Sent
3. **Client reviews** → (Optional) Digital acceptance
4. **Accepted** → Automatically create project
5. **Rejected** → Mark reason, archive

### 5.3 Module 3: Project Management

#### Purpose
Track active work, progress, timeline, and budget for each job. Central hub for all project-related information.

#### Key Features

- Create projects (manual or from accepted quotes)
- Project details: name, address, client, start/end dates, budget
- Project status workflow: Not Started, In Progress, On Hold, Completed, Cancelled
- Timeline view with milestones
- Budget tracking: estimated vs actual costs
- Upload progress photos with timestamps
- Attach documents (contracts, permits, plans)
- Internal notes and updates
- Link related quotes and invoices
- Project checklist/tasks (basic)
- Completion percentage tracking

#### User Stories

- As a contractor, I want to see all active projects on one screen, so I can plan my week
- As a contractor, I want to upload progress photos from my phone, so I document work as I go
- As a business owner, I want to track actual vs estimated costs, so I understand project profitability
- As a contractor, I want to mark a project complete, so it moves out of my active list

### 5.4 Module 4: Documents & Photos

#### Purpose
Organized file storage tied to specific projects, replacing camera roll chaos and lost email attachments.

#### Key Features

- Upload files (images, PDFs, documents)
- Organize by project
- Photo gallery view with thumbnails
- Add captions/descriptions to photos
- Automatic EXIF data capture (date, location)
- Before/after photo pairing
- Download originals
- Share photos with clients via portal
- File versioning for documents
- Bulk upload capability

#### User Stories

- As a contractor, I want to take photos on site and immediately upload them, so I document progress in real-time
- As a business owner, I want to show before/after photos to potential clients, so I demonstrate quality
- As a contractor, I want clients to see progress photos without emailing them individually, so I save time

### 5.5 Module 5: Invoice Management

#### Purpose
Professional invoice creation and payment tracking to eliminate lost revenue from forgotten bills.

#### Key Features

- Create invoices with line items
- Generate from quotes (inherit line items)
- Invoice details: number, date, due date, payment terms
- Status tracking: Draft, Sent, Paid, Partial, Overdue, Cancelled
- Payment tracking: record payment date, method, amount
- Automatic overdue detection and notifications
- Recurring invoices (Pro tier)
- Payment reminders (automated emails)
- Professional PDF generation
- Multiple tax rates support
- Deposit/partial payment handling
- Invoice numbering (auto-increment with custom prefix)

#### User Stories

- As a contractor, I want to create an invoice from a quote, so I don't re-enter information
- As a business owner, I want to see which invoices are overdue, so I can follow up on payments
- As a contractor, I want automatic reminders for unpaid invoices, so I don't forget to chase payment
- As a contractor, I want to record partial payments, so I track what's still owed

### 5.6 Module 6: Dashboard

#### Purpose
At-a-glance overview of business status, providing immediate situational awareness.

#### Key Widgets

- Active projects count and list
- Pending quotes requiring action
- Outstanding invoice total and list
- Recent activity feed
- This month's revenue (Paid invoices)
- Storage usage indicator
- Quick actions: Create Quote, New Project, New Invoice
- Upcoming deadlines and due dates
- Client portal activity (Pro tier)

#### User Stories

- As a contractor, I want to see my day's priorities when I open the app, so I know what needs attention
- As a business owner, I want to see revenue and outstanding payments, so I understand cash flow
- As a contractor, I want quick access to common actions, so I can start tasks immediately

### 5.7 Module 7: Client Portal

#### Purpose
Self-service client interface that reduces communication overhead while increasing transparency and professionalism. Premium feature that differentiates businesses from competitors.

#### Key Features

- Secure client login (email + password or magic link)
- View assigned projects and their status
- See project timeline and milestones
- Browse progress photos and documents
- View original quote and scope
- See invoices and payment status
- Payment link integration (future)
- Send messages to contractor
- Receive automated updates (project status changes)
- Mobile-friendly interface

#### User Stories

- As a client, I want to check project progress without calling, so I stay informed conveniently
- As a client, I want to see photos of work in progress, so I feel confident about quality
- As a contractor, I want clients to see updates automatically, so I reduce status update calls
- As a business owner, I want to look modern and professional, so I win more premium clients

#### Portal Access Control (Pro tier)

- Contractor grants/revokes client access per project
- Client can only see their own projects
- Contractor controls what's visible (photos, documents, timeline)
- Activity logging (who viewed what)

---

## 6. User Roles & Permissions

### 6.1 Role Definitions

#### Owner/Admin
Full system access. Can:
- Manage all clients, quotes, projects, invoices
- Manage user accounts and permissions
- Access billing and subscription settings
- Configure company settings and branding
- Export all data
- Grant/revoke client portal access

#### Project Manager (Business tier)
Manages day-to-day operations. Can:
- Create and manage clients, quotes, projects, invoices
- Upload documents and photos
- Grant client portal access
- View reports and analytics
- Cannot: manage users, change billing, export data

#### Field Worker (Business tier)
Limited access for on-site staff. Can:
- View assigned projects
- Upload photos and documents to assigned projects
- Add project notes
- View client contact information
- Cannot: create quotes/invoices, manage clients, access billing

#### Client (Portal User)
External user with limited read-only access. Can:
- View own projects only
- See progress photos and documents shared by contractor
- View quotes and invoices
- Send messages to contractor
- Cannot: see other clients' data, modify anything, access contractor's dashboard

### 6.2 Permission Matrix

| Feature | Owner | Project Manager | Field Worker | Client |
|---------|-------|-----------------|--------------|--------|
| Clients | Full CRUD | Full CRUD | Read only | No access |
| Quotes | Full CRUD | Full CRUD | Read assigned | Read own (portal) |
| Projects | Full CRUD | Full CRUD | Read/update assigned | Read own (portal) |
| Invoices | Full CRUD | Full CRUD | No access | Read own (portal) |
| Documents | Full CRUD | Full CRUD | Create/read assigned | Read shared |
| Users | Full CRUD | No access | No access | No access |
| Settings | Full access | View only | No access | No access |
| Portal Access | Grant/revoke | Grant/revoke | No access | Own projects |
| Reports | Full access | Full access | No access | No access |
| Billing | Full access | No access | No access | No access |

---

## 7. User Stories & Use Cases

### 7.1 Epic: First Day Setup

- As a new user, I want to complete setup in under 10 minutes, so I can start using the app immediately
- As a new user, I want a tutorial on first login, so I understand basic features
- As a new user, I want to import existing clients from CSV, so I don't manually re-enter data

### 7.2 Epic: Daily Workflow

- As a contractor arriving on site, I want to quickly take and upload progress photos, so documentation is effortless
- As a contractor finishing a job, I want to create an invoice from the quote in 2 clicks, so billing is fast
- As a contractor, I want to check my dashboard each morning, so I know my priorities for the day
- As a contractor receiving a quote request, I want to create and send a professional quote in 5 minutes, so I respond quickly

### 7.3 Epic: Client Interaction

- As a contractor, I want to give clients portal access, so they stop calling for status updates
- As a client, I want to log into a portal and see my project progress, so I feel informed and confident
- As a contractor, I want to email a quote directly from the app, so I avoid manual attachment handling
- As a contractor, I want clients to digitally accept quotes, so I have clear approval records

### 7.4 Epic: Business Intelligence

- As a business owner, I want to see which quotes were accepted vs rejected, so I optimize pricing
- As a business owner, I want to see actual vs estimated project costs, so I improve estimation
- As a business owner, I want to see total outstanding payments, so I understand receivables
- As a business owner, I want to export all data to Excel, so I can do custom analysis

### 7.5 Epic: Team Collaboration (Business tier)

- As an owner, I want to add my project manager, so they can help manage the workload
- As a project manager, I want to assign projects to field workers, so they know their responsibilities
- As a field worker, I want to see my assigned projects, so I know where to work
- As an owner, I want to see who did what via activity logs, so I have accountability

### 7.6 Use Case: Creating a Quote

**Actor:** Contractor  
**Preconditions:** User is logged in, client exists in system

**Main Flow:**
1. User navigates to Quotes section
2. User clicks 'Create New Quote'
3. User selects client from dropdown
4. User enters quote details (project name, address, valid until)
5. User adds line items one by one (or from template)
6. System automatically calculates subtotals and totals
7. User adds optional notes or terms
8. User saves quote as Draft
9. User generates PDF preview
10. User sends quote via email or marks as Sent
11. System records sent timestamp

**Alternate Flows:**
- 5a. User uses template to pre-fill common line items
- 8a. User saves and sends quote in one action
- 10a. Quote is sent via client portal notification (Pro tier)

### 7.7 Use Case: Client Checking Project Status (Portal)

**Actor:** Client  
**Preconditions:** Contractor has granted portal access

**Main Flow:**
1. Client receives email with portal access link
2. Client clicks link and logs in (or uses magic link)
3. Client sees dashboard with their project(s)
4. Client clicks on specific project
5. Client sees project timeline, current status, completion percentage
6. Client browses progress photos
7. Client views original quote and scope
8. Client sees related invoices and payment status
9. (Optional) Client sends message to contractor

**Alternate Flows:**
- 6a. No photos uploaded yet - client sees 'No photos available'
- 9a. Client makes payment via integrated payment link (future feature)

---

## 8. Data Model & Business Entities

### 8.1 Core Entities Overview

The system revolves around these primary business entities:

- **Organization** (tenant)
- **User** (internal team members)
- **Client** (customers)
- **Quote** (estimates/proposals)
- **Project** (active jobs)
- **Invoice** (bills)
- **Document** (files and photos)
- **Note** (text entries)
- **Activity Log** (audit trail)

### 8.2 Entity: Organization

Represents a single business/tenant in the system.

**Attributes:**
- ID (unique identifier)
- Name (company name)
- Email (primary contact)
- Phone
- Address (full address fields)
- Logo (uploaded image)
- Subscription tier (starter/pro/business)
- Subscription status (active/cancelled/past_due)
- Settings (JSON: preferences, defaults)
- Created at
- Updated at

**Relationships:**
- Has many: Users, Clients, Quotes, Projects, Invoices, Documents

### 8.3 Entity: User

Internal team members who use the system.

**Attributes:**
- ID
- Organization ID (foreign key)
- Email (unique within organization)
- Name
- Role (owner/manager/field_worker)
- Password (hashed)
- Avatar (uploaded image)
- Timezone
- Last login
- Status (active/inactive)
- Created at
- Updated at

**Relationships:**
- Belongs to: Organization
- Has many: Activity logs (as actor)

### 8.4 Entity: Client

External customers/contacts.

**Attributes:**
- ID
- Organization ID
- Name
- Company name (optional)
- Email
- Phone
- Mobile phone (optional)
- Address (full address fields)
- Tags (array: residential, commercial, lead, etc.)
- Source (how they found the business)
- Portal enabled (boolean)
- Portal password (hashed, if portal enabled)
- Portal last login
- Notes (rich text)
- Status (lead/active/past/archived)
- Created at
- Updated at

**Relationships:**
- Belongs to: Organization
- Has many: Quotes, Projects, Invoices

### 8.5 Entity: Quote

Estimates/proposals sent to clients.

**Attributes:**
- ID
- Organization ID
- Client ID
- Quote number (auto-generated)
- Title/Project name
- Description (rich text)
- Status (draft/sent/accepted/rejected/expired)
- Valid until (date)
- Subtotal (calculated)
- Tax rate (%)
- Tax amount (calculated)
- Discount amount
- Total (calculated)
- Terms and conditions (rich text)
- Notes (internal)
- Sent at (timestamp)
- Accepted at (timestamp)
- Rejected at (timestamp)
- Rejection reason
- Created by (User ID)
- Created at
- Updated at

**Relationships:**
- Belongs to: Organization, Client, Creator (User)
- Has many: Quote line items
- Has one: Project (if accepted)

### 8.6 Entity: Quote Line Item

Individual items within a quote.

**Attributes:**
- ID
- Quote ID
- Description
- Category (labor/materials/equipment/other)
- Quantity
- Unit (hours, units, etc.)
- Unit price
- Total (calculated: quantity * unit_price)
- Sort order
- Created at
- Updated at

**Relationships:**
- Belongs to: Quote

### 8.7 Entity: Project

Active jobs being executed.

**Attributes:**
- ID
- Organization ID
- Client ID
- Quote ID (optional, if created from quote)
- Project number (auto-generated)
- Name
- Description
- Address (full address fields)
- Status (not_started/in_progress/on_hold/completed/cancelled)
- Start date
- End date (estimated)
- Actual end date
- Budget (from quote or manual)
- Actual cost (tracked)
- Completion percentage (0-100)
- Created by (User ID)
- Assigned to (User IDs array - Business tier)
- Created at
- Updated at
- Completed at

**Relationships:**
- Belongs to: Organization, Client, Quote (optional)
- Has many: Documents, Notes, Invoices, Activity logs

### 8.8 Entity: Invoice

Bills sent to clients for payment.

**Attributes:**
- ID
- Organization ID
- Client ID
- Project ID (optional)
- Quote ID (optional)
- Invoice number (auto-generated)
- Issue date
- Due date
- Status (draft/sent/paid/partial/overdue/cancelled)
- Subtotal (calculated)
- Tax rate (%)
- Tax amount (calculated)
- Discount amount
- Total (calculated)
- Amount paid
- Balance due (calculated: total - amount_paid)
- Payment terms (net 30, etc.)
- Notes (internal)
- Client notes (visible to client)
- Sent at
- Paid at
- Created by (User ID)
- Created at
- Updated at

**Relationships:**
- Belongs to: Organization, Client, Project (optional), Quote (optional)
- Has many: Invoice line items, Payment records

### 8.9 Entity: Invoice Line Item

Individual items within an invoice.

**Attributes:**
- ID
- Invoice ID
- Description
- Quantity
- Unit price
- Total (calculated)
- Sort order
- Created at
- Updated at

**Relationships:**
- Belongs to: Invoice

### 8.10 Entity: Payment

Records of payments received.

**Attributes:**
- ID
- Invoice ID
- Amount
- Payment date
- Payment method (cash/check/transfer/card/other)
- Reference/transaction number
- Notes
- Recorded by (User ID)
- Created at
- Updated at

**Relationships:**
- Belongs to: Invoice, Recorder (User)

### 8.11 Entity: Document

Files, photos, and other attachments.

**Attributes:**
- ID
- Organization ID
- Uploadable type (Project/Client/Quote/Invoice)
- Uploadable ID
- File name
- Original file name
- File type/MIME type
- File size (bytes)
- File path/URL
- Thumbnail path/URL (for images)
- Category (photo/document/contract/permit/other)
- Caption/description
- EXIF data (JSON: date, location for photos)
- Visible to client (boolean - for portal)
- Uploaded by (User ID)
- Created at
- Updated at

**Relationships:**
- Belongs to: Organization, Uploadable (polymorphic)
- Uploaded by: User

### 8.12 Entity: Note

Text notes attached to various entities.

**Attributes:**
- ID
- Noteable type (Project/Client/Quote/Invoice)
- Noteable ID
- Content (rich text)
- Type (general/internal/client_message)
- Created by (User ID)
- Created at
- Updated at

**Relationships:**
- Belongs to: Noteable (polymorphic), Creator (User)

### 8.13 Entity: Activity Log

Audit trail of all actions in the system.

**Attributes:**
- ID
- Organization ID
- User ID (actor)
- Action type (created/updated/deleted/sent/accepted/etc.)
- Subject type (Quote/Project/Invoice/etc.)
- Subject ID
- Description (human-readable)
- Changes (JSON: before/after values)
- IP address
- User agent
- Created at

**Relationships:**
- Belongs to: Organization, User (actor), Subject (polymorphic)

### 8.14 Data Relationships Summary

**Key relationships:**
- Organization → Users (1:many)
- Organization → Clients (1:many)
- Client → Quotes (1:many)
- Client → Projects (1:many)
- Client → Invoices (1:many)
- Quote → Project (1:1 optional - if quote accepted)
- Quote → Quote Line Items (1:many)
- Project → Documents (1:many)
- Project → Notes (1:many)
- Project → Invoices (1:many)
- Invoice → Invoice Line Items (1:many)
- Invoice → Payments (1:many)
- All entities → Activity Logs (polymorphic)

---

## 9. Business Rules & Workflows

### 9.1 Quote Workflow

**States:** Draft → Sent → Accepted/Rejected/Expired

**Rules:**
- Draft quotes can be edited freely
- Sent quotes are locked from editing (must create new version)
- Quote numbers are auto-generated and immutable
- Valid until date must be in the future when sending
- Accepted quotes automatically create a project
- Only one quote version can be accepted per quote
- Quotes automatically expire after 'valid until' date
- Rejected quotes can include reason for future analysis

**Calculations:**
- Subtotal = Sum of all line item totals
- Tax amount = Subtotal × Tax rate
- Total = Subtotal + Tax amount - Discount amount

### 9.2 Project Workflow

**States:** Not Started → In Progress → Completed (or On Hold/Cancelled)

**Rules:**
- Projects created from accepted quotes inherit budget from quote total
- Completion percentage is manually set by user (0-100%)
- Projects in 'Completed' status cannot be edited (read-only)
- Moving to 'Completed' records completion timestamp
- Only active projects count against subscription limits
- Completed/cancelled projects don't count against active limit
- Projects must have at least: name, client, start date

### 9.3 Invoice Workflow

**States:** Draft → Sent → Paid/Partial (or Overdue/Cancelled)

**Rules:**
- Invoice numbers are auto-generated, sequential, immutable
- Due date must be after issue date
- Overdue status is automatically set when: due date < today AND balance due > 0
- Paid status requires: amount paid = total
- Partial status when: 0 < amount paid < total
- Recording payment updates amount paid and calculates balance
- Sent invoices can still be edited (unlike quotes)
- Paid invoices can be marked unpaid if payment fails

**Calculations:**
- Subtotal = Sum of line items
- Tax amount = Subtotal × Tax rate
- Total = Subtotal + Tax amount - Discount
- Balance due = Total - Amount paid
- Overdue if: Today > Due date AND Balance due > 0

### 9.4 Subscription & Limits

**Free Tier Enforcement:**
- Active projects limit: 3 (completed/cancelled don't count)
- Total clients limit: 10 (all statuses count)
- Quotes per month: 5 (resets on 1st of month)
- Invoices per month: 3 (resets on 1st of month)
- Storage: 100 MB total (all uploaded files)
- Users: 1 (owner only)

**Upgrade Prompts:**
- When creating 4th active project: "Upgrade to Pro for more projects"
- When adding 11th client: "Upgrade to add more clients"
- When creating 6th quote this month: "Limit reached, upgrade to continue"
- When storage reaches 90%: "Running low on storage, upgrade for 10GB"
- When trying to enable client portal: "Client portal available in Pro"

**Subscription Changes:**
- Upgrade: Immediate access to higher tier features
- Downgrade: Takes effect at end of billing period
- Cancellation: Organization goes read-only after grace period
- Data retention: 90 days after cancellation, then deleted

### 9.5 Client Portal Access

**Rules:**
- Portal access is per-client (not per-project)
- Enabling portal generates secure password or magic link
- Clients see only projects where they are the client
- Contractor controls visibility: photos, documents, timeline
- Portal users cannot modify anything, read-only access
- Free tier: No portal access
- Pro tier: Up to 5 clients with portal access
- Business tier: Unlimited client portal access

### 9.6 Document & Photo Management

**Rules:**
- Photos retain EXIF metadata (date, location, camera)
- Before/after pairing is manual (user selects pairs)
- Documents can be attached to: clients, quotes, projects, invoices
- Deleting a quote/project/invoice does NOT delete attached files (orphaned)
- Storage quota is organization-wide, not per-user
- Supported image formats: JPG, PNG, WebP, HEIC
- Supported document formats: PDF, DOC, DOCX, XLS, XLSX
- Max file size: 25 MB per file
- Thumbnails auto-generated for images

### 9.7 Notifications & Reminders

**Email notifications sent for:**
- Quote sent to client (to contractor as confirmation)
- Quote accepted by client (to contractor)
- Invoice overdue (to contractor) - 1, 7, 14, 30 days overdue
- Payment received (to contractor)
- Project status changed (to client if portal enabled)
- New client portal message (to contractor)
- Low storage warning at 80%, 95%

**Tier limits:**
- Free tier: 10 email notifications per month
- Pro/Business tier: Unlimited email notifications
- Business tier: SMS notifications available

### 9.8 Data Export & Backup

**Rules:**
- Free tier: No data export capability (view only in app)
- Pro tier: Export to CSV/Excel (clients, quotes, projects, invoices)
- Business tier: API access for custom integrations
- Exports include all data fields, not redacted
- Automatic daily backups (7-day retention)
- Manual backup on demand (Pro/Business tier)

---

## 10. Integration Requirements

### 10.1 MVP Integrations

#### Email Service

**Purpose:** Send transactional emails (quotes, invoices, notifications)

**Requirements:**
- Reliable delivery (99.9% uptime)
- Template support
- Attachment support (PDF quotes/invoices)
- Tracking: opens, clicks (optional)
- Bounce and complaint handling
- Suggested providers: SendGrid, AWS SES, Resend, Postmark

#### File Storage

**Purpose:** Store uploaded documents and photos

**Requirements:**
- Scalable, cost-effective storage
- Public URLs for sharing
- Image transformation/thumbnail generation
- CDN distribution for fast loading
- Secure access control
- Suggested providers: AWS S3, Cloudinary, UploadThing, Supabase Storage

#### PDF Generation

**Purpose:** Generate professional PDFs for quotes and invoices

**Requirements:**
- HTML to PDF conversion
- Custom styling and branding
- Multi-page support
- Header/footer support
- Consistent rendering across devices
- Suggested libraries: Puppeteer, Playwright, react-pdf, PDFKit

### 10.2 Future Integrations (Post-MVP)

#### Payment Processing

**Purpose:** Enable online payments from clients  
**Providers:** Stripe, PayPal, Square

**Features:**
- One-time payments
- Deposit/partial payment handling
- Payment link generation
- Webhook for payment confirmation
- Refund processing

#### Accounting Software

**Purpose:** Sync invoices and payments to accounting systems  
**Providers:** QuickBooks, Xero, FreshBooks

**Features:**
- Auto-create invoices in accounting software
- Sync payment status
- Match clients/customers
- Expense tracking integration

#### Calendar/Scheduling

**Purpose:** Sync project deadlines to calendar apps  
**Providers:** Google Calendar, Outlook Calendar

**Features:**
- Create calendar events for project milestones
- Reminder notifications
- Two-way sync (calendar → app)

#### SMS Notifications

**Purpose:** Send urgent notifications via SMS  
**Providers:** Twilio, AWS SNS

**Features:**
- Payment reminders
- Project status updates
- Appointment confirmations

### 10.3 API Architecture

For Business tier API access:

- RESTful API design
- JWT authentication
- Rate limiting: 1000 requests/hour
- Webhook support for real-time events
- OpenAPI/Swagger documentation
- Client SDKs: JavaScript, Python, PHP

---

## 11. Non-Functional Requirements

### 11.1 Performance

- Page load time: < 2 seconds on 4G connection
- Time to interactive: < 3 seconds
- Image upload: Support up to 25 MB files
- Concurrent users: Support 100+ per organization
- Database query time: < 100ms for common operations
- PDF generation: < 5 seconds for typical quote/invoice

### 11.2 Scalability

- Support 10,000+ organizations in MVP phase
- Horizontal scaling capability for future growth
- Database sharding ready for multi-region deployment
- CDN usage for static assets and images
- Background job processing for heavy tasks (PDF generation, email sending)

### 11.3 Security

- HTTPS only (TLS 1.2+)
- Password hashing: bcrypt or Argon2
- SQL injection prevention (parameterized queries)
- XSS prevention (input sanitization)
- CSRF protection on all forms
- Rate limiting on API endpoints
- Multi-tenant data isolation (row-level security)
- Regular security audits
- Dependency vulnerability scanning
- Two-factor authentication (Pro/Business tier)

### 11.4 Reliability & Availability

- Uptime target: 99.9% (8.76 hours downtime/year)
- Automated backups: Daily with 7-day retention
- Disaster recovery plan
- Monitoring and alerting for critical errors
- Graceful error handling and user-friendly error messages

### 11.5 Usability

- Mobile-first responsive design
- Touch-friendly UI elements (min 44x44px tap targets)
- Support for major browsers: Chrome, Firefox, Safari, Edge (latest 2 versions)
- Keyboard navigation support
- Loading states and progress indicators
- Intuitive navigation (< 3 clicks to any feature)
- In-app help and tooltips
- Onboarding tutorial for new users

### 11.6 Accessibility

- WCAG 2.1 Level AA compliance
- Screen reader compatibility
- Keyboard navigation support
- Sufficient color contrast (4.5:1 minimum)
- Alt text for all images
- Form labels and error messages
- Focus indicators visible

### 11.7 Localization

- UTF-8 encoding support
- Date format preferences (MM/DD/YYYY, DD/MM/YYYY, etc.)
- Number format preferences (1,000.00 vs 1.000,00)
- Currency support (symbol and code)
- Timezone handling
- Multi-language support (future): English as MVP, expandable

### 11.8 Compliance

- GDPR compliance (data privacy, right to erasure)
- Data retention policies
- Privacy policy and terms of service
- Cookie consent management
- Data export capability (GDPR right to portability)

### 11.9 Browser & Device Support

**Desktop:**
- Chrome/Edge (latest 2 versions)
- Firefox (latest 2 versions)
- Safari (latest 2 versions)

**Mobile:**
- iOS Safari (iOS 14+)
- Chrome on Android (Android 9+)
- Responsive design: 320px to 2560px width

**Tablets:**
- iPad (Safari)
- Android tablets
- Optimized layout for landscape/portrait

---

## 12. Success Metrics & KPIs

### 12.1 User Acquisition Metrics

- New signups per month
- Signup conversion rate (visitors → signups)
- Traffic sources (organic, referral, social, paid)
- Cost per acquisition (if paid marketing)
- Viral coefficient (referrals per user)

### 12.2 User Engagement Metrics

- Daily Active Users (DAU)
- Monthly Active Users (MAU)
- DAU/MAU ratio (stickiness)
- Average session duration
- Features usage rate (% of users using each module)
- Time to first value (first quote/project created)
- Onboarding completion rate

### 12.3 Business Metrics

- Monthly Recurring Revenue (MRR)
- Free to paid conversion rate (target: 5%)
- Average Revenue Per User (ARPU)
- Customer Lifetime Value (CLV)
- Churn rate (monthly)
- Upgrade rate (free → pro → business)
- Net revenue retention

### 12.4 Product Health Metrics

- Quote creation time (average)
- Invoice creation time (average)
- Number of quotes created per user per month
- Client portal adoption rate (% of Pro users using portal)
- Storage utilization per user
- Error rate (% of requests resulting in errors)
- Page load time (P50, P95, P99)

### 12.5 Customer Satisfaction Metrics

- Net Promoter Score (NPS) - target: 50+
- Customer Satisfaction Score (CSAT) - target: 4.5/5
- Support ticket volume
- Average time to resolution
- Feature request volume and themes
- User reviews/ratings (if on app stores)

### 12.6 Open Source Metrics

- GitHub stars
- Contributors count
- Pull requests per month
- Issues opened vs closed
- Community activity (discussions, Discord members)
- Forks and self-hosted deployments

### 12.7 Success Milestones (Year 1)

**Month 3:**
- 100 active users
- MVP feature complete
- First 5 paying customers

**Month 6:**
- 1,000 active users
- 50 paying customers
- 10 GitHub contributors
- NPS 40+

**Month 12:**
- 10,000 active users
- 500 paying customers (5% conversion)
- $10K+ MRR
- 50+ contributors
- Laravel and Symfony implementations launched

---

## 13. Roadmap & Phasing

### 13.1 Phase 1: MVP (Months 1-3)

**Goal:** Deliver core functionality enabling real usage by first customers

**Features:**
- User authentication and organization setup
- Client management (CRUD)
- Quote creation, PDF generation, sending
- Project tracking (basic status workflow)
- Document/photo upload and galleries
- Invoice creation, PDF generation, sending
- Payment tracking (manual)
- Dashboard with key metrics
- Email notifications (basic)
- Mobile-responsive UI

**Success criteria:**
- End-to-end workflow: client → quote → project → invoice
- First 10 beta users actively using the system
- Average quote creation time < 5 minutes

### 13.2 Phase 2: Client Portal & Polish (Months 4-6)

**Goal:** Add premium features and improve UX based on feedback

**Features:**
- Client portal (Pro tier feature)
- Client portal: project view, timeline, photos
- Client portal: messaging contractor
- Quote templates for common job types
- Invoice templates
- Advanced search and filtering
- Bulk operations
- Activity log and audit trail
- Improved onboarding tutorial
- Performance optimizations

**Success criteria:**
- 100+ active users
- First 5 Pro tier subscribers
- Client portal adoption: 50% of Pro users
- NPS 40+

### 13.3 Phase 3: Analytics & Team Features (Months 7-9)

**Goal:** Enable business intelligence and team collaboration

**Features:**
- Advanced reporting (profitability, conversion rates)
- Project cost tracking (actual vs estimated)
- Custom dashboards
- Multi-user support (Business tier)
- Role-based permissions (owner/manager/field worker)
- Project assignment to team members
- Calendar view of projects and deadlines
- Basic scheduling features
- Data export (CSV/Excel)

**Success criteria:**
- 1,000+ active users
- First 5 Business tier subscribers
- Average of 2 users per Business tier account

### 13.4 Phase 4: Integrations & Scale (Months 10-12)

**Goal:** Expand ecosystem and prepare for scale

**Features:**
- Payment integration (Stripe)
- Accounting software integration (QuickBooks/Xero)
- API for Business tier
- Webhook system
- SMS notifications (Business tier)
- Recurring invoices
- Invoice payment links
- Multi-language support foundation
- Performance and scaling improvements
- White-label options (Business tier)

**Success criteria:**
- 10,000+ active users
- 500+ paying customers
- $10K+ MRR
- 99.9% uptime
- Laravel implementation launched

### 13.5 Phase 5: Advanced Features (Year 2)

**Features to explore:**
- Mobile apps (iOS/Android native)
- Offline mode with sync
- Advanced project management (Gantt charts, dependencies)
- Material/inventory management
- Equipment tracking
- Subcontractor management
- Time tracking
- Employee management
- Marketing tools (email campaigns to leads)
- Client reviews and testimonials module
- AI features (quote suggestions, price optimization)
- Marketplace for integrations and templates

### 13.6 Technology Stack Implementations

**Parallel development tracks:**
- Next.js implementation (primary) - Months 1-12
- Laravel implementation - Months 6-12
- Symfony implementation - Year 2
- Mobile app (React Native) - Year 2

---

## 14. Appendix

### 14.1 Glossary

- **Client:** External customer who receives quotes and has projects
- **User:** Internal team member who uses the system (contractor, owner, etc.)
- **Organization:** Single business/tenant in the system (multi-tenant architecture)
- **Quote:** Estimate or proposal sent to client before work begins
- **Project:** Active job being executed
- **Invoice:** Bill sent to client for payment
- **Line Item:** Individual item within a quote or invoice
- **Portal:** Client-facing interface for viewing project information
- **Active Project:** Project in 'Not Started' or 'In Progress' status (counts against limits)
- **Watermark:** 'Made with BuildFlow' branding on free tier PDFs

### 14.2 Assumptions

- Users have smartphone or computer with internet access
- Users are comfortable with basic web/app interfaces
- Businesses primarily operate in local markets (not global shipping)
- Primary language is English (other languages future)
- Users want mobile access but can use desktop for detailed work
- Email is primary communication channel with clients
- Most projects are quoted in advance (not time & materials only)

### 14.3 Out of Scope (MVP)

- Time tracking and timesheets
- Payroll and employee management
- Material/inventory management
- Equipment tracking
- Subcontractor management
- Recurring subscriptions or contracts
- Purchase orders
- Multi-currency support
- Multi-language interface
- Country-specific compliance (tax, legal)
- Native mobile apps (iOS/Android)
- Offline mode
- Real-time collaboration (multiple users editing simultaneously)
- Video calls or screen sharing
- Marketing automation
- Client reviews/testimonials
- AI-powered features

### 14.4 Open Questions

- Should free tier allow limited client portal access (e.g., 1 client)?
- What's optimal quote/invoice monthly limit for free tier?
- Should we support multi-currency in MVP if targeting global market?
- Do we need offline mode given contractors work on-site with spotty internet?
- Should completed projects count against active project limits?
- How to handle quote revisions - new quote or versioning?
- Should clients be able to comment on progress photos?
- What's the right balance of features in free vs paid tiers?

### 14.5 References & Inspiration

- **Jobber** - Field service management software
- **Housecall Pro** - Home service business platform
- **ServiceTitan** - Service business software (enterprise-focused)
- **Square Appointments** - Booking and payments
- **FreshBooks** - Invoicing and accounting for small businesses
- **monday.com** - Project management (inspiration for UX)
- **Linear** - Issue tracking (inspiration for clean UI)
- **Notion** - All-in-one workspace (inspiration for flexibility)

### 14.6 Document History

**Version 1.0 - November 12, 2025**
- Initial business requirements document
- Covers MVP scope and first year roadmap
- Technology-agnostic for multiple implementations

---

**Document prepared for:** BuildFlow Open Source Project  
**Prepared by:** Piotr Świderski  
**Date:** November 12, 2025  
**Version:** 1.0

*This document is technology-agnostic and can be used as the foundation for implementations in Next.js, Laravel, Symfony, or other frameworks.*