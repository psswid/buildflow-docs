# BuildFlow - GitHub Roadmap
## Issues, Milestones & Project Board Structure

This document contains a complete GitHub project setup including:
- Labels taxonomy
- Milestones (Phases 0-4)
- Issues for each milestone
- Project board structure

---

## 📌 LABELS TAXONOMY

Create these labels in your GitHub repository:

### Priority Labels
- `priority: critical` - #d73a4a - Must be done ASAP
- `priority: high` - #e99695 - Important, do soon
- `priority: medium` - #fbca04 - Normal priority
- `priority: low` - #0e8a16 - Nice to have

### Type Labels
- `type: feature` - #a2eeef - New feature or enhancement
- `type: bug` - #d73a4a - Something isn't working
- `type: docs` - #0075ca - Documentation improvements
- `type: refactor` - #d4c5f9 - Code refactoring
- `type: test` - #bfd4f2 - Testing related
- `type: devops` - #5319e7 - CI/CD, deployment, infrastructure

### Module Labels
- `module: auth` - #c2e0c6 - Authentication & authorization
- `module: clients` - #c2e0c6 - Client management
- `module: quotes` - #c2e0c6 - Quote management
- `module: projects` - #c2e0c6 - Project management
- `module: invoices` - #c2e0c6 - Invoice management
- `module: documents` - #c2e0c6 - Document & photo management
- `module: dashboard` - #c2e0c6 - Dashboard & analytics
- `module: portal` - #c2e0c6 - Client portal
- `module: api` - #c2e0c6 - API development

### Status Labels
- `status: blocked` - #b60205 - Blocked by dependencies
- `status: in-progress` - #fbca04 - Currently being worked on
- `status: ready` - #0e8a16 - Ready to be picked up
- `status: needs-review` - #e99695 - Needs code review

### Stack Labels (for multi-stack implementations)
- `stack: nextjs` - #000000 - Next.js implementation
- `stack: laravel` - #ff2d20 - Laravel implementation
- `stack: symfony` - #000000 - Symfony implementation
- `stack: agnostic` - #ededed - Technology agnostic

### Other Labels
- `good first issue` - #7057ff - Good for newcomers
- `help wanted` - #008672 - Extra attention needed
- `breaking change` - #d93f0b - Breaking API changes
- `technical debt` - #d4c5f9 - Technical debt to address

---

## 🎯 MILESTONES

### Milestone 1: Phase 0 - Foundation (Weeks 1-2)
**Due Date:** 2 weeks from start  
**Description:**  
Project setup, infrastructure, and basic authentication. Goal is to have a solid foundation for development.

**Goals:**
- Repository structure and monorepo setup
- Development environment configuration
- Basic authentication working
- Database schema foundation
- UI component library setup

---

### Milestone 2: Phase 1 - MVP Core Features (Weeks 3-10)
**Due Date:** 8 weeks  
**Description:**  
Core business functionality: clients, quotes, projects, invoices, documents. End-to-end workflow from client to invoice.

**Goals:**
- Complete client management CRUD
- Quote creation with PDF generation
- Project tracking with status workflow
- Invoice generation and payment tracking
- Document upload and gallery
- Basic dashboard
- Email notifications

**Success Criteria:**
- First 10 beta users can use the system end-to-end
- Average quote creation time < 5 minutes
- Can track a project from quote to completion to invoice

---

### Milestone 3: Phase 2 - Client Portal & Polish (Weeks 11-18)
**Due Date:** 8 weeks  
**Description:**  
Premium features (client portal) and UX improvements based on MVP feedback.

**Goals:**
- Client portal with authentication
- Project timeline and status for clients
- Photo sharing via portal
- Quote and invoice templates
- Advanced search and filtering
- Performance optimizations
- Improved onboarding

**Success Criteria:**
- 100+ active users
- First 5 Pro tier subscribers
- Client portal adoption: 50% of Pro users
- NPS score 40+

---

### Milestone 4: Phase 3 - Analytics & Team (Weeks 19-26)
**Due Date:** 8 weeks  
**Description:**  
Business intelligence features and multi-user collaboration for growing businesses.

**Goals:**
- Advanced reporting and analytics
- Multi-user support with RBAC
- Team collaboration features
- Calendar and scheduling
- Cost tracking (actual vs estimated)
- Data export functionality

**Success Criteria:**
- 1,000+ active users
- First 5 Business tier subscribers
- Average 2 users per Business account

---

### Milestone 5: Phase 4 - Integrations & Scale (Weeks 27-40)
**Due Date:** 14 weeks  
**Description:**  
Third-party integrations, API, and scaling for growth.

**Goals:**
- Payment integration (Stripe)
- Accounting software integration
- Public API with documentation
- Webhook system
- SMS notifications
- Performance and scaling improvements
- White-label options

**Success Criteria:**
- 10,000+ active users
- 500+ paying customers
- $10K+ MRR
- 99.9% uptime

---

## 📋 GITHUB ISSUES

---

## PHASE 0: FOUNDATION

### Issue #1: Repository Structure & Monorepo Setup
**Labels:** `type: devops`, `priority: critical`, `stack: agnostic`  
**Milestone:** Phase 0 - Foundation  
**Assignees:** Unassigned

**Description:**
Setup repository structure with monorepo architecture (Turborepo/Nx) to support multiple implementations.

**Tasks:**
- [ ] Initialize monorepo structure
- [ ] Setup apps/ and packages/ directories
- [ ] Configure Turborepo/Nx
- [ ] Setup package.json with workspaces
- [ ] Create initial README.md with project overview
- [ ] Setup .gitignore
- [ ] Create CONTRIBUTING.md guidelines

**Acceptance Criteria:**
- Monorepo builds successfully
- Can add new apps and packages easily
- Documentation explains structure

---

### Issue #2: Development Environment & Tooling
**Labels:** `type: devops`, `priority: critical`, `stack: agnostic`  
**Milestone:** Phase 0 - Foundation

**Description:**
Setup development tools, linters, formatters, and pre-commit hooks.

**Tasks:**
- [ ] Setup ESLint configuration
- [ ] Setup Prettier configuration
- [ ] Configure Husky for git hooks
- [ ] Setup lint-staged for pre-commit
- [ ] Configure TypeScript (if applicable)
- [ ] Setup VS Code workspace settings
- [ ] Create .env.example files
- [ ] Document development setup in README

**Acceptance Criteria:**
- Code is automatically formatted on commit
- Linting passes on all files
- New contributors can setup environment in < 10 minutes

---

### Issue #3: Database Schema Design & Migrations
**Labels:** `type: feature`, `priority: critical`, `stack: agnostic`  
**Milestone:** Phase 0 - Foundation

**Description:**
Design and implement initial database schema for all core entities.

**Tasks:**
- [ ] Create ERD (Entity Relationship Diagram)
- [ ] Design schema for: organizations, users, clients, quotes, projects, invoices
- [ ] Setup migration system (Prisma/Eloquent/Doctrine)
- [ ] Create initial migration files
- [ ] Add indexes for performance
- [ ] Setup seed data for development
- [ ] Document schema in docs/

