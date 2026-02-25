# Microsoft Dynamics 365 CRM Database Architecture
## Enterprise Technical Documentation

---

# 📘 DOCUMENT STRUCTURE

1. Executive Summary
2. System Architecture Overview
3. Entity Classification & Module Mapping
4. Core Data Model Analysis
5. Security Architecture
6. Ownership & Sharing Model
7. Sales Pipeline Data Flow
8. Activity Management Architecture
9. Product & Pricing Architecture
10. Custom Entities Analysis
11. Audit & Logging Mechanism
12. Data Flow Diagrams (Text-based)
13. Performance & Scalability Analysis
14. Risk Assessment
15. Recommendations & Best Practices
16. Conclusion

---

# 1. Executive Summary

## Overview

This document provides comprehensive technical documentation for the **CANCRM_MSCRM** Microsoft Dynamics 365 CRM database implementation. The database represents a complete on-premise CRM solution with full enterprise capabilities including sales pipeline management, service case management, marketing automation, and custom extensions.


## Business Capabilities

| Module | Status | Entities |
|--------|--------|----------|
| Sales Pipeline | Active | Lead, Opportunity, Quote, Order, Invoice |
| Service Management | Active | Case, Contract, Entitlement |
| Marketing | Active | Campaign, List, Campaign Response |
| Custom Extensions | Active | canven_*, new_*, msdyn_* |
| Security & Audit | Active | Full RBAC + Field-Level Security |

---

# 2. System Architecture Overview

## Database Configuration

```sql
Database: CANCRM_MSCRM
Compatibility Level: 110 (SQL Server 2016)
Recovery Model: SIMPLE
Auto-Close: OFF
Auto-Shrink: OFF
Page Verify: CHECKSUM
```

### Key Architectural Patterns Identified

1. **GUID-Based Primary Keys**: All entities use `uniqueidentifier` as primary keys
2. **Polymorphic Lookups**: CustomerId + CustomerIdType pattern for flexible relationships
3. **ActivityPointer Pattern**: Unified activity storage with type-specific attributes
4. **Ownership Model**: OwnerId + OwnerIdType for User/Team ownership
5. **Business Unit Hierarchy**: Hierarchical security propagation through BusinessUnitMap
6. **Filtered Views**: Security-trimmed views for data access control

## Partitioning Strategy

The database implements **partitioning for the Audit table** using:
- Partition Function: `AuditPFN` (quarterly boundaries)
- Partition Scheme: `AuditPScheme`
- Archives: Quarterly partitions for audit data management

---

# 3. Entity Classification & Module Mapping

## A. Organization & Business Unit

### Tables
| Table | Purpose |
|-------|---------|
| `OrganizationBase` | Single organization root entity |
| `BusinessUnitBase` | Business unit hierarchy |
| `BusinessUnitMap` | BU-to-Parent BU mapping |
| `SiteBase` | Office/site locations |

### Key Attributes
- `OrganizationId`: Unique org identifier
- `BusinessUnitId`: BU primary key
- `ParentBusinessUnitId`: Hierarchical relationship

---

## B. Security Model

### Tables
| Table | Purpose |
|-------|---------|
| `SystemUserBase` | User accounts |
| `TeamBase` | Team grouping |
| `RoleBase` | Security roles |
| `SystemUserRoles` | User-to-Role mapping |
| `TeamRoles` | Team-to-Role mapping |
| `PrivilegeBase` | Individual privileges |
| `RolePrivileges` | Role-to-Privilege mapping |
| `SystemUserPrincipals` | User principal mapping |
| `SystemUserManagerMap` | Manager hierarchy for hierarchical security |

### Security Depth Levels
- **User (1)**: Single user access
- **Business Unit (2)**: BU-level access
- **Parent:Child BU (4)**: Deep BU hierarchy
- **Organization (8)**: Full organization access

---

## C. Ownership & Sharing

### Tables
| Table | Purpose |
|-------|---------|
| `OwnerBase` | Polymorphic owner lookup |
| `PrincipalObjectAccess` | Record sharing |
| `PrincipalEntityMap` | Principal-to-Entity mapping |
| `PrincipalObjectAttributeAccess` | Field-level security |

### Ownership Types
- **Type 8**: User ownership
- **Type 9**: Team ownership

---

## D. Sales Pipeline

