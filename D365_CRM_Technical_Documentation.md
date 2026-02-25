        # Microsoft Dynamics 365 CRM Sales Module - Enterprise Technical Documentation


---

## Executive Summary

This document provides comprehensive technical documentation for the Microsoft Dynamics 365 CRM Sales Module database structure. The database implements a full-featured CRM architecture with 300+ tables, supporting enterprise sales processes including Lead management, Opportunity tracking, Quote generation, Order processing, and Invoice management. The architecture follows standard Dynamics 365/Dataverse patterns with Base tables, logical views, and filtered security views.

---

## 1. Full Data Structure Analysis

### 1.1 Database Architecture Overview

The CRM_DB database implements a layered architecture typical of Microsoft Dynamics 365:

| Layer | Table Pattern | Purpose |
|-------|---------------|---------|
| **Base Layer** | `[EntityName]Base` | Physical storage of entity data |
| **Logical Layer** | `[EntityName]` | Logical view joining base tables with related entities |
| **Security Layer** | `Filtered[EntityName]` | Row-level security filtered views |
| **Audit Layer** | `AuditBase` | Comprehensive change tracking |

### 1.2 Core Sales Entities

The following table maps the primary sales lifecycle entities and their Object Type Codes (OTC):

| Entity | Table | OTC | Purpose |
|--------|-------|-----|---------|
| Account | AccountBase | 1 | Business customers/organizations |
| Contact | ContactBase | 2 | Individual customers |
| Lead | LeadBase | 4 | Prospective customer (pre-qualification) |
| Opportunity | OpportunityBase | 3 | Qualified sales opportunity |
| Quote | QuoteBase | 1084 | Sales quotation |
| Sales Order | SalesOrderBase | 1088 | Customer order |
| Invoice | InvoiceBase | 1090 | Customer invoice |
| Competitor | CompetitorBase | 14 | Competitive intelligence |
| Product | ProductBase | 1024 | Products/services catalog |
| Price Level | PriceLevelBase | 1022 | Pricing lists |

### 1.3 Custom Entities (canven_ prefix)

The database includes custom entities for property/onboarding management:

| Entity | Table | OTC | Purpose |
|--------|-------|-----|---------|
| canven_city | canven_cityBase | 10006 | City master data |
| canven_onboarding | canven_onboardingBase | 10005 | Customer onboarding workflow |
| canven_propertyonboard | canven_propertyonboardBase | 10008 | Property onboarding |
| new_usertype | new_usertypeBase | 10010 | User classification |
| new_userdetails | new_userdetailsBase | 10011 | Extended user information |
| new_customercomplaint | new_customercomplaintBase | 10012 | Customer complaint management |

---

## 2. Entity-Level Deep Dive

### 2.1 Lead Entity (LeadBase)

**Purpose**: Capture and manage prospective customer information before qualification.

**Business Role**: Lead management is the entry point in the sales cycle. Leads represent potential customers discovered through marketing campaigns, referrals, or cold outreach.

**Table Structure**:
```
LeadBase (
    LeadId (uniqueidentifier) - PK
    OwningBusinessUnit (uniqueidentifier) - FK to BusinessUnitBase
    OwnerId (uniqueidentifier) - FK to OwnerBase
    StateCode (int) - Entity state
    StatusCode (int) - Status reason
    Subject (nvarchar)
    CompanyName (nvarchar)
    FirstName (nvarchar)
    LastName (nvarchar)
    EmailAddress (nvarchar)
    Telephone (nvarchar)
    Website (nvarchar)
    IndustryCode (int)
    LeadSourceCode (int)
    CampaignId (uniqueidentifier) - FK to CampaignBase
    OriginatingLeadId (uniqueidentifier) - Self-referential FK
    ParentAccountId (uniqueidentifier) - FK to AccountBase
    ParentContactId (uniqueidentifier) - FK to ContactBase
    QualifyingOpportunityId (uniqueidentifier) - FK to OpportunityBase
    CreditOnHold (bit)
    DoNotPhone (bit)
    DoNotEmail (bit)
    DoNotPostalMail (bit)
    TraversedPath (nvarchar)
    ProcessId (uniqueidentifier)
    StageId (uniqueidentifier)
    ... (additional 60+ attributes)
)
```

**Key Attributes**:
- **LeadId**: Uniqueidentifier primary key with NEWSEQUENTIALID() default
- **StateCode/StatusCode**: State machine controlling lead lifecycle
- **CustomerId/CustomerIdType**: Polymorphic lookup supporting both Account and Contact
- **QualifyingOpportunityId**: Links to created Opportunity during qualification
- **TraversedPath**: Business Process Flow stage tracking

**Relationships**:
- N:1 with BusinessUnitBase (OwningBusinessUnit)
- N:1 with OwnerBase (OwnerId)
- N:1 with CampaignBase (CampaignId - lead source)
- N:1 with AccountBase (ParentAccountId)
- N:1 with ContactBase (ParentContactId)
- N:1 with OpportunityBase (QualifyingOpportunityId)
- 1:N with LeadAddressBase (child addresses)
- 1:N with LeadCompetitors (N:N via junction)
- 1:N with LeadProduct (N:N via junction)
- 1:N with AccountLeads (N:N via junction)
- 1:N with ContactLeads (N:N via junction)

**State Management**:
| StateCode | State | StatusCode Options |
|-----------|-------|-------------------|
| 0 | Open | 1=New, 2=Attempted to Contact, 3=Contacted, 4=Open |
| 1 | Qualified | 5=Qualified, 6=Disqualified |
| 2 | Disqualified | 7=Lost, 8=Cannot Contact, 9=Not Interested |

**Ownership Model**: User or Team ownership via OwnerBase polymorphic relationship

**Security Scope**: Row-level security via OwningBusinessUnit and OwnerId

---

### 2.2 Opportunity Entity (OpportunityBase)

**Purpose**: Track qualified sales opportunities through the sales process.

**Business Role**: Opportunity represents a qualified lead or existing customer interest that has measurable revenue potential and is being actively pursued.

**Table Structure**:
```
OpportunityBase (
    OpportunityId (uniqueidentifier) - PK
    OwningBusinessUnit (uniqueidentifier) - FK
    OwnerId (uniqueidentifier) - FK
    StateCode (int)
    StatusCode (int)
    Name (nvarchar) - Opportunity title
    CustomerId (uniqueidentifier) - Polymorphic (Account/Contact)
    CustomerIdType (int) - 1=Account, 2=Contact
    CustomerIdName (nvarchar) - Denormalized name
    CustomerIdYomiName (nvarchar) - Phonetic name
    PriceLevelId (uniqueidentifier) - FK to PriceLevelBase
    CurrencyId (uniqueidentifier) - FK to TransactionCurrencyBase
    EstimatedValue (money)
    EstimatedValue_Base (money) - Base currency equivalent
    ActualValue (money)
    EstimatedCloseDate (datetime)
    ActualCloseDate (datetime)
    CloseProbability (int) - Percentage 0-100
    StepId (uniqueidentifier)
    StepName (nvarchar)
    SalesStageCode (int)
    SalesStage (nvarchar)
    CampaignId (uniqueidentifier) - Source campaign
    OriginatingLeadId (uniqueidentifier) - Source lead
    ParentAccountId (uniqueidentifier)
    ParentContactId (uniqueidentifier)
    IsRevenueSystemCalculated (bit)
    
    // Financial Fields
    TotalAmount (money)
    TotalAmount_Base (money)
    DiscountAmount (money)
    DiscountPercentage (money)
    FreightAmount (money)
    TotalTax (money)
    TotalLineItemAmount (money)
    TotalDiscountAmount (money)
    TotalAmountLessFreight (money)
    
    // Business Process Flow
    TraversedPath (nvarchar)
    ProcessId (uniqueidentifier)
    StageId (uniqueidentifier)
    
    // Date Fields
    ScheduleFollowup_Prospect (datetime)
    ScheduleFollowup_Qualify (datetime)
    ScheduleProposalMeeting (datetime)
    DevelopProposal (datetime)
    CompleteInternalReview (datetime)
    PresentProposal (datetime)
    ... (80+ additional attributes)
)
```