**Acceptance Criteria:**
- All core entities are in database
- Migrations run successfully
- Relationships are properly defined
- Seed data populates test environment

**Reference:**
See section 8 of Business Requirements document for complete data model.

---

### Issue #4: Authentication System
**Labels:** `type: feature`, `module: auth`, `priority: critical`  
**Milestone:** Phase 0 - Foundation

**Description:**
Implement user authentication with email/password and session management.

**Tasks:**
- [ ] Setup authentication library (NextAuth/Laravel Sanctum/Symfony Security)
- [ ] Implement registration flow
- [ ] Implement login flow
- [ ] Implement logout
- [ ] Add password reset functionality
- [ ] Email verification (optional for MVP)
- [ ] Session management
- [ ] Protected route middleware
- [ ] Write tests for auth flows

**Acceptance Criteria:**
- Users can register with email/password
- Users can login and logout
- Sessions persist correctly
- Password reset works via email
- Protected routes redirect to login
- All auth flows have test coverage

---

### Issue #5: UI Component Library Setup
**Labels:** `type: feature`, `priority: high`, `stack: agnostic`  
**Milestone:** Phase 0 - Foundation

**Description:**
Setup UI component library and design system foundation.

**Tasks:**
- [ ] Choose component library (Shadcn/Tailwind, Blade components, Twig, etc.)
- [ ] Setup Tailwind CSS (or alternative)
- [ ] Create base layout components (Header, Sidebar, Footer)
- [ ] Create form components (Input, Select, Checkbox, etc.)
- [ ] Create button variants
- [ ] Create modal/dialog component
- [ ] Setup color palette and typography
- [ ] Create Storybook (optional)
- [ ] Document component usage

**Acceptance Criteria:**
- Consistent styling across all pages
- Reusable components documented
- Responsive design works on mobile
- Accessible components (WCAG AA)

---

### Issue #6: Multi-Tenancy Architecture
**Labels:** `type: feature`, `priority: critical`, `stack: agnostic`  
**Milestone:** Phase 0 - Foundation

**Description:**
Implement multi-tenant architecture to isolate organization data.

**Tasks:**
- [ ] Design tenant isolation strategy (row-level vs schema)
- [ ] Add organization_id to all relevant tables
- [ ] Implement global scope/query filters for tenant data
- [ ] Create middleware to set current organization context
- [ ] Test data isolation between organizations
- [ ] Add organization switcher (if supporting multiple orgs per user)
- [ ] Document tenancy approach

**Acceptance Criteria:**
- Each organization only sees their own data
- Queries are automatically scoped to current organization
- No data leakage between tenants
- Performance is not significantly impacted

---

### Issue #7: File Upload Infrastructure
**Labels:** `type: devops`, `priority: high`, `stack: agnostic`  
**Milestone:** Phase 0 - Foundation

**Description:**
Setup file storage infrastructure for document and photo uploads.

**Tasks:**
- [ ] Choose storage provider (S3/Cloudinary/Local)
- [ ] Configure storage credentials
- [ ] Create upload endpoint/handler
- [ ] Implement file validation (type, size)
- [ ] Generate thumbnails for images
- [ ] Setup CDN for file delivery (if needed)
- [ ] Add file deletion functionality
- [ ] Handle EXIF data extraction
- [ ] Write upload tests

**Acceptance Criteria:**
- Files upload successfully
- File size is limited to 25MB
- Images have thumbnails generated
- Files are served via CDN (if configured)
- Upload progress is shown to user

---

### Issue #8: Email Service Integration
**Labels:** `type: feature`, `priority: high`, `stack: agnostic`  
**Milestone:** Phase 0 - Foundation

**Description:**
Integrate email service for transactional emails.

**Tasks:**
- [ ] Choose email provider (SendGrid/Resend/AWS SES)
- [ ] Configure email credentials
- [ ] Create email templates system
- [ ] Design welcome email template
- [ ] Design password reset email template
- [ ] Implement email sending service
- [ ] Add email queue for background processing
- [ ] Test email delivery
- [ ] Document email configuration

**Acceptance Criteria:**
- Emails send successfully
- Email templates are customizable
- Emails are sent asynchronously (queued)
- Failed emails are logged and retried

---

### Issue #9: Error Handling & Logging
**Labels:** `type: devops`, `priority: medium`, `stack: agnostic`  
**Milestone:** Phase 0 - Foundation

**Description:**
Implement comprehensive error handling and logging system.

**Tasks:**
- [ ] Setup logging framework
- [ ] Configure log levels (debug, info, warning, error)
- [ ] Implement global error handler
- [ ] Create user-friendly error pages (404, 500)
- [ ] Setup error monitoring (Sentry/optional)
- [ ] Log database queries in development
- [ ] Document error codes and messages
- [ ] Add error boundary components (if React)

**Acceptance Criteria:**
- All errors are logged appropriately
- Users see friendly error messages
- Developers get detailed error info in logs
- Production errors are monitored

---

### Issue #10: Testing Infrastructure
**Labels:** `type: test`, `priority: medium`, `stack: agnostic`  
**Milestone:** Phase 0 - Foundation

**Description:**
Setup testing framework and CI pipeline.

**Tasks:**
- [ ] Choose testing framework (Jest/PHPUnit/etc.)
- [ ] Configure test environment
- [ ] Setup test database
- [ ] Create test factories/fixtures
- [ ] Write example unit test
- [ ] Write example integration test
- [ ] Setup GitHub Actions CI
- [ ] Configure test coverage reporting
- [ ] Document testing approach

**Acceptance Criteria:**
- Tests run successfully
- CI pipeline runs on every PR
- Test coverage is tracked
- Testing guide is documented

---

## PHASE 1: MVP CORE FEATURES

### Issue #11: Client Management - Create Client
**Labels:** `type: feature`, `module: clients`, `priority: critical`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Implement create client functionality with form validation.

**Tasks:**
- [ ] Create client model/entity
- [ ] Create client creation form UI
- [ ] Add form validation (name, email, phone required)
- [ ] Implement create client endpoint/action
- [ ] Add client to organization automatically
- [ ] Show success/error messages
- [ ] Write unit tests for client creation
- [ ] Write integration tests

**Acceptance Criteria:**
- User can create a client with required fields
- Email validation works
- Client is associated with current organization
- Success message is shown
- Validation errors are displayed clearly

**User Story:**
As a contractor, I want to add a new client, so I can start creating quotes for them.

---

### Issue #12: Client Management - List & View Clients
**Labels:** `type: feature`, `module: clients`, `priority: critical`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Display list of all clients with search and detail view.

**Tasks:**
- [ ] Create clients list page with table/cards
- [ ] Add pagination (20 clients per page)
- [ ] Implement search by name/email
- [ ] Add filter by status (lead/active/past)
- [ ] Create client detail view page
- [ ] Show related quotes/projects/invoices
- [ ] Add "Create Quote" button from client detail
- [ ] Optimize queries (N+1 problem)

**Acceptance Criteria:**
- All clients are displayed
- Search returns relevant results
- Pagination works correctly
- Client detail shows all related data
- Page loads in < 2 seconds

---

