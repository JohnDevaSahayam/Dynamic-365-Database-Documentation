# Microsoft Dynamics 365 CRM Sales Module Database
## Technical Architecture & Data Flow Documentation

---

**Author:** Microsoft Dynamics 365 CRM Sales Module Architect & Senior Database Analyst  
**Database:** CANCRM_MSCRM  
**Platform:** Microsoft SQL Server / Microsoft Dynamics 365 Dataverse  
**Version:** 1.0  
**Date:** February 2026

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Database Architecture Overview](#2-database-architecture-overview)
3. [Core Entity Deep Dive](#3-core-entity-deep-dive)
4. [Sales Process Data Flow](#4-sales-process-data-flow)
5. [Entity Relationships](#5-entity-relationships)
6. [Transaction & Pricing Flow](#6-transaction--pricing-flow)
7. [Security & Ownership Model](#7-security--ownership-model)
8. [Data Integrity & Validation](#8-data-integrity--validation)
9. [Performance Considerations](#9-performance-considerations)
10. [Custom Entities](#10-custom-entities)

---

## 1. Introduction

### 1.1 Purpose

This document provides a comprehensive technical analysis of the Microsoft Dynamics 365 CRM Sales Module database implementation. It serves as a complete reference for developers, database administrators, and system architects working with this CRM implementation.

### 1.2 Scope

The documentation covers:
- Complete database schema for Sales Module entities
- End-to-end data flow from Lead generation to Invoice payment
- Entity relationships and referential integrity
- Security architecture and ownership model
- Performance optimization strategies

---

## 2. Database Architecture Overview

### 2.1 Database Schema Pattern

Microsoft Dynamics 365 CRM uses a specific naming convention for database tables:

| Table Type | Naming Pattern | Example |
|------------|----------------|---------|
| Base Table | EntityName + Base | AccountBase |
| Extension Table | EntityName + Extension | AccountExtensionBase |
| History Table | EntityName + History | AccountHistory |
| Archive Table | EntityName + Archive | AccountArchive |
| Filtered View | Filtered + EntityName | FilteredAccount |

### 2.2 Base Table Structure Pattern

Every base table in Dynamics 365 follows a standardized structure:

```sql
CREATE TABLE [dbo].[EntityNameBase](
    -- Primary Key
    [EntityNameId] [uniqueidentifier] NOT NULL,
    
    -- Business Fields (entity-specific)
    [Field1] [nvarchar](100) NULL,
    [Field2] [int] NULL,
    
    -- Ownership (required for security)
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,           -- 8=User, 9=Team
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    
    -- State Management (required)
    [StateCode] [int] NOT NULL,             -- Active/Inactive status
    [StatusCode] [int] NULL,                -- Status reason
    
    -- Audit Fields (required)
    [CreatedOn] [datetime] NULL,
    [CreatedBy] [uniqueidentifier] NULL,
    [ModifiedOn] [datetime] NULL,
    [ModifiedBy] [uniqueidentifier] NULL,
    
    -- Concurrency Control
    [VersionNumber] [timestamp] NULL,
    
    CONSTRAINT [ndx_PrimaryKey_EntityName] PRIMARY KEY CLUSTERED (EntityNameId)
)
```

### 2.3 Field Naming Conventions

| Suffix | Purpose | Example |
|--------|---------|---------|
| `Id`   | Primary key identifier | AccountId |
| `Name` | Display name lookup | CustomerIdName |
| `Code`  | Picklist/OptionSet value | StatusCode |
| `_Base` | Base currency equivalent | Amount_Base |
| `_Yomi` | Phonetic reading (Japanese) | YomiFullName |
| `Address` | Address composite fields | Address1_Line1 |

---

## 3. Core Entity Deep Dive

### 3.1 Account Entity (AccountBase)

**Purpose:** Represents a business or organization

**Object Type Code:** 1

#### Schema Definition

```sql
CREATE TABLE [dbo].[AccountBase](
    -- Primary Key
    [AccountId] [uniqueidentifier] NOT NULL,
    
    -- Core Business Fields
    [Name] [nvarchar](160) NULL,              -- Account name (searchable)
    [AccountNumber] [nvarchar](20) NULL,     -- Business account number
    [AccountCategoryCode] [int] NULL,        -- Category classification
    [CustomerSizeCode] [int] NULL,           -- Size (Enterprise, SMB, etc.)
    [CustomerTypeCode] [int] NULL,           -- Type (Customer, Competitor)
    [AccountRatingCode] [int] NULL,          -- Rating (Hot, Warm, Cold)
    [IndustryCode] [int] NULL,               -- Industry classification
    [TerritoryCode] [int] NULL,              -- Geographic territory
    
    -- Financial Information
    [Revenue] [money] NULL,                   -- Annual revenue
    [NumberOfEmployees] [int] NULL,          -- Employee count
    [OwnershipCode] [int] NULL,              -- Ownership type
    
    -- Contact Information
    [EMailAddress1] [nvarchar](100) NULL,
    [Telephone1] [nvarchar](50) NULL,
    [Telephone2] [nvarchar](50) NULL,
    [Fax] [nvarchar](50) NULL,
    [WebSiteURL] [nvarchar](200) NULL,
    
    -- Primary Address (Address1)
    [Address1_Line1] [nvarchar](250) NULL,
    [Address1_Line2] [nvarchar](250) NULL,
    [Address1_Line3] [nvarchar](250) NULL,
    [Address1_City] [nvarchar](80) NULL,
    [Address1_StateOrProvince] [nvarchar](50) NULL,
    [Address1_PostalCode] [nvarchar](20) NULL,
    [Address1_Country] [nvarchar](100) NULL,
    
    -- Secondary Address (Address2)
    [Address2_Line1] [nvarchar](250) NULL,
    [Address2_City] [nvarchar](80) NULL,
    [Address2_StateOrProvince] [nvarchar](50) NULL,
    [Address2_PostalCode] [nvarchar](20) NULL,
    
    -- Hierarchy (Parent Account for organizational hierarchy)
    [ParentAccountId] [uniqueidentifier] NULL,
    
    -- Ownership & Security
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    
    -- State Management
    [StateCode] [int] NOT NULL DEFAULT 0,     -- 0=Active, 1=Inactive
    [StatusCode] [int] NULL,
    
    -- Audit Fields
    [CreatedOn] [datetime] NULL,
    [CreatedBy] [uniqueidentifier] NULL,
    [ModifiedOn] [datetime] NULL,
    [ModifiedBy] [uniqueidentifier] NULL,
    
    -- Concurrency
    [VersionNumber] [timestamp] NULL,
    
    CONSTRAINT [ndx_PrimaryKey_Account] PRIMARY KEY CLUSTERED (AccountId)
)
```

#### Key Business Rules

1. **Name** is the primary searchable field
2. **AccountNumber** can be auto-generated or manually assigned
3. **ParentAccountId** creates organizational hierarchy (supports multi-level)
4. **CustomerTypeCode** determines how the account is classified:
   - 1 = Customer
   - 2 = Competitor
   - 3 = Partner
   - 4 = Reseller

#### Indexes

| Index Name | Columns | Purpose |
|------------|---------|---------|
| ndx_PrimaryKey_Account | AccountId (PK) | Clustered primary key |
| ndx_Account_OwnerId | OwnerId | Ownership queries |
| ndx_Account_Name | Name | Search and lookup |
| ndx_Account_ParentAccountId | ParentAccountId | Hierarchy traversal |

---

### 3.2 Contact Entity (ContactBase)

**Purpose:** Represents an individual person, typically associated with an Account

**Object Type Code:** 2

#### Schema Definition

```sql
CREATE TABLE [dbo].[ContactBase](
    -- Primary Key
    [ContactId] [uniqueidentifier] NOT NULL,
    
    -- Name Components
    [FirstName] [nvarchar](50) NULL,
    [MiddleName] [nvarchar](50) NULL,
    [LastName] [nvarchar](50) NULL,
    [FullName] [nvarchar](160) NULL,           -- Computed: First + Middle + Last
    [YomiFirstName] [nvarchar](50) NULL,       -- Phonetic (Japanese)
    [YomiLastName] [nvarchar](50) NULL,
    [YomiFullName] [nvarchar](160) NULL,
    
    -- Contact Information
    [EMailAddress1] [nvarchar](100) NULL,
    [EMailAddress2] [nvarchar](100) NULL,
    [Telephone1] [nvarchar](50) NULL,
    [Telephone2] [nvarchar](50) NULL,
    [MobilePhone] [nvarchar](50) NULL,
    [Fax] [nvarchar](50) NULL,
    
    -- Professional Information
    [JobTitle] [nvarchar](100) NULL,
    [Department] [nvarchar](100) NULL,
    
    -- Parent Customer (Polymorphic - can be Account or Contact)
    [ParentCustomerId] [uniqueidentifier] NULL,
    [ParentCustomerIdType] [int] NULL,        -- 1=Account, 2=Contact
    
    -- Address Information
    [Address1_Line1] [nvarchar](250) NULL,
    [Address1_City] [nvarchar](80) NULL,
    [Address1_StateOrProvince] [nvarchar](50) NULL,
    [Address1_PostalCode] [nvarchar](20) NULL,
    [Address1_Country] [nvarchar](100) NULL,
    
    -- Birth/Anniversary
    [BirthDate] [datetime] NULL,
    [Anniversary] [datetime] NULL,
    
    -- Master Record (for duplicate merging)
    [MasterId] [uniqueidentifier] NULL,       -- Links to master record
    
    -- Ownership & Security
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    
    -- State Management
    [StateCode] [int] NOT NULL DEFAULT 0,
    [StatusCode] [int] NULL,
    
    -- Audit Fields
    [CreatedOn] [datetime] NULL,
    [CreatedBy] [uniqueidentifier] NULL,
    [ModifiedOn] [datetime] NULL,
    [ModifiedBy] [uniqueidentifier] NULL,
    [VersionNumber] [timestamp] NULL,
    
    CONSTRAINT [ndx_PrimaryKey_Contact] PRIMARY KEY CLUSTERED (ContactId)
)
```

#### Key Business Rules

1. **FullName** is auto-calculated from FirstName, MiddleName, LastName
2. **ParentCustomerId** creates relationship to Account (typical) or other Contact
3. **MasterId** supports duplicate record merging functionality

---

### 3.3 Lead Entity (LeadBase)

**Purpose:** Represents a prospective customer before qualification

**Object Type Code:** 4

#### Schema Definition

```sql
CREATE TABLE [dbo].[LeadBase](
    -- Primary Key
    [LeadId] [uniqueidentifier] NOT NULL,
    
    -- Basic Information
    [Subject] [nvarchar](200) NULL,
    [FirstName] [nvarchar](50) NULL,
    [LastName] [nvarchar](50) NULL,
    [CompanyName] [nvarchar](100) NULL,
    
    -- Contact Information
    [EMailAddress1] [nvarchar](100) NULL,
    [EMailAddress2] [nvarchar](100) NULL,
    [Telephone1] [nvarchar](50) NULL,
    [Telephone2] [nvarchar](50) NULL,
    [MobilePhone] [nvarchar](50) NULL,
    [Fax] [nvarchar](50) NULL,
    [WebSiteURL] [nvarchar](200) NULL,
    
    -- Address
    [Address1_Line1] [nvarchar](250) NULL,
    [Address1_City] [nvarchar](80) NULL,
    [Address1_StateOrProvince] [nvarchar](50) NULL,
    [Address1_PostalCode] [nvarchar](20) NULL,
    [Address1_Country] [nvarchar](100) NULL,
    
    -- Lead Classification
    [LeadSourceCode] [int] NULL,             -- Source of lead
    [RatingCode] [int] NULL,                 -- Hot, Warm, Cold
    
    -- Qualification Data
    [EstimatedAmount] [money] NULL,
    [EstimatedCloseDate] [datetime] NULL,
    [PurchaseTimeframeCode] [int] NULL,      -- When they plan to buy
    [PurchaseProcessCode] [int] NULL,        -- Decision process
    
    -- Business Information
    [NumberOfEmployees] [int] NULL,
    [Revenue] [money] NULL,
    [IndustryCode] [int] NULL,
    
    -- Qualification Status
    [StateCode] [int] NOT NULL DEFAULT 0,   -- 0=Open, 1=Qualified, 2=Disqualified
    [StatusCode] [int] NULL,
    
    -- Conversion Tracking (populated on qualification)
    [QualifiedFromContactId] [uniqueidentifier] NULL,  -- Created Contact
    [QualifiedFromAccountId] [uniqueidentifier] NULL,  -- Created Account
    [QualifiedFromOpportunityId] [uniqueidentifier] NULL, -- Created Opportunity
    
    -- Campaign Association
    [CampaignId] [uniqueidentifier] NULL,    -- Originating campaign
    
    -- Ownership & Security
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    
    -- Audit Fields
    [CreatedOn] [datetime] NULL,
    [CreatedBy] [uniqueidentifier] NULL,
    [ModifiedOn] [datetime] NULL,
    [ModifiedBy] [uniqueidentifier] NULL,
    [VersionNumber] [timestamp] NULL,
    
    CONSTRAINT [ndx_PrimaryKey_Lead] PRIMARY KEY CLUSTERED (LeadId)
)
```

#### Lead State Transitions

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         LEAD LIFECYCLE                                  │
└─────────────────────────────────────────────────────────────────────────┘

                         ┌─────────┐
                    ┌───▶│  OPEN  │◀───────┐
                    │    └─────────┘        │
                    │    StateCode = 0      │
                    │                        │
           Qualify │                        │ Disqualify
                    │                        │
                    ▼                        ▼
            ┌─────────────┐          ┌───────────────┐
            │ QUALIFIED   │          │DISQUALIFIED   │
            └─────────────┘          └───────────────┘
           StateCode = 1            StateCode = 2
           
    Converts to:
    - Contact (QualifiedFromContactId)
    - Account (QualifiedFromAccountId)
    - Opportunity (QualifiedFromOpportunityId)
```

#### Lead Status Codes

| StateCode | State | StatusCode | Status | Description |
|-----------|-------|------------|--------|-------------|
| 0 | Open  | 1 | New | Newly created lead |
| 0 | Open  | 2 | Contacted | Initial contact made |
| 0 | Open  | 3 | Qualified | Ready for conversion |
| 0 | Open  | 4 | In Progress | Being worked |
| 0 | Open  | 6 | Lost | Lead lost |
| 1 | Qualified | 7 | In Progress | Converted, opportunity active |
| 1 | Qualified | 8 | Won | Successfully converted |
| 2 | Disqualified | 9 | Cannot Contact | Unable to reach |
| 2 | Disqualified | 10 | Lost | Lost to competitor |
| 2 | Disqualified | 11 | Not Interested | Not interested |
| 2 | Disqualified | 12 | Already Enrolled | Already customer |

---

### 3.4 Opportunity Entity (OpportunityBase)

**Purpose:** Represents a qualified sales opportunity

**Object Type Code:** 3

#### Schema Definition

```sql
CREATE TABLE [dbo].[OpportunityBase](
    -- Primary Key
    [OpportunityId] [uniqueidentifier] NOT NULL,
    
    -- Basic Information
    [Name] [nvarchar](200) NULL,
    [Description] [nvarchar](max) NULL,
    
    -- Customer (Polymorphic - Account or Contact)
    [CustomerId] [uniqueidentifier] NULL,
    [CustomerIdType] [int] NULL,             -- 1=Account, 2=Contact
    [CustomerIdName] [nvarchar](160) NULL,   -- Denormalized for display
    
    -- Sales Information
    [EstimatedValue] [money] NULL,           -- Expected revenue
    [EstimatedRevenue] [money] NULL,        -- Calculated revenue
    [ActualValue] [money] NULL,              -- Won value
    [BudgetAmount] [money] NULL,             -- Customer's budget
    
    -- Sales Process
    [Probability] [int] NULL,                 -- 0-100%
    [StepId] [uniqueidentifier] NULL,        -- Current stage ID
    [StepName] [nvarchar](100) NULL,        -- Current stage name
    
    -- Timeline
    [EstimatedCloseDate] [datetime] NULL,
    [ActualCloseDate] [datetime] NULL,
    
    -- Rating & Classification
    [RatingCode] [int] NULL,                -- Hot, Warm, Cold
    
    -- Resolution (for closed opportunities)
    [ClosedReasonCode] [int] NULL,
    [CompetitorId] [uniqueidentifier] NULL,
    
    -- Pricing
    [PriceLevelId] [uniqueidentifier] NULL,
    [DiscountAmount] [money] NULL,
    [DiscountPercentage] [decimal](5, 2) NULL,
    
    -- Currency
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [ExchangeRate] [decimal](23, 10) NULL,
    
    -- Status
    [StateCode] [int] NOT NULL DEFAULT 0,    -- 0=Open, 1=Won, 2=Lost
    [StatusCode] [int] NULL,
    
    -- Source Tracking
    [OriginatingLeadId] [uniqueidentifier] NULL,
    [CampaignId] [uniqueidentifier] NULL,
    
    -- Ownership & Security
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    
    -- Audit Fields
    [CreatedOn] [datetime] NULL,
    [CreatedBy] [uniqueidentifier] NULL,
    [ModifiedOn] [datetime] NULL,
    [ModifiedBy] [uniqueidentifier] NULL,
    [VersionNumber] [timestamp] NULL,
    
    CONSTRAINT [ndx_PrimaryKey_Opportunity] PRIMARY KEY CLUSTERED (OpportunityId)
)
```

#### Opportunity Sales Stages

| Step Name | Step Order | Probability |
|-----------|------------|-------------|
| Qualify | 1 | 10% |
| Develop | 2 | 25% |
| Propose | 3 | 50% |
| Close | 4 | 100% |

#### Opportunity State Transitions

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     OPPORTUNITY LIFECYCLE                               │
└─────────────────────────────────────────────────────────────────────────┘

                         ┌─────────┐
                    ┌───▶│   OPEN  │◀──────────────┐
                    │    └─────────┘               │
                    │    StateCode = 0             │
                    │                             │
         Close Won │                             │ Close Lost
                    ▼                             ▼
            ┌─────────┐                   ┌─────────────┐
            │   WON   │                   │    LOST     │
            └─────────┘                   └─────────────┘
           StateCode = 1                  StateCode = 2
            
    Creates: Quote, Order, Invoice
```

---

### 3.5 Quote Entity (QuoteBase)

**Purpose:** Represents a formal sales quotation to a customer

**Object Type Code:** 1084

#### Schema Definition

```sql
CREATE TABLE [dbo].[QuoteBase](
    -- Primary Key
    [QuoteId] [uniqueidentifier] NOT NULL,
    
    -- Basic Information
    [Name] [nvarchar](200) NULL,
    [QuoteNumber] [nvarchar](100) NULL,       -- Auto-generated
    
    -- Customer (Polymorphic)
    [CustomerId] [uniqueidentifier] NULL,
    [CustomerIdType] [int] NULL,
    [CustomerIdName] [nvarchar](160) NULL,
    
    -- Source Opportunity
    [OpportunityId] [uniqueidentifier] NULL,
    
    -- Pricing
    [PriceLevelId] [uniqueidentifier] NULL,
    [TotalAmount] [money] NULL,                -- Total including details
    [TotalAmount_Base] [money] NULL,           -- Base currency
    [DiscountAmount] [money] NULL,
    [DiscountPercentage] [decimal](5, 2) NULL,
    [SubTotal] [money] NULL,
    [Tax] [money] NULL,
    
    -- Validity Period
    [EffectiveFrom] [datetime] NULL,
    [EffectiveTo] [datetime] NULL,
    
    -- Status
    [StateCode] [int] NOT NULL DEFAULT 0,     -- 0=Open, 1=Won, 2=Lost
    [StatusCode] [int] NULL,
    [ClosedDate] [datetime] NULL,
    
    -- Shipping Address
    [ShipTo_Name] [nvarchar](200) NULL,
    [ShipTo_Line1] [nvarchar](250) NULL,
    [ShipTo_City] [nvarchar](80) NULL,
    [ShipTo_StateOrProvince] [nvarchar](50) NULL,
    [ShipTo_PostalCode] [nvarchar](20) NULL,
    [ShipTo_Country] [nvarchar](100) NULL,
    
    -- Bill To Address
    [BillTo_Name] [nvarchar](200) NULL,
    [BillTo_Line1] [nvarchar](250) NULL,
    [BillTo_City] [nvarchar](80) NULL,
    
    -- Currency
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [ExchangeRate] [decimal](23, 10) NULL,
    
    -- Sales Order (if converted)
    [SalesOrderId] [uniqueidentifier] NULL,
    
    -- Ownership & Security
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    
    -- Audit Fields
    [CreatedOn] [datetime] NULL,
    [CreatedBy] [uniqueidentifier] NULL,
    [ModifiedOn] [datetime] NULL,
    [ModifiedBy] [uniqueidentifier] NULL,
    [VersionNumber] [timestamp] NULL,
    
    CONSTRAINT [ndx_PrimaryKey_Quote] PRIMARY KEY CLUSTERED (QuoteId)
)
```

#### Quote Status Codes

| StateCode | State | StatusCode | Status |
|-----------|-------|------------|--------|
| 0 | Open | 1 | Draft |
| 0 | Open | 2 | Active |
| 0 | Open | 3 | Revised |
| 0 | Open | 4 | Inactivated |
| 1 | Won | 5 | Accepted |
| 1 | Won | 6 | Revised (Accepted) |
| 1 | Won | 7 | Accepted (Converted) |
| 2 | Lost | 8 | Rejected |
| 2 | Lost | 9 | Superseded |

---

### 3.6 Sales Order Entity (SalesOrderBase)

**Purpose:** Represents a confirmed customer order

**Object Type Code:** 1088

#### Schema Definition

```sql
CREATE TABLE [dbo].[SalesOrderBase](
    -- Primary Key
    [SalesOrderId] [uniqueidentifier] NOT NULL,
    
    -- Basic Information
    [Name] [nvarchar](200) NULL,
    [OrderNumber] [nvarchar](100) NULL,        -- Auto-generated
    
    -- Customer (Polymorphic)
    [CustomerId] [uniqueidentifier] NULL,
    [CustomerIdType] [int] NULL,
    [CustomerIdName] [nvarchar](160) NULL,
    
    -- Source Documents
    [OpportunityId] [uniqueidentifier] NULL,
    [QuoteId] [uniqueidentifier] NULL,
    
    -- Pricing
    [PriceLevelId] [uniqueidentifier] NULL,
    [TotalAmount] [money] NULL,
    [TotalAmount_Base] [money] NULL,
    [DiscountAmount] [money] NULL,
    [DiscountPercentage] [decimal](5, 2) NULL,
    [FreightAmount] [money] NULL,
    [Tax] [money] NULL,
    [SubTotal] [money] NULL,
    
    -- Dates
    [OrderDate] [datetime] NULL,
    [EffectiveFrom] [datetime] NULL,
    [RequestDeliveryBy] [datetime] NULL,
    [DateFulfilled] [datetime] NULL,
    
    -- Shipping
    [ShippingMethodCode] [int] NULL,
    [ShipTo_Name] [nvarchar](200) NULL,
    [ShipTo_Line1] [nvarchar](250) NULL,
    [ShipTo_City] [nvarchar](80) NULL,
    [ShipTo_StateOrProvince] [nvarchar](50) NULL,
    [ShipTo_PostalCode] [nvarchar](20) NULL,
    [ShipTo_Country] [nvarchar](100) NULL,
    
    -- Billing
    [BillTo_Name] [nvarchar](200) NULL,
    [BillTo_Line1] [nvarchar](250) NULL,
    [BillTo_City] [nvarchar](80) NULL,
    [BillTo_StateOrProvince] [nvarchar](50) NULL,
    [BillTo_PostalCode] [nvarchar](20) NULL,
    [BillTo_Country] [nvarchar](100) NULL,
    
    -- Status
    [StateCode] [int] NOT NULL DEFAULT 0,
    -- 0=Active, 1=Submitted, 2=Cancelled, 3=Fulfilled, 4=Invoiced
    [StatusCode] [int] NULL,
    
    -- Invoice Link
    [InvoiceId] [uniqueidentifier] NULL,
    
    -- Currency
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [ExchangeRate] [decimal](23, 10) NULL,
    
    -- Ownership & Security
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    
    -- Audit Fields
    [CreatedOn] [datetime] NULL,
    [CreatedBy] [uniqueidentifier] NULL,
    [ModifiedOn] [datetime] NULL,
    [ModifiedBy] [uniqueidentifier] NULL,
    [VersionNumber] [timestamp] NULL,
    
    CONSTRAINT [ndx_PrimaryKey_SalesOrder] PRIMARY KEY CLUSTERED (SalesOrderId)
)
```

#### Order Status Flow

```
Order Status Lifecycle:
┌─────────┐  Submit   ┌──────────┐  Cancel  ┌───────────┐
│  ACTIVE │──────────▶│SUBMITTED │─────────▶│ CANCELLED │
└─────────┘           └──────────┘          └───────────┘
                          │
                    Fulfill Order
                          │
                          ▼
                   ┌──────────┐  Invoice  ┌──────────┐
                   │ FULFILLED │─────────▶│ INVOICED │
                   └──────────┘           └──────────┘
```

---

### 3.7 Invoice Entity (InvoiceBase)

**Purpose:** Represents a billing document requesting payment

**Object Type Code:** 1090

#### Schema Definition

```sql
CREATE TABLE [dbo].[InvoiceBase](
    -- Primary Key
    [InvoiceId] [uniqueidentifier] NOT NULL,
    
    -- Basic Information
    [Name] [nvarchar](200) NULL,
    [InvoiceNumber] [nvarchar](100) NULL,      -- Auto-generated
    
    -- Customer (Polymorphic)
    [CustomerId] [uniqueidentifier] NULL,
    [CustomerIdType] [int] NULL,
    [CustomerIdName] [nvarchar](160) NULL,
    
    -- Source Order
    [SalesOrderId] [uniqueidentifier] NULL,
    
    -- Pricing
    [PriceLevelId] [uniqueidentifier] NULL,
    [TotalAmount] [money] NULL,
    [TotalAmount_Base] [money] NULL,
    [DiscountAmount] [money] NULL,
    [DiscountPercentage] [decimal](5, 2) NULL,
    [FreightAmount] [money] NULL,
    [Tax] [money] NULL,
    [SubTotal] [money] NULL,
    
    -- Billing Terms
    [InvoiceDate] [datetime] NULL,
    [DueDate] [datetime] NULL,
    [PaymentTermsCode] [int] NULL,            -- 1=Net 30, 2=Net 60, etc.
    
    -- Status
    [StateCode] [int] NOT NULL DEFAULT 0,     -- 0=Active, 1=Paid, 2=Cancelled
    [StatusCode] [int] NULL,
    
    -- Addresses
    [BillTo_Name] [nvarchar](200) NULL,
    [BillTo_Line1] [nvarchar](250) NULL,
    [BillTo_City] [nvarchar](80) NULL,
    [ShipTo_Name] [nvarchar](200) NULL,
    [ShipTo_Line1] [nvarchar](250) NULL,
    
    -- Currency
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [ExchangeRate] [decimal](23, 10) NULL,
    
    -- Ownership & Security
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    
    -- Audit Fields
    [CreatedOn] [datetime] NULL,
    [CreatedBy] [uniqueidentifier] NULL,
    [ModifiedOn] [datetime] NULL,
    [ModifiedBy] [uniqueidentifier] NULL,
    [VersionNumber] [timestamp] NULL,
    
    CONSTRAINT [ndx_PrimaryKey_Invoice] PRIMARY KEY CLUSTERED (InvoiceId)
)
```

#### Payment Terms Codes

| Code | Term | Description |
|------|------|-------------|
| 1 | Net 30 | Payment due within 30 days |
| 2 | Net 60 | Payment due within 60 days |
| 3 | Net 90 | Payment due within 90 days |
| 4 | Due on Receipt | Payment due immediately |

---

### 3.8 Product Entity (ProductBase)

**Purpose:** Represents products or services sold

**Object Type Code:** 1024

#### Schema Definition

```sql
CREATE TABLE [dbo].[ProductBase](
    -- Primary Key
    [ProductId] [uniqueidentifier] NOT NULL,
    
    -- Basic Information
    [Name] [nvarchar](200) NULL,
    [ProductNumber] [nvarchar](100) NULL,      -- SKU/Part Number
    [VendorName] [nvarchar](100) NULL,
    [VendorPartNumber] [nvarchar](100) NULL,
    
    -- Classification
    [ProductTypeCode] [int] NULL,             -- 1=Product, 2=Service
    [ProductSubCategory] [nvarchar](100) NULL,
    [CategoryId] [uniqueidentifier] NULL,
    
    -- Pricing
    [DefaultUoMScheduleId] [uniqueidentifier] NULL,
    [DefaultUoMId] [uniqueidentifier] NULL,
    [Price] [money] NULL,                     -- List price
    [Cost] [money] NULL,                      -- Cost
    
    -- Inventory
    [QuantityOnHand] [decimal](8, 2) NULL,
    [CurrentCost] [money] NULL,
    [StandardCost] [money] NULL,
    
    -- Hierarchy
    [ParentProductId] [uniqueidentifier] NULL, -- For product families
    
    -- Status
    [StateCode] [int] NOT NULL DEFAULT 0,     -- 0=Active, 1=Retired
    [StatusCode] [int] NULL,
    
    -- Description
    [Description] [nvarchar](max) NULL,
    
    -- Image
    [ImageUrl] [nvarchar](200) NULL,
    
    -- Ownership & Security
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    
    -- Currency
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [ExchangeRate] [decimal](23, 10) NULL,
    
    -- Audit Fields
    [CreatedOn] [datetime] NULL,
    [CreatedBy] [uniqueidentifier] NULL,
    [ModifiedOn] [datetime] NULL,
    [ModifiedBy] [uniqueidentifier] NULL,
    [VersionNumber] [timestamp] NULL,
    
    CONSTRAINT [ndx_PrimaryKey_Product] PRIMARY KEY CLUSTERED (ProductId)
)
```

---

### 3.9 Price List Entity (PriceLevelBase)

**Purpose:** Defines pricing schedules for products

**Object Type Code:** 1022

#### Schema Definition

```sql
CREATE TABLE [dbo].[PriceLevelBase](
    -- Primary Key
    [PriceLevelId] [uniqueidentifier] NOT NULL,
    
    -- Basic Information
    [Name] [nvarchar](100) NULL,
    [Description] [nvarchar](max) NULL,
    
    -- Currency (required - each price list has one currency)
    [TransactionCurrencyId] [uniqueidentifier] NOT NULL,
    
    -- Status
    [StateCode] [int] NOT NULL DEFAULT 0,      -- 0=Active, 1=Inactive
    [StatusCode] [int] NULL,
    
    -- Validity Period
    [BeginDate] [datetime] NULL,
    [EndDate] [datetime] NULL,
    
    -- Ownership & Security
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    
    -- Audit Fields
    [CreatedOn] [datetime] NULL,
    [CreatedBy] [uniqueidentifier] NULL,
    [ModifiedOn] [datetime] NULL,
    [ModifiedBy] [uniqueidentifier] NULL,
    [VersionNumber] [timestamp] NULL,
    
    CONSTRAINT [ndx_PrimaryKey_PriceLevel] PRIMARY KEY CLUSTERED (PriceLevelId)
)
```

---

## 4. Sales Process Data Flow

### 4.1 Complete Lead-to-Cash Flow

```
┌──────────────────────────────────────────────────────────────────────────────────────┐
│                         MICROSOFT DYNAMICS 365 CRM                                  │
│                         LEAD-TO-CASH SALES PROCESS                                   │
└──────────────────────────────────────────────────────────────────────────────────────┘

    ┌──────────┐       ┌────────────┐       ┌─────────────┐       ┌────────┐       ┌──────────┐
    │   LEAD   │──────▶│ OPPORTUNITY│──────▶│   QUOTE    │──────▶│ ORDER  │──────▶│ INVOICE  │
    └──────────┘       └────────────┘       └─────────────┘       └────────┘       └──────────┘
         │                   │                    │                    │                 │
         ▼                   ▼                    ▼                    ▼                 ▼
    ┌──────────┐       ┌────────────┐       ┌─────────────┐       ┌────────┐       ┌──────────┐
    │ LeadBase │       │Opportunity │       │  QuoteBase │       │SalesOrd│       │ Invoice  │
    │          │       │   Base     │       │             │       │  erBase│       │   Base   │
    │ LeadId   │       │Opportunity │       │  QuoteId   │       │OrderId │       │InvoiceId │
    │Subject   │       │  Id        │       │ CustomerId │       │Customer│       │ Invoice  │
    │Company   │       │CustomerId  │       │TotalAmount │       │Id      │       │  Number  │
    │Name      │       │Est.Value   │       │PriceLevel  │       │Total   │       │ Customer │
    │StateCode │       │Probability │       │StateCode   │       │Amount  │       │   Id     │
    │Rating    │       │StepName    │       │EffectiveTo │       │State   │       │TotalAmt  │
    │LeadSource│       │StateCode   │       │            │       │Code    │       │DueDate   │
    └──────────┘       └────────────┘       └─────────────┘       └────────┘       └──────────┘
         
         │                   │                    │                    │                 │
         ▼                   ▼                    ▼                    ▼                 ▼
    Qualify/           Close as            Accept/               Fulfill          Receive
    Disqualify         Won/Lost            Close                  Order            Payment
```

### 4.2 Stage-by-Stage Data Flow

#### Stage 1: Lead Generation

```
Lead Creation Flow:

1. Lead Created
   └─▶ LeadBase.Insert(LeadId, Subject, Name, CompanyName, ContactInfo)
   
2. Lead Scored (optional)
   └─▶ LeadBase.Update(LeadScore, RatingCode)
   
3. Lead Assignment
   └─▶ LeadBase.Update(OwnerId, OwnerIdType, OwningBusinessUnit)

Database State:
- StateCode = 0 (Open)
- StatusCode = 1 (New)
```

#### Stage 2: Lead Qualification

```
Qualify Lead Flow:

1. Validate Required Fields
   └─▶ Check: LastName (B2C) OR CompanyName (B2B) required
   
2. Create Contact (optional)
   └─▶ ContactBase.Insert(ContactId, FirstName, LastName, EMailAddress)
   └─▶ LeadBase.Update(QualifiedFromContactId = ContactId)
   
3. Create Account (optional for B2B)
   └─▶ AccountBase.Insert(AccountId, CompanyName)
   └─▶ LeadBase.Update(QualifiedFromAccountId = AccountId)
   
4. Create Opportunity
   └─▶ OpportunityBase.Insert(OpportunityId, CustomerId = Account/Contact)
   └─▶ LeadBase.Update(QualifiedFromOpportunityId = OpportunityId)
   
5. Update Lead Status
   └─▶ LeadBase.Update(StateCode = 1, StatusCode = 7)

Database State:
- Lead.StateCode = 1 (Qualified)
- Lead.StatusCode = 7 (Converted)
- Contact/Account created with proper ownership
- Opportunity created with CustomerId linked
```

#### Stage 3: Opportunity Development

```
Opportunity Development Flow:

1. Add Products/Services
   └─▶ OpportunityProductBase.Insert(ProductId, Quantity, UnitPrice)
   └─▶ Calculated: ExtendedAmount = Quantity × UnitPrice
   
2. Update Probability
   └─▶ OpportunityBase.Update(Probability based on StepId)
   
3. Update Estimated Value
   └─▶ OpportunityBase.Update(EstimatedRevenue = SUM(ExtendedAmount))
   
4. Advance Sales Stage
   └─▶ OpportunityBase.Update(StepId = next stage)
   └─▶ Business Process Flow updated

Database State:
- Opportunity.StateCode = 0 (Open)
- OpportunityProducts linked to Opportunity
- EstimatedValue calculated from products
```

#### Stage 4: Quote Creation

```
Quote Creation Flow:

1. Create Quote Header
   └─▶ QuoteBase.Insert(QuoteId, CustomerId, OpportunityId, PriceLevelId)
   
2. Add Quote Details
   └─▶ QuoteDetailBase.Insert(QuoteDetailId, QuoteId, ProductId, Quantity)
   └─▶ Get UnitPrice from ProductPriceLevel (ProductId + PriceLevelId)
   └─▶ Calculate ExtendedAmount
   
3. Apply Header Discount (optional)
   └─▶ QuoteBase.Update(DiscountAmount OR DiscountPercentage)
   
4. Calculate Totals
   └─▶ SubTotal = SUM(QuoteDetail.ExtendedAmount)
   └─▶ TotalAmount = SubTotal - Discount + Tax + Freight

Database State:
- Quote.StateCode = 0 (Open)
- Quote.StatusCode = 2 (Active)
- QuoteDetails linked with pricing from Price List
```

#### Stage 5: Quote to Order Conversion

```
Quote to Order Conversion Flow:

1. Validate Quote Status
   └─▶ IF Quote.StateCode IN (0, 1) -- Open or Won
   
2. Create Order Header
   └─▶ Copy from Quote: CustomerId, PriceLevelId, Addresses
   └─▶ SalesOrderBase.Insert(SalesOrderId, QuoteId = OriginalQuoteId)
   
3. Create Order Details (from Quote Details)
   └─▶ For each QuoteDetail:
       └─▶ OrderDetailBase.Insert(OrderDetailId, SalesOrderId, ProductId, Quantity)
       
4. Update Quote Status
   └─▶ QuoteBase.Update(StateCode = 1, StatusCode = 7)
   
5. Update Order Status
   └─▶ SalesOrderBase.Update(StateCode = 0, StatusCode = 1)

Database State:
- Quote.StateCode = 1 (Won)
- Quote.StatusCode = 7 (Accepted - Converted)
- SalesOrder created with all details
- Order linked to Quote (QuoteId)
```

#### Stage 6: Order Fulfillment

```
Order Fulfillment Flow:

1. Validate Order Status
   └─▶ IF SalesOrder.StateCode = 1 (Submitted)
   
2. Update Order Detail Status
   └─▶ OrderDetailBase.Update(StateCode = 1) -- Fulfilled
   
3. Update Order Header Status
   └─▶ SalesOrderBase.Update(StateCode = 3, DateFulfilled = GETDATE())
   
4. Update Inventory (if tracked)
   └─▶ ProductBase.Update(QuantityOnHand - OrderedQuantity)

Database State:
- SalesOrder.StateCode = 3 (Fulfilled)
- SalesOrder.StatusCode = 6 (Fulfilled)
- Inventory updated (if applicable)
```

#### Stage 7: Invoice Generation

```
Invoice Generation Flow:

1. Validate Order Status
   └─▶ IF SalesOrder.StateCode = 3 (Fulfilled)
   
2. Create Invoice Header
   └─▶ Copy from Order: CustomerId, Addresses, PaymentTerms
   └─▶ InvoiceBase.Insert(InvoiceId, SalesOrderId = OriginalOrderId)
   └─▶ Calculate DueDate = InvoiceDate + PaymentTerms
   
3. Create Invoice Details (from Order Details)
   └─▶ For each OrderDetail:
       └─▶ InvoiceDetailBase.Insert(InvoiceDetailId, InvoiceId, ProductId, Quantity)
       
4. Update Order Status
   └─▶ SalesOrderBase.Update(StateCode = 4, InvoiceId = NewInvoiceId)
   
5. Update Invoice Status
   └─▶ InvoiceBase.Update(StateCode = 0, StatusCode = 1)

Database State:
- SalesOrder.StateCode = 4 (Invoiced)
- Invoice.StateCode = 0 (Active)
- Invoice linked to Order
```

#### Stage 8: Payment Receipt

```
Payment Receipt Flow:

1. Record Payment
   └─▶ Create Payment/InvoiceClose activity
   
2. Update Invoice Status
   └─▶ InvoiceBase.Update(StateCode = 1, StatusCode = 4)
   
3. Recognize Revenue (if applicable)
   └─▶ Update Opportunity.ActualValue if linked
   └─▶ Post to finance system (external integration)

Database State:
- Invoice.StateCode = 1 (Paid)
- Invoice.StatusCode = 4 (Paid)
- Revenue recognized
```

---

## 5. Entity Relationships

### 5.1 Primary Relationships

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                           ENTITY RELATIONSHIP DIAGRAM                                   │
└─────────────────────────────────────────────────────────────────────────────────────────┘

                                    ┌─────────────────┐
                                    │    CAMPAIGN    │
                                    │  CampaignBase  │
                                    └────────┬────────┘
                                             │
                        ┌────────────────────┼────────────────────┐
                        │                    │                    │
                        ▼                    ▼                    ▼
                 ┌────────────┐      ┌────────────┐       ┌────────────┐
                 │    LEAD     │      │    LEAD    │       │    LEAD    │
                 │  LeadBase  │      │  LeadBase  │       │  LeadBase  │
                 └─────┬───────┘      └─────┬───────┘       └─────┬───────┘
                       │                    │                    │
           ┌───────────┴───────────┐       │           ┌─────────┴─────────┐
           │                         │       │           │                   │
           ▼                         ▼       ▼           ▼                   ▼
    ┌────────────┐          ┌────────────┐ ┌──────────┐ ┌──────────────┐ ┌─────────────┐
    │   ACCOUNT   │          │  CONTACT   │ │  CONTACT │ │  OPPORTUNITY │ │  OPPORTUNITY│
    │ AccountBase │◀─────────│ ContactBase│ │ContactBs│ │OpportunityBs │ │ Opportunity │
    └──────┬──────┘          └──────┬─────┘ └──────────┘ └──────┬───────┘ └──────┬──────┘
           │                        │                            │                │
           │                        │                            │                │
           │                        │                            │                │
           ▼                        ▼                            ▼                ▼
    ┌────────────────────────────────────────────────────────────────────────────────┐
    │                            OPPORTUNITY                                         │
    │                          OpportunityBase                                        │
    │                                                                                 │
    │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐│
    │  │   QUOTE      │  │   QUOTE     │  │   ORDER    │  │  OPPORTUNITY PRODUCT   ││
    │  │  QuoteBase   │  │ QuoteBase   │  │SalesOrderBs│  │OpportunityProductBase   ││
    │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └───────────┬─────────────┘│
    └─────────┼────────────────┼────────────────┼─────────────────────┼──────────────┘
              │                │                │                     │
              │                │                │                     │
              ▼                ▼                ▼                     ▼
       ┌────────────┐    ┌────────────┐  ┌────────────┐       ┌──────────────────────┐
       │QUOTE DETAIL│    │   ORDER    │  │   ORDER   │       │      PRODUCT         │
       │QuoteDetail │    │ SalesOrder │  │SalesOrder │       │    ProductBase       │
       └─────┬──────┘    └──────┬─────┘  └─────┬──────┘       └──────────┬───────────┘
             │                  │              │                          │
             │                  │              │                          │
             ▼                  ▼              ▼                          ▼
       ┌────────────┐    ┌────────────┐  ┌────────────┐       ┌──────────────────────┐
       │   PRODUCT   │    │  INVOICE   │  │INVOICE DET │       │  PRICE LIST ITEM     │
       │  ProductBase│    │ InvoiceBase│  │InvoiceDetai│       │ProductPriceLevelBase│
       └─────────────┘    └──────┬─────┘  └────────────┘       └──────────┬───────────┘
                                 │                                       │
                                 │                                       ▼
                                 │                              ┌───────────────┐
                                 │                              │   PRICE LIST  │
                                 └─────────────────────────────▶│  PriceLevelBs │
                                                                 └───────────────┘
```

### 5.2 Cascade Delete Rules

The database implements cascade delete rules through the `fn_CollectForCascadeDelete` function:

| Parent Entity | Child Entity | Cascade Action |
|--------------|--------------|----------------|
| Account | Contact | Cascade (Delete) |
| Account | Opportunity | Cascade |
| Account | Quote | Cascade |
| Account | Order | Cascade |
| Account | Invoice | Cascade |
| Account | Case (Incident) | Cascade |
| Account | Child Account | Cascade |
| Contact | Opportunity | Cascade |
| Contact | Quote | Cascade |
| Contact | Order | Cascade |
| Contact | Invoice | Cascade |
| Lead | Contact (from qualification) | Remove Link |
| Lead | Account (from qualification) | Remove Link |
| Lead | Opportunity (from conversion) | Remove Link |
| Opportunity | Quote | Cascade |
| Opportunity | Order | Cascade |
| Opportunity | Invoice | Cascade |
| Quote | QuoteDetail | Cascade |
| Order | OrderDetail | Cascade |
| Order | Invoice | Cascade |
| Invoice | InvoiceDetail | Cascade |
| Product | ProductPriceLevel | Remove Link |
| PriceLevel | ProductPriceLevel | Remove Link |

### 5.3 Polymorphic Relationships

Some entities use polymorphic relationships where a single field can reference multiple entity types:

**Example: CustomerId**

```sql
-- CustomerId can reference Account (CustomerIdType = 1) or Contact (CustomerIdType = 2)

-- Query for all opportunities with Account customers:
SELECT o.Name, a.Name as AccountName
FROM OpportunityBase o
LEFT JOIN AccountBase a ON o.CustomerId = a.AccountId
WHERE o.CustomerIdType = 1

-- Query for all opportunities with Contact customers:
SELECT o.Name, c.FullName as ContactName
FROM OpportunityBase o
LEFT JOIN ContactBase c ON o.CustomerId = c.ContactId
WHERE o.CustomerIdType = 2
```

---

## 6. Transaction & Pricing Flow

### 6.1 Currency Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          CURRENCY PRICING FLOW                                  │
└─────────────────────────────────────────────────────────────────────────────────┘

                          TransactionCurrencyBase
                                │
                    ┌───────────┴───────────┐
                    │                       │
                    ▼                       ▼
            PriceLevelBase           OpportunityBase
            (PriceLevelId)           (TransactionCurrencyId)
                    │                       │
                    │                       │
                    ▼                       ▼
            ProductPriceLevelBase     QuoteBase
                    │                   (TransactionCurrencyId)
                    │                       │
                    │                       │
                    ▼                       ▼
            ProductBase ◀───────────── OrderDetailBase
                                        (UnitPrice_Base)
                                        
    Exchange Rate Flow:
    ┌────────────────────────────────────────────────────────────────────┐
    │ TransactionCurrencyId: Currency of the transaction                │
    │ ExchangeRate: Rate to convert to base currency                    │
    │ _Base fields: Values in base currency                             │
    └────────────────────────────────────────────────────────────────────┘
    
    Example:
    - Transaction in USD (TransactionCurrencyId = USD)
    - Base currency is CAD
    - ExchangeRate = 1.25 (1 USD = 1.25 CAD)
    - Quote.TotalAmount = $1,000 USD
    - Quote.TotalAmount_Base = $1,250 CAD
```

### 6.2 Price Determination Flow

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        PRICE DETERMINATION PROCESS                              │
└─────────────────────────────────────────────────────────────────────────────────┘

    1. Customer selects Price List (PriceLevel)
       └─▶ Quote.PriceLevelId = PriceLevelId
       
    2. For each product line item:
       a. Look up price in ProductPriceLevel
          └─▶ WHERE ProductId = LineItem.ProductId
              AND PriceLevelId = Quote.PriceLevelId
              
       b. Get Unit Price
          └─▶ UnitPrice = ProductPriceLevel.Amount
          
       c. Apply quantity discount (if configured)
          └─▶ ExtendedAmount = UnitPrice × Quantity
          
    3. Calculate header totals
       ├─▶ SubTotal = SUM(ExtendedAmount)
       ├─▶ DiscountAmount = SubTotal × DiscountPercentage
       │                      OR
       │                      Fixed DiscountAmount
       ├─▶ Tax = (SubTotal - Discount) × TaxRate
       ├─▶ Freight = Fixed FreightAmount
       └─▶ TotalAmount = SubTotal - Discount + Tax + Freight

    SQL Calculation:
    ┌────────────────────────────────────────────────────────────────┐
    │ UPDATE QuoteBase SET                                          │
    │     SubTotal = (SELECT SUM(ExtendedAmount)                   │
    │                  FROM QuoteDetailBase                         │
    │                  WHERE QuoteDetailBase.QuoteId = QuoteBase.QuoteId),|
    │     TotalAmount = SubTotal - ISNULL(DiscountAmount, 0)      │
    │                  + ISNULL(Tax, 0) + ISNULL(FreightAmount, 0) │
    │ WHERE QuoteId = @QuoteId                                      │
    └────────────────────────────────────────────────────────────────┘
```

---

## 7. Security & Ownership Model

### 7.1 Ownership Architecture

Every record in Dynamics 365 has an owner:

```sql
-- Ownership fields on every record
[OwnerId] [uniqueidentifier] NOT NULL,        -- User or Team
[OwnerIdType] [int] NOT NULL,                  -- 8 = User, 9 = Team
[OwningBusinessUnit] [uniqueidentifier] NULL   -- Business Unit hierarchy
```

**Ownership Type Codes:**

| OwnerIdType | Owner Type | Description |
|-------------|------------|-------------|
| 8 | SystemUser | Individual user |
| 9 | Team | Team-based ownership |

### 7.2 Security Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        SECURITY HIERARCHY                                       │
└─────────────────────────────────────────────────────────────────────────────────┘

                         Organization
                              │
                    ┌─────────┴─────────┐
                    │                   │
              Business Unit 1      Business Unit 2
                    │                   │
           ┌────────┴────────┐   ┌────┴────────┐
           │                 │   │             │
      User A            Team A      User B    Team B
      (Owner)           (Owner)    (Owner)    (Owner)
           │                 │   │             │
           └────────┬────────┘   └────┬────────┘
                    │                   │
                    ▼                   ▼
              Records              Records
              (OwnedBy)            (OwnedBy)
```

### 7.3 Privilege Depth Levels

| Depth | Value | Access Scope |
|-------|-------|--------------|
| None | 0 | No access |
| Basic | 1 | Own records only |
| Local | 2 | Business Unit |
| Deep | 4 | Business Unit + Children |
| Global | 8 | Entire Organization |

### 7.4 Field-Level Security

Field-level security is controlled through the `PrincipalObjectAttributeAccess` table:

```sql
-- Field-level security table
CREATE TABLE [dbo].[PrincipalObjectAttributeAccess](
    [AttributeId] [uniqueidentifier] NOT NULL,
    [ObjectId] [uniqueidentifier] NOT NULL,      -- Field (attribute) GUID
    [PrincipalId] [uniqueidentifier] NOT NULL,   -- User or Team
    [PrincipalTypeCode] [int] NOT NULL,          -- 8=User, 9=Team
    [AccessRightsMask] [int] NOT NULL,           -- 1=Read, 2=Create, 4=Update
    [InitializedOn] [datetime] NULL,
    [ModifiedOn] [datetime] NULL,
    [ModifiedBy] [uniqueidentifier] NULL
)
```

---

## 8. Data Integrity & Validation

### 8.1 Standard Audit Fields

All entities include audit tracking:

```sql
-- Creation tracking
[CreatedOn] [datetime] NULL,              -- Record creation timestamp
[CreatedBy] [uniqueidentifier] NULL,      -- User who created
[CreatedOnBehalfBy] [uniqueidentifier] NULL, -- Delegate creation

-- Modification tracking
[ModifiedOn] [datetime] NULL,            -- Last modification timestamp
[ModifiedBy] [uniqueidentifier] NULL,     -- User who modified
[ModifiedOnBehalfBy] [uniqueidentifier] NULL, -- Delegate modification

-- Concurrency control
[VersionNumber] [timestamp] NULL          -- Optimistic locking
```

### 8.2 Required Field Validation

| Entity | Required Fields |
|--------|-----------------|
| Account | Name |
| Contact | LastName (or FirstName) |
| Lead | Subject, LastName (B2C) OR CompanyName (B2B) |
| Opportunity | CustomerId, Name |
| Quote | CustomerId |
| Sales Order | CustomerId, PriceLevelId |
| Invoice | CustomerId, InvoiceDate |
| Product | Name, ProductNumber |

### 8.3 State Code Constraints

All entities use a standard state/status model:

```sql
-- StateCode: Overall status (always required, NOT NULL)
-- StatusCode: Specific reason (nullable, depends on StateCode)

-- Example: Opportunity
StateCode:
  0 = Open
  1 = Won
  2 = Lost

StatusCode (depends on StateCode):
  StateCode = 0: 1=New, 2=In Progress, 3=On Hold
  StateCode = 1: 6=Won
  StateCode = 2: 7=Cancelled, 8=Out-Sold, 9=Did not Budget, 10=Lost to Competition
```

---

## 9. Performance Considerations

### 9.1 Primary Key Strategy

**Current Implementation:**
- All tables use `uniqueidentifier` (GUID) as primary key
- Clustered index on GUID causes random page inserts
- Can lead to page fragmentation

**Recommendation:**
```sql
-- For new tables, consider using NEWSEQUENTIALID()
-- This generates sequential GUIDs reducing fragmentation

CREATE TABLE NewEntityBase(
    [NewEntityId] [uniqueidentifier] NOT NULL DEFAULT NEWSEQUENTIALID(),
    ...
    CONSTRAINT [PK_NewEntity] PRIMARY KEY CLUSTERED (NewEntityId)
)
```

### 9.2 Common Query Patterns

**Account Hierarchy Query:**
```sql
-- Get all accounts in hierarchy
WITH AccountHierarchy AS (
    SELECT AccountId, ParentAccountId, Name, 1 AS Level
    FROM AccountBase
    WHERE ParentAccountId IS NULL
    
    UNION ALL
    
    SELECT a.AccountId, a.ParentAccountId, a.Name, ah.Level + 1
    FROM AccountBase a
    INNER JOIN AccountHierarchy ah ON a.ParentAccountId = ah.AccountId
)
SELECT * FROM AccountHierarchy
```

**Opportunity Pipeline Query:**
```sql
-- Calculate pipeline value by stage
SELECT 
    StepName,
    COUNT(*) AS OpportunityCount,
    SUM(EstimatedValue) AS TotalValue,
    AVG(Probability) AS AvgProbability
FROM OpportunityBase
WHERE StateCode = 0  -- Open opportunities
GROUP BY StepName
ORDER BY SUM(EstimatedValue) DESC
```

### 9.3 Recommended Indexes

```sql
-- Account search index
CREATE NONCLUSTERED INDEX IX_Account_Search 
ON AccountBase(Name, AccountId, OwnerId, StateCode)
INCLUDE (AccountNumber, EMailAddress1)

-- Opportunity pipeline index
CREATE NONCLUSTERED INDEX IX_Opportunity_Pipeline 
ON OpportunityBase(StateCode, OwnerId, StepId)
INCLUDE (EstimatedValue, ActualValue, CustomerId)

-- Lead by source index
CREATE NONCLUSTERED INDEX IX_Lead_Source 
ON LeadBase(LeadSourceCode, StateCode, OwnerId)
INCLUDE (EstimatedAmount, CreatedOn)

-- Filtered index for active records
CREATE NONCLUSTERED INDEX IX_Account_Active 
ON AccountBase(StateCode, OwnerId)
WHERE StateCode = 0
```

---

## 10. Custom Entities

### 10.1 Custom Entity Naming Convention

Custom entities follow the pattern:
- **Table Name:** `EntityNameBase` (standard)
- **Entity Name:** `new_entityname` or `canven_entityname`

### 10.2 Identified Custom Entities

Based on database analysis:

| Entity Name | Table | Description |
|-------------|-------|-------------|
| Onboarding | canven_onboarding | Customer onboarding process |
| Property Onboard | canven_propertyonboard | Property onboarding |
| Customer Complaint | new_customercomplaint | Customer complaint tracking |

### 10.3 Custom Entity Schema Pattern

```sql
-- Custom entity example structure
CREATE TABLE [dbo].[canven_onboardingBase](
    [canven_onboardingId] [uniqueidentifier] NOT NULL,
    
    -- Custom fields
    [canven_Name] [nvarchar](100) NULL,
    [canven_CustomerId] [uniqueidentifier] NULL,
    [canven_Status] [int] NULL,
    [canven_StartDate] [datetime] NULL,
    [canven_EndDate] [datetime] NULL,
    [canven_AssignedTo] [uniqueidentifier] NULL,
    
    -- Standard fields (required)
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    [StateCode] [int] NOT NULL,
    [StatusCode] [int] NULL,
    [CreatedOn] [datetime] NULL,
    [CreatedBy] [uniqueidentifier] NULL,
    [ModifiedOn] [datetime] NULL,
    [ModifiedBy] [uniqueidentifier] NULL,
    [VersionNumber] [timestamp] NULL,
    
    CONSTRAINT [PK_canven_onboarding] 
    PRIMARY KEY CLUSTERED (canven_onboardingId)
)
```

---

## Appendix A: Object Type Codes Reference

| Code | Entity | Description |
|------|--------|-------------|
| 1 | Account | Business account |
| 2 | Contact | Individual contact |
| 3 | Opportunity | Sales opportunity |
| 4 | Lead | Sales lead |
| 5 | Annotation | Note/attachment |
| 8 | SystemUser | User |
| 9 | Team | Team |
| 10 | BusinessUnit | Business unit |
| 112 | Incident | Service case |
| 1001 | Email | Email activity |
| 1083 | OpportunityProduct | Opportunity line item |
| 1084 | Quote | Sales quotation |
| 1085 | QuoteDetail | Quote line item |
| 1088 | SalesOrder | Sales order |
| 1089 | SalesOrderDetail | Order line item |
| 1090 | Invoice | Invoice |
| 1091 | InvoiceDetail | Invoice line item |
| 1022 | PriceLevel | Price list |
| 1024 | Product | Product/service |
| 1026 | ProductPriceLevel | Product price |
| 1010 | Contract | Service contract |
| 4400 | Campaign | Marketing campaign |

---

## Appendix B: Quick Reference

### StateCode Values

| Entity | 0 | 1 | 2 | 3 | 4 |
|--------|---|---|---|---|---|
| Account | Active | Inactive | - | - | - |
| Contact | Active | Inactive | - | - | - |
| Lead | Open | Qualified | Disqualified | - | - |
| Opportunity | Open | Won | Lost | - | - |
| Quote | Open | Won | Lost | - | - |
| SalesOrder | Active | Submitted | Cancelled | Fulfilled | Invoiced |
| Invoice | Active | Paid | Cancelled | - | - |
| Product | Active | Retired | - | - | - |
| PriceLevel | Active | Inactive | - | - | - |

### OwnerIdType Values

| Value | Owner Type |
|-------|------------|
| 8 | User |
| 9 | Team |

---

**Document Version:** 1.0  
**Last Updated:** February 2026  
**Database:** CANCRM_MSCRM  
**Platform:** Microsoft Dynamics 365 / SQL Server

---

*This documentation provides a comprehensive technical reference for understanding the Microsoft Dynamics 365 CRM Sales Module database architecture and data flow patterns.*