**Key Attributes**:
- **CustomerId/CustomerIdType**: Polymorphic relationship supporting both Account and Contact
- **EstimatedValue/ActualValue**: Revenue tracking with base currency equivalents
- **CloseProbability**: Sales forecasting metric
- **PriceLevelId**: Links to pricing structure
- **TotalAmount fields**: Aggregate calculations from OpportunityProducts

**Relationships**:
- N:1 with AccountBase/ContactBase via CustomerId polymorphic
- N:1 with PriceLevelBase
- N:1 with TransactionCurrencyBase
- N:1 with LeadBase (OriginatingLeadId)
- 1:N with OpportunityProductBase
- 1:N with QuoteBase
- 1:N with SalesOrderBase
- 1:N with InvoiceBase
- 1:N with CustomerOpportunityRoleBase
- 1:N with ConnectionBase

**State Management**:
| StateCode | State | StatusCode Options |
|-----------|-------|-------------------|
| 0 | Open | 1=New, 2=In Progress, 3=On Hold |
| 1 | Won | 4=Won |
| 2 | Lost | 5=Lost, 6=Cancelled, 7=Out-Sold |

**Sales Stage Progression** (Default):
1. Qualify → 2. Develop → 3. Propose → 4. Close

---

### 2.3 Quote Entity (QuoteBase)

**Purpose**: Generate formal sales quotations for customers.

**Business Role**: Quote represents a formal price proposal sent to customers, typically created from an Opportunity and can convert to a Sales Order.

**Table Structure**:
```
QuoteBase (
    QuoteId (uniqueidentifier) - PK
    OwningBusinessUnit (uniqueidentifier)
    OwnerId (uniqueidentifier)
    StateCode (int)
    StatusCode (int)
    Name (nvarchar)
    QuoteNumber (nvarchar) - System-generated sequential number
    RevisionNumber (int) - Quote version
    
    // Customer
    CustomerId (uniqueidentifier) - Polymorphic
    CustomerIdType (int)
    CustomerIdName (nvarchar)
    CustomerIdYomiName (nvarchar)
    
    // Pricing
    PriceLevelId (uniqueidentifier)
    TransactionCurrencyId (uniqueidentifier)
    ExchangeRate (decimal)
    DiscountAmount (money)
    DiscountPercentage (money)
    FreightAmount (money)
    TotalAmount (money)
    TotalLineItemAmount (money)
    TotalTax (money)
    TotalDiscountAmount (money)
    TotalAmountLessFreight (money)
    
    // Date Control
    EffectiveFrom (datetime)
    EffectiveTo (datetime)
    ExpiresOn (datetime)
    RequestDeliveryBy (datetime)
    ClosedOn (datetime)
    
    // Shipping
    ShippingMethodCode (int)
    ShipTo_Name (nvarchar)
    ShipTo_Address (nvarchar)
    ShipTo_City (nvarchar)
    ShipTo_StateOrProvince (nvarchar)
    ShipTo_PostalCode (nvarchar)
    ShipTo_Country (nvarchar)
    WillCall (bit)
    
    // Billing (mirrors shipping)
    BillTo_Name (nvarchar)
    BillTo_Address (nvarchar)
    ... (similar billing fields)
    
    // Opportunity Link
    OpportunityId (uniqueidentifier)
    
    // Process
    ProcessId (uniqueidentifier)
    StageId (uniqueidentifier)
    TraversedPath (nvarchar)
)
```

**Key Attributes**:
- **QuoteNumber**: Auto-incremented system field (AK1_SalesOrderBase unique constraint pattern)
- **RevisionNumber**: Tracks quote iterations
- **EffectiveFrom/EffectiveTo**: Quote validity period
- **CustomerId**: Supports both Account and Contact

**Relationships**:
- N:1 with OpportunityBase (source opportunity)
- N:1 with PriceLevelBase
- N:1 with TransactionCurrencyBase
- 1:N with QuoteDetailBase (line items)
- N:N via ContactQuotes junction table

**State Management**:
| StateCode | State | StatusCode Options |
|-----------|-------|-------------------|
| 0 | Open | 1=In Progress, 2=In Progress |
| 1 | Won | 3=Won, 4=Accepted, 5=Revised |
| 2 | Lost | 6=Lost, 7=Declined, 8=Cancelled |

---

### 2.4 Sales Order Entity (SalesOrderBase)

**Purpose**: Record accepted customer orders.

**Business Role**: Sales Order represents a confirmed customer purchase. Created from Quote (upon acceptance) or directly from Opportunity.

**Table Structure**:
```
SalesOrderBase (
    SalesOrderId (uniqueidentifier) - PK
    OrderNumber (nvarchar) - System-generated
    OwningBusinessUnit (uniqueidentifier)
    OwnerId (uniqueidentifier)
    StateCode (int)
    StatusCode (int)
    
    // Customer
    CustomerId (uniqueidentifier)
    CustomerIdType (int)
    
    // Opportunity/Quote Source
    OpportunityId (uniqueidentifier)
    QuoteId (uniqueidentifier)
    
    // Pricing (mirrors Quote)
    PriceLevelId (uniqueidentifier)
    TransactionCurrencyId (uniqueidentifier)
    ExchangeRate (decimal)
    DiscountAmount (money)
    FreightAmount (money)
    TotalAmount (money)
    TotalTax (money)
    ... (pricing fields)
    
    // Fulfillment
    RequestDeliveryBy (datetime)
    ShippingMethodCode (int)
    ShipTo_* fields (name, address, city, etc.)
    BillTo_* fields
    
    // Backend Integration
    SubmitStatus (int)
    SubmitStatusDescription (nvarchar)
    SubmitDate (datetime)
    LastBackofficeSubmit (datetime)
    
    // Process
    ProcessId (uniqueidentifier)
    StageId (uniqueidentifier)
)
```

**Key Attributes**:
- **OrderNumber**: Sequential order identifier
- **SubmitStatus**: ERP integration status (0=None, 1=Submitted, 2=Accepted, 3=Rejected)
- **LastBackofficeSubmit**: Tracks ERP synchronization

**State Management**:
| StateCode | State | StatusCode Options |
|-----------|-------|-------------------|
| 0 | Active | 1=New, 2=Pending, 3=Submitted, 4=Cancelled |
| 1 | Fulfilled | 5=Fulfilled |
| 2 | Invoiced | 6=Invoiced |
| 3 | Paid | 7=Paid |
| 4 | Cancelled | 8=Cancelled |