### Issue #13: Client Management - Edit & Delete
**Labels:** `type: feature`, `module: clients`, `priority: high`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Allow editing and deleting client records.

**Tasks:**
- [ ] Create edit client form
- [ ] Pre-populate form with existing data
- [ ] Implement update client endpoint/action
- [ ] Add delete confirmation modal
- [ ] Implement soft delete (keep data, mark as deleted)
- [ ] Prevent deletion if client has active projects
- [ ] Add activity log for edits
- [ ] Write tests

**Acceptance Criteria:**
- User can edit all client fields
- Changes are saved successfully
- Cannot delete client with active projects
- Deletion requires confirmation
- Deleted clients don't appear in list

---

### Issue #14: Client Management - Tags & Categories
**Labels:** `type: feature`, `module: clients`, `priority: medium`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Add tagging system for client categorization.

**Tasks:**
- [ ] Create tags table/model
- [ ] Add tags input to client form (multi-select)
- [ ] Implement tag creation (create on-the-fly)
- [ ] Add tag filter on clients list
- [ ] Show tag badges on client cards
- [ ] Add tag management page (optional)
- [ ] Write tests for tagging

**Acceptance Criteria:**
- User can add multiple tags to client
- New tags are created automatically
- Can filter clients by tags
- Tags are displayed clearly

---

### Issue #15: Client Management - Import from CSV
**Labels:** `type: feature`, `module: clients`, `priority: low`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Allow bulk import of clients from CSV file.

**Tasks:**
- [ ] Create CSV import page
- [ ] Accept CSV file upload
- [ ] Parse CSV and validate data
- [ ] Map CSV columns to client fields
- [ ] Show preview of import
- [ ] Implement batch import
- [ ] Handle errors (duplicate emails, invalid data)
- [ ] Show import results summary
- [ ] Provide CSV template download

**Acceptance Criteria:**
- User can upload CSV with client data
- Preview shows what will be imported
- Errors are reported clearly
- Successful imports create clients
- Template CSV is available for download

---

### Issue #16: Quote Management - Create Quote with Line Items
**Labels:** `type: feature`, `module: quotes`, `priority: critical`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Implement quote creation with dynamic line items.

**Tasks:**
- [ ] Create quote model/entity and line items model
- [ ] Create quote creation form UI
- [ ] Add client selector dropdown
- [ ] Implement dynamic line item addition
- [ ] Add line item fields: description, qty, unit price
- [ ] Calculate totals automatically (subtotal, tax, total)
- [ ] Add tax rate configuration
- [ ] Add discount field (optional)
- [ ] Save quote as draft
- [ ] Write tests for quote calculations

**Acceptance Criteria:**
- User can select a client
- Can add/remove line items dynamically
- Totals calculate correctly in real-time
- Quote saves as draft
- Quote creation takes < 5 minutes

**User Story:**
As a contractor, I want to create a quote quickly, so I can respond to client requests.

---

### Issue #17: Quote Management - Quote Templates
**Labels:** `type: feature`, `module: quotes`, `priority: medium`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Create reusable quote templates for common job types.

**Tasks:**
- [ ] Create templates table/model
- [ ] Add "Save as template" option on quotes
- [ ] Create templates management page
- [ ] Allow creating quote from template
- [ ] Pre-fill line items from template
- [ ] Edit template feature
- [ ] Delete template feature
- [ ] Write tests

**Acceptance Criteria:**
- User can save quote as template
- Can create new quote from template
- Templates pre-fill line items
- Can manage templates (edit/delete)

---

### Issue #18: Quote Management - PDF Generation
**Labels:** `type: feature`, `module: quotes`, `priority: critical`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Generate professional PDF for quotes.

**Tasks:**
- [ ] Choose PDF generation library (Puppeteer/wkhtmltopdf/etc.)
- [ ] Design quote PDF template (HTML/CSS)
- [ ] Add company logo to PDF
- [ ] Include all quote details and line items
- [ ] Add terms and conditions section
- [ ] Generate PDF on demand
- [ ] Add watermark for free tier
- [ ] Cache generated PDFs
- [ ] Write tests for PDF generation

**Acceptance Criteria:**
- PDF looks professional
- All quote data is included
- PDF generates in < 5 seconds
- Free tier has watermark
- Pro tier has no watermark

---

### Issue #19: Quote Management - Send Quote via Email
**Labels:** `type: feature`, `module: quotes`, `priority: critical`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Email quotes to clients with PDF attachment.

**Tasks:**
- [ ] Create email template for quote
- [ ] Attach PDF to email
- [ ] Send to client email address
- [ ] Mark quote as "Sent" with timestamp
- [ ] Send copy to contractor (optional)
- [ ] Add resend quote feature
- [ ] Log email sending activity
- [ ] Write tests

**Acceptance Criteria:**
- Email is sent with PDF attached
- Quote status changes to "Sent"
- Timestamp is recorded
- Client receives email successfully

---

### Issue #20: Quote Management - Quote Status Workflow
**Labels:** `type: feature`, `module: quotes`, `priority: high`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Implement quote status workflow (Draft/Sent/Accepted/Rejected/Expired).

**Tasks:**
- [ ] Add status field to quotes
- [ ] Implement status change logic
- [ ] Add "Accept" action (manual for MVP)
- [ ] Add "Reject" action with reason field
- [ ] Automatic expiration based on valid_until date
- [ ] Lock editing for sent quotes
- [ ] Create activity log for status changes
- [ ] Write tests for workflow

**Acceptance Criteria:**
- Quotes progress through correct statuses
- Sent quotes cannot be edited
- Expired quotes are marked automatically
- Status changes are logged

---

### Issue #21: Quote Management - Convert Quote to Project
**Labels:** `type: feature`, `module: quotes`, `module: projects`, `priority: critical`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
One-click conversion of accepted quote to project.

**Tasks:**
- [ ] Add "Convert to Project" button on accepted quotes
- [ ] Create project from quote data
- [ ] Copy project name, client, address
- [ ] Copy budget from quote total
- [ ] Link project back to quote
- [ ] Set project status to "Not Started"
- [ ] Redirect to new project page
- [ ] Write tests

**Acceptance Criteria:**
- Accepted quote can be converted to project
- Project inherits all relevant data
- Bidirectional link between quote and project
- User is redirected to new project

**User Story:**
As a contractor, I want to convert an accepted quote to a project, so I can start tracking work.

---

### Issue #22: Project Management - Create & Edit Projects
**Labels:** `type: feature`, `module: projects`, `priority: critical`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Create and edit projects with basic information.

**Tasks:**
- [ ] Create project model/entity
- [ ] Create project form UI
- [ ] Add fields: name, client, address, dates, budget
- [ ] Add description (rich text editor)
- [ ] Implement create project action
- [ ] Implement edit project action
- [ ] Add project number auto-generation
- [ ] Write tests

**Acceptance Criteria:**
- User can create project manually
- All required fields are validated
- Project number is auto-generated
- Can edit project details
- Changes are saved successfully

---

### Issue #23: Project Management - Project Status Workflow
**Labels:** `type: feature`, `module: projects`, `priority: high`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Implement project status workflow and status changes.

