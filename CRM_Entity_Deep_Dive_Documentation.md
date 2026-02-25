# Microsoft Dynamics 365 CRM - Deep Entity & Data Flow Technical Documentation

## Document Information
- **Database Name**: CANCRM_MSCRM
- **Document Type**: Entity-Level Technical Architecture
- **Focus**: Comprehensive Deep Dive per Entity

---

## Table of Contents
1. [Account Entity Deep Dive](#1-account-entity-deep-dive)
2. [Contact Entity Deep Dive](#2-contact-entity-deep-dive)
3. [Lead Entity Deep Dive](#3-lead-entity-deep-dive)
4. [Opportunity Entity Deep Dive](#4-opportunity-entity-deep-dive)
5. [Quote, Sales Order & Invoice Deep Dive](#5-quote-sales-order--invoice-deep-dive)
6. [Product & Pricing Entity Deep Dive](#6-product--pricing-entity-deep-dive)
7. [Service Entities Deep Dive](#7-service-entities-deep-dive)
8. [Custom Entities Deep Dive](#8-custom-entities-deep-dive)
9. [Activity Entities Deep Dive](#9-activity-entities-deep-dive)
10. [Data Flow & Integration Mapping](#10-data-flow--integration-mapping)

---

## 1. ACCOUNT ENTITY DEEP DIVE

### 1.1 Entity Overview

| Attribute | Value |
|-----------|-------|
| **Entity Name** | Account |
| **Object Type Code** | 1 |
| **Primary Table** | AccountBase |
| **Filtered View** | FilteredAccount |
| **Ownership Type** | User/Team |
| **Entity Category** | Customer |

### 1.2 Table Structure: AccountBase

```sql
CREATE TABLE [dbo].[AccountBase](
    [AccountId] [uniqueidentifier] NOT NULL,          -- Primary Key
    [AccountCategoryCode] [int] NULL,                -- Category (e.g., Preferred Customer)
    [AccountClassificationCode] [int] NULL,           -- Classification
    [AccountId] [uniqueidentifier] NOT NULL,
    [AccountNumber] [nvarchar](20) NULL,             -- Customer Account Number
    [Address1_AddressId] [uniqueidentifier] NULL,     -- Address ID
    [Address1_AddressTypeCode] [int] NULL,           -- Address Type (Billing/Shipping)
    [Address1_City] [nvarchar](80) NULL,
    [Address1_Country] [nvarchar](80) NULL,
    [Address1_Fax] [nvarchar](50) NULL,
    [Address1_Latitude] [decimal](18, 0) NULL,
    [Address1_Line1] [nvarchar](250) NULL,
    [Address1_Line2] [nvarchar](250) NULL,
    [Address1_Line3] [nvarchar](250) NULL,
    [Address1_Longitude] [decimal](18, 0) NULL,
    [Address1_Name] [nvarchar](100) NULL,
    [Address1_PostalCode] [nvarchar](20) NULL,
    [Address1_PostOfficeBox] [nvarchar](20) NULL,
    [Address1_ShippingMethodCode] [int] NULL,
    [Address1_StateOrProvince] [nvarchar](50) NULL,
    [Address1_Telephone1] [nvarchar](50) NULL,
    [Address1_Telephone2] [nvarchar](50) NULL,
    [Address1_Telephone3] [nvarchar](50) NULL,
    [Address1_UPSZone] [nvarchar](4) NULL,
    [Address1_UTCConversionTimeZoneCode] [int] NULL,
    [Address2_AddressId] [uniqueidentifier] NULL,
    [Address2_AddressTypeCode] [int] NULL,
    [Address2_City] [nvarchar](80) NULL,
    [Address2_Country] [nvarchar](80) NULL,
    [BusinessTypeCode] [int] NULL,                   -- Business Type
    [CreatedOn] [datetime] NULL,
    [CreditLimit] [money] NULL,                       -- Credit Limit
    [CustomerSizeCode] [int] NULL,                   -- Company Size
    [CustomerTypeCode] [int] NULL,                   -- Account Type
    [Description] [nvarchar](max) NULL,
    [EMailAddress1] [nvarchar](100) NULL,
    [EMailAddress2] [nvarchar](100) NULL,
    [EMailAddress3] [nvarchar](100) NULL,
    [ExchangeRate] [decimal](23, 10) NULL,
    [Fax] [nvarchar](50) NULL,
    [FollowUpDate] [datetime] NULL,
    [IndustryCode] [int] NULL,                        -- Industry Classification
    [LastUsedInCampaign] [datetime] NULL,
    [MarketCap] [money] NULL,                         -- Market Capitalization
    [MasterId] [uniqueidentifier] NULL,               -- Master Record (for duplicate)
    [ModifiedOn] [datetime] NULL,
    [Name] [nvarchar](160) NOT NULL,                 -- Account Name
    [NumberOfEmployees] [int] NULL,                   -- Employee Count
    [OnHoldTime] [int] NULL,
    [OperatingHoursId] [uniqueidentifier] NULL,
    [OriginatingLeadId] [uniqueidentifier] NULL,      -- Originating Lead
    [OwnerId] [uniqueidentifier] NOT NULL,            -- Owner
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    [ParentAccountId] [uniqueidentifier] NULL,        -- Parent Account
    [PaymentTermsCode] [int] NULL,                    -- Payment Terms
    [PreferredContactMethodCode] [int] NULL,
    [PrimaryContactId] [uniqueidentifier] NULL,       -- Primary Contact
    [PrimarySatoriId] [nvarchar](200) NULL,
    [PrimaryTwitterId] [nvarchar](128) NULL,
    [Revenue] [money] NULL,                            -- Annual Revenue
    [SharesOutstanding] [int] NULL,
    [ShippingMethodCode] [int] NULL,
    [SIC] [nvarchar](20) NULL,                        -- SIC Code
    [StateCode] [int] NOT NULL,                       -- Status (0=Active, 1=Inactive)
    [StatusCode] [int] NULL,                          -- Status Reason
    [TerritoryId] [uniqueidentifier] NULL,           -- Sales Territory
    [TickerSymbol] [nvarchar](10) NULL,               -- Stock Ticker
    [TransactionCurrencyId] [uniqueidentifier] NULL,  -- Currency
    [WebsiteUrl] [nvarchar](200) NULL,
    [VersionNumber] [timestamp] NULL
)
```

### 1.3 Field Classification

| Field Category | Fields | Description |
|----------------|--------|-------------|
| **Identification** | AccountId, AccountNumber, Name | Core identifiers |
| **Address** | Address1_*, Address2_* | Multiple address support |
| **Business Info** | IndustryCode, NumberOfEmployees, Revenue, SIC | Company demographics |
| **Financial** | CreditLimit, Revenue, MarketCap, TransactionCurrencyId | Financial data |
| **Classification** | AccountCategoryCode, CustomerTypeCode, CustomerSizeCode | Categorization |
| **Ownership** | OwnerId, OwnerIdType, OwningBusinessUnit | Security/Ownership |
| **Relationships** | ParentAccountId, PrimaryContactId, OriginatingLeadId | Entity relationships |
| **Audit** | CreatedOn, ModifiedOn, VersionNumber | Change tracking |

### 1.4 Business Rules & Validation

| Rule | Implementation | Notes |
|------|---------------|-------|
| Name Required | NOT NULL | Account name is mandatory |
| Owner Required | NOT NULL | Every record must have owner |
| State/Status Pair | Application Logic | StatusCode must match StateCode |
| Currency Required | Application Logic | For monetary fields |
| Unique AccountNumber | Unique Constraint | If configured |

### 1.5 Entity Relationships

```
Account
├── 1:N → Contact (ParentCustomerId)
├── 1:N → Opportunity (CustomerId)
├── 1:N → Lead (CustomerId)
├── 1:N → Quote (CustomerId)
├── 1:N → SalesOrder (CustomerId)
├── 1:N → Invoice (CustomerId)
├── 1:N → Contract (CustomerId)
├── 1:N → Incident (CustomerId)
├── 1:N → Activity (RegardingObjectId)
├── N:N → Lead (AccountLeads)
├── 1:1 → AccountExtensionBase (custom fields)
└── 1:N → Child Account (ParentAccountId)
```

### 1.6 Data Flow Into/Out of Account

| Flow Type | Source/Destination | Trigger |
|-----------|-------------------|---------|
| **IN** | Lead Qualification | Convert Lead → Create Account |
| **IN** | Manual Entry | User creates account |
| **IN** | Import | Data import wizard |
| **IN** | Integration | Dynamics 365 integration |
| **OUT** | Account → Contact | Create associated contact |
| **OUT** | Account → Opportunity | Create opportunity from account |
| **OUT** | Account → Case | Create incident from account |
| **OUT** | Duplicate Detection | Match to existing records |

---

## 2. CONTACT ENTITY DEEP DIVE

### 2.1 Entity Overview

| Attribute | Value |
|-----------|-------|
| **Entity Name** | Contact |
| **Object Type Code** | 2 |
| **Primary Table** | ContactBase |
| **Filtered View** | FilteredContact |
| **Ownership Type** | User/Team |
| **Entity Category** | Customer |

### 2.2 Table Structure: ContactBase

```sql
CREATE TABLE [dbo].[ContactBase](
    [ContactId] [uniqueidentifier] NOT NULL,
    [AccountId] [uniqueidentifier] NULL,              -- Associated Account
    [AccountRoleCode] [int] NULL,                     -- Role in Account
    [Address1_AddressId] [uniqueidentifier] NULL,
    [Address1_AddressTypeCode] [int] NULL,
    [Address1_City] [nvarchar](80) NULL,
    [Address1_Country] [nvarchar](80) NULL,
    [Address1_Fax] [nvarchar](50) NULL,
    [Address1_Line1] [nvarchar](250) NULL,
    [Address1_Line2] [nvarchar](250) NULL,
    [Address1_Line3] [nvarchar](250) NULL,
    [Address1_Name] [nvarchar](100) NULL,
    [Address1_PostalCode] [nvarchar](20) NULL,
    [Address1_StateOrProvince] [nvarchar](50) NULL,
    [Address1_Telephone1] [nvarchar](50) NULL,
    [Address1_Telephone2] [nvarchar](50) NULL,
    [Address2_AddressId] [uniqueidentifier] NULL,
    [Address2_AddressTypeCode] [int] NULL,
    [AssistantId] [uniqueidentifier] NULL,            -- Assistant
    [BirthDate] [datetime] NULL,                       -- Date of Birth
    [BusinessCard] [varbinary](max) NULL,
    [BusinessPhone] [nvarchar](50) NULL,
    [Callback] [nvarchar](50) NULL,
    [Company] [nvarchar](100) NULL,
    [CreatedOn] [datetime] NULL,
    [CreditLimit] [money] NULL,
    [CustomerSizeCode] [int] NULL,
    [CustomerTypeCode] [int] NULL,
    [Department] [nvarchar](100) NULL,
    [Description] [nvarchar](max) NULL,
    [DoNotBulkEMail] [bit] NULL,                       -- Do Not Send Bulk Email
    [DoNotEMail] [bit] NULL,                          -- Do Not Email
    [DoNotFax] [bit] NULL,                            -- Do Not Fax
    [DoNotPhone] [bit] NULL,                          -- Do Not Call
    [DoNotPostalMail] [bit] NULL,                     -- Do Not Mail
    [EMailAddress1] [nvarchar](100) NULL,
    [EMailAddress2] [nvarchar](100) NULL,
    [EmployeeId] [nvarchar](50) NULL,                 -- Employee ID
    [EntityImageId] [uniqueidentifier] NULL,
    [ExchangeRate] [decimal](23, 10) NULL,
    [ExternalUserIdentifier] [nvarchar](max) NULL,
    [FamilyStatusCode] [int] NULL,                    -- Marital Status
    [Fax] [nvarchar](50) NULL,
    [FirstName] [nvarchar](50) NULL,
    [FollowUpDate] [datetime] NULL,
    [FullName] [nvarchar](160) NULL,                  -- Computed: First + Middle + Last
    [GenderCode] [int] NULL,
    [GovernmentId] [nvarchar](50) NULL,
    [HasChildrenCode] [int] NULL,
    [HomePhone] [nvarchar](50) NULL,
    [ImportSequenceNumber] [int] NULL,
    [IsAutoCreate] [bit] NULL,
    [JobTitle] [nvarchar](100) NULL,
    [LastName] [nvarchar](50) NULL,                   -- Last Name
    [LeadSourceCode] [int] NULL,
    [_MANAGED] [bit] NULL,
    [MarketingOnlyId] [uniqueidentifier] NULL,
    [MasterId] [uniqueidentifier] NULL,
    [MiddleName] [nvarchar](50) NULL,
    [MobilePhone] [nvarchar](50) NULL,
    [ModifiedOn] [datetime] NULL,
    [NickName] [nvarchar](50) NULL,
    [NumberOfChildren] [int] NULL,
    [OnHoldTime] [int] NULL,
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    [Pager] [nvarchar](50) NULL,
    [ParentContactId] [uniqueidentifier] NULL,        -- Parent Contact
    [ParentCustomerId] [uniqueidentifier] NULL,        -- Parent Account/Contact
    [ParentCustomerIdType] [int] NULL,
    [PaymentTermsCode] [int] NULL,
    [PreferredContactMethodCode] [int] NULL,
    [PreferredEquipmentId] [uniqueidentifier] NULL,
    [PreferredServiceId] [uniqueidentifier] NULL,
    [PreferredTimeZoneCode] [int] NULL,
    [PrimarySatoriId] [nvarchar](200) NULL,
    [PrimaryTwitterId] [nvarchar](128) NULL,
    [ProcessId] [uniqueidentifier] NULL,
    [PurchasingLevelCode] [int] NULL,
    [Revenue] [money] NULL,
    [Salutation] [nvarchar](100) NULL,                -- Salutation (Mr./Ms./Dr.)
    [ShippingMethodCode] [int] NULL,
    [SIC] [nvarchar](20) NULL,
    [SpouseName] [nvarchar](100) NULL,
    [StateCode] [int] NOT NULL,
    [StatusCode] [int] NULL,
    [Suffix] [nvarchar](50) NULL,                      -- Suffix (Jr./Sr.)
    [Telephone1] [nvarchar](50) NULL,
    [Telephone2] [nvarchar](50) NULL,
    [Telephone3] [nvarchar](50) NULL,
    [TerritoryId] [uniqueidentifier] NULL,
    [TimeZoneRuleVersionNumber] [int] NULL,
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [TraversedPath] [nvarchar](max) NULL,
    [UTCConversionTimeZoneCode] [int] NULL,
    [VersionNumber] [timestamp] NULL,
    [WebSiteUrl] [nvarchar](200) NULL,
    [YomiFirstName] [nvarchar](50) NULL,               -- Phonetic First Name
    [YomiFullName] [nvarchar](160) NULL,              -- Phonetic Full Name
    [YomiLastName] [nvarchar](50) NULL                 -- Phonetic Last Name
)
```

### 2.3 Field Classification

| Field Category | Fields | Purpose |
|----------------|--------|---------|
| **Personal Info** | FirstName, MiddleName, LastName, FullName, BirthDate, GenderCode, FamilyStatusCode | Personal demographics |
| **Contact Methods** | EMailAddress1/2/3, Phone, MobilePhone, Fax, Pager | Communication |
| **Professional** | JobTitle, Department, Company, EmployeeId | Employment info |
| **Address** | Address1_*, Address2_* | Multiple addresses |
| **Marketing Pref** | DoNotEMail, DoNotPhone, DoNotFax, DoNotPostalMail, DoNotBulkEMail | Consent management |
| **Preferences** | PreferredContactMethodCode, PreferredEquipmentId, PreferredServiceId | Contact preferences |
| **Ownership** | OwnerId, OwnerIdType, OwningBusinessUnit, ParentCustomerId | Security & hierarchy |
| **Audit** | CreatedOn, ModifiedOn, VersionNumber | Change tracking |

### 2.4 Business Rules

| Rule | Implementation |
|------|---------------|
| FullName Computation | FirstName + MiddleName + LastName (system calculated) |
| Yomi Fields | Phonetic readings for Japanese/Chinese names |
| Parent Customer | Can be Account or Contact (polymorphic) |
| Privacy Fields | DoNot* fields for CAN-SPAM/GDPR compliance |

### 2.5 Entity Relationships

```
Contact
├── N:1 → Account (AccountId) - if employed
├── 1:N → Contact (ParentContactId) - family hierarchy
├── 1:N → Opportunity (CustomerId)
├── 1:N → Lead (CustomerId)
├── 1:N → Quote (CustomerId)
├── 1:N → SalesOrder (CustomerId)
├── 1:N → Invoice (CustomerId)
├── N:N → Account (ContactLeads)
├── N:N → Lead (ContactLeads)
├── 1:N → Activity (RegardingObjectId)
└── 1:1 → ContactExtensionBase
```

---

## 3. LEAD ENTITY DEEP DIVE

### 3.1 Entity Overview

| Attribute | Value |
|-----------|-------|
| **Entity Name** | Lead |
| **Object Type Code** | 4 |
| **Primary Table** | LeadBase |
| **Filtered View** | FilteredLead |
| **Ownership Type** | User/Team |
| **Entity Category** | Sales - Lead Management |

### 3.2 Table Structure: LeadBase

```sql
CREATE TABLE [dbo].[LeadBase](
    [LeadId] [uniqueidentifier] NOT NULL,
    [AccountId] [uniqueidentifier] NULL,              -- Created Account
    [AccountName] [nvarchar](160) NULL,
    [Address1_AddressId] [uniqueidentifier] NULL,
    [Address1_AddressTypeCode] [int] NULL,
    [Address1_City] [nvarchar](80) NULL,
    [Address1_Country] [nvarchar](80) NULL,
    [Address1_Fax] [nvarchar](50) NULL,
    [Address1_Line1] [nvarchar](250) NULL,
    [Address1_Line2] [nvarchar](250) NULL,
    [Address1_Line3] [nvarchar](250) NULL,
    [Address1_Name] [nvarchar](100) NULL,
    [Address1_PostalCode] [nvarchar](20) NULL,
    [Address1_StateOrProvince] [nvarchar](50) NULL,
    [Address1_Telephone1] [nvarchar](50) NULL,
    [Address1_Telephone2] [nvarchar](50) NULL,
    [BudgetAmount] [money] NULL,                      -- Budget
    [BudgetQuality] [int] NULL,                       -- Budget confirmed?
    [BusinessCard] [varbinary](max) NULL,
    [BusinessNeeds] [nvarchar](max) NULL,
    [BusinessTypeCode] [int] NULL,
    [CampaignId] [uniqueidentifier] NULL,             -- Source Campaign
    [City] [nvarchar](80) NULL,
    [CompanyName] [nvarchar](250) NULL,               -- Company Name
    [ConfirmInterest] [bit] NULL,
    [ContactId] [uniqueidentifier] NULL,              -- Created Contact
    [Country] [nvarchar](80) NULL,
    [CreatedOn] [datetime] NULL,
    [CustomerId] [uniqueidentifier] NULL,            -- Converted Customer
    [CustomerIdType] [int] NULL,
    [CustomerNeed] [nvarchar](max) NULL,
    [CustomerPainPoints] [nvarchar](max) NULL,
    [DecisionMaker] [bit] NULL,
    [Description] [nvarchar](max) NULL,
    [DoNotBulkEMail] [bit] NULL,
    [DoNotEMail] [bit] NULL,
    [DoNotFax] [bit] NULL,
    [DoNotPhone] [bit] NULL,
    [DoNotPostalMail] [bit] NULL,
    [EMail] [nvarchar](100) NULL,
    [EMailAddress1] [nvarchar](100) NULL,
    [EstimatedAmount] [money] NULL,                    -- Estimated Value
    [EstimatedCloseDate] [datetime] NULL,             -- Expected Close
    [EvaluateFit] [bit] NULL,                         -- Evaluate Fit
    [ExchangeRate] [decimal](23, 10) NULL,
    [Fax] [nvarchar](50) NULL,
    [FirstName] [nvarchar](50) NULL,
    [IndustryCode] [int] NULL,
    [InitialCommunication] [int] NULL,
    [IsAutoCreate] [bit] NULL,
    [JobTitle] [nvarchar](100) NULL,
    [LastName] [nvarchar](50) NULL,
    [LeadQualityCode] [int] NULL,                    -- Rating (Hot/Warm/Cold)
    [LeadSourceCode] [int] NULL,                      -- Lead Source
    [MobilePhone] [nvarchar](50) NULL,
    [ModifiedOn] [datetime] NULL,
    [Need] [int] NULL,
    [NumberOfEmployees] [int] NULL,
    [OpportunityId] [uniqueidentifier] NULL,          -- Created Opportunity
    [OpportunityName] [nvarchar](220) NULL,
    [OrganizationId] [uniqueidentifier] NULL,
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    [ParticipatesInWorkflow] [bit] NULL,
    [PreferredContactMethodCode] [int] NULL,
    [PriceLevelId] [uniqueidentifier] NULL,
    [PriorityCode] [int] NULL,                        -- Priority
    [ProcessId] [uniqueidentifier] NULL,
    [PurchaseTimeframe] [int] NULL,
    [PurchaseProcess] [int] NULL,
    [QualificationComment] [nvarchar](max) NULL,
    [RatingCode] [int] NULL,
    [RejectionComment] [nvarchar](max) NULL,
    [Revenue] [money] NULL,
    [SalesStage] [nvarchar](50) NULL,
    [SalesStageCode] [int] NULL,
    [SIC] [nvarchar](20) NULL,
    [StateCode] [int] NOT NULL,                       -- Status
    [StatusCode] [int] NULL,                          -- Status Reason
    [Subject] [nvarchar](300) NULL,                   -- Topic
    [Telephone1] [nvarchar](50) NULL,
    [TerritoryId] [uniqueidentifier] NULL,
    [TimeZoneRuleVersionNumber] [int] NULL,
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [TraversedPath] [nvarchar](max) NULL,
    [UTCConversionTimeZoneCode] [int] NULL,
    [VersionNumber] [timestamp] NULL,
    [WebSiteUrl] [nvarchar](200) NULL,
    [YomiCompanyName] [nvarchar](250) NULL,
    [YomiFirstName] [nvarchar](50) NULL,
    [YomiLastName] [nvarchar](50) NULL
)
```

### 3.3 Lead Qualification Process

```
Lead States:
┌─────────────────────────────────────────────────────────────────────────────┐
│  OPEN LEAD                                                                  │
│  ┌─────────┐    ┌──────────┐    ┌────────────┐    ┌────────────────────┐  │
│  │  New    │───▶│ Attempted│───▶│ Contacted  │───▶│ Qualified           │  │
│  │         │    │ to       │    │            │    │                     │  │
│  └─────────┘    └──────────┘    └────────────┘    └──────────┬───────────┘  │
│                                                             │               │
│                                                             ▼               │
│  ┌─────────────────────────────────────────────────────────────────────────┐
│  │                          DISQUALIFIED                                   │  │
│  │  ┌────────────┐    ┌────────────┐    ┌────────────┐    ┌────────────┐  │  │
│  │  │ Lost       │    │ Not        │    │ No        │    │ Cannot     │  │  │
│  │  │ Competition│    │ Contacted  │    │ Budget    │    │ Contact    │  │  │
│  │  └────────────┘    └────────────┘    └────────────┘    └────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────┘
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.4 Lead Qualification Output

When a Lead is Qualified, the system can create:

| Created Entity | Source Field | Notes |
|----------------|--------------|-------|
| **Account** | AccountName | If company specified |
| **Contact** | FirstName, LastName, EMail | If contact details provided |
| **Opportunity** | OpportunityName, EstimatedAmount, EstimatedCloseDate | Always created |

### 3.5 Key Lead Fields

| Field | Purpose | Business Logic |
|-------|---------|---------------|
| LeadQualityCode | Rating (Hot/Warm/Cold) | Determines sales priority |
| LeadSourceCode | Origin (Web, Referral, Campaign, etc.) | Marketing attribution |
| BudgetAmount | Customer's budget | Qualification criteria |
| DecisionMaker | Is decision maker? | Sales approach |
| EstimatedAmount | Potential deal value | Pipeline value |
| EstimatedCloseDate | Expected close | Forecasting |
| PurchaseTimeframe | When buying | Sales timing |
| IndustryCode | Industry classification | Segmentation |

### 3.6 Entity Relationships

```
Lead
├── 1:N → LeadAddress (ParentId) - multiple addresses
├── N:N → Account (AccountLeads) - account interests
├── N:N → Contact (ContactLeads) - contact interests
├── N:N → Competitor (LeadCompetitors) - known competitors
├── N:N → Product (LeadProduct) - interested products
├── 1:1 → Opportunity (on qualification)
├── 1:1 → Account (on qualification)
├── 1:1 → Contact (on qualification)
├── 1:N → CampaignResponse (RegardingObjectId)
└── N:N → Campaign (CampaignItem)
```

---

## 4. OPPORTUNITY ENTITY DEEP DIVE

### 4.1 Entity Overview

| Attribute | Value |
|-----------|-------|
| **Entity Name** | Opportunity |
| **Object Type Code** | 3 |
| **Primary Table** | OpportunityBase |
| **Filtered View** | FilteredOpportunity |
| **Ownership Type** | User/Team |
| **Entity Category** | Sales - Pipeline |

### 4.2 Table Structure: OpportunityBase

```sql
CREATE TABLE [dbo].[OpportunityBase](
    [OpportunityId] [uniqueidentifier] NOT NULL,
    [AccountId] [uniqueidentifier] NULL,              -- Customer Account
    [ActualCloseDate] [datetime] NULL,                -- Actual Close Date
    [ActualValue] [money] NULL,                        -- Actual Revenue
    [Amount] [money] NULL,                             -- Est. Revenue
    [BillingContactId] [uniqueidentifier] NULL,
    [CampaignId] [uniqueidentifier] NULL,              -- Source Campaign
    [Captured] [bit] NULL,
    [CloseProbability] [int] NULL,                    -- Probability %
    [ContactId] [uniqueidentifier] NULL,               -- Customer Contact
    [CreatedOn] [datetime] NULL,
    [CustomerId] [uniqueidentifier] NULL,              -- Customer (Account/Contact)
    [CustomerIdType] [int] NULL,                      -- 1=Account, 2=Contact
    [Description] [nvarchar](max) NULL,
    [DiscountAmount] [money] NULL,
    [DiscountPercentage] [decimal](18, 0) NULL,
    [EstimatedValue] [money] NULL,
    [ExchangeRate] [decimal](23, 10) NULL,
    [FinalDecisionDate] [datetime] NULL,
    [IsRevenueSystemCalculated] [bit] NULL,
    [LeadSourceCode] [int] NULL,
    [ModifiedOn] [datetime] NULL,
    [Name] [nvarchar](220) NULL,                       -- Opportunity Name
    [Need] [int] NULL,
    [OnHoldTime] [int] NULL,
    [OpportunityId] [uniqueidentifier] NOT NULL,
    [OpportunityRatingCode] [int] NULL,               -- Rating
    [OrderId] [uniqueidentifier] NULL,                -- Created Order
    [OriginatingLeadId] [uniqueidentifier] NULL,      -- Source Lead
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    [PriceLevelId] [uniqueidentifier] NULL,            -- Pricing Level
    [ProcessId] [uniqueidentifier] NULL,
    [PurchaseTimeframe] [int] NULL,
    [QuoteId] [uniqueidentifier] NULL,                -- Winning Quote
    [ResolveInfo] [nvarchar](max) NULL,
    [SalesStage] [nvarchar](50) NULL,                 -- Current Stage
    [SalesStageCode] [int] NULL,                      -- Stage Reason
    [ScheduleFollowup_Expr1] [datetime] NULL,
    [ScheduleMeeting_Expr1] [datetime] NULL,
    [SLAId] [uniqueidentifier] NULL,
    [SLAInvarianceId] [uniqueidentifier] NULL,
    [StageId] [uniqueidentifier] NULL,
    [StateCode] [int] NOT NULL,                       -- Status
    [StatusCode] [int] NULL,                          -- Status Reason
    [StepId] [uniqueidentifier] NULL,
    [StepName] [nvarchar](200) NULL,
    [TerritoryId] [uniqueidentifier] NULL,
    [Timeline] [int] NULL,
    [TotalAmount] [money] NULL,                       -- Total with products
    [TotalAmountLessFreight] [money] NULL,
    [TotalDiscountAmount] [money] NULL,
    [TotalLineItemAmount] [money] NULL,
    [TotalProductAmount] [money] NULL,
    [TotalTax] [money] NULL,
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [TraversedPath] [nvarchar](max) NULL,
    [UTCConversionTimeZoneCode] [int] NULL,
    [VersionNumber] [timestamp] NULL,
    [WillSendBonuses] [bit] NULL
)
```

### 4.3 Sales Process Flow

```
OPPORTUNITY LIFECYCLE
══════════════════════════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────────────────────────────┐
  │                           OPEN OPPORTUNITIES                            │
  └─────────────────────────────────────────────────────────────────────────┘

  Stage 1: Qualify ──────────▶ Stage 2: Develop ──────────▶ Stage 3: Propose
       (10-30%)                    (30-60%)                   (60-90%)
  ┌───────────────┐          ┌───────────────┐           ┌───────────────┐
  │ Identify      │          │ Build         │           │ Present      │────┐
  │ Customer      │─────────▶│ Solution      │─────────▶│ Proposal     │    │
  │ Needs         │          │ & Value       │          │              │    │
  └───────────────┘          └───────────────┘           └───────────────┘    │
                                                                        │    │
  ┌─────────────────────────────────────────────────────────────────────────┘    │
  │                                                                             │
  ▼                                                                             │
  Stage 4: Close ──────────────────────────────────────────────────────────────►│
       (100%)                                                                      │
  ┌───────────────┐                                                              │
  │ Negotiate     │──────▶  WIN or LOST                                         │
  │ & Contract    │         ┌─────────────┐                                     │
  └───────────────┘         │ Opportunity  │                                    │
                            │ Closed - Won │                                    │
                            └─────────────┘                                    │
```

### 4.4 Opportunity vs. OpportunityProduct Data Flow

```
OpportunityBase (Header Level)
│
├── [OpportunityId] - PK
├── [Name] - Opportunity name
├── [Amount] - Manual estimated value
├── [TotalLineItemAmount] - Sum of products (system calc)
├── [TotalDiscountAmount] - Line item + header discount
├── [TotalTax] - Tax calculation
├── [TotalAmount] - Final amount (TotalLineItemAmount + Discount + Tax)
│
└── 1:N → OpportunityProductBase (Line Items)
    │
    ├── [OpportunityProductId] - PK
    ├── [OpportunityId] - FK to Opportunity
    ├── [ProductId] - FK to Product
    ├── [UoMId] - Unit of Measure
    ├── [Quantity] - Quantity
    ├── [BaseAmount] - Unit Price × Quantity
    ├── [ExtendedAmount] - Amount - Discount
    ├── [PriceLevelId] - Pricing
    ├── [Tax] - Tax amount
    └── [IsPriceOverridden] - Manual price override?
```

### 4.5 Entity Relationships

```
Opportunity
├── N:1 → Account (AccountId)
├── N:1 → Contact (ContactId)
├── N:1 → TransactionCurrency (TransactionCurrencyId)
├── N:1 → PriceLevel (PriceLevelId)
├── N:1 → Lead (OriginatingLeadId)
├── N:1 → Campaign (CampaignId)
├── 1:N → OpportunityProduct (Products)
├── 1:N → OpportunityCompetitors (Competitors)
├── N:N → SalesLiterature (for sharing)
├── 1:N → Quote (OpportunityId)
├── 1:1 → SalesOrder (Closed Won)
├── 1:N → Activity (RegardingObjectId)
└── 1:1 → OpportunityExtensionBase
```

---

## 5. QUOTE, SALES ORDER & INVOICE DEEP DIVE

### 5.1 Quote Entity

| Attribute | Value |
|-----------|-------|
| **Entity Name** | Quote |
| **Object Type Code** | 1088 |
| **Primary Table** | QuoteBase |
| **Ownership Type** | User/Team |

#### QuoteBase Structure

```sql
CREATE TABLE [dbo].[QuoteBase](
    [QuoteId] [uniqueidentifier] NOT NULL,
    [AccountId] [uniqueidentifier] NULL,
    [BillTo_AddressId] [uniqueidentifier] NULL,
    [BillTo_City] [nvarchar](80) NULL,
    [BillTo_ContactName] [nvarchar](200) NULL,
    [BillTo_Country] [nvarchar](80) NULL,
    [BillTo_Fax] [nvarchar](50) NULL,
    [BillTo_Line1] [nvarchar](250) NULL,
    [BillTo_Line2] [nvarchar](250) NULL,
    [BillTo_Line3] [nvarchar](250) NULL,
    [BillTo_Name] [nvarchar](200) NULL,
    [BillTo_PostalCode] [nvarchar](20) NULL,
    [BillTo_StateOrProvince] [nvarchar](50) NULL,
    [BillTo_Telephone] [nvarchar](50) NULL,
    [CampaignId] [uniqueidentifier] NULL,
    [ClosedOn] [datetime] NULL,
    [ContactId] [uniqueidentifier] NULL,
    [CreatedOn] [datetime] NULL,
    [CustomerId] [uniqueidentifier] NULL,
    [CustomerIdType] [int] NULL,
    [Description] [nvarchar](max) NULL,
    [DiscountAmount] [money] NULL,
    [DiscountPercentage] [decimal](18, 0) NULL,
    [EffectiveFrom] [datetime] NULL,
    [EffectiveTo] [datetime] NULL,
    [ExchangeRate] [decimal](23, 10) NULL,
    [FreightTermCode] [int] NULL,
    [IsPriceLocked] [bit] NULL,
    [LastAppliedOn] [datetime] NULL,
    [ModifiedOn] [datetime] NULL,
    [Name] [nvarchar](200) NULL,                    -- Quote Name
    [OpportunityId] [uniqueidentifier] NULL,        -- Source Opportunity
    [OrderId] [uniqueidentifier] NULL,              -- Converted Order
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    [PaymentTermCode] [int] NULL,
    [PriceLevelId] [uniqueidentifier] NULL,
    [PricingErrorCode] [int] NULL,
    [ProcessId] [uniqueidentifier] NULL,
    [QuoteNumber] [nvarchar](20) NULL,               -- Auto-generated
    [RequestDeliveryBy] [datetime] NULL,
    [ShipTo_AddressId] [uniqueidentifier] NULL,
    [ShipTo_City] [nvarchar](80) NULL,
    [ShipTo_ContactName] [nvarchar](200) NULL,
    [ShipTo_Country] [nvarchar](80) NULL,
    [ShipTo_Fax] [nvarchar](50) NULL,
    [ShipTo_Line1] [nvarchar](250) NULL,
    [ShipTo_Line2] [nvarchar](250) NULL,
    [ShipTo_Line3] [nvarchar](250) NULL,
    [ShipTo_Name] [nvarchar](200) NULL,
    [ShipTo_PostalCode] [nvarchar](20) NULL,
    [ShipTo_StateOrProvince] [nvarchar](50) NULL,
    [ShipTo_Telephone] [nvarchar](50) NULL,
    [ShippingMethodCode] [int] NULL,
    [StateCode] [int] NOT NULL,                      -- Status
    [StatusCode] [int] NULL,                         -- Reason
    [TotalAmount] [money] NULL,                       -- Total with products/tax
    [TotalAmountLessFreight] [money] NULL,
    [TotalDiscountAmount] [money] NULL,
    [TotalLineItemAmount] [money] NULL,
    [TotalTax] [money] NULL,
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [TraversedPath] [nvarchar](max) NULL,
    [UTCConversionTimeZoneCode] [int] NULL,
    [VersionNumber] [timestamp] NULL,
    [WillCall] [bit] NULL
)
```

### 5.2 Quote Detail (Line Items)

```sql
CREATE TABLE [dbo].[QuoteDetailBase](
    [QuoteDetailId] [uniqueidentifier] NOT NULL,
    [BaseAmount] [money] NULL,                         -- Unit Price × Quantity
    [Description] [nvarchar](max) NULL,
    [ExtendedAmount] [money] NULL,                    -- Amount - Discount
    [InvoiceId] [uniqueidentifier] NULL,              -- Invoiced (if won)
    [IsPriceOverridden] [bit] NULL,                   -- Manual price?
    [LineItemNumber] [int] NULL,                      -- Line order
    [LotNumber] [nvarchar](100) NULL,
    [OpportunityProductId] [uniqueidentifier] NULL,  -- From Opportunity
    [PriceLevelId] [uniqueidentifier] NULL,
    [ProductId] [uniqueidentifier] NULL,             -- Product
    [ProductDescription] [nvarchar](max) NULL,        -- Description override
    [ProductName] [nvarchar](200) NULL,               -- Name override
    [Quantity] [decimal](18, 0) NULL,                  -- Quantity
    [QuoteId] [uniqueidentifier] NOT NULL,            -- Parent Quote
    [RequestDeliveryBy] [datetime] NULL,
    [SalesOrderDetailId] [uniqueidentifier] NULL,    -- Ordered (if won)
    [ShipTo_AddressId] [uniqueidentifier] NULL,
    [ShippingMethodCode] [int] NULL,
    [Tax] [money] NULL,
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [UoMId] [uniqueidentifier] NULL,                  -- Unit of Measure
    [VersionNumber] [timestamp] NULL,
    [VolumeDiscountAmount] [money] NULL               -- Volume discount
)
```

### 5.3 Sales Order Entity

| Attribute | Value |
|-----------|-------|
| **Entity Name** | SalesOrder (Order) |
| **Object Type Code** | 1088 |
| **Primary Table** | SalesOrderBase |

#### Order Status Flow

```
Order Status Lifecycle:
════════════════════════════════════════════════════════════════════

  Active (0) ──────────▶ Fulfilled (1) ──────────▶ Cancelled (2)
  ┌─────────────┐      ┌─────────────┐           ┌─────────────┐
  │ New Order   │      │ Products    │           │ Order       │
  │ Created     │─────▶│ Shipped     │           │ Cancelled   │
  └─────────────┘      └─────────────┘           └─────────────┘
```

### 5.4 Invoice Entity

| Attribute | Value |
|-----------|-------|
| **Entity Name** | Invoice |
| **Object Type Code** | 1090 |
| **Primary Table** | InvoiceBase |

#### Invoice Status Flow

```
Invoice Status Lifecycle:
════════════════════════════════════════════════════════════════════

  Active (0) ──────────▶ Paid (1) ──────────▶ Cancelled (2)
  ┌─────────────┐      ┌─────────────┐       ┌─────────────┐
  │ Invoice     │      │ Payment     │       │ Invoice     │
  │ Created     │─────▶│ Received    │       │ Cancelled   │
  └─────────────┘      └─────────────┘       └─────────────┘
```

### 5.5 Quote → Order → Invoice Data Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          QUOTE TO CASH FLOW                                │
└─────────────────────────────────────────────────────────────────────────────┘

    ┌─────────┐       ┌────────────┐       ┌─────────────┐       ┌─────────┐
    │  QUOTE  │       │ SALES      │       │  INVOICE    │       │  CASH   │
    │         │       │ ORDER      │       │             │       │         │
    │ Draft   │       │            │       │             │       │         │
    │    │    │       │            │       │             │       │         │
    │    ▼    │       │            │       │             │       │         │
    │ Active  │──────▶│ Active     │──────▶│ Active      │──────▶│         │
    │    │    │       │    │       │       │    │        │       │         │
    │    ▼    │       │    ▼       │       │    ▼        │       │         │
    │ Won    │       │ Fulfilled   │       │ Paid        │◀──────│ Payment │
    │    │    │       │    │       │       │    │        │       │ Received│
    │    ▼    │       │    ▼       │       │    ▼        │       │         │
    │ Revised │       │ Cancelled  │       │ Cancelled   │       │         │
    └─────────┘       └────────────┘       └─────────────┘       └─────────┘

    QuoteDetail ────▶ SalesOrderDetail ────▶ InvoiceDetail
    (Line Items)      (Copied on order)      (Copied on invoice)
```

---

## 6. PRODUCT & PRICING ENTITY DEEP DIVE

### 6.1 Product Entity

| Attribute | Value |
|-----------|-------|
| **Entity Name** | Product |
| **Object Type Code** | 1024 |
| **Primary Table** | ProductBase |

#### ProductBase Structure

```sql
CREATE TABLE [dbo].[ProductBase](
    [ProductId] [uniqueidentifier] NOT NULL,
    [Amount] [money] NULL,                            -- Cost
    [Barcode] [nvarchar](100) NULL,
    [BaseUnit] [nvarchar](100) NULL,
    [Comments] [nvarchar](max) NULL,
    [CreatedOn] [datetime] NULL,
    [CurrentCost] [money] NULL,                       -- Current Cost
    [DefaultUoMScheduleId] [uniqueidentifier] NULL,  -- Default Unit Schedule
    [Description] [nvarchar](max) NULL,
    [DoesContainMedia] [bit] NULL,
    [ExchangeRate] [decimal](23, 10) NULL,
    [HasVariants] [bit] NULL,                         -- Has Product Variants
    [HierarchyPath] [nvarchar](max) NULL,
    [ImageUrl] [nvarchar](200) NULL,
    [IsKit] [bit] NULL,                               -- Is Product Bundle
    [IsStockItem] [bit] NULL,                         -- Is Stocked
    [ItemId] [nvarchar](100) NULL,                    -- Product ID
    [ItemType] [int] NULL,                            -- Product Type
    [LoadTime] [datetime] NULL,
    [Manufacturer] [nvarchar](100) NULL,
    [ManufacturerPartNumber] [nvarchar](100) NULL,
    [MasterProductId] [uniqueidentifier] NULL,        -- Parent Product
    [ModifiedOn] [datetime] NULL,
    [Name] [nvarchar](200) NOT NULL,                  -- Product Name
    [Price] [money] NULL,                             -- List Price
    [ProductNumber] [nvarchar](100) NULL,             -- SKU
    [ProductPricingPrecision] [int] NULL,
    [ProductStructure] [int] NULL,                   -- 1=Product, 2=Bundle, 3=Kit
    [PropertyConfigurationId] [uniqueidentifier] NULL,
    [QuantityDecimal] [int] NULL,
    [SearchBy] [nvarchar](100) NULL,
    [ShipTo_AddressId] [uniqueidentifier] NULL,
    [StateCode] [int] NOT NULL,
    [StatusCode] [int] NULL,
    [StockVolume] [nvarchar](200) NULL,
    [StockWeight] [nvarchar](200) NULL,
    [SubjectId] [uniqueidentifier] NULL,
    [SupplierId] [uniqueidentifier] NULL,            -- Default Vendor
    [Taxable] [bit] NULL,                             -- Is Taxable
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [UTCConversionTimeZoneCode] [int] NULL,
    [VendorId] [nvarchar](100) NULL,
    [VendorPartNumber] [nvarchar](100) NULL,
    [VersionNumber] [timestamp] NULL,
    [Volume] [nvarchar](200) NULL,
    [Weight] [nvarchar](200) NULL
)
```

### 6.2 Pricing Architecture

```
PRICING MODEL
═══════════════════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────────────────┐
│                        PRICING COMPONENTS                           │
└──────────────────────────────────────────────────────────────────────┘

  PriceLevelBase (Price List)
  │
  ├── [PriceLevelId] - PK
  ├── [Name] - Price List Name
  ├── [BeginDate] - Effective From
  ├── [EndDate] - Effective To
  ├── [TransactionCurrencyId] - Currency
  └── 1:N → ProductPriceLevelBase (Prices per Product)

  ProductPriceLevelBase (Price List Item)
  │
  ├── [ProductPriceLevelId] - PK
  ├── [PriceLevelId] - FK to PriceLevel
  ├── [ProductId] - FK to Product
  ├── [UoMId] - Unit of Measure
  ├── [Amount] - List Price
  ├── [RoundingOptionCode] - Rounding
  ├── [RoundingPolicyCode] - Policy
  ├── [RoundingAmount] - Rounding Value
  └── [Percentage] - % off list

  UoMBase (Unit of Measure)
  │
  ├── [UoMId] - PK
  ├── [Name] - Unit Name (Each, Box, Case, etc.)
  └── 1:N → UoMScheduleBase (Schedule)

  DiscountBase
  │
  ├── [DiscountId] - PK
  ├── [DiscountTypeId] - FK to DiscountType
  ├── [LowAmount] - Minimum
  ├── [HighAmount] - Maximum
  ├── [Percentage] - % Discount
  └── [Amount] - Fixed Discount
```

### 6.3 Product Types

| ProductStructure | Type | Description |
|-----------------|------|-------------|
| 1 | Product | Standard product |
| 2 | Bundle | Product bundle (fixed items) |
| 3 | Kit | Product kit (selectable components) |

### 6.4 Price Calculation Logic

```
PRICE CALCULATION FLOW
═══════════════════════════════════════════════════════════════════════

  OpportunityProduct / QuoteDetail / OrderDetail / InvoiceDetail
  │
  ├── Input: ProductId, Quantity, UoMId
  │
  ├── Step 1: Get List Price
  │   Lookup: ProductPriceLevelBase
  │   Where: ProductId + PriceLevelId + UoMId
  │   Result: Amount (list price per unit)
  │
  ├── Step 2: Calculate Line Amount
  │   Formula: Quantity × Unit Price = ExtendedAmount
  │
  ├── Step 3: Apply Discounts
  │   Check: DiscountBase (volume-based)
  │   Check: Header Discount (%)
  │   Result: DiscountAmount
  │
  ├── Step 4: Calculate Tax
  │   Lookup: Tax table based on shipping address
  │   Formula: Taxable Amount × Tax Rate
  │
  └── Step 5: Calculate Total
      Formula: ExtendedAmount - DiscountAmount + Tax
```

---

## 7. SERVICE ENTITIES DEEP DIVE

### 7.1 Incident (Case) Entity

| Attribute | Value |
|-----------|-------|
| **Entity Name** | Incident (Case) |
| **Object Type Code** | 112 |
| **Primary Table** | IncidentBase |

#### IncidentBase Structure

```sql
CREATE TABLE [dbo].[IncidentBase](
    [IncidentId] [uniqueidentifier] NOT NULL,
    [AccountId] [uniqueidentifier] NULL,              -- Customer Account
    [ActiveOn] [datetime] NULL,
    [BilledServiceCases_Id] [uniqueidentifier] NULL,
    [CaseOriginCode] [int] NULL,                      -- Origin (Phone, Email, Web)
    [CaseTypeCode] [int] NULL,                        -- Case Type
    [ContactId] [uniqueidentifier] NULL,             -- Customer Contact
    [ContractId] [uniqueidentifier] NULL,            -- Associated Contract
    [ContractServiceLevelCode] [int] NULL,
    [ContractId] [uniqueidentifier] NULL,
    [CreatedOn] [datetime] NULL,
    [CustomerId] [uniqueidentifier] NULL,            -- Customer (Account/Contact)
    [CustomerIdType] [int] NULL,
    [CustomerSatisfactionCode] [int] NULL,          -- CSAT Score
    [Description] [nvarchar](max) NULL,
    [EmailAddress] [nvarchar](100) NULL,
    [EntityImageId] [uniqueidentifier] NULL,
    [EscalateOn] [datetime] NULL,
    [ExchangeRate] [decimal](23, 10) NULL,
    [ExistingTicket] [nvarchar](50) NULL,
    [FollowUpTaskCreated] [bit] NULL,
    [InboundChannelCode] [int] NULL,
    [IncidentId] [uniqueidentifier] NOT NULL,
    [IsDecrementing] [bit] NULL,
    [IsSLAEnabled] [bit] NULL,
    [ItemId] [nvarchar](100) NULL,
    [LastOnHoldTime] [int] NULL,
    [Mobile] [nvarchar](50) NULL,
    [ModifiedOn] [datetime] NULL,
    [Name] [nvarchar](200) NULL,                     -- Case Title
    [Number] [nvarchar](20) NULL,                     -- Case Number
    [OnHoldTime] [int] NULL,
    [OriginCode] [int] NULL,
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    [ParentCaseId] [uniqueidentifier] NULL,          -- Parent Case
    [PaymentTermsCode] [int] NULL,
    [PriceListId] [uniqueidentifier] NULL,
    [PriorityCode] [int] NULL,                       -- Priority
    [ProductId] [uniqueidentifier] NULL,             -- Product
    [ResolveBy] [datetime] NULL,                      -- SLA Target Date
    [ResolveByBase] [datetime] NULL,
    [Resolution] [nvarchar](max) NULL,               -- Resolution Description
    [ResponseBy] [datetime] NULL,                     -- First Response Target
    [ResponseByBase] [datetime] NULL,
    [SLAId] [uniqueidentifier] NULL,
    [SLAInvarianceId] [uniqueidentifier] NULL,
    [SocialProfileId] [uniqueidentifier] NULL,
    [SourceCaseId] [uniqueidentifier] NULL,
    [StateCode] [int] NOT NULL,                      -- Status
    [StatusCode] [int] NULL,                         -- Status Reason
    [SubjectId] [uniqueidentifier] NULL,            -- Subject/Category
    [TicketNumber] [nvarchar](20) NULL,
    [Title] [nvarchar](200) NULL,
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [TraversedPath] [nvarchar](max) NULL,
    [UTCConversionTimeZoneCode] [int] NULL,
    [VersionNumber] [timestamp] NULL
)
```

### 7.2 Case Lifecycle

```
CASE LIFECYCLE
═══════════════════════════════════════════════════════════════════════

  ┌───────────────────────────────────────────────────────────────────┐
  │  ACTIVE CASES                                                     │
  └───────────────────────────────────────────────────────────────────┘

  ┌─────────────┐     ┌─────────────┐     ┌─────────────────────────┐
  │ In Progress │────▶│ On Hold     │────▶│ Waiting for Details     │
  └─────────────┘     └─────────────┘     └─────────────────────────┘
         │                   │                       │
         │                   │                       │
         ▼                   ▼                       ▼
  ┌───────────────────────────────────────────────────────────────────┐
  │  RESOLVED / CLOSED                                               │
  └───────────────────────────────────────────────────────────────────┘

  ┌─────────────┐     ┌─────────────┐
  │ Problem     │     │ Information │
  │ Solved      │     │ Provided     │
  └─────────────┘     └─────────────┘
```

### 7.3 Contract Entity

| Attribute | Value |
|-----------|-------|
| **Entity Name** | Contract |
| **Object Type Code** | 1010 |
| **Primary Table** | ContractBase |

#### ContractBase Key Fields

```sql
CREATE TABLE [dbo].[ContractBase](
    [ContractId] [uniqueidentifier] NOT NULL,
    [AccountId] [uniqueidentifier] NULL,              -- Customer Account
    [ActiveOn] [datetime] NULL,                      -- Start Date
    [AllotmentTypeCode] [int] NULL,                   -- 1=Number of Cases, 2=Time
    [BillTo] [uniqueidentifier] NULL,
    [BillToContactId] [uniqueidentifier] NULL,
    [BillingCustomerId] [uniqueidentifier] NULL,
    [BillingEnd] [datetime] NULL,
    [BillingFrequencyCode] [int] NULL,              -- Billing Frequency
    [BillingStart] [datetime] NULL,
    [CancelOn] [datetime] NULL,                       -- End Date
    [ContractNumber] [nvarchar](20) NULL,
    [ContractTemplateId] [uniqueidentifier] NULL,   -- Contract Template
    [CreatedOn] [datetime] NULL,
    [CustomerId] [uniqueidentifier] NULL,            -- Customer
    [CustomerIdType] [int] NULL,
    [Duration] [int] NULL,                           -- Duration (months)
    [ExchangeRate] [decimal](23, 10) NULL,
    [ModifiedOn] [datetime] NULL,
    [Name] [nvarchar](100) NULL,
    [NetPrice] [money] NULL,                         -- Total Value
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    [PriceLevelId] [uniqueidentifier] NULL,
    [ProcessId] [uniqueidentifier] NULL,
    [ServiceAddress] [uniqueidentifier] NULL,
    [StateCode] [int] NOT NULL,
    [StatusCode] [int] NULL,
    [Terms] [nvarchar](2000) NULL,
    [Title] [nvarchar](200) NULL,
    [TotalDiscount] [money] NULL,
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [UseAsPriceList] [bit] NULL,
    [VersionNumber] [timestamp] NULL
)
```

### 7.4 Service Level Agreement (SLA)

| Attribute | Value |
|-----------|-------|
| **Entity Name** | SLA |
| **Object Type Code** | 9750 |
| **Primary Table** | SLABase |

---

## 8. CUSTOM ENTITIES DEEP DIVE

### 8.1 canven_onboarding Entity

This is a custom entity for property onboarding workflow.

```sql
CREATE TABLE [dbo].[canven_onboardingBase](
    [canven_onboardingId] [uniqueidentifier] NOT NULL,
    [canven_name] [nvarchar](100) NULL,                -- Onboarding Name
    [canven_propertyid] [uniqueidentifier] NULL,      -- Related Property
    [canven_status] [int] NULL,                       -- Status
    [canven_startdate] [datetime] NULL,               -- Start Date
    [canven_completiondate] [datetime] NULL,         -- Completion Date
    [canven_assignedto] [uniqueidentifier] NULL,      -- Assigned User
    [canven_estimatedvalue] [money] NULL,           -- Estimated Value
    [canven_notes] [nvarchar](max) NULL,             -- Notes
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    [StateCode] [int] NOT NULL,
    [StatusCode] [int] NULL,
    [CreatedOn] [datetime] NULL,
    [ModifiedOn] [datetime] NULL,
    [VersionNumber] [timestamp] NULL
)
```

### 8.2 canven_propertyonboard Entity

Related entity for property onboarding details.

```sql
CREATE TABLE [dbo].[canven_propertyonboardBase](
    [canven_propertyonboardId] [uniqueidentifier] NOT NULL,
    [canven_propertyname] [nvarchar](200) NULL,       -- Property Name
    [canven_propertytype] [int] NULL,                 -- Property Type
    [canven_address] [nvarchar](500) NULL,           -- Address
    [canven_cityid] [uniqueidentifier] NULL,         -- City Lookup
    [canven_size] [decimal](18, 0) NULL,            -- Size (sqft)
    [canven_valuation] [money] NULL,                 -- Valuation
    [canven_onboardingid] [uniqueidentifier] NULL,  -- Parent Onboarding
    [canven_documentstatus] [int] NULL,              -- Document Status
    [canven_inspectionstatus] [int] NULL,            -- Inspection Status
    [canven_approvalstatus] [int] NULL,              -- Approval Status
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    [StateCode] [int] NOT NULL,
    [StatusCode] [int] NULL,
    [CreatedOn] [datetime] NULL,
    [ModifiedOn] [datetime] NULL,
    [VersionNumber] [timestamp] NULL
)
```

### 8.3 canven_city Reference Entity

```sql
CREATE TABLE [dbo].[canven_cityBase](
    [canven_cityId] [uniqueidentifier] NOT NULL,
    [canven_cityname] [nvarchar](100) NULL,          -- City Name
    [canven_state] [nvarchar](100) NULL,             -- State
    [canven_country] [nvarchar](100) NULL,           -- Country
    [canven_postalcode] [nvarchar](20) NULL,        -- Postal Code
    [canven_regionid] [uniqueidentifier] NULL,       -- Region
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    [StateCode] [int] NOT NULL,
    [StatusCode] [int] NULL,
    [CreatedOn] [datetime] NULL,
    [ModifiedOn] [datetime] NULL,
    [VersionNumber] [timestamp] NULL
)
```

### 8.4 Custom Entities Summary

| Entity | Prefix | Type | Purpose |
|--------|--------|------|---------|
| canven_onboarding | canven_ | Custom | Property onboarding workflow |
| canven_propertyonboard | canven_ | Custom | Property details |
| canven_city | canven_ | Reference | City master data |
| new_usertype | new_ | Custom | User classification |
| new_customercomplaint | new_ | Custom | Complaint tracking |
| new_userdetails | new_ | Custom | Extended user info |

---

## 9. ACTIVITY ENTITIES DEEP DIVE

### 9.1 Activity Pointer (Activity)

The ActivityPointerBase is the base table for all activities.

```sql
CREATE TABLE [dbo].[ActivityPointerBase](
    [ActivityId] [uniqueidentifier] NOT NULL,
    [ActualDurationMinutes] [int] NULL,              -- Duration
    [ActualEnd] [datetime] NULL,                     -- Actual End
    [ActualStart] [datetime] NULL,                   -- Actual Start
    [Category] [nvarchar](250) NULL,                 -- Category
    [Community] [int] NULL,
    [ConversationIndex] [varbinary](max) NULL,
    [CreatedOn] [datetime] NULL,
    [DeliveryPriorityCode] [int] NULL,
    [Description] [nvarchar](max) NULL,
    [ExchangeRate] [decimal](23, 10) NULL,
    [From] [nvarchar](250) NULL,                     -- From (ActivityParty)
    [ImportSequenceNumber] [int] NULL,
    [IsBilled] [bit] NULL,
    [IsMapiPrivate] [bit] NULL,
    [IsWorkflowCreated] [bit] NULL,
    [LastOnHoldTime] [int] NULL,
    [ModifiedOn] [datetime] NULL,
    [OnHoldTime] [int] NULL,
    [OptionalAttendees] [nvarchar](max) NULL,
    [Organizer] [nvarchar](250) NULL,
    [OwnerId] [uniqueidentifier] NOT NULL,
    [OwnerIdType] [int] NOT NULL,
    [OwningBusinessUnit] [uniqueidentifier] NULL,
    [PostponeUntil] [datetime] NULL,
    [PriorityCode] [int] NULL,                       -- Priority
    [ProcessId] [uniqueidentifier] NULL,
    [RegardingObjectId] [uniqueidentifier] NULL,     -- Related Record
    [RegardingObjectIdName] [nvarchar](4000) NULL,   -- Denormalized Name
    [RegardingObjectIdYomiName] [nvarchar](4000) NULL,
    [RegardingObjectTypeCode] [int] NULL,           -- Related Entity Type
    [RequiredAttendees] [nvarchar](max) NULL,
    [Resources] [nvarchar](max) NULL,
    [ScheduledDurationMinutes] [int] NULL,           -- Scheduled Duration
    [ScheduledEnd] [datetime] NULL,                  -- Due Date
    [ScheduledStart] [datetime] NULL,                -- Start Date
    [SendDirectlyToTeam] [bit] NULL,
    [SeriesId] [uniqueidentifier] NULL,              -- Recurring Series
    [ServiceId] [uniqueidentifier] NULL,
    [SLAId] [uniqueidentifier] NULL,
    [SLAInvarianceId] [uniqueidentifier] NULL,
    [SortDate] [datetime] NULL,
    [StateCode] [int] NOT NULL,                      -- Status
    [StatusCode] [int] NULL,                         -- Status Reason
    [Subcategory] [nvarchar](250) NULL,
    [Subject] [nvarchar](200) NOT NULL,             -- Subject
    [TimeZoneRuleVersionNumber] [int] NULL,
    [To] [nvarchar](250) NULL,                       -- To (ActivityParty)
    [TransactionCurrencyId] [uniqueidentifier] NULL,
    [TraversedPath] [nvarchar](max) NULL,
    [UTCConversionTimeZoneCode] [int] NULL,
    [VersionNumber] [timestamp] NULL
)
```

### 9.2 Activity Party

The ActivityPartyBase stores participants in activities.

```sql
CREATE TABLE [dbo].[ActivityPartyBase](
    [ActivityId] [uniqueidentifier] NOT NULL,         -- Activity Reference
    [ActivityPartyId] [uniqueidentifier] NOT NULL,  -- PK
    [PartyId] [uniqueidentifier] NULL,               -- Party (Contact/Account/User)
    [PartyIdName] [nvarchar](4000) NULL,
    [PartyIdTypeCode] [int] NULL,                   -- Party Type
    [ParticipationTypeMask] [int] NOT NULL,          -- Role in Activity
    [AddressUsed] [nvarchar](250) NULL,
    [AddressUsedEmailColumnNumber] [int] NULL,
    [DoNotEmail] [bit] NULL,
    [DoNotPhone] [bit] NULL,
    [DoNotFax] [bit] NULL,
    [EmailAddress] [nvarchar](250) NULL,
    [InstanceTypeCode] [int] NULL,
    [IsPrimary] [bit] NULL,                          -- Primary Contact
    [Unresolved] [bit] NULL,
    [VersionNumber] [timestamp] NULL
)
```

### 9.3 Participation Type Codes

| Code | Meaning |
|------|---------|
| 1 | Sender |
| 2 | To Recipient |
| 3 | CC Recipient |
| 4 | BCC Recipient |
| 5 | Required Attendee |
| 6 | Optional Attendee |
| 7 | Organizer |
| 8 | Regarding |
| 9 | Owner |
| 10 | Resource |
| 11 | Customer |

### 9.4 Activity Types

| Activity | ObjectTypeCode | Base Table |
|----------|----------------|------------|
| Appointment | 4201 | AppointmentBase |
| Email | 4202 | EmailBase |
| Fax | 4204 | FaxBase |
| Letter | 4207 | LetterBase |
| Phone Call | 4210 | PhoneCallBase |
| Task | 4212 | TaskBase |
| Service Appointment | 4214 | ServiceAppointmentBase |
| Campaign Response | 4401 | CampaignResponseBase |

---

## 10. DATA FLOW & INTEGRATION MAPPING

### 10.1 Complete Sales Process Data Flow

```
LEAD TO CASH - COMPLETE DATA FLOW
════════════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────────────┐
│  LEAD STAGE                                                                       │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                            │
│  │ LeadBase    │───▶│ LeadBase    │───▶│ LeadBase    │                            │
│  │ (New)       │    │ (Contacted) │    │ (Qualified) │                           │
│  └─────────────┘    └─────────────┘    └──────┬──────┘                            │
│                                                │                                    │
│                                                ▼                                    │
│  ON QUALIFICATION ────────────────────────────────────────────────────────────────│
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                            │
│  │ AccountBase│    │ContactBase  │    │Opportunity  │                            │
│  │ Created    │    │ Created     │    │ Created     │                            │
│  └─────┬───────┘    └──────┬──────┘    └──────┬──────┘                            │
│        │                   │                   │                                   │
│        │                   │                   │                                   │
│        ▼                   ▼                   ▼                                   │
│  ┌─────────────────────────────────────────────────────────────────────────────┐  │
│  │  OPPORTUNITY STAGE                                                           │  │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  │  │
│  │  │Opportunity  │───▶│Opportunity  │───▶│Opportunity  │───▶│ Opportunity │  │  │
│  │  │ProductBase  │    │ProductBase  │    │ Quote       │    │ Won         │  │  │
│  │  │(Add Items)  │    │             │    │             │    │             │  │  │
│  │  └─────────────┘    └─────────────┘    └──────┬──────┘    └──────┬──────┘  │  │
│  └─────────────────────────────────────────────────────────────────────┴────────┘  │
│                                                                                   │
│        │                   │                   │                   │                │
│        │                   │                   │                   │                │
│        ▼                   ▼                   ▼                   ▼                │
│  ┌───────────────────────────────────────────────────────────────────────────────┐│
│  │  POST-SALE STAGE                                                             ││
│  │                                                                              ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  ││
│  │  │SalesOrder   │───▶│SalesOrder   │───▶│ Invoice     │───▶│ Payment     │  ││
│  │  │Created      │    │ Fulfilled   │    │ Created     │    │ Received    │  ││
│  │  └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘  ││
│  │                                                                              ││
│  └──────────────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 10.2 Master vs. Transactional Data

| Data Type | Entities | Characteristics |
|-----------|----------|----------------|
| **Master Data** | Account, Contact, Product, PriceLevel, UoM, User, Team | Reference data, low change frequency, used across transactions |
| **Transactional** | Lead, Opportunity, Quote, Order, Invoice, Case | High change frequency, workflow-driven, time-sensitive |
| **Activity** | Email, PhoneCall, Task, Appointment | Communication logs, high volume |
| **Configuration** | StringMap, BusinessUnit, Role, Privilege | System setup, infrequent changes |

### 10.3 Entity State Transitions Summary

| Entity | Initial State | Intermediate States | Terminal States |
|--------|--------------|---------------------|-----------------|
| Lead | Open/New | Open/Contacted, Open/Qualified | Disqualified |
| Opportunity | Open | Open/In Progress, Open/On Hold | Won, Lost |
| Quote | Draft | Active, Won | Revised, Closed |
| Order | Active | Fulfilled | Cancelled |
| Invoice | Active | Paid | Cancelled |
| Case | Active | In Progress, On Hold, Waiting | Resolved, Cancelled |
| Contract | Draft | Invoiced, Active | Expired, Cancelled |

### 10.4 Key Integration Points

| Integration Type | Source | Target | Mechanism |
|-----------------|--------|--------|-----------|
| Lead Qualification | Lead | Account/Contact/Opportunity | Auto-create |
| Quote to Order | Quote | SalesOrder | Copy + Status change |
| Order to Invoice | SalesOrder | Invoice | Copy + Status change |
| Contract to Case | Contract | Incident | Entitlement check |
| Campaign Response | Campaign | Lead/Opportunity | Response tracking |

---

## Appendix A: Field Suffix Conventions

| Suffix | Meaning | Example |
|--------|---------|---------|
| _Id | Foreign Key | AccountId, ContactId |
| _IdName | Denormalized Name | RegardingObjectIdName |
| _Code | Picklist/Status | StatusCode, PriorityCode |
| _Type | Type Indicator | OwnerIdType, CustomerIdType |
| _Base | Base Currency | Amount_Base |
| Yomi* | Phonetic Reading | YomiFirstName |
| UTC* | Timezone Info | UTCConversionTimeZoneCode |
| VersionNumber | Concurrency | timestamp |

---

## Appendix B: Object Type Codes

| Code | Entity |
|------|--------|
| 1 | Account |
| 2 | Contact |
| 3 | Opportunity |
| 4 | Lead |
| 1011 | Contract |
| 1088 | Quote |
| 1088 | SalesOrder |
| 1090 | Invoice |
| 112 | Incident (Case) |
| 1024 | Product |
| 1025 | PriceLevel |
| 1026 | UoM |
| 8 | SystemUser |
| 9 | Team |
| 10 | BusinessUnit |

---

*This document provides deep entity-level analysis of the CANCRM_MSCRM database following Microsoft Dynamics 365 CRM architecture patterns.*