---

### 2.5 Invoice Entity (InvoiceBase)

**Purpose**: Record customer invoices for payment.

**Business Role**: Invoice represents a billing document requesting payment. Created from Sales Order upon fulfillment.

**Table Structure**:
```
InvoiceBase (
    InvoiceId (uniqueidentifier) - PK
    InvoiceNumber (nvarchar)
    OwningBusinessUnit (uniqueidentifier)
    OwnerId (uniqueidentifier)
    StateCode (int)
    StatusCode (int)
    
    // Customer
    CustomerId (uniqueidentifier)
    CustomerIdType (int)
    
    // Source Order
    SalesOrderId (uniqueidentifier)
    OpportunityId (uniqueidentifier)
    
    // Pricing
    PriceLevelId (uniqueidentifier)
    TransactionCurrencyId (uniqueidentifier)
    ExchangeRate (decimal)
    TotalAmount (money)
    TotalTax (money)
    ... (pricing fields)
    
    // Payment Terms
    PaymentTermsCode (int)
    DueDate (datetime)
    
    // Date Tracking
    DateSent (datetime)
    DueDate (datetime)
    PaidOn (datetime)
    
    // Process
    ProcessId (uniqueidentifier)
    StageId (uniqueidentifier)
)
```

**State Management**:
| StateCode | State | StatusCode Options |
|-----------|-------|-------------------|
| 0 | Active | 1=New, 2=Partial, 3=Complete |
| 1 | Paid | 4=Paid, 5=Partial |
| 2 | Cancelled | 6=Cancelled |

---

### 2.6 Account Entity (AccountBase)

**Purpose**: Represent business organizations.

**Business Role**: Account is the primary entity for managing business customers. Can have multiple Contacts, Opportunities, and Orders.

**Table Structure**:
```
AccountBase (
    AccountId (uniqueidentifier) - PK
    AccountNumber (nvarchar) - Business ID
    Name (nvarchar)
    OwningBusinessUnit (uniqueidentifier)
    OwnerId (uniqueidentifier)
    StateCode (int)
    StatusCode (int)
    
    // Hierarchy
    ParentAccountId (uniqueidentifier) - Self-referential (account hierarchy)
    MasterId (uniqueidentifier) - For duplicate merging
    
    // Classification
    AccountRatingCode (int)
    IndustryCode (int)
    BusinessTypeCode (int)
    
    // Primary Contact
    PrimaryContactId (uniqueidentifier)
    
    // Contact Information
    Telephone1 (nvarchar)
    Telephone2 (nvarchar)
    Telephone3 (nvarchar)
    Fax (nvarchar)
    WebsiteUrl (nvarchar)
    EMailAddress1 (nvarchar)
    EMailAddress2 (nvarchar)
    EMailAddress3 (nvarchar)
    
    // Address (multiple via CustomerAddressBase)
    Address1_* fields (line1-9, city, state, postalcode, country)
    Address2_* fields
    
    // Preferences
    PreferredSystemUserId (uniqueidentifier) - Primary sales rep
    PreferredServiceId (uniqueidentifier)
    PreferredEquipmentId (uniqueresources)
    TerritoryId (uniqueidentifier)
    DefaultPriceLevelId (uniqueidentifier)
    
    // Financial
    Revenue (money)
    NumberOfEmployees (int)
    CreditLimit (money)
    CreditOnHold (bit)
    
    // Source
    OriginatingLeadId (uniqueidentifier)
    
    // Process
    ProcessId (uniqueidentifier)
    StageId (uniqueidentifier)
    TraversedPath (nvarchar)
)
```

**Relationships**:
- Self-referential 1:N via ParentAccountId (organizational hierarchy)
- 1:N with ContactBase (multiple contacts)
- 1:N with OpportunityBase
- 1:N with QuoteBase
- 1:N with SalesOrderBase
- 1:N with InvoiceBase
- 1:N with ContractBase
- 1:N with CustomerAddressBase
- N:N via AccountLeads

---

### 2.7 Contact Entity (ContactBase)

**Purpose**: Represent individual people.

**Business Role**: Contact represents individuals, typically associated with Accounts. Can be decision-makers, influencers, or end-users.

**Table Structure**:
```
ContactBase (
    ContactId (uniqueidentifier) - PK
    FullName (nvarchar)
    FirstName (nvarchar)
    LastName (nvarchar)
    MiddleName (nvarchar)
    Suffix (nvarchar)
    YomiFullName (nvarchar) - Phonetic spelling
    
    OwningBusinessUnit (uniqueidentifier)
    OwnerId (uniqueidentifier)
    StateCode (int)
    StatusCode (int)
    
    // Customer Relationship
    ParentCustomerId (uniqueidentifier) - Polymorphic (Account/Contact)
    ParentCustomerIdType (int)
    
    // Contact Info
    EMailAddress1/2/3 (nvarchar)
    Telephone1/2/3 (nvarchar)
    MobilePhone (nvarchar)
    Pager (nvarchar)
    Fax (nvarchar)
    WebsiteUrl (nvarchar)
    
    // Address
    Address1_* fields
    Address2_* fields
    
    // Personal
    BirthDate (datetime)
    GenderCode (int)
    MaritalStatusCode (int)
    
    // Professional
    JobTitle (nvarchar)
    Department (nvarchar)
    
    // Hierarchy
    MasterId (uniqueidentifier) - For duplicate merge
    
    // Source
    OriginatingLeadId (uniqueidentifier)
    
    // Preferences
    PreferredSystemUserId (uniqueidentifier)
    PreferredServiceId (uniqueidentifier)
    PreferredEquipmentId (uniqueidentifier)
    DefaultPriceLevelId (uniqueidentifier)
    TerritoryId (uniqueidentifier)
)
```

---

### 2.8 Activity Entities (ActivityPointerBase and Derived)

**Purpose**: Track time-based interactions with customers.

**Business Role**: Activities represent communications and meetings. The ActivityPointerBase is the base entity for all activity types.

**Activity Types**:
| Activity Type | Table | OTC | Description |
|---------------|-------|-----|-------------|
| Appointment | AppointmentBase | 4201 | Meeting/calendar entry |
| Email | EmailBase | 4202 | Email message |
| Phone Call | PhoneCallBase | 4210 | Phone conversation |
| Task | TaskBase | 4212 | To-do item |
| Letter | LetterBase | 4207 | Physical letter |
| Fax | FaxBase | 4204 | Fax transmission |
| Campaign Response | CampaignResponseBase | 4402 | Campaign response |
| Campaign Activity | CampaignActivityBase | 4404 | Marketing activity |
| Service Appointment | ServiceAppointmentBase | 4214 | Service booking |
| Social Activity | SocialActivityBase | 4216 | Social post |