**Tasks:**
- [ ] Add status field (not_started/in_progress/on_hold/completed/cancelled)
- [ ] Create status dropdown on project page
- [ ] Implement status change logic
- [ ] Record status change timestamp
- [ ] Lock completed projects from editing
- [ ] Add completion percentage field
- [ ] Create activity log for status changes
- [ ] Write tests

**Acceptance Criteria:**
- User can change project status easily
- Status changes are logged
- Completed projects are read-only
- Only active projects count against limits

---

### Issue #24: Project Management - Projects List & Dashboard
**Labels:** `type: feature`, `module: projects`, `priority: critical`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Display all projects with filtering and search.

**Tasks:**
- [ ] Create projects list page
- [ ] Add Kanban board view (optional)
- [ ] Add table view
- [ ] Implement filter by status
- [ ] Implement search by name/client
- [ ] Sort by start date, end date, status
- [ ] Show completion percentage
- [ ] Add "Quick actions" (change status, view)
- [ ] Optimize queries

**Acceptance Criteria:**
- All projects are displayed
- Can switch between views
- Filters work correctly
- Page loads quickly
- User can quickly see active projects

**User Story:**
As a contractor, I want to see all active projects, so I can plan my week.

---

### Issue #25: Document Management - File Upload
**Labels:** `type: feature`, `module: documents`, `priority: critical`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Upload documents and photos to projects.

**Tasks:**
- [ ] Create documents model/entity
- [ ] Create upload UI (drag & drop)
- [ ] Implement file upload endpoint
- [ ] Validate file types and sizes
- [ ] Store files in storage (S3/local)
- [ ] Extract EXIF data from photos
- [ ] Generate thumbnails for images
- [ ] Associate files with project
- [ ] Show upload progress
- [ ] Write tests

**Acceptance Criteria:**
- User can upload multiple files
- Drag & drop works
- Progress is shown during upload
- Files are stored securely
- Thumbnails are generated for images

**User Story:**
As a contractor, I want to upload photos from my phone, so I document progress.

---

### Issue #26: Document Management - Photo Gallery
**Labels:** `type: feature`, `module: documents`, `priority: high`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Display uploaded photos in gallery view.

**Tasks:**
- [ ] Create gallery component
- [ ] Display thumbnails in grid
- [ ] Add lightbox for full-size view
- [ ] Add photo captions/descriptions
- [ ] Show EXIF data (date, location)
- [ ] Add before/after pairing (manual)
- [ ] Implement photo sorting (date, name)
- [ ] Add delete photo action
- [ ] Write tests

**Acceptance Criteria:**
- Photos display in attractive grid
- Clicking photo opens lightbox
- EXIF data is visible
- Can add captions to photos
- Can delete photos

---

### Issue #27: Document Management - Document List & Download
**Labels:** `type: feature`, `module: documents`, `priority: medium`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
List all documents and allow downloads.

**Tasks:**
- [ ] Create documents list view
- [ ] Show document icon based on type
- [ ] Display file name, size, upload date
- [ ] Add download button
- [ ] Add delete button
- [ ] Filter by document type
- [ ] Add document preview (PDF inline)
- [ ] Write tests

**Acceptance Criteria:**
- All documents are listed
- User can download any file
- File types are visually distinguished
- Can filter by type (photo/document/contract)

---

### Issue #28: Invoice Management - Create Invoice
**Labels:** `type: feature`, `module: invoices`, `priority: critical`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Create invoices with line items similar to quotes.

**Tasks:**
- [ ] Create invoice model/entity and line items
- [ ] Create invoice creation form UI
- [ ] Add client selector
- [ ] Add project selector (optional)
- [ ] Implement dynamic line items
- [ ] Calculate totals (subtotal, tax, total)
- [ ] Add payment terms field
- [ ] Add invoice number auto-generation
- [ ] Save invoice as draft
- [ ] Write tests

**Acceptance Criteria:**
- User can create invoice
- Line items work like quotes
- Totals calculate correctly
- Invoice number is auto-generated
- Invoice saves successfully

**User Story:**
As a contractor, I want to create an invoice quickly, so I can bill clients.

---

### Issue #29: Invoice Management - Generate from Quote
**Labels:** `type: feature`, `module: invoices`, `module: quotes`, `priority: critical`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Create invoice from existing quote with one click.

**Tasks:**
- [ ] Add "Create Invoice" button on quotes
- [ ] Copy line items from quote to invoice
- [ ] Copy client and project info
- [ ] Set invoice date and due date
- [ ] Link invoice back to quote
- [ ] Redirect to invoice edit page
- [ ] Write tests

**Acceptance Criteria:**
- Invoice is pre-filled from quote
- All line items are copied
- User can edit before saving
- Link between quote and invoice exists

**User Story:**
As a contractor, I want to create an invoice from a quote, so I don't re-enter information.

---

### Issue #30: Invoice Management - Invoice PDF Generation
**Labels:** `type: feature`, `module: invoices`, `priority: critical`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Generate professional invoice PDF.

**Tasks:**
- [ ] Design invoice PDF template
- [ ] Include company details and logo
- [ ] Show invoice number, date, due date
- [ ] List all line items with totals
- [ ] Show payment terms
- [ ] Add watermark for free tier
- [ ] Generate PDF on demand
- [ ] Write tests

**Acceptance Criteria:**
- Invoice PDF looks professional
- All details are included
- PDF generates quickly
- Free tier has watermark

---

### Issue #31: Invoice Management - Payment Tracking
**Labels:** `type: feature`, `module: invoices`, `priority: high`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Track payments received against invoices.

**Tasks:**
- [ ] Create payments model/entity
- [ ] Add "Record Payment" button on invoices
- [ ] Create payment recording form
- [ ] Add fields: amount, date, method, reference
- [ ] Calculate balance due (total - amount paid)
- [ ] Update invoice status (paid/partial/overdue)
- [ ] Show payment history on invoice
- [ ] Write tests

**Acceptance Criteria:**
- User can record payments
- Balance due updates correctly
- Invoice status changes appropriately
- Payment history is visible

**User Story:**
As a contractor, I want to record partial payments, so I track what's still owed.

---

### Issue #32: Invoice Management - Overdue Detection & Reminders
**Labels:** `type: feature`, `module: invoices`, `priority: medium`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Automatically detect overdue invoices and send reminders.

**Tasks:**
- [ ] Create scheduled job for overdue detection
- [ ] Run daily to check due dates
- [ ] Update status to "overdue" when due_date < today
- [ ] Create email reminder template
- [ ] Send reminder at 1, 7, 14, 30 days overdue
- [ ] Track reminder sent dates
- [ ] Add manual "Send Reminder" button
- [ ] Write tests

**Acceptance Criteria:**
- Overdue invoices are detected automatically
- Reminders are sent on schedule
- Can manually send reminder
- Reminder history is logged

---

### Issue #33: Dashboard - Overview Widgets
**Labels:** `type: feature`, `module: dashboard`, `priority: high`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Create dashboard with key metrics and quick actions.

