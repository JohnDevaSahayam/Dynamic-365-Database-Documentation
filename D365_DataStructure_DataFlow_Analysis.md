# Microsoft Dynamics 365 CRM Sales Module
## Data Structure & Data Flow Technical Analysis

**Document Classification**: Enterprise Architecture Technical Documentation  
**Author**: Senior Microsoft Dynamics 365 CRM Architect & Dataverse Database Expert  
**Database**: CRM_DB (SQL Server)  
**Date**: January 2026

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Data Structure Architecture](#2-data-structure-architecture)
3. [Core Entity Deep Dive](#3-core-entity-deep-dive)
4. [Entity Relationship Mapping](#4-entity-relationship-mapping)
5. [Data Flow Analysis](#5-data-flow-analysis)
6. [Sales Pipeline Flow](#6-sales-pipeline-flow)
7. [Financial Transaction Flow](#7-financial-transaction-flow)
8. [Activity & Communication Flow](#8-activity--communication-flow)
9. [Custom Entities Analysis](#9-custom-entities-analysis)
10. [Technical Specifications](#10-technical-specifications)

---

## 1. Executive Summary

This document provides a **deep technical analysis** of the Microsoft Dynamics 365 CRM Sales Module database structure and data flow patterns. The analysis is based on examination of the CRM_DB.sql script containing approximately **300+ tables** implementing a full-featured enterprise CRM system.

### Key Findings

- **Database Pattern**: Standard Dynamics 365 three-layer architecture (Base tables → Logical views → Filtered security views)
- **Primary Entities**: Account, Contact, Lead, Opportunity, Quote, Sales Order, Invoice
- **Relationship Model**: Complex polymorphic relationships with 1:N, N:1, and N:N mappings
- **Sales Process**: Complete Lead-to-Cash pipeline implementation
- **Custom Extensions**: Property/Real estate industry-specific entities

---

## 2. Data Structure Architecture

### 2.1 Three-Layer Table Architecture

The database follows the standard Microsoft Dynamics 365 pattern with three distinct table layers:

#### Layer 1: Base Tables (`[EntityName]Base`)
Physical storage tables containing all entity data:

```
┌─────────────────────────────────────────────────────────────┐
│                    BASE TABLE LAYER                          │
├─────────────────────────────────────────────────────────────┤
│  AccountBase      │ Physical storage                         │
│  ContactBase      │ All attributes                          │
│  LeadBase         │ Foreign keys                            │
│  OpportunityBase  │ State codes                             │
│  QuoteBase        │ Money fields with _Base equivalents    │
│  ...              │ Version numbers for concurrency         │
└─────────────────────────────────────────────────────────────┘
```

#### Layer 2: Logical Views (`[EntityName]`)
Logical views joining base tables with related entities:

```sql
-- Example: Opportunity View Pattern
CREATE VIEW [dbo].[Opportunity] AS
SELECT 
    -- Physical attributes from OpportunityBase
    [OpportunityBase].[OpportunityId],
    [OpportunityBase].[Name],
    ...
    -- Logical attributes (joined from related tables)
    [account_opportunities].[Name] AS CustomerIdName,
    [price_level_opportunties].[Name] AS PriceLevelIdName,
    -- Ownership entries
    OwnerId = [OpportunityBase].OwnerId,
    OwnerName = XXowner.Name
FROM [OpportunityBase]
    LEFT JOIN [AccountBase] [account_opportunities] 
        ON ([OpportunityBase].[CustomerId] = [account_opportunities].[AccountId])
    LEFT JOIN [PriceLevelBase] [price_level_opportunties] 
        ON ([OpportunityBase].[PriceLevelId] = [price_level_opportunties].[PriceLevelId])
    LEFT JOIN OwnerBase XXowner 
        ON ([OpportunityBase].OwnerId = XXowner.OwnerId)
```

#### Layer 3: Filtered Security Views (`Filtered[EntityName]`)
Row-level security enforced views:

```sql
-- Security filtering pattern
CREATE VIEW [dbo].[FilteredOpportunity] AS
SELECT * FROM Opportunity
WHERE 
    -- Owner-based access
    Opportunity.OwnerId IN (
        SELECT OwnerId FROM fn_GetOwnerIdsForFilteredView(u.SystemUserId)
    )
    OR
    -- Business Unit access
    Opportunity.OwningBusinessUnit IN (
        SELECT BusinessUnitId FROM SystemUserBusinessUnitEntityMap 
        WHERE SystemUserId = u.SystemUserId AND ObjectTypeCode = 3
    )
    OR
    -- Shared records
    Opportunity.OpportunityId IN (
        SELECT ObjectId FROM fn_GetSharedRecordIdsForFilteredView(u.SystemUserId, 3)
    )
```

### 2.2 Standard Entity Attributes

Every entity in the database follows a consistent attribute pattern:

| Attribute Category | Attributes | Purpose |
|-------------------|------------|---------|
| **Primary Key** | `[Entity]Id` (uniqueidentifier) | Globally unique identifier |
| **Ownership** | `OwnerId`, `OwningBusinessUnit` | Security control |
| **State Management** | `StateCode`, `StatusCode` | Lifecycle control |
| **Timestamps** | `CreatedOn`, `ModifiedOn` | Audit trail |
| **Users** | `CreatedBy`, `ModifiedBy` | Attribution |
| **Delegation** | `CreatedOnBehalfBy`, `ModifiedOnBehalfBy` | Delegate actions |
| **Concurrency** | `VersionNumber` (timestamp) | Optimistic locking |
| **Import** | `ImportSequenceNumber` | Data import ordering |
| **Timezone** | `UTCConversionTimeZoneCode`, `TimeZoneRuleVersionNumber` | Global support |

### 2.3 Currency Handling Architecture

Multi-currency support implemented across all transactional entities:

```sql
-- Standard pattern for money fields
OpportunityBase (
    EstimatedValue money,              -- Transaction currency
    EstimatedValue_Base money,          -- Base currency (USD)
    ExchangeRate decimal(23,10),       -- At transaction time
    TransactionCurrencyId uniqueidentifier  -- Currency reference
)
```

**Calculation**:
```
EstimatedValue_Base = EstimatedValue × ExchangeRate
```

This ensures:
- Historical exchange rate preservation
- Accurate reporting in organization's base currency
- Audit trail for currency fluctuations

---

## 3. Core Entity Deep Dive

### 3.1 Lead Entity (LeadBase) - Object Type Code: 4

**Purpose**: Entry point for prospective customers in the sales cycle

**Business Function**: Capture and qualify potential customers discovered through marketing campaigns, referrals, or cold outreach

**Table Structure**:

```sql
CREATE TABLE [dbo].[LeadBase] (
    -- Primary Key
    LeadId uniqueidentifier NOT NULL,
    
    -- Ownership
    OwningBusinessUnit uniqueidentifier,
    OwnerId uniqueidentifier,
    OwnerIdType int,
    
    -- State Management
    StateCode int NOT NULL,          -- 0=Open, 1=Qualified, 2=Disqualified
    StatusCode int NULL,
    
    -- Core Business Fields
    Subject nvarchar(200),
    FirstName nvarchar(100),
    LastName nvarchar(100),
    CompanyName nvarchar(200),
    EmailAddress nvarchar(100),
    Telephone nvarchar(50),
    
    -- Qualification Fields
    IndustryCode int,
    LeadSourceCode int,
    RatingCode int,
    
    -- Source Tracking
    CampaignId uniqueidentifier,
    OriginatingLeadId uniqueidentifier,    -- Self-referential
    
    -- Conversion Links
    ParentAccountId uniqueidentifier,
    ParentContactId uniqueidentifier,
    QualifyingOpportunityId uniqueidentifier,  -- Created opportunity
    
    -- Contact Preferences
    DoNotPhone bit,
    DoNotEmail bit,
    DoNotPostalMail bit,
    DoNotFax bit,
    
    -- Process Tracking
    TraversedPath nvarchar(1250),    -- BPF stage history
    ProcessId uniqueidentifier,
    StageId uniqueidentifier,
    
    -- Standard System Attributes
    CreatedOn datetime,
    CreatedBy uniqueidentifier,
    ModifiedOn datetime,
    ModifiedBy uniqueidentifier,
    VersionNumber timestamp
)
```

**State Transitions**:

```
┌─────────────────────────────────────────────────────────────────┐
│                     LEAD STATE MACHINE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    ┌──────┐     ┌──────────┐      ┌───────────┐                │
│    │ OPEN │────▶│QUALIFIED │────▶│  WON      │                │
│    │(0-4) │     │  (5)     │      │(Opportunity)              │
│    └──┬───┘     └────┬─────┘      └───────────┘                │
│       │              │                                           │
│       │              │                                           │
│       ▼              ▼                                           │
│    ┌──────────────┐                                              │
│    │DISQUALIFIED │                                              │
│    │  (7-9)      │                                              │
│    └─────────────┘                                              │
│                                                                  │
│    States:                                                       │
│    - Open: New(1), Attempted to Contact(2), Contacted(3),     │
│            Open(4)                                              │
│    - Qualified: Qualified(5), Disqualified(6)                  │
│    - Disqualified: Lost(7), Cannot Contact(8),                │
│                    Not Interested(9)                            │
└─────────────────────────────────────────────────────────────────┘
```

**Key Relationships**:

| Relationship | Type | Target Entity |
|--------------|------|---------------|
| OwningBusinessUnit | N:1 | BusinessUnitBase |
| OwnerId | N:1 | OwnerBase |
| CampaignId | N:1 | CampaignBase |
| ParentAccountId | N:1 | AccountBase |
| ParentContactId | N:1 | ContactBase |
| QualifyingOpportunityId | N:1 | OpportunityBase |
| AccountLeads | N:N | AccountBase (via junction) |
| ContactLeads | N:N | ContactBase (via junction) |
| LeadCompetitors | N:N | CompetitorBase (via junction) |
| LeadProduct | N:N | ProductBase (via junction) |

---

### 3.2 Opportunity Entity (OpportunityBase) - Object Type Code: 3

**Purpose**: Track qualified sales opportunities through the sales process

**Business Function**: Manage actively pursued sales with revenue potential, from qualification through close

**Table Structure**:

```sql
CREATE TABLE [dbo].[OpportunityBase] (
    -- Primary Key
    OpportunityId uniqueidentifier NOT NULL,
    
    -- Ownership
    OwningBusinessUnit uniqueidentifier,
    OwnerId uniqueidentifier,
    
    -- State Management
    StateCode int NOT NULL,          -- 0=Open, 1=Won, 2=Lost
    StatusCode int NULL,
    
    -- Identity
    Name nvarchar(200),
    
    -- Customer (Polymorphic: Account or Contact)
    CustomerId uniqueidentifier,      -- Can be Account or Contact
    CustomerIdType int,               -- 1=Account, 2=Contact
    CustomerIdName nvarchar(200),     -- Denormalized for display
    CustomerIdYomiName nvarchar(200), -- Phonetic
    
    -- Financial
    EstimatedValue money,
    EstimatedValue_Base money,
    ActualValue money,
    ActualValue_Base money,
    CloseProbability int,             -- 0-100
    
    -- Pricing
    PriceLevelId uniqueidentifier,
    TransactionCurrencyId uniqueidentifier,
    ExchangeRate decimal(23,10),
    
    -- Dates
    EstimatedCloseDate datetime,
    ActualCloseDate datetime,
    
    -- Sales Process
    StepId uniqueidentifier,
    StepName nvarchar(100),
    SalesStageCode int,
    SalesStage nvarchar(100),
    
    -- Source
    CampaignId uniqueidentifier,
    OriginatingLeadId uniqueidentifier,  -- From converted lead
    
    -- Parent References
    ParentAccountId uniqueidentifier,
    ParentContactId uniqueidentifier,
    
    -- Aggregated Financials
    TotalAmount money,
    TotalAmount_Base money,
    DiscountAmount money,
    DiscountPercentage money,
    FreightAmount money,
    TotalTax money,
    TotalLineItemAmount money,
    TotalDiscountAmount money,
    TotalAmountLessFreight money,
    
    -- BPF Tracking
    TraversedPath nvarchar(1250),
    ProcessId uniqueidentifier,
    StageId uniqueidentifier,
    
    -- Date Fields for BPF Stages
    ScheduleFollowup_Prospect datetime,
    ScheduleFollowup_Qualify datetime,
    ScheduleProposalMeeting datetime,
    DevelopProposal datetime,
    CompleteInternalReview datetime,
    PresentProposal datetime,
    
    -- System
    CreatedOn datetime,
    ModifiedOn datetime,
    VersionNumber timestamp
)
```

**State Transitions**:

```
┌─────────────────────────────────────────────────────────────────┐
│                  OPPORTUNITY STATE MACHINE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    ┌─────────┐     ┌─────────┐                                 │
│    │   NEW   │────▶│IN PROGRESS│────▶ ┌─────┐                 │
│    │   (1)   │     │   (2)    │      │ WON │                 │
│    └────┬────┘     └─────┬─────┘      │ (4) │                 │
│         │                │             └─────┘                 │
│         │                │                                        │
│         │                ▼                                        │
│         │            ┌───────┐      ┌──────────┐                │
│         │            │ ON    │────▶│   LOST   │                │
│         └──────────▶│ HOLD  │      │  (5-7)   │                │
│                      │ (3)   │      └──────────┘                │
│                      └───────┘                                   │
│                                                                  │
│    Open States: New(1), In Progress(2), On Hold(3)            │
│    Won: Won(4)                                                │
│    Lost: Lost(5), Cancelled(6), Out-Sold(7)                  │
└─────────────────────────────────────────────────────────────────┘
```

**Financial Calculation Flow**:

```
OpportunityProductBase (Line Items)
         │
         ▼
┌─────────────────────────────────────────────┐
│  SUM(Quantity × UnitPrice)                  │
│  = TotalLineItemAmount                      │
└─────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────┐
│  Apply Line Discounts                        │
│  = TotalLineItemDiscountAmount              │
└─────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────┐
│  TotalLineItemAmount - Discount             │
│  = Subtotal                                 │
└─────────────────────────────────────────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐ ┌─────────┐
│ Freight│ │  Tax    │
│Amount │ │ Calculate│
└───┬───┘ └───┬─────┘
    │         │
    └────┬────┘
         ▼
┌─────────────────────┐
│   TotalAmount       │
│ (EstimatedValue)    │
└─────────────────────┘
```

---

### 3.3 Quote Entity (QuoteBase) - Object Type Code: 1084

**Purpose**: Generate formal sales quotations for customers

**Business Function**: Create detailed price proposals that can be sent to customers and converted to orders

**Key Attributes**:

| Attribute | Type | Purpose |
|-----------|------|---------|
| QuoteId | uniqueidentifier | Primary key |
| QuoteNumber | nvarchar | System-generated sequential number |
| RevisionNumber | int | Quote version tracking |
| CustomerId | uniqueidentifier | Polymorphic (Account/Contact) |
| PriceLevelId | uniqueidentifier | Pricing list reference |
| EffectiveFrom | datetime | Quote validity start |
| EffectiveTo | datetime | Quote validity end |
| TotalAmount | money | Total including tax/freight |
| TotalTax | money | Calculated tax |
| FreightAmount | money | Shipping cost |

**Relationships**:
- N:1 with OpportunityBase (source opportunity)
- N:1 with PriceLevelBase
- 1:N with QuoteDetailBase (line items)
- N:N via ContactQuotes junction

---

### 3.4 Sales Order Entity (SalesOrderBase) - Object Type Code: 1088

**Purpose**: Record confirmed customer purchases

**Business Function**: Track accepted orders from fulfillment through completion

**Key Attributes**:

| Attribute | Type | Purpose |
|-----------|------|---------|
| SalesOrderId | uniqueidentifier | Primary key |
| OrderNumber | nvarchar | System-generated order number |
| CustomerId | uniqueidentifier | Billing customer |
| OpportunityId | uniqueidentifier | Source opportunity |
| QuoteId | uniqueidentifier | Source quote |
| SubmitStatus | int | ERP integration status |
| RequestDeliveryBy | datetime | Requested delivery date |
| TotalAmount | money | Order total |

**State Flow**:
```
Active (0) → Fulfilled (1) → Invoiced (2) → Paid (3)
                    ↓
               Cancelled (4)
```

---

### 3.5 Invoice Entity (InvoiceBase) - Object Type Code: 1090

**Purpose**: Record customer invoices for payment

**Key Attributes**:

| Attribute | Type | Purpose |
|-----------|------|---------|
| InvoiceId | uniqueidentifier | Primary key |
| InvoiceNumber | nvarchar | Invoice number |
| SalesOrderId | uniqueidentifier | Source order |
| TotalAmount | money | Invoice total |
| PaymentTermsCode | int | Payment terms |
| DueDate | datetime | Payment due date |

**State Flow**:
```
Active (0) → Paid (1) → Cancelled (2)
```

---

### 3.6 Account Entity (AccountBase) - Object Type Code: 1

**Purpose**: Represent business organizations/customers

**Key Attributes**:

```sql
CREATE TABLE [dbo].[AccountBase] (
    AccountId uniqueidentifier,
    AccountNumber nvarchar(20),        -- Business ID
    Name nvarchar(160),
    
    -- Hierarchy
    ParentAccountId uniqueidentifier,   -- Self-referential (org hierarchy)
    MasterId uniqueidentifier,          -- For duplicate merging
    
    -- Classification
    IndustryCode int,
    BusinessTypeCode int,
    AccountRatingCode int,
    
    -- Primary Contact
    PrimaryContactId uniqueidentifier,
    
    -- Contact Information
    Telephone1 nvarchar(50),
    EmailAddress1 nvarchar(100),
    WebsiteUrl nvarchar(200),
    
    -- Financial
    Revenue money,
    NumberOfEmployees int,
    CreditLimit money,
    
    -- Ownership
    OwnerId uniqueidentifier,
    OwningBusinessUnit uniqueidentifier,
    
    -- Process
    ProcessId uniqueidentifier,
    StageId uniqueidentifier,
    TraversedPath nvarchar(1250)
)
```

**Relationship Hierarchy**:

```
Account (Parent)
    │
    ├──► Contact (Primary Contact)
    │
    ├──► Opportunity (Customer)
    │         │
    │         ├──► Quote
    │         │
    │         ├──► Sales Order
    │         │
    │         └──► Invoice
    │
    ├──► Address (Multiple via CustomerAddressBase)
    │
    └──► Child Account (ParentAccountId self-reference)
```

---

### 3.7 Contact Entity (ContactBase) - Object Type Code: 2

**Purpose**: Represent individual people

**Key Attributes**:

```sql
CREATE TABLE [dbo].[ContactBase] (
    ContactId uniqueidentifier,
    FirstName nvarchar(100),
    LastName nvarchar(100),
    FullName nvarchar(200),           -- Computed
    YomiFullName nvarchar(200),       -- Phonetic
    
    -- Customer Link
    ParentCustomerId uniqueidentifier, -- Polymorphic (Account/Contact)
    ParentCustomerIdType int,          -- 1=Account, 2=Contact
    
    -- Contact Info
    EmailAddress1 nvarchar(100),
    Telephone1 nvarchar(50),
    MobilePhone nvarchar(50),
    
    -- Personal
    BirthDate datetime,
    GenderCode int,
    JobTitle nvarchar(100),
    Department nvarchar(100),
    
    -- Ownership
    OwnerId uniqueidentifier,
    OwningBusinessUnit uniqueidentifier
)
```

---

## 4. Entity Relationship Mapping

### 4.1 Relationship Types Overview

| Type | Implementation | Example |
|------|---------------|---------|
| **1:N (One-to-Many)** | FK on child table | Account → Contacts |
| **N:1 (Many-to-One)** | FK on child table | Opportunity → Account |
| **N:N (Many-to-Many)** | Junction table | Lead → Competitor |
| **Polymorphic** | Id + Type columns | Opportunity.CustomerId |

### 4.2 One-to-Many (1:N) Relationships

```sql
-- Account to Contact: Many contacts can belong to one account
ALTER TABLE [dbo].[ContactBase] ADD CONSTRAINT 
    [contact_parent_customer_account] 
    FOREIGN KEY([ParentCustomerId])
    REFERENCES [dbo].[AccountBase] ([AccountBase].[AccountId])
```

### 4.3 Many-to-Many (N:N) Relationships

```sql
-- Junction table for Account-Lead relationship
CREATE TABLE [dbo].[AccountLeads] (
    AccountId uniqueidentifier NOT NULL,
    LeadId uniqueidentifier NOT NULL,
    CONSTRAINT [PK_AccountLeads] PRIMARY KEY (AccountId, LeadId)
)
```

**Standard N:N Junction Tables**:

| Junction Table | Entity 1 | Entity 2 |
|---------------|----------|----------|
| AccountLeads | Account | Lead |
| ContactLeads | Contact | Lead |
| ContactQuotes | Contact | Quote |
| ContactOrders | Contact | SalesOrder |
| ContactInvoices | Contact | Invoice |
| LeadCompetitors | Lead | Competitor |
| LeadProduct | Lead | Product |

### 4.4 Polymorphic Relationships

```sql
-- CustomerId can reference either Account or Contact
OpportunityBase (
    CustomerId uniqueidentifier,      -- Can be AccountId or ContactId
    CustomerIdType int,               -- 1=Account, 2=Contact
    CustomerIdName nvarchar,         -- Denormalized for display
    CustomerIdYomiName nvarchar      -- Phonetic for display
)
```

**Resolution in Views**:

```sql
-- Logical view resolves polymorphic relationship
[AccountId] = case 
    when [OpportunityBase].[CustomerIdType] = 1 
    then [OpportunityBase].[CustomerId] 
    else NULL end,
[ContactId] = case 
    when [OpportunityBase].[CustomerIdType] = 2 
    then [OpportunityBase].[CustomerId] 
    else NULL end
```

---

## 5. Data Flow Analysis

### 5.1 Lead Qualification Flow

```
┌──────────────────────────────────────────────────────────────────────┐
│                      LEAD QUALIFICATION FLOW                           │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌──────────┐                                                        │
│  │   LEAD   │  1. Sales representative works the lead               │
│  │  (Open)  │  2. Gathers requirements and qualifies               │
│  └────┬─────┘  3. Identifies decision maker                          │
│       │                                                            │
│       │ Qualify Action                                             │
│       ▼                                                            │
│  ┌──────────────┐                                                  │
│  │ OPPORTUNITY  │  • Lead.QualifyingOpportunityId → OpportunityId  │
│  │   CREATED    │  • Lead.StateCode → 1 (Qualified)                │
│  └──────────────┘  • Lead.StatusCode → 5                           │
│                    • Opportunity.OriginatingLeadId → LeadId         │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

**Data Transformation**:

| Lead Field | → | Opportunity Field |
|-----------|---|-------------------|
| LeadId | → | OriginatingLeadId |
| CompanyName | → | (Used in Account creation) |
| Contact info | → | ParentContactId |
| EstimatedValue | → | EstimatedValue |
| - | → | CustomerId (new Account/Contact) |

---

### 5.2 Opportunity to Close Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    OPPORTUNITY TO CLOSE FLOW                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐          │
│  │  QUALIFY    │────▶│  DEVELOP   │────▶│  PROPOSE    │          │
│  │  Stage 1    │     │  Stage 2   │     │  Stage 3    │          │
│  └─────────────┘     └─────────────┘     └──────┬──────┘          │
│                                                   │                  │
│                                                   │                  │
│                      ┌─────────────┐              │                  │
│                      │    CLOSE    │◀─────────────┘                  │
│                      │  (Won/Lost) │                                  │
│                      └──────┬──────┘                                  │
│                             │                                         │
│              ┌──────────────┼──────────────┐                         │
│              │              │              │                         │
│              ▼              ▼              ▼                         │
│       ┌──────────┐   ┌──────────┐   ┌──────────────┐                 │
│       │  QUOTE   │   │  ORDER   │   │ NO SALE      │                 │
│       │          │   │          │   │ (Lost/Cancel)│                 │
│       └──────────┘   └──────────┘   └──────────────┘                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.3 Quote to Order Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                      QUOTE TO ORDER FLOW                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────┐         ┌─────────┐         ┌─────────────┐           │
│   │  QUOTE  │────────▶│ QUOTE   │────────▶│  SALES      │           │
│   │ (Open)  │ Accept  │ (Won)   │  Close  │   ORDER     │           │
│   └─────────┘         └─────────┘         └─────────────┘           │
│        │                                            │                 │
│        │                                            │                 │
│        │  Field Mapping                             │                 │
│        │                                            │                 │
│        ├────────────────────┬───────────────────────┘                 │
│        │                    │                                         │
│        ▼                    ▼                                         │
│   QuoteDetails         SalesOrderDetails                             │
│        │                    │                                         │
│   - ProductId          - ProductId                                  │
│   - Quantity           - Quantity                                   │
│   - UnitPrice          - UnitPrice                                  │
│   - Discount%          - Discount%                                   │
│   - Tax                - Tax                                        │
│        │                    │                                         │
│        │                    │                                         │
│        ▼                    ▼                                         │
│   QuoteDetailId ──────▶ SalesOrderDetailId                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Data Copy Rules**:

| Quote Field | → | Sales Order Field |
|-------------|---|-------------------|
| CustomerId | → | CustomerId |
| PriceLevelId | → | PriceLevelId |
| TransactionCurrencyId | → | TransactionCurrencyId |
| ExchangeRate | → | ExchangeRate |
| TotalAmount | → | TotalAmount |
| QuoteId | → | QuoteId |
| QuoteNumber | → | OrderNumber (system-generated) |

---

### 5.4 Order to Invoice Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                     ORDER TO INVOICE FLOW                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────┐         ┌─────────────┐         ┌──────────┐    │
│   │  SALES     │────────▶│  SALES     │────────▶│ INVOICE  │    │
│   │  ORDER     │  Fulfill │  ORDER     │  Invoice │          │    │
│   │ (Active)   │         │(Fulfilled) │          │ (Active) │    │
│   └─────────────┘         └─────────────┘         └────┬─────┘    │
│        │                                                  │          │
│        │                                                  │          │
│        │  Items and Totals Copied                        │          │
│        │                                                  │          │
│        ├──────────────────────────────────────────────────┘          │
│        │                                                            │
│        ▼                                                            │
│   SalesOrderDetail ──────────────▶ InvoiceDetail                    │
│        │                                                            │
│   All line item fields copied                                      │
│        │                                                            │
│        ▼                                                            │
│   SalesOrderId ─────────────────▶ InvoiceId                         │
│                                       SalesOrderId = @SalesOrderId  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Sales Pipeline Flow

### 6.1 Complete Lead-to-Cash Process

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        COMPLETE SALES PIPELINE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    │
│  │  LEAD   │───▶│OPPORTUNITY│───▶│  QUOTE  │───▶│  ORDER  │───▶│ INVOICE │    │
│  │         │    │          │    │         │    │         │    │         │    │
│  │Qualify  │    │ Develop  │    │ Accept  │    │ Fulfill │    │ Collect │    │
│  └────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘    │
│       │              │              │              │              │          │
│       ▼              ▼              ▼              ▼              ▼          │
│   Open/Qualified   Open/Won      Open/Won       Active         Active      │
│                    (Lost)        (Lost)         Fulfilled      Paid        │
│                                                  Cancelled     Cancelled   │
│                                                                              │
│  KEY METRICS:                                                                │
│  ┌────────────────────────────────────────────────────────────────────┐   │
│  │  Lead-to-Opportunity Rate = Qualified Leads / Total Leads           │   │
│  │  Opportunity-to-Quote Rate = Quotes Created / Opportunities Won      │   │
│  │  Quote-to-Order Rate = Orders / Quotes Accepted                      │   │
│  │  Order-to-Invoice Rate = Invoices Created / Orders Fulfilled       │   │
│  │  Average Cycle Time = Sum(Close Date - Created Date) / Won Deals    │   │
│  └────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Data Transformation at Each Stage

| Stage | Input | Transformation | Output |
|-------|-------|----------------|--------|
| **Lead → Opportunity** | Lead record | Convert, create Account/Contact | New Opportunity, Qualified Lead |
| **Opportunity → Quote** | Opportunity + Products | Add pricing, validity dates | Quote with Details |
| **Quote → Order** | Accepted Quote | Copy all fields | Sales Order with Details |
| **Order → Invoice** | Fulfilled Order | Generate invoice request | Invoice with Details |
| **Invoice → Payment** | Paid Invoice | Record payment | Closed Invoice |

---

## 7. Financial Transaction Flow

### 7.1 Pricing Engine Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                      PRICING ENGINE FLOW                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────┐                                                 │
│  │  PRICE LEVEL  │  (PriceLevelBase - Price List)                 │
│  └───────┬────────┘                                                 │
│          │                                                           │
│          │                                                          │
│          ▼                                                           │
│  ┌───────────────────────────────────────┐                         │
│  │     PRODUCT PRICE LEVEL               │                         │
│  │  (ProductPriceLevelBase)               │                         │
│  │                                        │                         │
│  │  ProductId + PriceLevelId ──────▶ Price │                         │
│  └───────────────────────────────────────┘                         │
│          │                                                           │
│          │ Apply to Quote/Order Line                                │
│          ▼                                                           │
│  ┌────────────────────────────────────────┐                        │
│  │           LINE CALCULATION              │                        │
│  │                                         │                        │
│  │  Quantity × UnitPrice = LineAmount       │                        │
│  │  LineAmount × Discount% = LineDiscount   │                        │
│  │  LineAmount - LineDiscount = NetLine    │                        │
│  └───────────────┬────────────────────────┘                        │
│                  │                                                   │
│      ┌───────────┴────────────┐                                      │
│      ▼                        ▼                                      │
│ ┌─────────────┐      ┌────────────────┐                            │
│ │ DISCOUNTS  │      │    TAXES       │                            │
│ │ (DiscountBase)    │ (Calculation)   │                            │
│ │                │      │              │                            │
│ │ Volume-based │      │ Line × Rate % │                            │
│ │ Percentage   │      │ = Tax Amount  │                            │
│ └──────────────┘      └───────────────┘                            │
│      │                        │                                      │
│      └───────────┬────────────┘                                      │
│                  ▼                                                   │
│          ┌──────────────┐                                            │
│          │   TOTALS    │                                            │
│          │              │                                            │
│          │ NetTotal    │                                            │
│          │ + Freight   │                                            │
│          │ + Tax       │                                            │
│          │ ─ Discount  │                                            │
│          │ ─────────── │                                            │
│          │ TotalAmount │                                            │
│          └──────────────┘                                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 7.2 Currency Conversion Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                   CURRENCY CONVERSION FLOW                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Transaction Currency                  Base Currency (USD)           │
│  ┌─────────────────┐              ┌─────────────────┐                │
│  │ EstimatedValue │   ×          │ EstimatedValue │                │
│  │   = 10,000     │   Exchange   │     _Base      │                │
│  │      EUR       │    Rate      │    = 10,900    │                │
│  └─────────────────┘   1.09       └─────────────────┘                │
│                                                                      │
│  Exchange Rate Sources:                                              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  1. Microsoft Dynamics 365 Currency Entity                  │    │
│  │  2. Exchange Rate Table (TransactionCurrencyBase)           │    │
│  │  3. Manual Entry / Integration                               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Historical Rate Preservation:                                       │
│  - ExchangeRate stored at transaction time                          │
│  - Enables accurate historical reporting                            │
│  - Supports currency revaluation                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 8. Activity & Communication Flow

### 8.1 Activity Party Model

```
┌─────────────────────────────────────────────────────────────────────┐
│                      ACTIVITY PARTY ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ActivityPointerBase (Parent)                                       │
│         │                                                            │
│         │ 1:N                                                       │
│         ▼                                                            │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │              ACTIVITYPARTYBASE                             │        │
│  │                                                          │        │
│  │  ActivityId ───────────▶ FK to ActivityPointer          │        │
│  │  PartyId ──────────────▶ FK to Party (Account/Contact/  │        │
│  │                           User)                           │        │
│  │  ParticipationTypeMask ──▶ Role in activity              │        │
│  │                                                          │        │
│  │  ParticipationTypeMask Values:                            │        │
│  │  ┌────────────────────────────────────────────────────┐  │        │
│  │  │ 1 = Sender                                         │  │        │
│  │  │ 2 = To (Primary Recipient)                         │  │        │
│  │  │ 3 = CC                                             │  │        │
│  │  │ 4 = BCC                                            │  │        │
│  │  │ 5 = Required Attendee                              │  │        │
│  │  │ 6 = Optional Attendee                              │  │        │
│  │  │ 7 = Organizer                                      │  │        │
│  │  │ 8 = Regarding (Record linked to activity)          │  │        │
│  │  │ 9 = Owner                                          │  │        │
│  │  │10 = Resource                                       │  │        │
│  │  └────────────────────────────────────────────────────┘  │        │
│  └─────────────────────────────────────────────────────────┘        │
│                                                                      │
│  Activity Types (Derived Tables):                                    │
│  ┌────────────┬────────────┬──────────────────────────────────┐     │
│  │  Type      │    OTC     │            Description           │     │
│  ├────────────┼────────────┼──────────────────────────────────┤     │
│  │ Appointment│   4201     │ Calendar/Meeting                  │     │
│  │ Email      │   4202     │ Email message                   │     │
│  │ PhoneCall  │   4210     │ Phone conversation              │     │
│  │ Task       │   4212     │ To-do item                      │     │
│  │ Letter     │   4207     │ Physical letter                  │     │
│  │ Fax        │   4204     │ Fax transmission                │     │
│  │ CampaignResponse│4402  │ Campaign response                │     │
│  │ CampaignActivity│4404 │ Marketing activity               │     │
│  │ ServiceAppointment│4214│ Service booking                 │     │
│  │ SocialActivity│  4216  │ Social post                     │     │
│  └────────────┴────────────┴──────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 8.2 Regarding Relationship

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REGARDING RELATIONSHIP                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ActivityPointerBase                                                │
│  ┌─────────────────────────────────────────────┐                   │
│  │  RegardingObjectId uniqueidentifier          │──────┐            │
│  │  RegardingObjectTypeCode int                │      │            │
│  └─────────────────────────────────────────────┘      │            │
│                                                        │            │
│                   ┌───────────────────────────────────┘            │
│                   │                                                 │
│                   ▼                                                 │
│  ┌─────────────────────────────────────────────┐                   │
│  │     POLYMORPHIC LOOKUP                      │                   │
│  │                                              │                   │
│  │  RegardingObjectTypeCode:                   │                   │
│  │  ┌────┬───────────────────────────────┐     │                   │
│  │  │  1 │ Account                      │     │                   │
│  │  │  2 │ Contact                      │     │                   │
│  │  │  3 │ Opportunity                 │     │                   │
│  │  │  4 │ Lead                         │     │                   │
│  │  │  5 │ Case (Incident)             │     │                   │
│  │  │... │ ...                          │     │                   │
│  │  └────┴───────────────────────────────┘     │                   │
│  └─────────────────────────────────────────────┘                   │
│                                                                      │
│  Query Example:                                                     │
│  SELECT * FROM ActivityPointerBase                                   │
│  WHERE RegardingObjectId = 'ABC123'                                 │
│    AND RegardingObjectTypeCode = 1  -- Account                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 9. Custom Entities Analysis

### 9.1 Custom Entity Overview

The database includes custom entities with `canven_` and `new_` prefixes:

| Entity | Table | OTC | Purpose |
|--------|-------|-----|---------|
| canven_city | canven_cityBase | 10006 | City master data |
| canven_onboarding | canven_onboardingBase | 10005 | Customer onboarding |
| canven_propertyonboard | canven_propertyonboardBase | 10008 | Property onboarding |
| new_usertype | new_usertypeBase | 10010 | User classification |
| new_userdetails | new_userdetailsBase | 10011 | User profile extension |
| new_customercomplaint | new_customercomplaintBase | 10012 | Complaint management |

### 9.2 canven_onboarding Entity

**Purpose**: Customer onboarding workflow management

**Key Attributes**:

```sql
CREATE TABLE [dbo].[canven_onboardingBase] (
    canven_onboardingId uniqueidentifier,
    canven_name nvarchar(100),
    canven_City uniqueidentifier,           -- FK to canven_cityBase
    canven_FinalApprovalOwner uniqueidentifier, -- FK to SystemUserBase
    canven_ReviewerOwner uniqueidentifier,  -- FK to SystemUserBase
    
    -- Standard CRM attributes
    OwningBusinessUnit uniqueidentifier,
    OwnerId uniqueidentifier,
    StateCode int,
    StatusCode int,
    CreatedOn datetime,
    ModifiedOn datetime,
    VersionNumber timestamp
)
```

**Relationships**:

| Relationship | Target | Type |
|--------------|--------|------|
| canven_City | canven_cityBase | N:1 |
| canven_FinalApprovalOwner | SystemUserBase | N:1 |
| canven_ReviewerOwner | SystemUserBase | N:1 |
| OwningBusinessUnit | BusinessUnitBase | N:1 |
| OwnerId | OwnerBase | N:1 |

**Foreign Keys**:

```sql
ALTER TABLE [dbo].[canven_onboardingBase] ADD CONSTRAINT 
    [canven_canven_city_canven_onboarding_City] 
    FOREIGN KEY([canven_City])
    REFERENCES [dbo].[canven_cityBase] ([canven_cityId])

ALTER TABLE [dbo].[canven_onboardingBase] ADD CONSTRAINT 
    [canven_systemuser_canven_onboarding_FinalApprovalOwner] 
    FOREIGN KEY([canven_FinalApprovalOwner])
    REFERENCES [dbo].[SystemUserBase] ([SystemUserId])

ALTER TABLE [dbo].[canven_onboardingBase] ADD CONSTRAINT 
    [canven_systemuser_canven_onboarding_ReviewerOwner] 
    FOREIGN KEY([canven_ReviewerOwner])
    REFERENCES [dbo].[SystemUserBase] ([SystemUserId])
```

**Industry Context**: The `canven_*` entities suggest a **real estate/property** industry focus with:
- City-based property routing
- Multi-level approval workflow (Reviewer → Final Approval)
- Property onboarding tracking

---

## 10. Technical Specifications

### 10.1 Database Architecture Summary

| Component | Specification |
|-----------|---------------|
| **Total Tables** | ~300+ |
| **Core Sales Tables** | 15+ |
| **Custom Tables** | 6 |
| **Activity Tables** | 10+ |
| **Primary Key Type** | uniqueidentifier (GUID) |
| **Clustered Index** | On Primary Key |
| **Naming Convention** | [EntityName]Base |

### 10.2 Index Strategy

| Index Type | Prefix | Purpose |
|-----------|--------|---------|
| Quick Find | fndx_ | Global search |
| Standard | ndx_ | Query optimization |
| Covering | cndx_ | Composite queries |
| Cascade | fndx_for_cascaderelationship_ | Relationship support |
| Sync | fndx_Sync_ | Offline synchronization |

### 10.3 Security Model

| Security Layer | Implementation |
|---------------|----------------|
| **Row-Level** | Filtered views (Filtered[Entity]) |
| **Field-Level** | FieldPermissionBase |
| **Ownership** | OwnerBase (User/Team/Organization) |
| **Business Unit** | BusinessUnitBase hierarchy |
| **Role-Based** | RoleBase + RolePrivilegesBase |

### 10.4 Concurrency Control

| Mechanism | Implementation |
|-----------|----------------|
| **Optimistic Locking** | VersionNumber (timestamp) |
| **Conflict Detection** | WHERE VersionNumber = @Original |
| **Sync Support** | fndx_Sync_VersionNumber unique index |

---

## Conclusion

This document provides a comprehensive analysis of the Microsoft Dynamics 365 CRM Sales Module database structure and data flow. The architecture demonstrates:

1. **Enterprise-Grade Design**: 300+ tables implementing full CRM functionality
2. **Standard Patterns**: Follows Microsoft Dynamics 365 best practices
3. **Scalable Architecture**: GUID-based keys, layered views, comprehensive indexing
4. **Complete Sales Process**: Lead-to-Cash implementation with full pipeline tracking
5. **Custom Extensions**: Industry-specific customization for property/real estate

The data structure supports complex business processes while maintaining data integrity through foreign key constraints, optimistic concurrency, and comprehensive audit trails.

---

*Document prepared by Senior Microsoft Dynamics 365 CRM Architect & Dataverse Database Expert*