**ActivityPointerBase Structure**:
```
ActivityPointerBase (
    ActivityId (uniqueidentifier) - PK
    OwningBusinessUnit (uniqueidentifier)
    OwnerId (uniqueidentifier)
    StateCode (int) - 0=Open, 1=Completed, 2=Cancelled
    StatusCode (int)
    ActivityTypeCode (nvarchar) - Activity type identifier
    IsBilled (bit)
    IsWorkflowCreated (bit)
    PriorityCode (int) - 1=Low, 2=Normal, 3=High
    ServiceId (uniqueidentifier)
    SiteId (uniqueidentifier)
    
    // Timing
    ScheduledStart (datetime)
    ScheduledEnd (datetime)
    ActualStart (datetime)
    ActualEnd (datetime)
    
    // Content
    Subject (nvarchar)
    Description (nvarchar)
    
    // Direction
    DirectionCode (bit) - 0=Inbound, 1=Outbound
    
    // Regarding (polymorphic)
    RegardingObjectId (uniqueidentifier)
    RegardingObjectTypeCode (int)
    
    // Sender/Recipient
    Sender (nvarchar)
    From (nvarchar) - ActivityParty
    To (nvarchar) - ActivityParty
    
    // Threading
    ThreadId (nvarchar)
    ParentActivityId (uniqueidentifier) - Email thread
    OriginalActivityId (uniqueidentifier)
    
    // Exchange
    ExchangeRate (decimal)
    ProcessId (uniqueidentifier)
    StageId (uniqueidentifier)
)
```

**ActivityPartyBase Structure** (Participants):
```
ActivityPartyBase (
    ActivityId (uniqueidentifier) - FK to ActivityPointerBase
    PartyId (uniqueidentifier) - FK to Account/Contact/User
    ParticipationTypeMask (int) - 1=Sender, 2=To, 3=Cc, 4=Bcc, etc.
    AddressUsed (nvarchar)
    AddressUsedEmailColumn (int)
    DoNotSend (bit)
)
```

---

## 3. Field-Level Explanation

### 3.1 Standard Attribute Types

The database implements standard Dataverse attribute types:

| Type | SQL Type | Storage | Usage |
|------|----------|---------|-------|
| Uniqueidentifier | uniqueidentifier | 16 bytes | Primary keys, lookups |
| String | nvarchar(n) | Variable Unicode | Names, descriptions |
| Memo | nvarchar(max) | Large text | Long descriptions |
| Integer | int | 4 bytes | Codes, counts |
| BigInt | bigint | 8 bytes | Large counters |
| Decimal | decimal(p,s) | Variable | Precise numbers |
| Money | money | 8 bytes | Currency (4 decimal) |
| Float | float | 8 bytes | Approximate numbers |
| Bit | bit | 1-2 bytes | Boolean flags |
| Datetime | datetime | 8 bytes | Date/time |
| Timestamp | timestamp | 8 bytes | Row versioning |
| Binary | varbinary(max) | BLOB | Files, images |

### 3.2 Currency Handling

**Multi-Currency Architecture**:

Every transactional entity includes:
1. **TransactionCurrencyId**: Reference to TransactionCurrencyBase
2. **ExchangeRate**: Rate at transaction time
3. **[Field]**: Value in transaction currency
4. **[Field]_Base**: Value in organization's base currency

Example from OpportunityBase:
```sql
EstimatedValue (money)          -- In customer's currency
EstimatedValue_Base (money)     -- In base currency (USD)
```

This ensures:
- Accurate reporting in base currency
- Historical exchange rate preservation
- Audit trail for currency fluctuations

### 3.3 Key System Attributes

All entities include these common attributes:

| Attribute | Type | Purpose |
|-----------|------|---------|
| [Entity]Id | uniqueidentifier | Primary key |
| OwnerId | uniqueidentifier | Security principal |
| OwningBusinessUnit | uniqueidentifier | Business unit |
| StateCode | int | Entity state |
| StatusCode | int | State reason |
| CreatedOn | datetime | Creation timestamp |
| CreatedBy | uniqueidentifier | Creator |
| ModifiedOn | datetime | Last modification |
| ModifiedBy | uniqueidentifier | Last modifier |
| CreatedOnBehalfBy | uniqueidentifier | Delegate creator |
| ModifiedOnBehalfBy | uniqueidentifier | Delegate modifier |
| VersionNumber | timestamp | Concurrency control |
| ImportSequenceNumber | int | Import ordering |
| OverriddenCreatedOn | datetime | Backdated creation |
| UTCConversionTimeZoneCode | int | Timezone |
| TimeZoneRuleVersionNumber | int | DST rules |

---

## 4. Relationship Mapping

### 4.1 Relationship Types

#### 4.1.1 One-to-Many (1:N) Relationships

Standard parent-child relationships enforced via FK:

```sql
-- Example: Account to Contact
ALTER TABLE [dbo].[ContactBase] ADD CONSTRAINT 
    [contact_parent_customer_account] FOREIGN KEY([ParentCustomerId])
    REFERENCES [dbo].[AccountBase] ([AccountId])
```

**Common 1:N Relationships**:
- Account → Contacts, Opportunities, Orders, Invoices
- Contact → Activities, Opportunities
- Opportunity → Quotes, Orders, Invoices, Products
- Quote → QuoteDetails
- SalesOrder → SalesOrderDetails
- Invoice → InvoiceDetails

#### 4.1.2 Many-to-One (N:1) Relationships

Lookup-based relationships from child to parent:

```sql
-- Example: Many Opportunities reference one Account
-- FK defined on OpportunityBase
ALTER TABLE [dbo].[OpportunityBase] ADD CONSTRAINT 
    [opportunity_customer_account] FOREIGN KEY([CustomerId])
    REFERENCES [dbo].[AccountBase] ([AccountId])
```

#### 4.1.3 Many-to-Many (N:N) Relationships

Junction tables with composite PKs:

**Example: AccountLeads**:
```sql
CREATE TABLE [dbo].[AccountLeads] (
    AccountId uniqueidentifier NOT NULL,
    LeadId uniqueidentifier NOT NULL,
    CONSTRAINT [account_leads] PRIMARY KEY (AccountId, LeadId)
)
```

**Common N:N Relationships**:
- AccountLeads: Accounts and Leads
- ContactLeads: Contacts and Leads
- ContactQuotes: Contacts and Quotes
- ContactOrders: Contacts and Orders
- ContactInvoices: Contacts and Invoices
- LeadCompetitors: Leads and Competitors
- LeadProduct: Leads and Products

#### 4.1.4 Polymorphic Relationships

Some relationships use polymorphic lookups (can reference multiple entity types):

**Example: CustomerId in OpportunityBase**:
```sql
CustomerId (uniqueidentifier)      -- Can be Account or Contact
CustomerIdType (int)              -- 1=Account, 2=Contact
```

This requires:
- CASE statements in views to resolve
- Separate indexes on each potential FK column
- Application logic to enforce referential integrity

---

### 4.2 Cascade Configuration

Relationships support cascade behaviors:

| Cascade Type | Behavior |
|--------------|----------|
| Cascade | Delete/Assign/Share/Reparent all children |
| Restricted | Prevent operation if children exist |
| RemoveLink | Only remove relationship |
| None | No cascade behavior |

**Example from Account relationships**:
```sql
-- When Account deleted, child Accounts (ParentAccountId) must be reassigned
-- Enforced via restricted relationship
```

---

## 5. Primary Keys & Foreign Keys

### 5.1 Primary Key Strategy

**GUID-Based Primary Keys**:
All entities use uniqueidentifier as primary key:
```sql
[LeadId] [uniqueidentifier] NOT NULL
CONSTRAINT [PK_LeadBase] PRIMARY KEY CLUSTERED (LeadId)
```

