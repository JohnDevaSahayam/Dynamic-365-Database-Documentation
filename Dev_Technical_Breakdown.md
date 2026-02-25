# Microsoft Dynamics 365 CRM
## Low-Level Technical Breakdown
### For Development Teams

---

## 1. Database Schema Analysis

### Primary Key Pattern
```sql
-- All entities use GUID primary keys
[EntityId] UNIQUEIDENTIFIER NOT NULL

-- Example: Account
[AccountId] UNIQUEIDENTIFIER NOT NULL

-- Example: Opportunity  
[OpportunityId] UNIQUEIDENTIFIER NOT NULL
```

### Common Audit Columns
Every Base table includes:
```sql
CreatedOn           DATETIME
CreatedBy           UNIQUEIDENTIFIER
ModifiedOn          DATETIME
ModifiedBy          UNIQUEIDENTIFIER
VersionNumber       TIMESTAMP  -- Optimistic locking
```

---

## 2. Ownership Model

### OwnerId + OwnerIdType Pattern
```sql
OwnerId         UNIQUEIDENTIFIER NOT NULL
OwnerIdType     INT NOT NULL     -- 8=User, 9=Team
OwningBusinessUnit UNIQUEIDENTIFIER NULL
```

### Access Rights Mask
```sql
0x01 = Read
0x02 = Write
0x04 = Delete
0x08 = Append
0x10 = AppendTo
0x20 = Assign
0x40 = Share
```

---

## 3. Security Tables

### Role-Based Access
```sql
-- User to Role mapping
SystemUserRoles (SystemUserId, RoleId)

-- Team to Role mapping  
TeamRoles (TeamId, RoleId)

-- Role privileges
RolePrivileges (RoleId, PrivilegeDepth, PrivilegeId)
```

### Privilege Depth
```
1  = User
2  = Business Unit
4  = Parent:Child BU
8  = Organization
```

### Field-Level Security
```sql
PrincipalObjectAttributeAccessBase
- ObjectId
- AttributeId  
- PrincipalId
- ReadAccess
- CreateAccess
```

---

## 4. Sales Pipeline Tables

### Lead → Opportunity Conversion
```sql
LeadBase
- LeadId (PK)
- CustomerId (Account/Contact polymorphic)
- CustomerIdType (1=Account, 2=Contact)
- StateCode, StatusCode
- QualifiedContactId
- QualifiedAccountId

OpportunityBase
- OpportunityId (PK)
- CustomerId (Account/Contact polymorphic)
- CustomerIdType
- StepId (Sales stage)
- EstRevenue (Money)
- CloseDate
```

### Quote → Order → Invoice
```sql
QuoteBase
- QuoteId (PK)
- OpportunityId (FK)
- StatusCode (1=Draft, 2=Active, 3=Won, 4=Lost)

SalesOrderBase  
- SalesOrderId (PK)
- QuoteId (FK)
- StatusCode (1=Draft, 2=Submitted, 3=Fulfilled, 4=Cancelled)

InvoiceBase
- InvoiceId (PK)
- SalesOrderId (FK)
- StatusCode (1=Draft, 2=Paid, 3=Cancelled)
```

### Line Items
```sql
QuoteDetailBase
- QuoteDetailId (PK)
- QuoteId (FK)
- ProductId (FK)
- Quantity
- PricePerUnit
- ExtendedAmount

SalesOrderDetailBase
- SalesOrderDetailId (PK)
- SalesOrderId (FK)
- ProductId (FK)

InvoiceDetailBase
- InvoiceDetailId (PK)
- InvoiceId (FK)
```

---

## 5. Activity Architecture

### ActivityPointer (Unified Activity)
```sql
ActivityPointerBase
- ActivityId (PK)
- ActivityTypeCode  -- 4201=Appointment, 4202=Email, etc.
- Subject
- Description
- ScheduledEnd
- ScheduledStart
- StateCode, StatusCode
- OwnerId, OwnerIdType
```

### ActivityParty (Participants)
```sql
ActivityPartyBase
- ActivityId (FK)
- PartyId (Account/Contact/Lead/User FK)
- PartyObjectTypeCode
- ParticipationType  -- 1=Sender, 2=To, 3=CC, etc.
```

---

## 6. Product & Pricing

### Pricing Hierarchy
```sql
PriceLevelBase (Price List)
- PriceLevelId (PK)
- Name
- Currency

ProductPriceLevelBase
- ProductPriceLevelId (PK)
- ProductId (FK)
- PriceLevelId (FK)
- Amount (Money)

ProductBase
- ProductId (PK)
- Name
- ProductNumber
- Price (Money)
- StandardCost (Money)
```

---

## 7. Service Management

### Case Lifecycle
```sql
IncidentBase
- IncidentId (PK)
- Title
- CustomerId
- CaseOriginCode
- StatusCode (1=Active, 2=Resolved, 3=Cancelled)
- PriorityCode (1=Low, 2=Normal, 3=High)
```

### SLA Tracking
```sql
SLABase
- SLAId (PK)
- Name
- EntityId (Entity this SLA applies to)
- ApplicableWhenXml

SLAItemBase
- SLAItemId (PK)
- SLAId (FK)
- Name
- ApplicableWhenXml
```

---

## 8. Custom Entities

### CANVEN Extensions
```sql
canven_onboardingBase
- canven_onboardingId (PK)
- canven_propertyname
- canven_propertyaddress
- canven_approvalstatus

canven_propertyonboardBase
- canven_propertyonboardId (PK)
- canven_onboardingid (FK)
- canven_reviewstatus
```

### User-Defined Entities
```sql
new_usertypeBase
new_customercomplaintBase
new_userdetailsBase
```

---

## 9. Key Database Objects

### Partitioned Tables
```sql
-- Audit table with quarterly partitions
AuditBase
- Partitioned by CreatedOn quarterly
- Uses AuditPFN function
- AuditPScheme partition scheme
```

### Filtered Views
All user-facing queries should use filtered views:
- AccountFilteredView
- ContactFilteredView
- OpportunityFilteredView
- etc.

---

## 10. Common Queries

### Get User's Records
```sql
SELECT * FROM AccountFilteredView 
WHERE OwnerId = @UserId 
OR OwningBusinessUnit IN 
    (SELECT BusinessUnitId FROM fn_Get BUChildren(@UserId))
```

### Get Record Sharing
```sql
SELECT * FROM PrincipalObjectAccess 
WHERE ObjectId = @RecordId
AND PrincipalId = @UserId
```

### Get Activity Parties
```sql
SELECT ap.*, p.FullName 
FROM ActivityPartyBase ap
LEFT JOIN ContactBase p ON ap.PartyId = p.ContactId
WHERE ap.ActivityId = @ActivityId
```

---

## 11. Integration Points

### Web Services
- Organization.svc - Metadata
- XRMServices/2011/Organization.svc - CRUD operations

### Plugin Context
```csharp
IPluginExecutionContext context = ...
Entity entity = (Entity)inputParameters["Target"];
Guid userId = context.InitiatingUserId;
```

---

## 12. Performance Considerations

### Index Strategy
- Clustered on PK (GUID)
- Non-clustered on OwnerId
- Non-clustered on OwningBusinessUnit
- Composite indexes for common queries

### Query Guidelines
- Use filtered views for security
- Avoid SELECT * 
- Use pagination (PageInfo)
- Consider NOLOCK for read-only

---

## 13. Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Slow queries | Check missing indexes |
| Locking contention | Use optimistic locking |
| Audit bloat | Archive quarterly |
| Security checks slow | Pre-compute POA |

---

*Technical Breakdown - January 2026*
*For Development Team Reference*