**Tasks:**
- [ ] Create dashboard page
- [ ] Add "Active Projects" widget with count
- [ ] Add "Pending Quotes" widget
- [ ] Add "Outstanding Invoices" widget with total
- [ ] Add "This Month Revenue" widget
- [ ] Add recent activity feed
- [ ] Add quick action buttons
- [ ] Add storage usage indicator
- [ ] Optimize dashboard queries

**Acceptance Criteria:**
- Dashboard loads in < 2 seconds
- All widgets show correct data
- Quick actions work
- Dashboard is mobile-friendly

**User Story:**
As a contractor, I want to see my priorities when I open the app.

---

### Issue #34: Email Notifications System
**Labels:** `type: feature`, `priority: medium`, `stack: agnostic`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Implement email notifications for key events.

**Tasks:**
- [ ] Create notification system architecture
- [ ] Send email when quote is sent
- [ ] Send email when quote is accepted
- [ ] Send email when invoice is overdue
- [ ] Send email when payment is received
- [ ] Track notification count per month (free tier limit)
- [ ] Add notification preferences page
- [ ] Write tests

**Acceptance Criteria:**
- Emails are sent for key events
- Free tier respects 10 email/month limit
- Users can customize notification preferences
- Emails are queued and sent asynchronously

---

### Issue #35: Subscription & Limits Enforcement
**Labels:** `type: feature`, `priority: high`, `stack: agnostic`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Enforce free tier limits and show upgrade prompts.

**Tasks:**
- [ ] Add subscription tier field to organizations
- [ ] Implement limit checks (projects, clients, quotes, invoices)
- [ ] Block actions when limit reached
- [ ] Show upgrade modal with clear messaging
- [ ] Track monthly usage (quotes, invoices, notifications)
- [ ] Reset monthly counters on 1st of month
- [ ] Add storage usage tracking
- [ ] Write tests for limit enforcement

**Acceptance Criteria:**
- Free tier limits are enforced
- User sees clear message when limit reached
- Monthly counters reset correctly
- Upgrade path is obvious

---

### Issue #36: Responsive Mobile Design
**Labels:** `type: feature`, `priority: critical`, `stack: agnostic`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Ensure all pages are mobile-responsive and touch-friendly.

**Tasks:**
- [ ] Test all pages on mobile devices
- [ ] Fix layout issues on small screens
- [ ] Ensure tap targets are 44x44px minimum
- [ ] Test forms on mobile
- [ ] Optimize image loading on mobile
- [ ] Add mobile navigation menu
- [ ] Test upload functionality on mobile
- [ ] Conduct mobile usability testing

**Acceptance Criteria:**
- All pages work on mobile (320px width)
- Forms are usable on mobile
- Photos can be uploaded from phone camera
- Navigation is intuitive on mobile

---

### Issue #37: Onboarding Tutorial
**Labels:** `type: feature`, `priority: medium`, `stack: agnostic`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Create first-time user onboarding tutorial.

**Tasks:**
- [ ] Design onboarding flow
- [ ] Add welcome modal on first login
- [ ] Create interactive tutorial (tooltips)
- [ ] Highlight key features (clients, quotes, projects)
- [ ] Add "Skip tutorial" option
- [ ] Mark tutorial as completed
- [ ] Add "Restart tutorial" in settings
- [ ] Write tests

**Acceptance Criteria:**
- New users see tutorial on first login
- Tutorial explains core features
- Can skip or restart tutorial
- Tutorial doesn't annoy returning users

---

### Issue #38: Activity Log & Audit Trail
**Labels:** `type: feature`, `priority: low`, `stack: agnostic`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Log all important actions for audit trail.

**Tasks:**
- [ ] Create activity_logs table
- [ ] Log client CRUD actions
- [ ] Log quote status changes
- [ ] Log project status changes
- [ ] Log invoice creation and payments
- [ ] Show activity feed on dashboard
- [ ] Add detailed activity log page
- [ ] Write tests

**Acceptance Criteria:**
- All important actions are logged
- Activity includes who, what, when
- Activity is visible on dashboard
- Full activity log is searchable

---

### Issue #39: Performance Optimization
**Labels:** `type: refactor`, `priority: medium`, `stack: agnostic`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Optimize application performance for better UX.

**Tasks:**
- [ ] Profile slow database queries
- [ ] Add database indexes
- [ ] Implement eager loading (N+1 fix)
- [ ] Add caching for dashboard widgets
- [ ] Optimize image loading (lazy load)
- [ ] Minify CSS and JavaScript
- [ ] Enable gzip compression
- [ ] Conduct performance testing

**Acceptance Criteria:**
- Dashboard loads in < 2 seconds
- No N+1 query problems
- Images load progressively
- Lighthouse score > 80

---

### Issue #40: MVP Documentation
**Labels:** `type: docs`, `priority: medium`, `stack: agnostic`  
**Milestone:** Phase 1 - MVP Core Features

**Description:**
Document MVP features for users and developers.

**Tasks:**
- [ ] Write user guide for all MVP features
- [ ] Create video tutorials (optional)
- [ ] Document API endpoints
- [ ] Update README with setup instructions
- [ ] Create FAQ page
- [ ] Document deployment process
- [ ] Add screenshots to documentation
- [ ] Review and publish docs

**Acceptance Criteria:**
- All MVP features are documented
- New users can follow user guide
- Developers can setup project from docs
- FAQ answers common questions

---

## PHASE 2: CLIENT PORTAL & POLISH

### Issue #41: Client Portal - Authentication System
**Labels:** `type: feature`, `module: portal`, `priority: critical`  
**Milestone:** Phase 2 - Client Portal & Polish

**Description:**
Implement secure login for clients to access portal.

**Tasks:**
- [ ] Create separate portal subdomain/path
- [ ] Implement magic link authentication
- [ ] Add password authentication (optional)
- [ ] Send portal invite email to client
- [ ] Create portal login page
- [ ] Add "Forgot password" flow
- [ ] Implement session management
- [ ] Write tests for auth flows

**Acceptance Criteria:**
- Clients can login via magic link
- Login is secure and session-based
- Invite emails are sent successfully
- Portal is separate from main app

---

### Issue #42: Client Portal - Project Dashboard
**Labels:** `type: feature`, `module: portal`, `priority: critical`  
**Milestone:** Phase 2 - Client Portal & Polish

**Description:**
Show client their projects with status and timeline.

**Tasks:**
- [ ] Create portal dashboard page
- [ ] List all projects for logged-in client
- [ ] Show project status and completion %
- [ ] Display project timeline with milestones
- [ ] Add project details page
- [ ] Show original quote
- [ ] Link to related invoices
- [ ] Write tests

**Acceptance Criteria:**
- Client sees only their own projects
- Project status is up-to-date
- Timeline is easy to understand
- Can view full project details

**User Story:**
As a client, I want to check project progress, so I stay informed.

---

### Issue #43: Client Portal - Photo Gallery
**Labels:** `type: feature`, `module: portal`, `priority: high`  
**Milestone:** Phase 2 - Client Portal & Polish

**Description:**
Allow clients to view progress photos shared by contractor.

**Tasks:**
- [ ] Show photos marked as "visible to client"
- [ ] Display in chronological order
- [ ] Add photo gallery with lightbox
- [ ] Show photo captions and dates
- [ ] Add before/after view
- [ ] Optimize image loading
- [ ] Write tests