**Advantages**:
- Globally unique across distributed systems
- No identity/sequence management overhead
- Supports offline/online sync scenarios

**Default Values**:
```sql
-- NEWSEQUENTIALID() for auto-generation
CONSTRAINT [DF_LeadBase_LeadId] DEFAULT (newid()) FOR [LeadId]
```

### 5.2 Foreign Key Implementation

**Standard Pattern**:
```sql
ALTER TABLE [dbo].[OpportunityBase] ADD CONSTRAINT 
    [business_unit_opportunities] FOREIGN KEY([OwningBusinessUnit])
    REFERENCES [dbo].[BusinessUnitBase] ([BusinessUnitId])
```

**Foreign Key Constraints**:
- With CHECK for data integrity
- Named convention: `[source]_[destination]`
- Support cascading deletes where appropriate

### 5.3 Unique Constraints

**AK (Alternate Key) Constraints**:
```sql
-- Quote Number uniqueness
CONSTRAINT [AK1_QuoteBase] UNIQUE NONCLUSTERED (QuoteNumber)
```

**Composite Unique Keys**:
```sql
-- N:N junction tables
CONSTRAINT [PK_AccountLeads] PRIMARY KEY (AccountId, LeadId)
```

---

## 6. Indexing Strategy

### 6.1 Index Types

#### 6.1.1 Clustered Primary Index
```sql
-- On primary key (GUID)
CONSTRAINT [PK_LeadBase] PRIMARY KEY CLUSTERED (LeadId)
```

#### 6.1.2 Nonclustered Indexes

**Name Indexes**:
```sql
CREATE NONCLUSTERED INDEX [fndx_Account_Name] 
ON [dbo].[AccountBase] (Name)
```

**Quick Find Indexes (fndx_)**:
```sql
CREATE NONCLUSTERED INDEX [fndx_Account_AccountNumber] 
ON [dbo].[AccountBase] (AccountNumber)
```

**Cascade Relationship Indexes**:
```sql
CREATE NONCLUSTERED INDEX [fndx_for_cascaderelationship_account_parent_account] 
ON [dbo].[AccountBase] (ParentAccountId)
```

**Security Indexes (ndx_Security)**:
```sql
CREATE NONCLUSTERED INDEX [ndx_Security] 
ON [dbo].[AccountBase] (OwningBusinessUnit, OwnerId)
```

**Sync Version Indexes (fndx_Sync_VersionNumber)**:
```sql
CREATE UNIQUE NONCLUSTERED INDEX [fndx_Sync_VersionNumber] 
ON [dbo].[AccountBase] (VersionNumber)
```

### 6.2 Index Naming Conventions

| Prefix | Purpose |
|--------|---------|
| `fndx_` | Quick Find/Search indexes |
| `ndx_` | Standard indexes |
| `cndx_` | Covering/Composite indexes |
| `fndx_for_cascaderelationship_` | Cascade relationship support |
| `fndx_Sync_` | Offline sync optimization |

### 6.3 Index Optimization Principles

1. **FILLFACTOR = 80**: Pages leave 20% free for updates
2. **ALLOW_ROW_LOCKS/ALLOW_PAGE_LOCKS = ON**: Minimal locking
3. **PAD_INDEX = OFF**: Default padding
4. **Composite indexes support common queries**: StateCode + OwnerId
5. **Include columns for covering**: Include denormalized Name columns

---

## 7. StateCode & StatusCode Design Logic

### 7.1 State Machine Architecture

Dynamics 365 implements a two-level state model:

#### 7.1.1 StateCode (High-Level State)
- **Type**: Integer, NOT NULL
- **Purpose**: Primary entity lifecycle state
- **Scope**: Entity-specific values (typically 0-3)

#### 7.1.2 StatusCode (Detailed Status)
- **Type**: Integer, nullable
- **Purpose**: Reason for current state
- **Scope**: State-specific options

### 7.2 Lead State Transitions

| StateCode | StateName | StatusCode | StatusName | Valid Transitions |
|-----------|-----------|------------|------------|------------------|
| 0 | Open | 1 | New | → Qualified |
| 0 | Open | 2 | Attempted to Contact | → Qualified, Disqualified |
| 0 | Open | 3 | Contacted | → Qualified, Disqualified |
| 0 | Open | 4 | Open | → Qualified, Disqualified |
| 1 | Qualified | 5 | Qualified | → Won (Opportunity) |
| 1 | Qualified | 6 | Disqualified | → Disqualified |
| 2 | Disqualified | 7 | Lost | Terminal |
| 2 | Disqualified | 8 | Cannot Contact | Terminal |
| 2 | Disqualified | 9 | Not Interested | Terminal |

### 7.3 Opportunity State Transitions

| StateCode | StateName | StatusCode | StatusName |
|-----------|-----------|------------|------------|
| 0 | Open | 1 | New |
| 0 | Open | 2 | In Progress |
| 0 | Open | 3 | On Hold |
| 1 | Won | 4 | Won |
| 2 | Lost | 5 | Lost |
| 2 | Lost | 6 | Cancelled |
| 2 | Lost | 7 | Out-Sold |

### 7.4 Order/Invoice State Flow

**SalesOrderBase States**:
- 0 (Active) → 1 (Fulfilled) → 2 (Invoiced) → 3 (Paid)
- Any → 4 (Cancelled)

### 7.5 Implementation Pattern

```sql
-- StateCode constraint
[StateCode] [int] NOT NULL DEFAULT 0

-- StatusCode constraint  
[StatusCode] [int] NULL

-- Status maps in StringMapBase
-- StateMap in StatusMapBase
```

---

## 8. Business Process Flow (Lead → Close)

### 8.1 Lead-to-Opportunity Process

The standard Dynamics 365 sales process follows:

```
[Lead] → [Qualify] → [Opportunity] → [Develop] → [Propose] → [Close]
```

**Stage Details**:

| Process | Stage | Fields | Criteria |
|---------|-------|--------|----------|
| Lead | Qualify | ScheduleFollowup_Qualify | Contact info complete |
| Opportunity | Qualify | Need, Budget, Timeline | Decision maker identified |
| Opportunity | Develop | CustomerNeed, CurrentSituation | Requirements documented |
| Opportunity | Propose | ProposedSolution, QuoteComments | Pricing finalized |
| Opportunity | Close | ActualCloseDate | Contract signed |

### 8.2 BusinessProcessFlowInstanceBase

**Purpose**: Track active process instances per record.

**Structure**:
```sql
BusinessProcessFlowInstanceBase (
    BusinessProcessFlowInstanceId (uniqueidentifier)
    ProcessId (uniqueidentifier) - FK to workflow
    ProcessStageId (uniqueidentifier) - FK to ProcessStageBase
    
    // Entity References (supports multi-entity BPF)
    Entity1Id (uniqueidentifier) - Primary entity
    Entity1ObjectTypeCode (int)
    Entity2Id (uniqueidentifier)
    Entity2ObjectTypeCode (int)
    ...
    
    TraversedPath (nvarchar) - Stage history
    VersionNumber (timestamp)
)
```

**Indexes**:
- Unique on Entity1 + ProcessId for single-process enforcement
- Additional indexes for Entity2-5 for multi-entity scenarios

### 8.3 Stage Progression

**Lead Stages**:
1. LeadCreated → 2. Qualify → 3. Develop → 4. Propose → 5. Close

