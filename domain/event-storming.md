# BuildFlow - Event Storming & Domain Analysis

## 🎯 Co to jest Event Storming?

Event Storming to warsztatowa technika do odkrywania domeny biznesowej przez:
- Identyfikację **Domain Events** (co się dzieje w systemie)
- Odkrycie **Commands** (co user robi)
- Znalezienie **Bounded Contexts** (granic modułów)
- Zrozumienie **Business Processes** (przepływów)

## 🏗️ Nasza Domena: Construction Business Management

### Kluczowe Procesy Biznesowe

#### 1. **Lead to Client Journey**
```
[Lead Contact] 
    → [Client Registered]
    → [Client Profile Updated]
    → [Client Tagged]
    → [Client Activated]
```

#### 2. **Quote to Project Journey** (CORE FLOW)
```
[Quote Requested]
    → [Quote Draft Created]
    → [Line Items Added]
    → [Quote Calculated]
    → [Quote Sent to Client]
    → [Client Reviewed Quote]
    → [Quote Accepted] ✨ CRITICAL EVENT
        → [Project Created]
        → [Schedule Planned]
        → [Team Assigned]
    → [Quote Rejected]
        → [Reason Recorded]
        → [Follow-up Scheduled]
```

#### 3. **Project Execution Journey**
```
[Project Started]
    → [Work Scheduled]
    → [Materials Ordered]
    → [Work Logged]
    → [Progress Photo Uploaded]
    → [Milestone Completed]
    → [Issue Reported]
    → [Issue Resolved]
    → [Project Completed]
    → [Final Photos Uploaded]
    → [Client Notified]
```

#### 4. **Invoicing & Payment Journey**
```
[Invoice Generated]
    → [Invoice Sent]
    → [Client Viewed Invoice]
    → [Payment Received]
        → [Payment Recorded]
        → [Receipt Sent]
    → [Payment Overdue]
        → [Reminder Sent]
        → [Follow-up Scheduled]
    → [Invoice Disputed]
        → [Resolution Started]
```

#### 5. **Client Portal Journey** (Premium Feature)
```
[Portal Access Granted]
    → [Client Logged In]
    → [Project Status Viewed]
    → [Photo Gallery Browsed]
    → [Message Sent to Contractor]
    → [Invoice Viewed]
    → [Document Downloaded]
```

---

## 🎨 Bounded Contexts (Moduły Domeny)

### Identyfikowane Konteksty:

### 1. **Client Management Context** 👥
**Odpowiedzialność:** Zarządzanie relacjami z klientami

**Entities:**
- Client (Aggregate Root)
- ClientContact
- ClientNote
- ClientTag

**Value Objects:**
- Email
- PhoneNumber
- Address
- ClientStatus (Active, Inactive, Archived)

**Domain Events:**
- ClientRegistered
- ClientProfileUpdated
- ClientContactAdded
- ClientTagged
- ClientActivated
- ClientDeactivated

**Use Cases:**
- Register new client
- Update client profile
- Add contact information
- Tag client for segmentation
- Search clients
- Import clients from CSV

---

### 2. **Quote Management Context** 💼
**Odpowiedzialność:** Tworzenie i zarządzanie ofertami

**Entities:**
- Quote (Aggregate Root)
- QuoteLineItem
- QuoteVersion (for revisions)

**Value Objects:**
- Money (amount + currency)
- QuoteNumber (auto-generated)
- QuoteStatus (Draft, Sent, Accepted, Rejected, Expired)
- TaxRate
- Discount

**Domain Events:**
- QuoteDraftCreated
- LineItemAdded
- LineItemRemoved
- LineItemUpdated
- QuoteCalculated
- QuoteSentToClient
- QuoteViewed (by client)
- QuoteAccepted ⭐ CRITICAL
- QuoteRejected
- QuoteExpired
- QuoteRevisionCreated

**Business Rules:**
- Quote in "Sent" status cannot be edited
- Only one version can be "Accepted"
- Expired quotes cannot be accepted
- Total must be calculated correctly (subtotal + tax - discount)
- Quote number must be unique and sequential