**Acceptance Criteria:**
- Client sees photos contractor shared
- Gallery loads quickly
- Photos have dates and captions
- Before/after comparison works

**User Story:**
As a client, I want to see progress photos, so I feel confident about quality.

---

### Issue #44: Client Portal - Messaging Contractor
**Labels:** `type: feature`, `module: portal`, `priority: medium`  
**Milestone:** Phase 2 - Client Portal & Polish

**Description:**
Enable clients to send messages to contractor via portal.

**Tasks:**
- [ ] Create messages table/model
- [ ] Add message form on portal
- [ ] Send message to contractor's email
- [ ] Show message history on portal
- [ ] Add contractor reply functionality (main app)
- [ ] Send email notification to client when replied
- [ ] Write tests

**Acceptance Criteria:**
- Client can send messages
- Contractor receives email notification
- Message history is visible
- Contractor can reply from main app

---

### Issue #45: Client Portal - Invoice & Payment Status
**Labels:** `type: feature`, `module: portal`, `priority: high`  
**Milestone:** Phase 2 - Client Portal & Polish

**Description:**
Show clients their invoices and payment status.

**Tasks:**
- [ ] List all invoices for client
- [ ] Show invoice status (paid/unpaid/overdue)
- [ ] Display balance due
- [ ] Allow invoice PDF download
- [ ] Add payment link (future: integrate Stripe)
- [ ] Show payment history
- [ ] Write tests

**Acceptance Criteria:**
- Client sees all their invoices
- Can download invoice PDFs
- Payment status is clear
- Balance due is visible

---

### Issue #46: Quote & Invoice Templates System
**Labels:** `type: feature`, `module: quotes`, `module: invoices`, `priority: medium`  
**Milestone:** Phase 2 - Client Portal & Polish

**Description:**
Create customizable templates for quotes and invoices.

**Tasks:**
- [ ] Create templates table for quotes/invoices
- [ ] Add template builder UI
- [ ] Allow customizing PDF layout
- [ ] Add custom fields to templates
- [ ] Save templates per organization
- [ ] Select template when creating quote/invoice
- [ ] Provide default templates
- [ ] Write tests

**Acceptance Criteria:**
- User can create custom templates
- Templates are organization-specific
- Can select template when creating document
- Default templates are provided

---

### Issue #47: Advanced Search & Filtering
**Labels:** `type: feature`, `priority: medium`, `stack: agnostic`  
**Milestone:** Phase 2 - Client Portal & Polish

**Description:**
Add powerful search across all entities.

**Tasks:**
- [ ] Implement global search bar
- [ ] Search across clients, quotes, projects, invoices
- [ ] Add advanced filters (date range, status, amount)
- [ ] Show search results grouped by type
- [ ] Add saved searches/filters
- [ ] Optimize search queries
- [ ] Add search history
- [ ] Write tests

**Acceptance Criteria:**
- Can search from anywhere in app
- Results are fast (< 1 second)
- Filters narrow results effectively
- Can save common searches

---

### Issue #48: Bulk Operations
**Labels:** `type: feature`, `priority: low`, `stack: agnostic`  
**Milestone:** Phase 2 - Client Portal & Polish

**Description:**
Enable bulk actions on multiple items.

**Tasks:**
- [ ] Add checkboxes to list views
- [ ] Implement "select all" functionality
- [ ] Add bulk delete action
- [ ] Add bulk status change (projects, quotes)
- [ ] Add bulk export
- [ ] Show progress for bulk operations
- [ ] Write tests

**Acceptance Criteria:**
- Can select multiple items
- Bulk actions work correctly
- Progress is shown for long operations
- Can undo accidental bulk delete (confirmation)

---

### Issue #49: Performance Monitoring & Optimization
**Labels:** `type: devops`, `priority: medium`, `stack: agnostic`  
**Milestone:** Phase 2 - Client Portal & Polish

**Description:**
Add performance monitoring and optimize slow areas.

**Tasks:**
- [ ] Setup performance monitoring tool
- [ ] Identify slow database queries
- [ ] Add missing database indexes
- [ ] Implement result caching
- [ ] Optimize large file uploads
- [ ] Add CDN for static assets
- [ ] Lazy load images and components
- [ ] Conduct load testing

**Acceptance Criteria:**
- All pages load in < 2 seconds
- Slow queries are identified and fixed
- Monitoring dashboard is accessible
- Can handle 100+ concurrent users

---

### Issue #50: Improved Onboarding & UX Polish
**Labels:** `type: feature`, `priority: high`, `stack: agnostic`  
**Milestone:** Phase 2 - Client Portal & Polish

**Description:**
Enhance onboarding flow and polish overall UX.

**Tasks:**
- [ ] Redesign welcome screen
- [ ] Add interactive product tour
- [ ] Improve form validation messages
- [ ] Add inline help and tooltips
- [ ] Improve error messages
- [ ] Add empty states with CTAs
- [ ] Conduct UX testing with real users
- [ ] Implement feedback

**Acceptance Criteria:**
- New users complete onboarding easily
- Error messages are helpful
- Empty states guide users
- Overall UX feels polished

---

## PHASE 3: ANALYTICS & TEAM

### Issue #51: Multi-User Support - Team Management
**Labels:** `type: feature`, `priority: critical`, `stack: agnostic`  
**Milestone:** Phase 3 - Analytics & Team

**Description:**
Allow adding team members to organization.

**Tasks:**
- [ ] Add team members management page
- [ ] Implement invite user flow
- [ ] Send invitation email
- [ ] User accepts invite and creates account
- [ ] Assign roles (owner/manager/field_worker)
- [ ] Implement role-based permissions
- [ ] Add remove team member action
- [ ] Write tests

**Acceptance Criteria:**
- Owner can invite team members
- Invitations work via email
- Roles are assigned correctly
- Permissions are enforced

---

### Issue #52: Role-Based Access Control (RBAC)
**Labels:** `type: feature`, `priority: critical`, `stack: agnostic`  
**Milestone:** Phase 3 - Analytics & Team

**Description:**
Implement granular permissions based on roles.

**Tasks:**
- [ ] Create permission matrix
- [ ] Implement middleware for permission checks
- [ ] Restrict routes based on roles
- [ ] Hide UI elements based on permissions
- [ ] Add permission checks on all actions
- [ ] Test all permission scenarios
- [ ] Document permission system

**Acceptance Criteria:**
- Field workers can't access billing
- Managers can't delete users
- Permissions are enforced server-side
- UI reflects user permissions

---

### Issue #53: Project Assignment to Team Members
**Labels:** `type: feature`, `module: projects`, `priority: high`  
**Milestone:** Phase 3 - Analytics & Team

**Description:**
Assign projects to specific team members.

**Tasks:**
- [ ] Add "Assigned to" field on projects
- [ ] Allow multiple team members per project
- [ ] Add project assignment UI
- [ ] Filter projects by assigned user
- [ ] Show "My Projects" view
- [ ] Send notification when assigned
- [ ] Write tests

**Acceptance Criteria:**
- Can assign multiple users to project
- Team members see their assigned projects
- Notifications are sent on assignment
- Can filter by assigned user