**Tracking via TraversedPath**:
```sql
TraversedPath (nvarchar) -- Format: "{StageId1},{StageId2},..."
```

---

## 9. Security Model

### 9.1 Ownership Model

#### 9.1.1 User Ownership
Records assigned to individual SystemUsers:
```sql
OwnerId → SystemUserBase (User)
OwnerIdType = 0
```

#### 9.1.2 Team Ownership
Records assigned to teams:
```sql
OwnerId → TeamBase (Team)
OwnerIdType = 1
```

#### 9.1.3 Organization Ownership
Organization-level records:
```sql
OwnerId → OrganizationBase
OwnerIdType = 2
```

### 9.2 OwnerBase Polymorphic Table

**Purpose**: Single join point for all ownership types.

```sql
OwnerBase (
    OwnerId (uniqueidentifier) - PK (composite from User/Team/Org)
    OwnerIdType (int) - 0=User, 1=Team, 2=Organization
    Name (nvarchar) - Owner name
)
```

### 9.3 Business Unit Hierarchy

```sql
BusinessUnitBase (
    BusinessUnitId (uniqueidentifier) - PK
    Name (nvarchar)
    ParentBusinessUnitId (uniqueidentifier) - Self-ref FK
    DivisionName (nvarchar)
    OrganizationId (uniqueidentifier)
    CalendarId (uniqueidentifier)
    TransactionCurrencyId (uniqueidentifier)
)
```

**Security Propagation**: Child Business Units inherit parent permissions.

### 9.4 Role-Based Security

```sql
RoleBase (
    RoleId (uniqueidentifier)
    Name (nvarchar)
    BusinessUnitId (uniqueidentifier)
    RoleTemplateId (uniqueidentifier)
)

RolePrivilegesBase (
    RoleId (uniqueidentifier)
    PrivilegeId (uniqueidentifier)
    PrivilegeDepthMask (int) -- 1=User, 2=BusinessUnit, 4=Parent/Child, 8=Organization
)
```

### 9.5 Field-Level Security

```sql
FieldPermissionBase (
    FieldPermissionId (uniqueidentifier)
    FieldSecurityProfileId (uniqueidentifier)
    EntityName (nvarchar)
    AttributeName (nvarchar)
    CanRead (bit)
    CanUpdate (bit)
)
```

### 9.6 Filtered Views (Row-Level Security)

**Example Pattern**:
```sql
CREATE VIEW [dbo].[FilteredAccount] AS
SELECT * FROM Account
WHERE 
    -- User is record owner
    Account.OwnerId IN (SELECT OwnerId FROM fn_GetOwnerIdsForFilteredView(@UserId))
    OR
    -- User's business unit
    Account.OwningBusinessUnit IN (
        SELECT BusinessUnitId FROM SystemUserBusinessUnitEntityMap 
        WHERE SystemUserId = @UserId
    )
    OR
    -- Shared to user
    Account.AccountId IN (
        SELECT ObjectId FROM fn_GetSharedRecordIdsForFilteredView(@UserId)
    )
```

---

## 10. Audit & Ownership Architecture

### 10.1 Audit Infrastructure

**AuditBase Table**:
```sql
AuditBase (
    AuditId (uniqueidentifier)
    Action (int) -- 0=Create, 1=Update, 2=Delete
    EntityId (uniqueidentifier) -- ObjectTypeCode
    ObjectId (uniqueidentifier) -- Record ID
    ObjectIdName (nvarchar) -- Record name
    AttributeMask (nvarchar) -- Changed fields
    CreatedOn (datetime) -- Audit timestamp
    UserId (uniqueidentifier) -- Who made change
    Operation (nvarchar) -- Operation details
)
```

**Audit Indexes**:
```sql
CREATE NONCLUSTERED INDEX [ndx_ObjectId] ON AuditBase (ObjectId)
CREATE NONCLUSTERED INDEX [ndx_UserId] ON AuditBase (UserId)
CREATE NONCLUSTERED INDEX [fndx_ObjectTypeCode] ON AuditBase (EntityId)
CREATE UNIQUE NONCLUSTERED INDEX [ndx_PrimaryKey_Audit_Primary] 
    ON AuditBase (AuditId, ObjectId, CreatedOn)
```

### 10.2 Ownership Tracking

Every record includes:
- **CreatedBy/CreatedOn**: Original creator
- **ModifiedBy/ModifiedOn**: Last modifier
- **CreatedOnBehalfBy/ModifiedOnBehalfBy**: Delegate actions

### 10.3 Version Control

**VersionNumber (timestamp)**:
- Auto-incremented on each update
- Used for optimistic concurrency
- Enables change detection

```sql
-- Concurrency check example
WHERE VersionNumber = @OriginalVersionNumber
-- If no rows updated, another user modified the record
```

---

## 11. Transaction Flow Between Modules

### 11.1 Lead → Opportunity → Quote → Order → Invoice Flow

```
Lead (Qualified)
    ↓ [Qualify Action]
Opportunity (Open)
    ↓ [Create Quote]
Quote (Open)
    ↓ [Accept Quote]
Sales Order (Active)
    ↓ [Fulfill]
Sales Order (Fulfilled)
    ↓ [Invoice]
Invoice (Active)
    ↓ [Payment]
Invoice (Paid)
```

### 11.2 Financial Cascade

**Quote to Order**:
```sql
-- Quote fields copied to Order
Quote.CustomerId → Order.CustomerId
Quote.PriceLevelId → Order.PriceLevelId
Quote.TotalAmount → Order.TotalAmount
-- QuoteDetails copied to OrderDetails
```

**Order to Invoice**:
```sql
-- Order fields copied to Invoice
Order.SalesOrderId → Invoice.SalesOrderId
Order.TotalAmount → Invoice.TotalAmount
-- OrderDetails copied to InvoiceDetails
```

### 11.3 Revenue Recognition

**Opportunity Close**:
- StateCode changes to 1 (Won)
- ActualValue calculated
- ActualCloseDate set
- Pipeline metrics updated

---

## 12. Data Validation Rules

### 12.1 Required Field Validation

Enforced at database level via NOT NULL constraints:
```sql
[LeadId] [uniqueidentifier] NOT NULL
[StateCode] [int] NOT NULL
[Name] [nvarchar](160) NOT NULL
```

### 12.2 Business Rules (Implemented in Application)

| Rule | Implementation | Example |
|------|----------------|---------|
| Required Fields | NOT NULL on critical fields | Lead.Subject |
| Picklist Values | Foreign key to StringMapBase | Account.IndustryCode |
| Email Format | Regex validation | Contact.EMailAddress1 |
| Phone Format | Regex validation | Contact.Telephone1 |
| Date Validation | Range checks | Opportunity.EstimatedCloseDate |
| Currency Link | FK to TransactionCurrencyBase | All money fields |

### 12.3 Duplicate Detection

**DuplicateRuleBase**:
```sql
DuplicateRuleBase (
    DuplicateRuleId (uniqueidentifier)
    Name (nvarchar)
    BaseEntityTypeCode (int)
    MatchingEntityTypeCode (int)
    BaseAttributeId (uniqueidentifier)
    MatchingAttributeId (uniqueidentifier)
)
```