### Entities Flow
```
Lead → Opportunity → Quote → Order → Invoice
  ↓           ↓          ↓        ↓         ↓
LeadBase  OpportunityBase QuoteBase SalesOrderBase InvoiceBase
```

### Tables
| Entity | Table | Key Fields |
|--------|-------|------------|
| Lead | `LeadBase` | LeadId, CustomerId, Status |
| Opportunity | `OpportunityBase` | OpportunityId, CustomerId, EstRevenue, Status |
| Quote | `QuoteBase` | QuoteId, OpportunityId, Status |
| Order | `SalesOrderBase` | SalesOrderId, CustomerId, Status |
| Invoice | `InvoiceBase` | InvoiceId, CustomerId, Status |

---

## E. Activities & ActivityParty

### Core Tables
| Table | Purpose |
|-------|---------|
| `ActivityPointerBase` | Unified activity storage |
| `ActivityPartyBase` | Party participation in activities |

### Activity Types (ActivityTypeCode)
- **4201**: Appointment
- **4202**: Email
- **4204**: Fax
- **4207**: Letter
- **4210**: PhoneCall
- **4212**: Task
- **4214**: ServiceAppointment
- **4216**: SocialActivity
- **4401**: CampaignResponse
- **4402**: CampaignActivity
- **4208**: OpportunityClose
- **4209**: OrderClose
- **4211**: QuoteClose

---

## F. Product & Pricing

### Tables
| Table | Purpose |
|-------|---------|
| `ProductBase` | Product catalog |
| `ProductPriceLevelBase` | Product pricing per price list |
| `PriceLevelBase` | Price lists |
| `UoMBase` | Unit of Measure |
| `UoMScheduleBase` | UoM schedules |
| `DiscountBase` | Discount records |
| `DiscountTypeBase` | Discount types |

---

## G. Service Management

### Tables
| Table | Purpose |
|-------|---------|
| `IncidentBase` | Cases |
| `ContractBase` | Service contracts |
| `EntitlementBase` | Customer entitlements |
| `EntitlementTemplateBase` | Entitlement templates |
| `SLABase` | Service Level Agreements |

---

## H. Marketing

### Tables
| Table | Purpose |
|-------|---------|
| `CampaignBase` | Marketing campaigns |
| `CampaignActivityBase` | Campaign activities |
| `CampaignResponseBase` | Campaign responses |
| `ListBase` | Marketing lists |
| `ListMemberBase` | List membership |

---

## I. Custom Entities

### Vendor-Specific (canven_*)
- `canven_onboarding` - Onboarding process
- `canven_propertyonboard` - Property onboarding
- `canven_city` - City reference data

### User-Defined (new_*)
- `new_usertype` - User type classification
- `new_customercomplaint` - Customer complaints
- `new_userdetails` - User details extension

### Dynamics Extensions (msdyn_*)
- `msdyn_wallsavedquery` - Wall saved queries
- `msdyn_wallsavedqueryusersettings` - User wall settings
- `msdyn_postconfig` - Post configuration
- `msdyn_postalbum` - Post album
- `msdyn_PostRuleConfig` - Post rule configuration

---

# 4. Core Data Model Analysis

## Primary Key Architecture

All primary keys follow the pattern:

```sql
[EntityName]Id UNIQUEIDENTIFIER NOT NULL
```

### Example Key Structure
```sql
-- Account
[AccountId] [uniqueidentifier] NOT NULL

-- Contact  
[ContactId] [uniqueidentifier] NOT NULL

-- Opportunity
[OpportunityId] [uniqueidentifier] NOT NULL
```

## Common Columns Across Entities

| Column | Type | Purpose |
|--------|------|---------|
| `CreatedOn` | datetime | Record creation timestamp |
| `CreatedBy` | uniqueidentifier | Creator user reference |
| `ModifiedOn` | datetime | Last modification timestamp |
| `ModifiedBy` | uniqueidentifier | Modifier user reference |
| `OwnerId` | uniqueidentifier | Record owner |
| `OwnerIdType` | int | Owner type (User/Team) |
| `OwningBusinessUnit` | uniqueidentifier | Owning BU reference |
| `StateCode` | int | Record state (Active/Inactive) |
| `StatusCode` | int | Status reason |
| `VersionNumber` | timestamp | Concurrency control |

---

# 5. Security Architecture

## Role-Based Access Control (RBAC)

### Privilege Model