---

### Issue #54: Calendar & Scheduling View
**Labels:** `type: feature`, `priority: medium`, `stack: agnostic`  
**Milestone:** Phase 3 - Analytics & Team

**Description:**
Add calendar view for projects and deadlines.

**Tasks:**
- [ ] Create calendar component
- [ ] Show projects on calendar by date range
- [ ] Display deadlines and milestones
- [ ] Add month/week/day views
- [ ] Allow drag & drop to reschedule (optional)
- [ ] Sync with project dates
- [ ] Export to Google Calendar/iCal
- [ ] Write tests

**Acceptance Criteria:**
- Calendar displays all projects
- Can switch between views
- Deadlines are highlighted
- Calendar is mobile-friendly

---

### Issue #55: Advanced Reporting - Profitability Analysis
**Labels:** `type: feature`, `module: dashboard`, `priority: high`  
**Milestone:** Phase 3 - Analytics & Team

**Description:**
Show profitability reports for projects.

**Tasks:**
- [ ] Create reports page
- [ ] Calculate profit per project (budget - actual cost)
- [ ] Show profit margin percentage
- [ ] Display in table and chart format
- [ ] Filter by date range
- [ ] Compare estimated vs actual
- [ ] Export report to CSV
- [ ] Write tests

**Acceptance Criteria:**
- Shows profit/loss per project
- Charts are easy to understand
- Can filter by date range
- Reports are accurate

**User Story:**
As a business owner, I want to see project profitability, so I improve estimation.

---

### Issue #56: Advanced Reporting - Quote Conversion Analysis
**Labels:** `type: feature`, `module: dashboard`, `priority: medium`  
**Milestone:** Phase 3 - Analytics & Team

**Description:**
Analyze quote acceptance vs rejection rates.

**Tasks:**
- [ ] Calculate quote conversion rate
- [ ] Show accepted vs rejected breakdown
- [ ] Display reasons for rejection
- [ ] Show conversion rate over time (chart)
- [ ] Compare by client or project type
- [ ] Export data
- [ ] Write tests

**Acceptance Criteria:**
- Shows overall conversion rate
- Rejection reasons are visible
- Can see trends over time
- Data helps improve pricing

---

### Issue #57: Advanced Reporting - Cash Flow Overview
**Labels:** `type: feature`, `module: dashboard`, `priority: high`  
**Milestone:** Phase 3 - Analytics & Team

**Description:**
Show cash flow and outstanding payments.

**Tasks:**
- [ ] Calculate total outstanding invoices
- [ ] Show aging report (30/60/90 days overdue)
- [ ] Display upcoming payments due
- [ ] Show revenue by month (chart)
- [ ] Compare revenue year-over-year
- [ ] Export report
- [ ] Write tests

**Acceptance Criteria:**
- Cash flow is clearly visualized
- Aging report shows overdue breakdowns
- Revenue trends are visible
- Reports guide collection efforts

---

### Issue #58: Project Cost Tracking (Actual vs Estimated)
**Labels:** `type: feature`, `module: projects`, `priority: medium`  
**Milestone:** Phase 3 - Analytics & Team

**Description:**
Track actual costs against estimated budget.

**Tasks:**
- [ ] Add "actual cost" field to projects
- [ ] Allow recording expenses per project
- [ ] Calculate variance (actual - estimated)
- [ ] Show cost breakdown (labor, materials)
- [ ] Display on project detail page
- [ ] Add to profitability reports
- [ ] Write tests

**Acceptance Criteria:**
- Can track actual costs
- Variance is calculated automatically
- Cost breakdown is visible
- Helps improve future estimates

---

### Issue #59: Custom Dashboards
**Labels:** `type: feature`, `module: dashboard`, `priority: low`  
**Milestone:** Phase 3 - Analytics & Team

**Description:**
Allow users to customize dashboard widgets.

**Tasks:**
- [ ] Create widget library
- [ ] Add drag & drop dashboard builder
- [ ] Save dashboard configuration per user
- [ ] Add widget: revenue chart
- [ ] Add widget: project timeline
- [ ] Add widget: top clients
- [ ] Reset to default dashboard option
- [ ] Write tests

**Acceptance Criteria:**
- Users can customize their dashboard
- Changes persist across sessions
- Can reset to default
- Dashboard loads quickly

---

### Issue #60: Data Export Functionality
**Labels:** `type: feature`, `priority: medium`, `stack: agnostic`  
**Milestone:** Phase 3 - Analytics & Team

**Description:**
Export data to CSV/Excel for external analysis.

**Tasks:**
- [ ] Add export button on all list pages
- [ ] Export clients to CSV
- [ ] Export quotes to CSV
- [ ] Export projects to CSV
- [ ] Export invoices to CSV
- [ ] Include all fields in export
- [ ] Handle large datasets (background job)
- [ ] Write tests

**Acceptance Criteria:**
- All entity types can be exported
- CSV includes all relevant fields
- Large exports don't timeout
- Exported data is accurate

---

## PHASE 4: INTEGRATIONS & SCALE

### Issue #61: Payment Integration - Stripe Setup
**Labels:** `type: feature`, `priority: critical`, `stack: agnostic`  
**Milestone:** Phase 4 - Integrations & Scale

**Description:**
Integrate Stripe for online payments.

**Tasks:**
- [ ] Setup Stripe account and API keys
- [ ] Install Stripe SDK
- [ ] Create payment link generation
- [ ] Add payment link to invoices
- [ ] Implement webhook for payment confirmation
- [ ] Update invoice status on payment
- [ ] Handle refunds
- [ ] Write tests

**Acceptance Criteria:**
- Payment links are generated correctly
- Clients can pay via Stripe
- Invoices auto-update on payment
- Webhooks work reliably

---

### Issue #62: Accounting Integration - QuickBooks/Xero
**Labels:** `type: feature`, `priority: medium`, `stack: agnostic`  
**Milestone:** Phase 4 - Integrations & Scale

**Description:**
Sync invoices to QuickBooks or Xero.

**Tasks:**
- [ ] Setup OAuth for QuickBooks/Xero
- [ ] Create invoice sync endpoint
- [ ] Map invoice fields to accounting software
- [ ] Sync payment status
- [ ] Handle sync errors gracefully
- [ ] Add sync status indicator
- [ ] Write tests

**Acceptance Criteria:**
- User can connect accounting software
- Invoices sync automatically
- Payment status syncs back
- Errors are reported clearly

---

### Issue #63: Public API Development
**Labels:** `type: feature`, `module: api`, `priority: high`  
**Milestone:** Phase 4 - Integrations & Scale

**Description:**
Create public API for Business tier customers.

**Tasks:**
- [ ] Design RESTful API endpoints
- [ ] Implement JWT authentication
- [ ] Create API key management
- [ ] Document all endpoints (OpenAPI/Swagger)
- [ ] Add rate limiting (1000 req/hour)
- [ ] Version API (v1)
- [ ] Create API playground
- [ ] Write comprehensive tests

**Acceptance Criteria:**
- API is fully documented
- Authentication works securely
- Rate limiting is enforced
- API is versioned

---