**DuplicateRecordBase**:
```sql
DuplicateRecordBase (
    DuplicateRecordId (uniqueidentifier)
    DuplicateRuleId (uniqueidentifier)
    RecordId1 (uniqueidentifier)
    RecordId2 (uniqueidentifier)
    CreatedOn (datetime)
)
```

---

## 13. Financial Calculation Logic

### 13.1 Pricing Engine

**PriceLevelBase** (Price List):
```sql
PriceLevelBase (
    PriceLevelId (uniqueidentifier)
    Name (nvarchar)
    BeginDate (datetime)
    EndDate (datetime)
    TransactionCurrencyId (uniqueidentifier)
    IsDefault (bit)
)
```

**ProductPriceLevelBase** (Price List Items):
```sql
ProductPriceLevelBase (
    ProductPriceLevelId (uniqueidentifier)
    PriceLevelId (uniqueidentifier) -- FK
    ProductId (uniqueidentifier) -- FK
    Amount (money) -- List price
    Percentage (decimal) -- Discount %
    RoundingPolicyCode (int)
    FloorPrice (money)
    CeilingPrice (money)
)
```

### 13.2 Discount Application

**DiscountTypeBase**:
```sql
DiscountTypeBase (
    DiscountTypeId (uniqueidentifier)
    Name (nvarchar)
    TransactionCurrencyId (uniqueidentifier)
    Type (int) -- 0=Percentage, 1=Amount
)
```

**DiscountBase**:
```sql
DiscountBase (
    DiscountId (uniqueidentifier)
    DiscountTypeId (uniqueidentifier)
    LowQuantity (int)
    HighQuantity (int)
    Percentage (decimal)
    Amount (money)
)
```

### 13.3 Calculation Formula

**Quote Total Calculation**:
```
TotalLineItemAmount = Σ(Quantity × UnitPrice)
TotalLineItemDiscountAmount = Σ(Quantity × UnitPrice × LineDiscount%)
DiscountAmount = TotalLineItemAmount - TotalLineItemDiscountAmount
FreightAmount = Manual or Calculated
TotalTax = Σ(LineTax)
TotalAmount = TotalLineItemAmount - DiscountAmount + FreightAmount + TotalTax
TotalAmountLessFreight = TotalLineItemAmount - DiscountAmount + TotalTax

-- Base Currency equivalents
TotalAmount_Base = TotalAmount × ExchangeRate
```

---

## 14. Concurrency Handling

### 14.1 Optimistic Concurrency

**Implementation via VersionNumber**:
```sql
-- timestamp column automatically updated
[VersionNumber] [timestamp] NULL

-- In UPDATE operations
UPDATE AccountBase 
SET Name = @Name, 
    ModifiedBy = @UserId, 
    ModifiedOn = GETUTCDATE()
WHERE AccountId = @AccountId 
  AND VersionNumber = @OriginalVersionNumber

-- Check rows affected
IF @@ROWCOUNT = 0
    -- Concurrency conflict - another user modified record
```

### 14.2 Pessimistic Locking Options

**NOLOCK hints** for read operations (in views):
```sql
SELECT * FROM AccountBase WITH (NOLOCK)
-- Used in logical views to prevent blocking
```

### 14.3 Sync Version Number

**fndx_Sync_VersionNumber**:
- Unique index on VersionNumber
- Supports offline/online sync
- Prevents duplicate sync conflicts

---

## 15. Soft Delete Strategy

### 15.1 Delete State Implementation

**Standard Soft Delete**:
- Records not physically deleted
- StatusCode set to indicate deletion
- StateCode set to indicate inactive

**Alternative: IsDeleted attribute**:
```sql
-- Some entities include
IsDeleted (bit) DEFAULT 0
```

### 15.2 Cascading Soft Delete

**Delete Subcomponents**:
```sql
-- When Opportunity deleted
-- Delete OpportunityProducts (Cascade)
-- Delete Quotes (RemoveLink or Cascade)
-- Keep relationship history for reporting
```

### 15.3 Recycle Bin Pattern

**Audit Table for Deleted Records**:
```sql
-- Soft-deleted records can be recovered
-- From AuditBase: Action = 2 (Delete)
-- Re-create record from audit history
```

---

## 16. Activity & Timeline Model

### 16.1 Activity Party Architecture

**ActivityPartyBase** stores all participants:
```sql
ParticipationTypeMask Values:
1 = Sender
2 = To Recipient  
3 = CC Recipient
4 = BCC Recipient
5 = Required Attendee
6 = Optional Attendee
7 = Organizer
8 = Regarding
9 = Owner (of activity)
10 = Resource
```

### 16.2 Timeline Integration

**Regarding Relationship**:
```sql
-- Activities linked to any entity
RegardingObjectId (uniqueidentifier)  -- Record ID
RegardingObjectTypeCode (int)         -- Entity type
```

**Query Pattern**:
```sql
-- Get all activities for an account
SELECT * FROM ActivityPointerBase
WHERE RegardingObjectId = @AccountId
  AND RegardingObjectTypeCode = 1
```

### 16.3 Activity State Management

| StateCode | Meaning | StatusCode Options |
|-----------|---------|-------------------|
| 0 | Open | Scheduled, In Progress |
| 1 | Completed | Completed, Sent |
| 2 | Cancelled | Cancelled |

---

## 17. Metadata Architecture

### 17.1 Entity Metadata

**EntityMetadata in StringMapBase**:
```sql
StringMapBase (
    AttributeValue (int)
    AttributeName (nvarchar)
    ObjectTypeCode (int) -- Entity ID
    Value (nvarchar) -- Display value
    LangId (int) -- Language
)
```

### 17.2 Option Set Metadata

**Picklist Values Stored**:
```sql
-- Example: IndustryCode values
ObjectTypeCode = 1 (Account)
AttributeName = 'IndustryCode'
AttributeValue = 1 -> 'Agriculture'
AttributeValue = 2 -> 'Apparel'
...
```

### 17.3 Relationship Metadata

**EntityMapBase**:
```sql
EntityMapBase (
    EntityMapId (uniqueidentifier)
    SourceEntityName (nvarchar)
    TargetEntityName (nvarchar)
)
```

**AttributeMapBase**:
```sql
AttributeMapBase (
    AttributeMapId (uniqueidentifier)
    EntityMapId (uniqueidentifier)
    SourceAttributeName (nvarchar)
    TargetAttributeName (nvarchar)
)
```

---

## 18. Data Integrity Enforcement

### 18.1 Constraint Categories

#### 18.1.1 Primary Key Constraints
```sql
CONSTRAINT [PK_LeadBase] PRIMARY KEY CLUSTERED (LeadId)
```

#### 18.1.2 Foreign Key Constraints
```sql
ALTER TABLE [dbo].[OpportunityBase] ADD CONSTRAINT 
    [business_unit_opportunities] FOREIGN KEY([OwningBusinessUnit])
    REFERENCES [dbo].[BusinessUnitBase] ([BusinessUnitId])
```

#### 18.1.3 Unique Constraints
```sql
CONSTRAINT [AK1_QuoteBase] UNIQUE NONCLUSTERED (QuoteNumber)
```

#### 18.1.4 Default Constraints
```sql
CONSTRAINT [DF_LeadBase_StateCode] DEFAULT (0) FOR [StateCode]
```

#### 18.1.5 Check Constraints
```sql
-- Probability must be 0-100
-- Validated at application layer
```