```
Privilege Depth Mask:
├── Bit 0 (1):  User-level access
├── Bit 1 (2):  Business Unit access
├── Bit 2 (4):  Parent:Child BU access (Deep)
└── Bit 3 (8):  Organization-level access
```

### Privilege Types

| AccessRight | Permission |
|------------|------------|
| 0x01 | Read |
| 0x02 | Write |
| 0x04 | Delete |
| 0x08 | Append |
| 0x10 | AppendTo |
| 0x20 | Assign |
| 0x40 | Share |

## Field-Level Security

Implemented via `PrincipalObjectAttributeAccess` table:
- Controls read/write access at field level
- Supports sharing with specific users/teams

---

# 6. Ownership & Sharing Model

## Ownership Architecture

### OwnerId + OwnerIdType Pattern

```sql
OwnerId UNIQUEIDENTIFIER NOT NULL
OwnerIdType INT NOT NULL  -- 8=User, 9=Team
```

### Record Sharing

The `PrincipalObjectAccess` table enables:

```sql
PrincipalObjectAccess (
    PrincipalId,      -- User/Team receiving access
    ObjectId,          -- Record being shared
    ObjectTypeCode,    -- Entity type
    AccessRightsMask,  -- Read, Write, Delete, etc.
    InheritedAccessRightsMask
)
```

### Cascade Behaviors

| Behavior | Code | Description |
|----------|------|-------------|
| Cascade | 1 | Copy ownership to child records |
| Remove Link | 2 | Remove relationship |
| Restrict | 3 | Prevent deletion if children exist |
| None | 4 | No cascading |

---

# 7. Sales Pipeline Data Flow

## Lead Qualification Flow

```
┌─────────────┐
│   Lead     │ (LeadBase)
│  Created   │
└──────┬──────┘
       │ Qualify
       ↓
┌─────────────┐
│Opportunity  │ (OpportunityBase)
│  Created    │
└──────┬──────┘
       │ Close (Won)
       ↓
┌─────────────┐
│   Quote    │ (QuoteBase)
│  Created    │
└──────┬──────┘
       │ Accept
       ↓
┌─────────────┐
│   Order    │ (SalesOrderBase)
│  Created    │
└──────┬──────┘
       │ Fulfill
       ↓
┌─────────────┐
│  Invoice   │ (InvoiceBase)
│  Created    │
└─────────────┘
```

### Key State Transitions

| Stage | StateCode | StatusCode |
|-------|-----------|------------|
| Lead - Open | 0 | 1 (New) |
| Lead - Qualified | 0 | 3 (Qualified) |
| Lead - Disqualified | 1 | 4 (Disqualified) |
| Opportunity - Open | 0 | 1 (In Progress) |
| Opportunity - Won | 1 | 3 (Won) |
| Opportunity - Lost | 2 | 4 (Lost) |

---

# 8. Activity Management Architecture

## ActivityPointer Pattern

All activities stored in single `ActivityPointerBase` table with:
- Common attributes in main table
- Type-specific attributes as optional columns
- `ActivityTypeCode` differentiates types

## ActivityParty Structure

```sql
ActivityPartyBase (
    ActivityId,        -- Reference to activity
    PartyId,           -- Reference to participant (Account/Contact/Lead/User)
    ParticipationType, -- 1=Sender, 2=To, 3=CC, etc.
    PartyObjectTypeCode -- Type of participant
)
```

### Participation Types

| Type | Meaning |
|------|---------|
| 1 | Sender |
| 2 | To Recipient |
| 3 | CC Recipient |
| 4 | BCC Recipient |
| 5 | Required Attendee |
| 6 | Optional Attendee |
| 7 | Organizer |
| 8 | Regarding |

---

# 9. Product & Pricing Architecture

## Pricing Model

```
PriceLevel (Price List)
    ↓
ProductPriceLevel (Product × Price List pricing)
    ↓
QuoteDetail / OrderDetail / InvoiceDetail (Line items)
```

## Discount Architecture

- **DiscountType**: Category of discount (e.g., Volume, Trade)
- **Discount**: Specific discount values linked to DiscountType

---

# 10. Custom Entities Analysis

## CANVEN Custom Entities

### canven_onboarding
- **Purpose**: Property onboarding workflow
- **Custom Fields**: Multiple vendor-specific fields
- **Relationships**: Linked to Account, Contact

### canven_propertyonboard
- **Purpose**: Property onboarding detail records
- **Ownership**: Includes Reviewer and Approval owners