**Use Cases:**
- Create quote draft
- Add/remove/update line items
- Calculate totals
- Send quote to client
- Accept quote
- Reject quote
- Create quote revision
- Convert quote to project

---

### 3. **Project Management Context** 🏗️
**Odpowiedzialność:** Śledzenie realizacji projektów

**Entities:**
- Project (Aggregate Root)
- Milestone
- WorkLog
- Issue

**Value Objects:**
- ProjectStatus (NotStarted, InProgress, OnHold, Completed, Cancelled)
- Budget (estimated vs actual)
- CompletionPercentage (0-100)
- Priority (Low, Medium, High)

**Domain Events:**
- ProjectCreated (from accepted quote)
- ProjectStarted
- MilestoneAdded
- MilestoneCompleted
- WorkLogged
- ProgressUpdated
- IssueReported
- IssueResolved
- ProjectPaused
- ProjectResumed
- ProjectCompleted
- ProjectCancelled

**Business Rules:**
- Project created only from accepted quote
- Cannot complete project with open issues
- Actual cost cannot exceed budget by more than X% without approval
- Only active projects count against subscription limits

**Use Cases:**
- Create project from quote
- Start project
- Log work hours
- Update progress
- Report issue
- Complete milestone
- Pause/resume project
- Complete project
- Cancel project

---

### 4. **Document Management Context** 📄
**Odpowiedzialność:** Zarządzanie plikami i zdjęciami

**Entities:**
- Document (Aggregate Root)
- DocumentVersion

**Value Objects:**
- FileName
- FileSize
- MimeType
- StorageUrl
- DocumentType (Photo, PDF, Contract, Invoice, Other)

**Domain Events:**
- DocumentUploaded
- DocumentCategorized
- DocumentShared (with client)
- DocumentDeleted
- DocumentVersionCreated

**Business Rules:**
- Max file size per tier (100MB free, 10GB pro)
- Storage quota enforcement
- EXIF data preserved for photos
- Thumbnails auto-generated for images

**Use Cases:**
- Upload document
- Categorize document
- Share with client (via portal)
- Create before/after pairs
- Generate thumbnail
- Delete document

---

### 5. **Invoice & Payment Context** 💰
**Odpowiedzialность:** Fakturowanie i płatności

**Entities:**
- Invoice (Aggregate Root)
- InvoiceLineItem
- Payment

**Value Objects:**
- InvoiceNumber (auto-generated)
- InvoiceStatus (Draft, Sent, Paid, PartiallyPaid, Overdue, Cancelled)
- Money
- PaymentMethod (Cash, Card, Transfer, Check)
- PaymentStatus

**Domain Events:**
- InvoiceGenerated (from quote)
- InvoiceSent
- InvoiceViewed (by client)
- PaymentReceived ⭐
- PaymentRecorded
- InvoiceMarkedAsPaid
- InvoiceBecameOverdue
- ReminderSent
- InvoiceDisputed
- InvoiceCancelled

**Business Rules:**
- Invoice number sequential and unique
- Balance = Total - AmountPaid
- Overdue if: today > due_date AND balance > 0
- Cannot delete invoice with payments
- Partial payments must be <= remaining balance

**Use Cases:**
- Generate invoice from quote
- Send invoice to client
- Record payment
- Send payment reminder
- Dispute invoice
- Cancel invoice
- Generate receipt

---

### 6. **Client Portal Context** 🌐 (Premium)
**Odpowiedzialność:** Samoobsługa klienta

**Entities:**
- PortalAccess (Aggregate Root)
- ClientMessage

**Value Objects:**
- PortalCredentials
- AccessToken
- MessageStatus (Sent, Read, Replied)

**Domain Events:**
- PortalAccessGranted
- PortalAccessRevoked
- ClientLoggedIn
- ClientViewedProject
- ClientViewedDocument
- ClientSentMessage
- ContractorRepliedToMessage