### 18.2 Referential Integrity

**Parent-Child Integrity**:
- Cascade delete where appropriate
- Restricted delete for critical relationships
- Set Null/Set Default for non-critical FKs

### 18.3 Data Quality Rules

| Rule | Implementation | Example |
|------|----------------|---------|
| Unique Values | Unique constraints | QuoteNumber |
| Range Validation | Check constraints | Probability 0-100 |
| Format Validation | Application regex | Email addresses |
| Cross-field | Plugin/business rule | End date > Start date |

---

## 19. Performance Optimization Design

### 19.1 Index Strategy

#### Covering Indexes
```sql
CREATE NONCLUSTERED INDEX [ndx_Cover] 
ON [dbo].[AccountBase] (StateCode, OwnerId)
INCLUDE (Name, AccountId)
```

#### Quick Find Indexes
```sql
-- Supports global search
CREATE NONCLUSTERED INDEX [fndx_Account_Name] 
ON [dbo].[AccountBase] (Name)
```

### 19.2 Query Optimization

#### 19.2.1 Optimized View Joins
```sql
-- LEFT JOINs with NOLOCK hints
LEFT JOIN [AccountBase] [account_opportunities] 
    WITH (NOLOCK) ON (...)
```

#### 19.2.2 Filtered Views
```sql
-- Security filtering at database level
-- Reduces data transfer for thin clients
```

### 19.3 Partitioning Considerations

**Future Optimization**:
- Transaction tables could be partitioned by date
- Archive strategy for historical data

### 19.4 Denormalization Strategy

**Performance Denormalization**:
- Name columns stored in FK tables
- Eliminates JOIN for display
- Updated via triggers/application

```sql
-- Example: CustomerIdName in OpportunityBase
-- Populated when CustomerId set
-- Avoids join to AccountBase for display
```

---

## 20. Comparison with Standard Dynamics 365 Sales Architecture

### 20.1 Standard Architecture Compliance

| Aspect | Standard D365 | This Database | Status |
|--------|--------------|---------------|--------|
| Entity Model | Same entities | Sales entities implemented | ✓ Compliant |
| Table Naming | [Entity]Base | [Entity]Base | ✓ Compliant |
| View Layer | Base/Filtered views | Implemented | ✓ Compliant |
| Primary Keys | GUID | GUID | ✓ Compliant |
| Ownership | OwnerBase | OwnerBase | ✓ Compliant |
| Currency | Multi-currency | Multi-currency | ✓ Compliant |
| Activities | ActivityPointer | ActivityPointer | ✓ Compliant |
| BPF | ProcessInstance | BusinessProcessFlowInstance | ✓ Compliant |

### 20.2 Custom Extensions

| Custom Entity | Purpose | Relationship |
|---------------|---------|--------------|
| canven_city | City master | Referenced by canven_onboarding |
| canven_onboarding | Customer onboarding | N:1 with canven_city, owned by User |
| canven_propertyonboard | Property onboarding | Similar to canven_onboarding |
| new_usertype | Extended classification | Standard ownership |
| new_userdetails | User profile extension | Standard ownership |
| new_customercomplaint | Complaint management | N:1 with Account |

### 20.3 Industry-Specific Extensions

The canven_* entities suggest a **real estate/property** industry focus:
- Property onboarding workflow
- City-based routing
- Approval workflows (ReviewerOwner, FinalApprovalOwner)

### 20.4 Differences from Default

| Feature | Default | This Implementation |
|---------|---------|---------------------|
| Field Security | Optional | Implemented in filtered views |
| Duplicate Detection | Enabled | Infrastructure present |
| Categories | Standard | Extended via Subject |
| Territory | Standard | Full implementation |
| Teams | Optional | Infrastructure present |

---

## Appendix A: Relationship Reference

### A.1 Account Relationships
```
AccountBase
├── 1:N → ContactBase (ParentCustomerId)
├── 1:N → OpportunityBase (CustomerId)
├── 1:N → QuoteBase (CustomerId)
├── 1:N → SalesOrderBase (CustomerId)
├── 1:N → InvoiceBase (CustomerId)
├── 1:N → ContractBase (CustomerId)
├── 1:N → CustomerAddressBase (ParentId)
├── 1:N → AccountBase (ParentAccountId - self)
├── N:N → LeadBase (AccountLeads)
├── N:1 → BusinessUnitBase (OwningBusinessUnit)
├── N:1 → OwnerBase (OwnerId)
├── N:1 → LeadBase (OriginatingLeadId)
├── N:1 → ContactBase (PrimaryContactId)
└── N:1 → TransactionCurrencyBase
```

### A.2 Opportunity Relationships
```
OpportunityBase
├── 1:N → QuoteBase (OpportunityId)
├── 1:N → SalesOrderBase (OpportunityId)
├── 1:N → InvoiceBase (OpportunityId)
├── 1:N → OpportunityProductBase (OpportunityId)
├── N:1 → AccountBase (CustomerId when Type=1)
├── N:1 → ContactBase (CustomerId when Type=2)
├── N:1 → LeadBase (OriginatingLeadId)
├── N:1 → PriceLevelBase (PriceLevelId)
├── N:1 → TransactionCurrencyBase
├── N:1 → BusinessUnitBase (OwningBusinessUnit)
├── N:1 → OwnerBase (OwnerId)
└── N:1 → CampaignBase (CampaignId)
```

---

## Appendix B: State/Status Code Reference

### B.1 Lead
| State | Value | Status | Value |
|-------|-------|--------|-------|
| Open | 0 | New | 1 |
| Open | 0 | Attempted to Contact | 2 |
| Open | 0 | Contacted | 3 |
| Open | 0 | Open | 4 |
| Qualified | 1 | Qualified | 5 |
| Qualified | 1 | Disqualified | 6 |
| Disqualified | 2 | Lost | 7 |
| Disqualified | 2 | Cannot Contact | 8 |
| Disqualified | 2 | Not Interested | 9 |

### B.2 Opportunity
| State | Value | Status | Value |
|-------|-------|--------|-------|
| Open | 0 | New | 1 |
| Open | 0 | In Progress | 2 |
| Open | 0 | On Hold | 3 |
| Won | 1 | Won | 4 |
| Lost | 2 | Lost | 5 |
| Lost | 2 | Cancelled | 6 |
| Lost | 2 | Out-Sold | 7 |

---

## Appendix C: Technical Specifications

### C.1 Database Parameters
- **Collation**: Database default
- **Compatibility Level**: SQL Server 2016+
- **Recovery Model**: Full (implied for production)
- **Auto-Close**: OFF
- **Auto-Shrink**: OFF

### C.2 Table Statistics
- **Total Tables**: ~300+
- **Core Sales Tables**: 15+
- **Activity Tables**: 10+
- **Custom Tables**: 6

### C.3 Index Summary
- **Primary Key Indexes**: Clustered, GUID-based
- **Secondary Indexes**: Non-clustered, various purposes
- **Unique Indexes**: For alternate keys and sync
- **Covering Indexes**: For common query patterns

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Jan 2026 | CRM Architect | Initial release |

---

*This document provides enterprise-level technical documentation for the Dynamics 365 CRM Sales Module database architecture. For implementation-specific questions, refer to the Microsoft Dynamics 365 Implementation Guide.*