### canven_city
- **Purpose**: Geographic reference data
- **Used By**: Multiple related entities

---

# 11. Audit & Logging Mechanism

## Audit Table Structure

```sql
AuditBase (
    AuditId,
    Operation,          -- Create=1, Update=2, Delete=3
    EntityId,           -- Record affected
    ObjectTypeCode,     -- Entity type
    AttributeMask,      -- Changed fields
    NewValue,          -- New value (for updates)
    OldValue,          -- Previous value (for updates)
    CreatedOn,
    UserId
)
```

## Audit Partitioning

Quarterly partitions ensure:
- Efficient querying for recent data
- Archival of historical audit data
- Performance optimization for large audit tables

---

# 12. Data Flow Diagrams (Text-Based)

## Record Creation Flow

```
User Request
    ↓
Application Layer
    ↓
Plugin/Workflow Execution
    ↓
Base Table Insert
    ↓
[Trigger: Cascade Operations]
    ↓
[Trigger: POA Update]
    ↓
Audit Log (if enabled)
    ↓
Filtered View Result
```

## Security Check Flow

```
User Query Request
    ↓
Filtered View Execution
    ↓
fn_GetOwnerIdsForFilteredView
    ↓
Check PrincipalEntityMap
    ↓
Check PrincipalObjectAccess (sharing)
    ↓
Apply Field-Level Security
    ↓
Return Filtered Results
```

---

# 13. Performance & Scalability Analysis

## Strengths

1. **GUID Primary Keys**: Distributed uniqueness, no hot spots
2. **Clustered Indexes**: Appropriate on primary keys
3. **Filtered Views**: Server-side security filtering
4. **Partitioning**: Audit table optimization
5. **Optimistic Locking**: VersionNumber for concurrency

## Potential Risks

1. **Fragmented Indexes**: 80% fill factor may cause page splits
2. **Missing Composite Indexes**: Common query patterns may lack coverage
3. **Large Table Scans**: Some FK relationships may cause full scans
4. **NOLOCK Hints**: Excessive use may cause dirty reads

---

# 14. Risk Assessment

## Security Risks

| Risk | Severity | Mitigation |
|------|----------|------------|
| Over-privileged roles | Medium | Regular role audits |
| Field-level security gaps | Medium | Review FLS profiles |
| Audit data exposure | Low | Partition and archive |

## Data Integrity Risks

| Risk | Severity | Mitigation |
|------|----------|------------|
| Cascade delete issues | Medium | Test cascade rules |
| Orphaned records | Low | Referential integrity |
| Duplicate detection | Low | Enable duplicate rules |

---

# 15. Recommendations & Best Practices

## Performance Recommendations

1. **Index Optimization**
   - Create covering indexes for filtered views
   - Consider filtered indexes for archived data

2. **Query Optimization**
   - Use filtered views instead of direct table access
   - Leverage pre-computed privilege masks

3. **Maintenance**
   - Regular index rebuilds (weekly)
   - Archive audit data quarterly
   - Monitor large table growth

## Security Recommendations

1. **Privilege Review**: Quarterly access audits
2. **Field Security**: Apply to sensitive data fields
3. **Sharing Limits**: Monitor excessive sharing

## Data Management

1. **Backup Strategy**: Full daily + transaction log backups
2. **Archiving**: Partition and archive audit quarterly
3. **Integration**: Use staging tables for data imports

---

# 16. Conclusion

## Summary

The **CANCRM_MSCRM** database represents a comprehensive Microsoft Dynamics 365 CRM implementation with:

- ✅ Complete sales pipeline management
- ✅ Service case management capabilities
- ✅ Marketing automation features
- ✅ Custom vendor extensions
- ✅ Enterprise-grade security model
- ✅ Audit and compliance tracking

## Key Takeaways

1. **Architecture**: Follows standard Dynamics 365 patterns with GUID-based keys and polymorphic relationships
2. **Scalability**: Partitioned audit table, optimized filtered views
3. **Customization**: Well-documented custom entities (canven_*, new_*)
4. **Security**: Multi-layered RBAC with field-level security support

## Next Steps

- Conduct performance baseline analysis
- Review custom entity relationships
- Validate security role configurations
- Implement monitoring for key tables

---

*Document Generated: January 2026*
*Database: CANCRM_MSCRM*
*Platform: Microsoft Dynamics 365 CRM On-Premise*