### Issue #64: Webhook System
**Labels:** `type: feature`, `module: api`, `priority: medium`  
**Milestone:** Phase 4 - Integrations & Scale

**Description:**
Allow customers to receive real-time event notifications.

**Tasks:**
- [ ] Create webhook configuration UI
- [ ] Add webhook endpoints management
- [ ] Implement webhook signing (security)
- [ ] Send webhooks for key events (quote accepted, invoice paid)
- [ ] Retry failed webhooks
- [ ] Log webhook delivery attempts
- [ ] Write tests

**Acceptance Criteria:**
- Customers can register webhook URLs
- Webhooks are sent for key events
- Failed webhooks are retried
- Webhook signatures can be verified

---

### Issue #65: SMS Notifications (Twilio)
**Labels:** `type: feature`, `priority: low`, `stack: agnostic`  
**Milestone:** Phase 4 - Integrations & Scale

**Description:**
Send SMS notifications for urgent updates (Business tier).

**Tasks:**
- [ ] Setup Twilio account and API
- [ ] Implement SMS sending service
- [ ] Send SMS for overdue invoices
- [ ] Send SMS for project status changes
- [ ] Add SMS preferences to user settings
- [ ] Track SMS usage per organization
- [ ] Write tests

**Acceptance Criteria:**
- SMS are sent successfully
- Users can opt in/out of SMS
- SMS usage is tracked
- Only Business tier has access

---

### Issue #66: Recurring Invoices
**Labels:** `type: feature`, `module: invoices`, `priority: medium`  
**Milestone:** Phase 4 - Integrations & Scale

**Description:**
Create invoices that recur automatically (monthly, etc.).

**Tasks:**
- [ ] Add recurring invoice template
- [ ] Configure recurrence: frequency, start/end dates
- [ ] Create scheduled job to generate invoices
- [ ] Send recurring invoices automatically
- [ ] Allow pausing/stopping recurrence
- [ ] Show next invoice date
- [ ] Write tests

**Acceptance Criteria:**
- Recurring invoices generate automatically
- Recurrence schedule is configurable
- Can pause or stop recurrence
- Next invoice date is visible

---

### Issue #67: White-Label Options (Business Tier)
**Labels:** `type: feature`, `priority: low`, `stack: agnostic`  
**Milestone:** Phase 4 - Integrations & Scale

**Description:**
Allow Business tier to customize branding.

**Tasks:**
- [ ] Add custom logo upload
- [ ] Allow custom color scheme
- [ ] Custom domain for client portal
- [ ] Remove "BuildFlow" branding
- [ ] Custom email sender name
- [ ] Custom PDF headers/footers
- [ ] Write tests

**Acceptance Criteria:**
- Business tier can upload logo
- Custom colors apply throughout app
- Client portal uses custom domain
- All branding is customizable

---

### Issue #68: Multi-Language Support Foundation
**Labels:** `type: feature`, `priority: low`, `stack: agnostic`  
**Milestone:** Phase 4 - Integrations & Scale

**Description:**
Setup i18n infrastructure for future language support.

**Tasks:**
- [ ] Setup i18n library (i18next/similar)
- [ ] Extract all strings to translation files
- [ ] Create English translation file
- [ ] Implement language switcher
- [ ] Detect browser language
- [ ] Allow user to select language
- [ ] Document translation process
- [ ] Write tests

**Acceptance Criteria:**
- All strings are translatable
- Can switch languages (even if only English available)
- Translation files are organized
- Ready for community translations

---

### Issue #69: Horizontal Scaling & Performance
**Labels:** `type: devops`, `priority: critical`, `stack: agnostic`  
**Milestone:** Phase 4 - Integrations & Scale

**Description:**
Prepare infrastructure for handling 10,000+ users.

**Tasks:**
- [ ] Setup load balancer
- [ ] Configure database read replicas
- [ ] Implement Redis caching
- [ ] Setup background job queue
- [ ] Configure CDN for static assets
- [ ] Add auto-scaling rules
- [ ] Conduct load testing (1000+ concurrent users)
- [ ] Document infrastructure

**Acceptance Criteria:**
- Can handle 10,000+ organizations
- Load testing passes
- No single point of failure
- 99.9% uptime achieved

---

### Issue #70: Security Audit & Hardening
**Labels:** `type: devops`, `priority: critical`, `stack: agnostic`  
**Milestone:** Phase 4 - Integrations & Scale

**Description:**
Comprehensive security review and improvements.

**Tasks:**
- [ ] Conduct security audit
- [ ] Fix identified vulnerabilities
- [ ] Implement 2FA for owner accounts
- [ ] Add security headers (CSP, etc.)
- [ ] Setup automated vulnerability scanning
- [ ] Implement rate limiting on all endpoints
- [ ] Add CAPTCHA to public forms
- [ ] Document security practices

**Acceptance Criteria:**
- No critical vulnerabilities
- 2FA is available for all users
- Security headers are configured
- Automated scanning is active

---

## 📊 PROJECT BOARD STRUCTURE

Create a GitHub Project Board with these columns:

1. **📋 Backlog** - Issues not yet prioritized
2. **🎯 Ready** - Issues ready to be worked on
3. **🚧 In Progress** - Currently being developed
4. **👀 Review** - Awaiting code review
5. **✅ Done** - Completed and merged

---

## 🏷️ ISSUE TEMPLATE

Use this template for creating new issues:

```markdown
## Description
[Brief description of the feature/bug]

## User Story (if applicable)
As a [type of user], I want [goal], so that [benefit].

## Tasks
- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

## Acceptance Criteria
- [ ] Criteria 1
- [ ] Criteria 2
- [ ] Criteria 3

## Technical Notes
[Any technical considerations, dependencies, or constraints]

## Related Issues
- Related to #[issue number]
- Depends on #[issue number]
- Blocks #[issue number]

## Screenshots/Mockups (if applicable)
[Add any relevant visuals]
```

---

## 📈 PROGRESS TRACKING

Track progress using GitHub Milestones:

```
Phase 0: [2/10 issues completed] ████░░░░░░ 20%
Phase 1: [0/30 issues completed] ░░░░░░░░░░ 0%
Phase 2: [0/10 issues completed] ░░░░░░░░░░ 0%
Phase 3: [0/10 issues completed] ░░░░░░░░░░ 0%
Phase 4: [0/10 issues completed] ░░░░░░░░░░ 0%
```

---

## 🎯 SPRINT PLANNING

Organize issues into 2-week sprints:

**Sprint 1-2:** Issues #1-10 (Phase 0)  
**Sprint 3-10:** Issues #11-40 (Phase 1 MVP)  
**Sprint 11-14:** Issues #41-50 (Phase 2)  
**Sprint 15-18:** Issues #51-60 (Phase 3)  
**Sprint 19-26:** Issues #61-70 (Phase 4)

---

## 📝 NOTES

- Each issue should be < 5 days of work
- Issues can be broken down further if needed
- Adjust priorities based on user feedback
- Milestone dates are flexible
- Community contributors should start with "good first issue" labels

---

**Last Updated:** November 12, 2025  
**Total Issues:** 70  
**Estimated Duration:** 40 weeks (~10 months)