**Business Rules:**
- Only Pro/Business tiers
- Client can only see their projects
- Read-only access to data
- Can send messages to contractor

**Use Cases:**
- Grant portal access
- Revoke portal access
- Client logs in
- Client views project status
- Client browses photos
- Client sends message
- Client views invoice

---

### 7. **Team Management Context** 👨‍👩‍👧‍👦 (Business Tier)
**Odpowiedzialność:** Zarządzanie zespołem

**Entities:**
- User (Aggregate Root)
- Role
- Permission

**Value Objects:**
- UserRole (Owner, Manager, FieldWorker)
- PermissionSet

**Domain Events:**
- UserInvited
- UserJoined
- UserRoleChanged
- UserAssignedToProject
- UserUnassignedFromProject
- UserDeactivated

**Business Rules:**
- Only Business tier has multi-user
- Owner has all permissions
- Manager can manage projects
- Field worker can update status only

---

### 8. **Notification Context** 📧
**Odpowiedzialność:** Komunikacja (cross-cutting)

**Entities:**
- Notification (Aggregate Root)

**Value Objects:**
- NotificationChannel (Email, SMS, InApp)
- NotificationPriority

**Domain Events:**
- NotificationScheduled
- NotificationSent
- NotificationFailed
- NotificationOpened

**Business Rules:**
- Free tier: 10 emails/month
- Pro: Unlimited emails
- Business: Emails + SMS

---

## 🔗 Context Relationships

```
┌─────────────────┐
│  Client Mgmt    │───┐
└─────────────────┘   │
                      │ provides Client
                      ↓
┌─────────────────┐  ┌─────────────────┐
│  Quote Mgmt     │→ │  Project Mgmt   │
└─────────────────┘  └─────────────────┘
         │                    │
         │ generates          │ tracks
         ↓                    ↓
┌─────────────────┐  ┌─────────────────┐
│ Invoice & Pay   │  │   Document      │
└─────────────────┘  └─────────────────┘
         │                    │
         │                    │ shared via
         └────────┬───────────┘
                  ↓
         ┌─────────────────┐
         │  Client Portal  │
         └─────────────────┘
```

### Zależności między kontekstami:

**Client Management → All**
- Dostarcza Client jako shared concept

**Quote Management → Project Management**
- QuoteAccepted triggers ProjectCreated
- Anti-corruption layer: Quote ≠ Project (różne modele)

**Quote Management → Invoice & Payment**
- Quote może wygenerować Invoice
- Współdzielone: LineItems concept

**Project Management → Document Management**
- Project ma Documents
- Weak coupling: tylko przez ID

**All → Notification**
- Wszystkie konteksty publikują eventy
- Notification nasłuchuje i wysyła powiadomienia

---

## 🎭 Domain Events (Complete List)

### Client Management
- ClientRegistered
- ClientProfileUpdated
- ClientContactAdded
- ClientTagged

### Quote Management
- QuoteDraftCreated
- LineItemAdded
- QuoteCalculated
- QuoteSentToClient
- **QuoteAccepted** ⭐ (triggers ProjectCreated)
- QuoteRejected
- QuoteExpired

### Project Management
- **ProjectCreated** (from QuoteAccepted)
- ProjectStarted
- MilestoneCompleted
- ProgressUpdated
- ProjectCompleted

### Invoice & Payment
- InvoiceGenerated (from Quote)
- InvoiceSent
- **PaymentReceived** ⭐
- InvoiceBecameOverdue

### Document
- DocumentUploaded
- DocumentShared

### Client Portal
- PortalAccessGranted
- ClientLoggedIn
- ClientSentMessage

### Notifications (reactions to other events)
- NotificationScheduled
- NotificationSent

---

## 🎯 Critical User Journeys

