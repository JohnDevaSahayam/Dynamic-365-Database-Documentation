# DATABASE ARCHITECTURE ANALYSIS REPORT
## Microsoft Dynamics 365 CRM Database - CANCRM_MSCRM

---

## TABLE OF CONTENTS

1. [Database Overview](#1-database-overview)
2. [Table Structure Analysis](#2-table-structure-analysis)
3. [Relationship Structure](#3-relationship-structure)
4. [Data Integrity Model](#4-data-integrity-model)
5. [Indexing & Performance Structure](#5-indexing--performance-structure)
6. [Security Structure](#6-security-structure)
7. [Normalization Analysis](#7-normalization-analysis)
8. [Data Flow Readiness](#8-data-flow-readiness)
9. [Architectural Quality Review](#9-architectural-quality-review)

---

# 1. DATABASE OVERVIEW

## 1.1 Purpose of the Database

The **CANCRM_MSCRM** database is a Microsoft Dynamics 365 CRM (Customer Relationship Management) system designed for enterprise-scale customer relationship management. This database serves as the backbone for:

- **Sales Management**: Lead, opportunity, quote, order, and invoice tracking
- **Service Management**: Case management, entitlements, and service scheduling
- **Marketing Automation**: Campaign management, list management, and campaign responses
- **Activity Tracking**: Emails, appointments, tasks, phone calls, and social activities
- **Document Management**: SharePoint integration and annotation storage
- **Custom Entities**: Organization-specific extensions (canven_, new_, msdyn_ prefixes)

## 1.2 Architecture Type

| Attribute | Value |
|-----------|-------|
| **Type** | OLTP (Online Transaction Processing) |
| **SQL Server Version** | SQL Server 2016+ |
| **Compatibility Level** | 110 |
| **Recovery Model** | SIMPLE |
| **Total Size** | ~652MB initial allocation |

### Key OLTP Characteristics:
- High-volume transactional processing
- Optimistic concurrency via row versioning
- Row-level security for multi-tenant isolation
- Timestamp-based change tracking

## 1.3 Design Pattern

| Pattern | Implementation |
|---------|----------------|
| **Primary** | Normalized (3NF) |
| **Secondary** | Denormalized for Performance |
| **Metadata** | Entity-Attribute-Value (EAV) pattern |
| **Temporal** | Partitioned audit tables |

### Normalized Design Rationale:
- Separate lookup tables (StringMapBase, TransactionCurrencyBase)
- Audit trail columns on every table
- Polymorphic relationships via TypeCode columns

---

# 2. TABLE STRUCTURE ANALYSIS

## 2.1 Core Entity Tables

### 2.1.1 ActivityPointerBase (Polymorphic Activity Container)

| Property | Details |
|----------|---------|
| **Table Name** | ActivityPointerBase |
| **Business Purpose** | Stores all types of activities (Email, Task, Appointment, PhoneCall, etc.) in a single polymorphic table |
| **Primary Key** | ActivityId (uniqueidentifier) |
| **Identity Column** | None - Uses GUID |
| **Clustered Index** | PK on ActivityId |
| **Estimated Cardinality** | HIGH (millions of rows) |

#### Column Categories:
- **Common Activity Columns**: Subject, Description, ScheduledStart, ScheduledEnd, PriorityCode, StatusCode, StateCode
- **Polymorphic Columns**: RegardingObjectId, RegardingObjectTypeCode (supports any entity)
- **Owner Columns**: OwnerId, OwnerIdType (User=8, Team=9)
- **Activity-Specific Columns**: Email-specific (MessageId, Sender, ToRecipients), Task-specific (PercentComplete), Appointment-specific (Location)

#### Data Types Justification:
- **uniqueidentifier**: Required for distributed system compatibility
- **datetime**: UTC timestamps for global operations
- **nvarchar(max)**: Description fields with variable length
- **bit**: Boolean flags (IsBilled, IsWorkflowCreated)

---

### 2.1.2 SystemUserBase (User Management)

| Property | Details |
|----------|---------|
| **Table Name** | SystemUserBase |
| **Business Purpose** | Stores CRM users, their credentials, and system preferences |
| **Primary Key** | SystemUserId (uniqueidentifier) |
| **Foreign Keys** | BusinessUnitId → BusinessUnitBase, TerritoryId → TerritoryBase, OrganizationId → OrganizationBase |
| **Unique Constraints** | ActiveDirectoryGuid |
| **Estimated Cardinality** | LOW (hundreds to thousands) |

#### Key Columns:
```sql
SystemUserId        uniqueidentifier    -- PK
DomainName          nvarchar(1024)     -- AD integration
BusinessUnitId      uniqueidentifier    -- FK to Organization
OrganizationId      uniqueidentifier    -- FK to Organization
IsDisabled         bit                 -- User status
AccessMode         int                -- 0=Read, 1=Write, 2=Delegate, 3=Admin
CALType            int                -- License type
```

---

### 2.1.3 OwnerBase (Unified Ownership)

| Property | Details |
|----------|---------|
| **Table Name** | OwnerBase |
| **Business Purpose** | Polymorphic ownership - supports both users and teams |
| **Primary Key** | OwnerId (uniqueidentifier) |
| **Type System** | OwnerIdType (8=User, 9=Team) |
| **Estimated Cardinality** | MEDIUM |

#### Why This Design?
All CRM entities reference this table for ownership, enabling:
- Single security check for both user and team ownership
- Dynamic reassignment between users and teams
- Simplified privilege calculation

---

### 2.1.4 TransactionCurrencyBase (Multi-Currency)

| Property | Details |
|----------|---------|
| **Table Name** | TransactionCurrencyBase |
| **Business Purpose** | Global currency management with exchange rates |
| **Primary Key** | TransactionCurrencyId (uniqueidentifier) |
| **Required Columns** | CurrencySymbol, CurrencyName, ISOCurrencyCode, CurrencyPrecision |
| **Estimated Cardinality** | LOW (10-100 currencies) |

---

### 2.1.5 OrganizationBase (Global Configuration)

| Property | Details |
|----------|---------|
| **Table Name** | OrganizationBase |
| **Business Purpose** | Global organization settings, numbering sequences, fiscal calendars |
| **Primary Key** | OrganizationId (uniqueidentifier) |
| **Estimated Cardinality** | LOW (1 per database) |

#### Critical Configuration Columns:
- **Numbering Sequences**: CurrentCaseNumber, CurrentQuoteNumber, CurrentOrderNumber, NextTrackingNumber
- **Fiscal Settings**: FiscalCalendarStart, FiscalPeriodType, FiscalYearDisplayCode
- **Email Settings**: AllowOutlookScheduledSyncs, IncomingEmailExchangeEmailRetrievalBatchSize

---

### 2.1.6 UserSettingsBase (Per-User Preferences)

| Property | Details |
|----------|---------|
| **Table Name** | UserSettingsBase |
| **Business Purpose** | User-specific preferences: timezone, language, UI settings |
| **Primary Key** | SystemUserId (uniqueidentifier) - FK to SystemUserBase |
| **Unique Constraints** | TrackingTokenId |
| **Estimated Cardinality** | LOW (matches user count) |

---

### 2.1.7 StringMapBase (Option Set Storage)

| Property | Details |
|----------|---------|
| **Table Name** | StringMapBase |
| **Business Purpose** | Stores localized option set (picklist) values |
| **Primary Key** | StringMapId (uniqueidentifier) |
| **Unique Constraints** | Composite (ObjectTypeCode, AttributeName, AttributeValue, LangId, OrganizationId) |
| **Estimated Cardinality** | MEDIUM |

---

## 2.2 Custom Entity Tables

### 2.2.1 canven_onboarding (Custom Entity - Onboarding Process)

| Property | Details |
|----------|---------|
| **Table Name** | canven_onboarding |
| **Business Purpose** | Property/vendor onboarding workflow |
| **ObjectTypeCode** | 10005 |
| **Estimated Cardinality** | MEDIUM |

### 2.2.2 canven_propertyonboard (Property Onboarding)

| Property | Details |
|----------|---------|
| **Table Name** | canven_propertyonboard |
| **Business Purpose** | Property onboarding details |
| **ObjectTypeCode** | 10008 |
| **Estimated Cardinality** | MEDIUM |

### 2.2.3 canven_city (Geographic Reference)

| Property | Details |
|----------|---------|
| **Table Name** | canven_city |
| **Business Purpose** | City master data |
| **ObjectTypeCode** | 10006 |
| **Estimated Cardinality** | LOW |

### 2.2.4 new_customercomplaint (Service Entity)

| Property | Details |
|----------|---------|
| **Table Name** | new_customercomplaint |
| **Business Purpose** | Customer complaint tracking |
| **ObjectTypeCode** | 10012 |
| **Estimated Cardinality** | MEDIUM |

### 2.2.5 msdyn_wallsavedquery (Activity Wall)

| Property | Details |
|----------|---------|
| **Table Name** | msdyn_wallsavedquery |
| **Business Purpose** | Activity wall configurations |
| **ObjectTypeCode** | 10003 |
| **Estimated Cardinality** | LOW |

---

# 3. RELATIONSHIP STRUCTURE

## 3.1 One-to-Many Relationships

```
OrganizationBase (1) ──────< SystemUserBase
    │                        │
    │                        ├─> UserSettingsBase
    │                        │
    │                        └─> SystemUserBusinessUnitEntityMap
    │
    ├─> TransactionCurrencyBase (1) ──< Many Entities
    │
    └─> BusinessUnitBase (1) ──────< SystemUserBase

SystemUserBase (1) ──────< OwnerBase (via UserTeamMembership)
```

## 3.2 Polymorphic Relationships (Many-to-One with Type Code)

### Pattern:
```sql
RegardingObjectId     uniqueidentifier  -- Foreign entity's PK
RegardingObjectTypeCode int            -- Entity type (1=Account, 2=Contact, etc.)
```

### Supported Entity Types:
| OTC | Entity | Description |
|-----|--------|-------------|
| 1 | Account | Business accounts |
| 2 | Contact | Individual contacts |
| 3 | Opportunity | Sales opportunities |
| 4 | Lead | Sales leads |
| 112 | Incident | Service cases |
| 1084 | Quote | Sales quotes |
| 1088 | SalesOrder | Sales orders |
| 1090 | Invoice | Invoices |

## 3.3 Cascade Rules

### Implemented via Functions:
| Function | Purpose |
|----------|---------|
| fn_CollectForCascadeDelete | Determines cascade delete order |
| fn_CollectForCascadeAssign | Cascades ownership changes |
| fn_CollectForCascadeReparent | Cascades parent reassignments |
| fn_CollectForCascadeShare/UnShare | Cascades sharing rules |

### Cascade Behavior by Entity Type:
- **Account**: Cascade to child Accounts, Contacts, Opportunities
- **Contact**: Cascade to related Activities
- **Case (Incident)**: Cascade to Resolutions, Articles
- **Sales Entities**: Cascade to Products, Line Items

## 3.4 Self-Referencing Relationships

```sql
AccountBase.ParentAccountId ──────< AccountBase (AccountId)
ContactBase.ParentCustomerId ──────< ContactBase (ContactId)
SystemUserBase.ParentSystemUserId ──────< SystemUserBase (SystemUserId)
BusinessUnitBase.ParentBusinessUnitId ──────< BusinessUnitBase (BusinessUnitId)
```

---

# 4. DATA INTEGRITY MODEL

## 4.1 Referential Integrity

### Implementation Approach:
- **Application-Level RI**: Foreign keys are enforced by the CRM application layer, NOT database constraints
- **Polymorphic FKs**: Uses TypeCode columns to identify referenced entity

### RI Columns Present:
| Column Pattern | Purpose |
|---------------|---------|
| CreatedBy | User who created the record |
| ModifiedBy | User who last modified |
| OwnerId | Current record owner |
| RegardingObjectId | Related entity reference |

## 4.2 Validation via Database Scoped Configurations

```sql
-- Optimistic Concurrency
READ_COMMITTED_SNAPSHOT = ON
ALLOW_SNAPSHOT_ISOLATION = ON

-- String Handling
ANSI_NULLS = ON
QUOTED_IDENTIFIER = ON
ARITHABORT = ON
```

## 4.3 Business Rule Constraints

### State Management:
- **StateCode**: Active (0) / Inactive (1)
- **StatusCode**: State-specific status values
- Enforced via StringMapBase option sets

### Required Fields (NOT NULL):
- Primary keys (uniqueidentifier)
- OwnerId on all owner entities
- StateCode on all state-managed entities
- OrganizationId on organization-scoped entities

## 4.4 Soft Delete Model

| Attribute | Implementation |
|-----------|----------------|
| **Delete Pattern** | Soft delete (StateCode = 1) |
| **Physical Delete** | Only via data retention jobs |
| **Audit Trail** | Deleted tables or partitioning |

## 4.5 Temporal Data

### Timestamp Columns:
| Column | Purpose |
|--------|---------|
| CreatedOn | Record creation time (UTC) |
| ModifiedOn | Last modification time |
| VersionNumber | Timestamp for optimistic locking |
| ImportSequenceNumber | Migration/source tracking |

---

# 5. INDEXING & PERFORMANCE STRUCTURE

## 5.1 Clustered Index Strategy

### Standard Pattern:
```sql
PRIMARY KEY CLUSTERED (UniqueIdentifierColumn)
WITH (FILLFACTOR = 80, PAD_INDEX = ON)
```

### Rationale:
- GUIDs as PKs distribute inserts across pages
- FILLFACTOR 80 reserves page space for updates
- Optimized for random insert patterns

## 5.2 Non-Clustered Indexes

### Covering Indexes:
- Composite indexes include frequently retrieved columns
- Index includes: CreatedOn, ModifiedOn, OwnerId for common queries

### Security Indexes:
```sql
-- SystemUserBusinessUnitEntityMap
CREATE CLUSTERED INDEX cndx_Cover ON (SystemUserId, ObjectTypeCode)
```

## 5.3 Unique Constraints

| Table | Constraint | Columns |
|-------|------------|---------|
| SystemUserBase | UQ_SystemUserBaseActiveDirectoryGuid | ActiveDirectoryGuid |
| UserSettingsBase | AK1_UserSettingsBase_TrackingTokenId | TrackingTokenId |
| StringMapBase | UQ_StringMap | ObjectTypeCode, AttributeName, AttributeValue, LangId, OrganizationId |

## 5.4 Missing Index Suggestions

Based on common query patterns:

```sql
-- Activity Queries
CREATE INDEX IX_ActivityPointer_RegardingObject 
ON ActivityPointerBase (RegardingObjectId, RegardingObjectTypeCode) 
INCLUDE (Subject, StateCode, OwnerId)

-- User Queries
CREATE INDEX IX_SystemUser_Organization 
ON SystemUserBase (OrganizationId, IsDisabled)

-- Owner Queries
CREATE INDEX IX_ActivityPointer_Owner 
ON ActivityPointerBase (OwnerId, OwnerIdType, StateCode)
```

## 5.5 Performance Risk Areas

| Risk | Impact | Mitigation |
|------|--------|------------|
| GUID Primary Keys | Page splits, fragmentation | Consider NEWSEQUENTIALID() |
| Wide Tables (200+ columns) | Index overhead | Columnstore for analytics |
| Polymorphic Queries | Complex joins | Indexed view materialization |
| Cascade Operations | Deadlock potential | Batch processing |

---

# 6. SECURITY STRUCTURE

## 6.1 Authentication & Authorization

### Users Defined:
```sql
-- Service Accounts
NT AUTHORITY\SYSTEM
NT AUTHORITY\NETWORK SERVICE

-- Application Groups
CANVENDOR\SQLAccessGroup
CANVENDOR\ReportingGroup
CANVENDOR\PrivReportingGroup
```

### Database Roles:
| Role | Purpose |
|------|---------|
| SQLArcExtensionUserRole | SQL-based extensions |
| CRMReaderRole | Read-only reporting access |
| db_owner | Full database control |

## 6.2 Schema Separation

| Schema | Purpose |
|--------|---------|
| dbo | Main CRM tables and objects |
| MetadataSchema | Entity metadata |
| Custom Schemas | Reporting group isolation |

## 6.3 Row-Level Security (RLS)

### Implementation via Filtered Views:
```sql
-- Example: FilteredImport security predicate
WHERE 
    -- Owner-based access
    OwnerId IN (SELECT OwnerId FROM fn_GetOwnerIdsForFilteredView(UserId, EntityId))
    OR
    -- Role-based access
    PrivilegeDepthMask & RequiredDepth > 0
    OR
    -- Shared records
    RecordId IN (SELECT ObjectId FROM fn_GetSharedRecordIdsForFilteredView(UserId, EntityId))
```

### Security Functions:
| Function | Purpose |
|----------|---------|
| fn_GetOwnerIdsForFilteredView | Owner's records |
| fn_GetSharedRecordIdsForFilteredView | Shared to user |
| fn_GetMaxPrivilegeDepthMask | User's privilege level |

## 6.4 Field-Level Security

Tracked via: PrincipalObjectAttributeAccess table
- Per-user/team attribute access
- Read/Write permission masks

## 6.5 Audit Structure

### Audit Columns (Standard):
```sql
CreatedOn          datetime    -- UTC creation time
CreatedBy          uniqueidentifier  -- Creator user ID
ModifiedOn         datetime    -- UTC modification time  
ModifiedBy         uniqueidentifier  -- Modifier user ID
VersionNumber      timestamp   -- Concurrency control
ImportSequenceNumber int      -- Data import tracking
```

---

# 7. NORMALIZATION ANALYSIS

## 7.1 Normal Form Assessment

| Level | Status | Notes |
|-------|--------|-------|
| 1NF | ✅ PASS | Atomic columns, no repeating groups |
| 2NF | ✅ PASS | Non-key columns depend on entire PK |
| 3NF | ✅ PASS | No transitive dependencies |
| BCNF | ⚠️ PARTIAL | Polymorphic FKs deviate from strict BCNF |

## 7.2 Denormalization Areas

### Purposeful Denormalization:

| Area | Denormalization | Justification |
|------|-----------------|---------------|
| ActivityPointerBase | Single table for all activities | Polymorphic pattern for CRM flexibility |
| StringMapBase | Option set values in single table | Dynamic metadata without schema changes |
| OwnerBase | Combined user/team ownership | Simplified security model |

## 7.3 Potential Redundancy

| Column Pattern | Frequency | Risk Level |
|---------------|-----------|------------|
| *Name columns | Many tables | LOW - Cached for performance |
| *YomiName columns | Contact/Account | LOW - Japanese phonetic data |
| *_Base columns | All entities | LOW - Required for filtered views |

## 7.4 Improvement Suggestions

### 1. Add Foreign Key Constraints (Optional)
```sql
ALTER TABLE SystemUserBase 
ADD CONSTRAINT FK_SystemUser_Organization 
FOREIGN KEY (OrganizationId) REFERENCES OrganizationBase(OrganizationId)
```

### 2. Consider Materialized Views for Analytics
```sql
CREATE MATERIALIZED VIEW SalesSummary AS
SELECT 
    o.OwningBusinessUnit,
    o.PriceLevelId,
    SUM(o.TotalAmount) AS TotalSales
FROM SalesOrderBase o
GROUP BY o.OwningBusinessUnit, o.PriceLevelId
```

### 3. Archive Historical Data
- Implement table partitioning for large tables
- Consider columnstore for historical reporting

---

# 8. DATA FLOW READINESS

## 8.1 Insert Flow Sequence

### Standard Entity Insert:
```
1. Generate GUID (NEWID or NEWSEQUENTIALID)
2. Set CreatedOn = GETUTCDATE()
3. Set CreatedBy = CURRENT_USER
4. Set OwnerId = Current User or Default
5. Set OrganizationId = User's Organization
6. Insert into Base table
7. Insert into Audit tables (if enabled)
8. Update Sequence counters (if applicable)
```

### Activity Insert:
```
1. Generate ActivityId (GUID)
2. Determine ActivityTypeCode
3. Map TypeCode-specific columns (Email vs Task vs Appointment)
4. Set polymorphic references (RegardingObjectId, CustomerId)
5. Insert into ActivityPointerBase
6. Insert TypeCode-specific table (EmailBase, TaskBase, etc.)
```

## 8.2 Update Flow Logic

```sql
-- Standard Update Pattern
UPDATE TableName
SET 
    Column1 = @Value1,
    ModifiedOn = GETUTCDATE(),
    ModifiedBy = @CurrentUserId,
    VersionNumber = VersionNumber + 1
WHERE 
    PrimaryKey = @PKValue
    AND VersionNumber = @OriginalVersion  -- Optimistic locking
```

### Cascade Update Behavior:
- **Ownership Change**: fn_CollectForCascadeAssign propagates to child records
- **Parent Change**: fn_CollectForCascadeReparent updates hierarchical relationships
- **Status Change**: Workflows and plugins may trigger side effects

## 8.3 Delete Flow Impact

### Cascade Delete Order (via fn_CollectForCascadeDelete):
```
1. Identify all dependent records
2. Topological sort (children first)
3. Execute in sequence:
   - Update child references to NULL (RemoveLink)
   - OR Delete child records (CascadeDelete)
   - Finally delete parent record
4. Audit trail entries created
```

### Delete Restrictions:
- **Restrict**: Prevent deletion if children exist (e.g., Account with Contacts)
- **Cascade**: Delete all children (e.g., Case with Resolutions)
- **RemoveLink**: Nullify relationship (e.g., Disconnect from Queue)

## 8.4 Dependency Chain

```
OrganizationBase (Root)
    ├── BusinessUnitBase
    │       ├── SystemUserBase ──> UserSettingsBase
    │       │           └── OwnerBase
    │       └── TeamBase
    │               └── TeamMembership
    │
    ├── TransactionCurrencyBase
    │       └── <All transactional entities>
    │
    └── Entity Tables (Account, Contact, Opportunity, etc.)
            ├── ActivityPointerBase (polymorphic)
            ├── AnnotationBase (notes/attachments)
            └── <Custom entities>
```

---

# 9. ARCHITECTURAL QUALITY REVIEW

## 9.1 Enterprise Readiness Score: **8.5/10**

## 9.2 Scalability Analysis

| Dimension | Score | Assessment |
|-----------|-------|------------|
| **Horizontal Scaling** | 9/10 | Organization-based partitioning ready |
| **Vertical Scaling** | 8/10 | Partitioning support, indexed appropriately |
| **Data Volume** | 8/10 | Handles millions of records |
| **User Concurrency** | 9/10 | Optimistic locking, row versioning |
| **Geographic Distribution** | 9/10 | UTC storage, timezone support |

## 9.3 Maintainability Analysis

| Dimension | Score | Assessment |
|-----------|-------|------------|
| **Code Organization** | 9/10 | Clear naming, documented functions |
| **Change Management** | 8/10 | Metadata-driven, solution framework |
| **Testing** | 7/10 | Unit test coverage varies |
| **Deployment** | 8/10 | Supports managed solutions |

## 9.4 Strengths

✅ **Enterprise Features**:
- Multi-currency support
- Global timezone handling
- Hierarchical security model
- Audit and tracking

✅ **Microsoft Best Practices**:
- Filtered views for security
- Optimistic concurrency
- Metadata-driven architecture
- Partitioned audit tables

✅ **Performance Optimized**:
- Fill factor on indexes
- Covering indexes for common queries
- Table-valued functions for complex logic

## 9.5 Areas for Improvement

⚠️ **Recommendations**:

1. **Add Foreign Key Constraints** (Low Priority)
   - Currently application-enforced only
   - Add for stricter data integrity

2. **Consider Sequential GUIDs**
   - Current NEWID() causes page splits
   - NEWSEQUENTIALID() improves insert performance

3. **Implement Columnstore for Analytics**
   - Large tables would benefit from hybrid columnstore
   - Improves aggregation query performance

4. **Archive Strategy**
   - Implement data lifecycle management
   - Move historical data to cheaper storage

5. **Extended Events Monitoring**
   - Add for production performance tuning
   - Monitor cascade operations

## 9.6 Summary

The **CANCRM_MSCRM** database is a **well-architected enterprise CRM system** following Microsoft Dynamics 365 best practices. It demonstrates:

- **Robust security model** with row-level filtering
- **Flexible data model** supporting polymorphic relationships
- **Enterprise-grade features** (multi-currency, timezone, localization)
- **Performance-conscious design** with appropriate indexing

The architecture is **production-ready** and **scalable** for Microsoft Dynamics 365 CRM scale deployments.

---

# APPENDIX: KEY METADATA

## Object Type Codes (OTC)

| OTC | Entity Name | Table Name |
|-----|-------------|------------|
| 1 | Account | AccountBase |
| 2 | Contact | ContactBase |
| 3 | Opportunity | OpportunityBase |
| 4 | Lead | LeadBase |
| 112 | Incident (Case) | IncidentBase |
| 10005 | canven_onboarding | canven_onboardingBase |
| 10006 | canven_city | canven_cityBase |
| 10008 | canven_propertyonboard | canven_propertyonboardBase |
| 10010 | new_usertype | new_usertypeBase |
| 10011 | new_userdetails | new_userdetailsBase |
| 10012 | new_customercomplaint | new_customercomplaintBase |

## Owner Types

| Type Code | Meaning |
|-----------|---------|
| 8 | SystemUser (Individual User) |
| 9 | Team |

## Activity Type Codes

| OTC | Activity Type |
|-----|--------------|
| 4200 | Email |
| 4201 | Appointment |
| 4202 | Task |
| 4204 | Fax |
| 4207 | Letter |
| 4210 | PhoneCall |
| 4212 | ServiceAppointment |
| 4214 | Case Resolution |
| 4401 | Campaign Response |
| 4402 | Campaign Activity |

---

*Report Generated: January 2026*
*Database: CANCRM_MSCRM*
*Analysis Tool: SQL Server Database Architect Assessment*