### Journey 1: Quote to Project (Happy Path)
```
User: Create Quote
  → QuoteDraftCreated
  → Add Line Items
  → LineItemsAdded
  → Calculate Total
  → QuoteCalculated
User: Send Quote
  → QuoteSentToClient
  → NotificationScheduled (email to client)
Client: Reviews Quote
  → QuoteViewed
User: Mark as Accepted
  → QuoteAccepted ⭐
    ├─ ProjectCreated (async)
    ├─ NotificationScheduled (confirmation email)
    └─ InvoiceGenerated (optional)
```

### Journey 2: Project Completion Flow
```
User: Start Project
  → ProjectStarted
  → NotificationScheduled (to client)
User: Upload Progress Photos
  → DocumentUploaded (multiple)
User: Log Work
  → WorkLogged
  → ProgressUpdated
User: Complete Milestone
  → MilestoneCompleted
  → NotificationScheduled
User: Mark Project Complete
  → ProjectCompleted
  → NotificationScheduled
  → PortalNotificationSent (if portal enabled)
```

### Journey 3: Payment Flow
```
System: Generate Invoice
  → InvoiceGenerated
  → NotificationScheduled
User: Send Invoice
  → InvoiceSent
  → NotificationScheduled (email with PDF)
System: Check Due Date Daily
  → InvoiceBecameOverdue
  → NotificationScheduled (reminder)
User: Record Payment
  → PaymentReceived ⭐
  → InvoiceMarkedAsPaid
  → NotificationScheduled (receipt)
```

---

## 📊 Aggregate Design

### Quote Aggregate
```
Quote (Root)
  ├─ QuoteId (identity)
  ├─ ClientId (reference)
  ├─ OrganizationId (tenancy)
  ├─ QuoteNumber (value object)
  ├─ Status (value object)
  ├─ LineItems (entity collection) ⚠️ Part of aggregate
  │   ├─ Description
  │   ├─ Quantity
  │   ├─ UnitPrice
  │   └─ Total
  ├─ Subtotal (calculated)
  ├─ TaxRate (value object)
  ├─ Discount (value object)
  └─ Total (calculated)

Business Invariants:
- LineItems cannot be empty when sending
- Total must equal sum(lineItems) + tax - discount
- Cannot edit when status = Sent
- Cannot accept when status = Expired
```

### Project Aggregate
```
Project (Root)
  ├─ ProjectId
  ├─ ClientId (reference)
  ├─ OriginQuoteId (reference - where it came from)
  ├─ Status (value object)
  ├─ Budget (value object)
  ├─ Milestones (entity collection)
  ├─ ActualCost (tracked)
  └─ CompletionPercentage

Business Invariants:
- Budget cannot be null
- ActualCost cannot exceed budget + overage limit
- Cannot complete with open issues
- CompletionPercentage 0-100
```

---

## 🚀 Implementation Priorities

### Phase 1: Core Aggregates
1. Client (simple aggregate, good starting point)
2. Quote (complex aggregate, business critical)
3. Project (medium complexity)

### Phase 2: Supporting Contexts
4. Invoice & Payment
5. Document Management

### Phase 3: Advanced Features
6. Client Portal
7. Team Management
8. Notifications (reactive)

---

## 💡 Key Insights

### What Makes This Enterprise-Grade?

1. **Clear Boundaries**: Each context has well-defined responsibilities
2. **Domain Events**: Loose coupling between contexts
3. **Business Rules**: Encapsulated in aggregates
4. **Ubiquitous Language**: Terms from business domain
5. **Aggregate Design**: Proper boundaries and invariants

### Anti-Patterns to Avoid

❌ **Don't:**
- Share entities between contexts
- Directly call other context's repositories
- Put business logic in controllers
- Use database transactions across contexts

✅ **Do:**
- Communicate via Domain Events
- Use Anti-Corruption Layers
- Keep aggregates small
- Enforce business rules in domain

---

## 📚 Next Steps

1. Create detailed ADRs for DDD approach
2. Design folder structure for modular monolith
3. Implement Quote aggregate (showcase)
4. Setup Event Bus
5. Implement first saga (QuoteAccepted → ProjectCreated)

---

**This is the foundation for a truly enterprise-grade Laravel application! 🚀**
