# Microsoft Dynamics 365 CRM Sales Module
## Complete Enterprise Technical Documentation

**Document Classification**: Enterprise Architecture Technical Documentation  
**Author**: Senior Microsoft Dynamics 365 CRM Architect & Dataverse Database Expert  
**Database**: CRM_DB (SQL Server)  
**Date**: January 2026

---

# SECTION 1 – System Overview

## 1.1 Database Architecture Overview

The CRM_DB database implements a comprehensive Microsoft Dynamics 365 CRM Sales Module with approximately **300+ tables** following the standard Dataverse three-layer architecture pattern. This database serves as the foundation for enterprise customer relationship management, supporting the complete sales lifecycle from lead generation through invoice payment.

The architecture demonstrates a mature, enterprise-grade implementation with full support for:
- Multi-currency transactions with historical exchange rate preservation
- Complex polymorphic relationships enabling flexible customer modeling
- Comprehensive audit trails for regulatory compliance
- Row-level and field-level security for data protection
- Business process flows guiding users through standardized sales stages

The database schema follows Microsoft Dynamics 365 conventions where every entity table includes standard system attributes for tracking ownership, state management, concurrency, and audit information. The three-layer architecture consisting of Base tables (physical storage), Logical views (joined representations), and Filtered security views (row-level security) enables both high performance and robust security.

## 1.2 CRM Sales Flow Architecture

The database supports the complete Lead-to-Cash business process:

```
Lead (Qualification) → Opportunity (Development) → Quote (Proposal) → 
Sales Order (Fulfillment) → Invoice (Billing) → Payment (Close)
```

Each stage in this pipeline maintains referential integrity through foreign key constraints, cascading behaviors, and state machine transitions enforced by StateCode and StatusCode fields.

---

# SECTION 2 – Entity Deep Dive

This section provides comprehensive field-level analysis for every major entity in the CRM sales module.

---

## ENTITY 1: ACCOUNT

### Business Purpose

The Account entity represents business organizations or companies that serve as customers within the CRM system. It is one of the core entities in any sales application, serving as a parent container for multiple related records including Contacts, Opportunities, Quotes, Orders, Invoices, and Activities. Accounts maintain hierarchical relationships enabling representation of corporate structures, and they integrate with marketing campaigns through leadorigination tracking.

### Table Information

- **Table Name**: AccountBase
- **Object Type Code**: 1
- **Primary Key**: AccountId (uniqueidentifier)
- **Ownership Model**: User or Team ownership via OwnerBase polymorphic relationship
- **Business Unit**: Required for row-level security

### Primary Key Definition

```sql
[AccountId] [uniqueidentifier] NOT NULL
CONSTRAINT [PK_AccountBase] PRIMARY KEY CLUSTERED ([AccountId])
    WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, 
          IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, 
          ALLOW_PAGE_LOCKS = ON, FILLFACTOR = 80)
```

### Foreign Keys

| Foreign Key Column | Referenced Table | Constraint Name | Behavior |
|-------------------|------------------|-----------------|----------|
| OwningBusinessUnit | BusinessUnitBase | business_unit_accounts | Restrict Delete |
| OwnerId | OwnerBase | owner_accounts | Restrict Delete |
| ParentAccountId | AccountBase | account_parent_account | Cascade |
| PrimaryContactId | ContactBase | account_primary_contact | RemoveLink |
| MasterId | AccountBase | account_master_account | RemoveLink |
| DefaultPriceLevelId | PriceLevelBase | price_level_accounts | RemoveLink |
| PreferredSystemUserId | SystemUserBase | system_user_accounts | RemoveLink |
| PreferredEquipmentId | EquipmentBase | equipment_accounts | RemoveLink |
| PreferredServiceId | ServiceBase | service_accounts | RemoveLink |
| TerritoryId | TerritoryBase | territory_accounts | RemoveLink |
| TransactionCurrencyId | TransactionCurrencyBase | transactioncurrency_account | RemoveLink |

### Field Analysis Table

| Field Name | Data Type | Nullable | Default | Validation | Business Meaning |
|------------|-----------|---------|---------|------------|-----------------|
| AccountId | uniqueidentifier | No | NEWSEQUENTIALID() | Required, Unique | Globally unique identifier for account record |
| AccountNumber | nvarchar(20) | Yes | NULL | Unique if populated | External business identifier |
| Name | nvarchar(160) | No | None | Required, Max 160 chars | Primary account name |
| OwningBusinessUnit | uniqueidentifier | Yes | NULL | FK validation | Security boundary for record access |
| OwnerId | uniqueidentifier | No | None | FK validation | User/Team owning this record |
| OwnerIdType | int | No | None | 0=User, 1=Team | Type of owner |
| StateCode | int | No | 0 | 0=Active, 1=Inactive | Entity active state |
| StatusCode | int | Yes | NULL | State-dependent | Reason for current state |
| ParentAccountId | uniqueidentifier | Yes | NULL | Self-reFK | Parent account for hierarchy |
| PrimaryContactId | uniqueidentifier | Yes | NULL | FK validation | Main contact person |
| MasterId | uniqueidentifier | Yes | NULL | Self-reFK | For duplicate record management |
| IndustryCode | int | Yes | NULL | Picklist | Industry classification |
| BusinessTypeCode | int | Yes | NULL | Picklist | Type of business entity |
| AccountRatingCode | int | Yes | NULL | Picklist | Rating/category |
| Revenue | money | Yes | NULL | >= 0 | Annual revenue |
| NumberOfEmployees | int | Yes | NULL | > 0 | Employee count |
| CreditLimit | money | Yes | NULL | >= 0 | Credit limit |
| CreditOnHold | bit | Yes | 0 | 0/1 | Credit hold status |
| Telephone1 | nvarchar(50) | Yes | NULL | Phone format | Primary phone |
| EmailAddress1 | nvarchar(100) | Yes | NULL | Email format | Primary email |
| WebsiteUrl | nvarchar(200) | Yes | NULL | URL format | Website |
| Address1_Line1 | nvarchar(250) | Yes | NULL | Max 250 | Primary address line 1 |
| Address1_City | nvarchar(80) | Yes | NULL | Max 80 | Primary city |
| Address1_StateOrProvince | nvarchar(50) | Yes | NULL | Max 50 | Primary state/province |
| Address1_PostalCode | nvarchar(40) | Yes | NULL | Max 40 | Primary postal code |
| Address1_Country | nvarchar(80) | Yes | NULL | Max 80 | Primary country |
| TraversedPath | nvarchar(1250) | Yes | NULL | BPF tracking | Business process flow stage history |
| ProcessId | uniqueidentifier | Yes | NULL | FK validation | Active BPF instance |
| StageId | uniqueidentifier | Yes | NULL | FK validation | Current BPF stage |
| CreatedOn | datetime | Yes | GETUTCDATE() | Valid datetime | Creation timestamp |
| CreatedBy | uniqueidentifier | Yes | NULL | FK validation | Creator user |
| ModifiedOn | datetime | Yes | GETUTCDATE() | Valid datetime | Last modification |
| ModifiedBy | uniqueidentifier | Yes | NULL | FK validation | Last modifier |
| CreatedOnBehalfBy | uniqueidentifier | Yes | NULL | FK validation | Delegate creator |
| ModifiedOnBehalfBy | uniqueidentifier | Yes | NULL | FK validation | Delegate modifier |
| VersionNumber | timestamp | Yes | NULL | Auto-increment | Concurrency control |
| ImportSequenceNumber | int | Yes | NULL | Import ordering | Sequence for data import |
| OverriddenCreatedOn | datetime | Yes | NULL | Backdated creation | For data migration |
| UTCConversionTimeZoneCode | int | Yes | NULL | Timezone mapping | UTC conversion |
| TimeZoneRuleVersionNumber | int | Yes | NULL | DST rules | Timezone version |

### Validation Explanation

**Required Fields**:
- AccountId: Cannot be null, auto-generated via NEWSEQUENTIALID()
- Name: Required nvarchar(160), provides business identifier
- OwnerId: Required for security enforcement
- OwningBusinessUnit: Required for security context

**Check Constraints**:
- Revenue and CreditLimit must be >= 0 (money type enforces this implicitly)
- NumberOfEmployees should be > 0 (application-level validation)
- Credit bitOnHold is type allowing only 0 or 1

**Lookup Validations**:
- All foreign key relationships validated via constraints
- ParentAccountId self-referential FK enforces hierarchy
- TransactionCurrencyId links to currency master for multi-currency support

### Relationship Explanation

**Parent Entity**: None (root entity for customer hierarchy)

**Child Entities** (One-to-Many):
- Contact (ParentCustomerId) - Multiple contacts per account
- Opportunity (CustomerId) - Sales opportunities
- Quote (CustomerId) - Sales quotes
- SalesOrder (CustomerId) - Customer orders
- Invoice (CustomerId) - Customer invoices
- Contract (CustomerId) - Service contracts
- CustomerAddress (ParentId) - Multiple addresses
- Lead (OriginatingLeadId) - Leads from this account

**Junction Tables**:
- AccountLeads - N:N relationship between accounts and leads

**Cascade Behavior**:
- Cascade on ParentAccountId for hierarchical deletes
- RemoveLink on primary contact (keeps contact record)
- Restricted on OwningBusinessUnit (must reassign records)

### State & Lifecycle

**StateCode Values**:
| Value | State | Meaning |
|-------|-------|---------|
| 0 | Active | Account is active and accessible |
| 1 | Inactive | Account is inactive (typically from merge) |

**StatusCode Values** (mapped to StateCode):
| StatusCode | State | Meaning |
|------------|-------|---------|
| 1 | Active | Default active status |
| 2 | Inactive | Inactive status |

### Security & Ownership

**OwnerId Behavior**: Records can be owned by users or teams. The OwnerId column references OwnerBase which is a polymorphic table containing both SystemUser and Team records. This enables flexible security where records can be individually assigned or team-based.

**BusinessUnit Impact**: The OwningBusinessUnit column establishes the primary security boundary. Users can only access records within their business unit hierarchy unless granted additional access through sharing or team membership.

**Role-Based Access**: Access controlled through SecurityRoleBase and RolePrivilegesBase. Typical account access levels include:
- Read: View account information
- Write: Modify account details
- Append: Add related records
- AppendTo: Be added as related to other records
- Delete: Remove account records
- Assign: Transfer ownership
- Share: Grant access to other users

---

## ENTITY 2: CONTACT

### Business Purpose

The Contact entity represents individual people, typically associated with Accounts as customers, prospects, or stakeholders. Contacts maintain personal information, communication preferences, and serve as the human interface within the CRM system. They participate in sales processes as decision-makers, influencers, or end-users, and are critical for marketing personalization and communication.

### Table Information

- **Table Name**: ContactBase
- **Object Type Code**: 2
- **Primary Key**: ContactId (uniqueidentifier)
- **Ownership Model**: User or Team ownership
- **Business Unit**: Required for row-level security

### Primary Key Definition

```sql
[ContactId] [uniqueidentifier] NOT NULL
CONSTRAINT [PK_ContactBase] PRIMARY KEY CLUSTERED ([ContactId])
```

### Foreign Keys

| Foreign Key Column | Referenced Table | Constraint Name | Behavior |
|-------------------|------------------|-----------------|----------|
| OwningBusinessUnit | BusinessUnitBase | business_unit_contacts | Restrict Delete |
| OwnerId | OwnerBase | owner_contacts | Restrict Delete |
| ParentCustomerId | AccountBase/ContactBase | contact_customer_account | RemoveLink |
| ParentCustomerIdType | int | N/A | Polymorphic type |
| MasterId | ContactBase | contact_master_contact | RemoveLink |
| OriginatingLeadId | LeadBase | contact_originating_lead | RemoveLink |
| DefaultPriceLevelId | PriceLevelBase | price_level_contacts | RemoveLink |
| PreferredSystemUserId | SystemUserBase | system_user_contacts | RemoveLink |
| PreferredEquipmentId | EquipmentBase | equipment_contacts | RemoveLink |
| PreferredServiceId | ServiceBase | service_contacts | RemoveLink |
| TransactionCurrencyId | TransactionCurrencyBase | transactioncurrency_contact | RemoveLink |

### Field Analysis Table

| Field Name | Data Type | Nullable | Default | Validation | Business Meaning |
|------------|-----------|---------|---------|------------|-----------------|
| ContactId | uniqueidentifier | No | NEWSEQUENTIALID() | Required, Unique | Unique identifier |
| FirstName | nvarchar(50) | Yes | NULL | Max 50 chars | First name |
| LastName | nvarchar(50) | No | None | Required, Max 50 | Last name |
| FullName | nvarchar(160) | Yes | NULL | Computed | First + Last name |
| YomiFullName | nvarchar(160) | Yes | NULL | Phonetic | Phonetic full name |
| EmailAddress1 | nvarchar(100) | Yes | NULL | Email format | Primary email |
| EmailAddress2 | nvarchar(100) | Yes | NULL | Email format | Secondary email |
| Telephone1 | nvarchar(50) | Yes | NULL | Phone format | Primary phone |
| Telephone2 | nvarchar(50) | Yes | NULL | Phone format | Secondary phone |
| MobilePhone | nvarchar(50) | Yes | NULL | Phone format | Mobile number |
| OwningBusinessUnit | uniqueidentifier | Yes | NULL | FK validation | Security boundary |
| OwnerId | uniqueidentifier | No | None | FK validation | Record owner |
| OwnerIdType | int | No | None | 0=User, 1=Team | Owner type |
| ParentCustomerId | uniqueidentifier | Yes | NULL | Polymorphic FK | Associated account |
| ParentCustomerIdType | int | Yes | NULL | 1=Account, 2=Contact | Customer type |
| StateCode | int | No | 0 | 0=Active, 1=Inactive | Contact state |
| StatusCode | int | Yes | NULL | State-dependent | State reason |
| BirthDate | datetime | Yes | NULL | Valid date | Date of birth |
| GenderCode | int | Yes | NULL | Picklist | Gender |
| JobTitle | nvarchar(100) | Yes | NULL | Max 100 | Position title |
| Department | nvarchar(100) | Yes | NULL | Max 100 | Department |
| DoNotEmail | bit | Yes | 0 | 0/1 | No email opt-out |
| DoNotPhone | bit | Yes | 0 | 0/1 | No phone opt-out |
| DoNotPostalMail | bit | Yes | 0 | 0/1 | No mail optout |
| VersionNumber | timestamp | Yes | NULL | Auto-increment | Concurrency |
| TraversedPath | nvarchar(1250) | Yes | NULL | BPF tracking | BPF stage history |

### Validation Explanation

**Required Fields**:
- ContactId: Auto-generated uniqueidentifier
- LastName: Required for contact identification
- OwnerId: Required for security

**Check Constraints**:
- Email format validation at application layer
- Phone number format validation at application layer
- BirthDate should be in past (application validation)

**Polymorphic Relationship**: ParentCustomerId can reference either AccountBase or ContactBase (for household contacts), with ParentCustomerIdType indicating the target entity type (1=Account, 2=Contact).

### Relationship Explanation

**Parent Entity**: 
- Account (via ParentCustomerId) - Optional account association

**Child Entities**:
- Opportunity - Associated opportunities
- Lead - Qualifying leads
- Quote - Received quotes
- SalesOrder - Customer orders
- Invoice - Customer invoices

**N:N Relationships**:
- ContactLeads - Junction for contact-lead relationships
- ContactQuotes - Junction for contact-quote associations
- ContactOrders - Junction for contact-order associations
- ContactInvoices - Junction for contact-invoice associations

### Security & Ownership

Standard ownership model applies. Contacts can be individually owned or team-owned, with business unit determining baseline visibility.

---

## ENTITY 3: LEAD

### Business Purpose

The Lead entity serves as the entry point in the sales cycle, capturing prospective customer information before qualification. Leads represent potential customers discovered through marketing campaigns, referrals, cold outreach, or website inquiries. The entity supports the qualification process where leads are evaluated and converted to opportunities when they meet criteria for active sales pursuit.

### Table Information

- **Table Name**: LeadBase
- **Object Type Code**: 4
- **Primary Key**: LeadId (uniqueidentifier)
- **Ownership Model**: User or Team ownership
- **Business Unit**: Required

### Primary Key Definition

```sql
[LeadId] [uniqueidentifier] NOT NULL
CONSTRAINT [PK_LeadBase] PRIMARY KEY CLUSTERED ([LeadId])
```

### Foreign Keys

| Foreign Key Column | Referenced Table | Constraint Name | Behavior |
|-------------------|------------------|-----------------|----------|
| OwningBusinessUnit | BusinessUnitBase | business_unit_leads | Restrict Delete |
| OwnerId | OwnerBase | owner_leads | Restrict Delete |
| CampaignId | CampaignBase | campaign_leads | RemoveLink |
| ParentAccountId | AccountBase | lead_parent_account | RemoveLink |
| ParentContactId | ContactBase | lead_parent_contact | RemoveLink |
| OriginatingLeadId | LeadBase | lead_master_lead | Self-referential |
| QualifyingOpportunityId | OpportunityBase | lead_qualifying_opportunity | RemoveLink |
| OriginatingCaseId | IncidentBase | OriginatingCase_Lead | RemoveLink |
| TransactionCurrencyId | TransactionCurrencyBase | transactioncurrency_lead | RemoveLink |

### Field Analysis Table

| Field Name | Data Type | Nullable | Default | Validation | Business Meaning |
|------------|-----------|---------|---------|------------|-----------------|
| LeadId | uniqueidentifier | No | NEWSEQUENTIALID() | Required | Unique identifier |
| Subject | nvarchar(200) | No | None | Required, Max 200 | Lead topic/summary |
| FirstName | nvarchar(100) | Yes | NULL | Max 100 | First name |
| LastName | nvarchar(100) | Yes | NULL | Max 100 | Last name |
| CompanyName | nvarchar(200) | Yes | NULL | Max 200 | Company name |
| EmailAddress | nvarchar(100) | Yes | NULL | Email format | Email address |
| Telephone | nvarchar(50) | Yes | NULL | Phone format | Phone number |
| Website | nvarchar(200) | Yes | NULL | URL format | Company website |
| OwningBusinessUnit | uniqueidentifier | Yes | NULL | FK validation | Security boundary |
| OwnerId | uniqueidentifier | No | None | FK validation | Record owner |
| OwnerIdType | int | No | None | Owner type | |
| StateCode | int | No | 0 | State machine | Lead lifecycle state |
| StatusCode | int | Yes | NULL | State-dependent | Status reason |
| IndustryCode | int | Yes | NULL | Picklist | Industry classification |
| LeadSourceCode | int | Yes | NULL | Picklist | Source of lead |
| RatingCode | int | Yes | NULL | Picklist | Lead quality rating |
| BudgetAmount | money | Yes | NULL | >= 0 | Customer budget |
| EstimatedAmount | money | Yes | NULL | >= 0 | Estimated value |
| PurchaseTimeframe | int | Yes | NULL | Picklist | Purchase timing |
| PurchaseProcess | int | Yes | NULL | Picklist | Decision process |
| DecisionMaker | bit | Yes | NULL | 0/1 | Has decision authority |
| DoNotEmail | bit | Yes | 0 | 0/1 | Opt-out email |
| DoNotPhone | bit | Yes | 0 | 0/1 | Opt-out phone |
| DoNotPostalMail | bit | Yes | 0 | 0/1 | Opt-out mail |
| DoNotFax | bit | Yes | 0 | 0/1 | Opt-out fax |
| TraversedPath | nvarchar(1250) | Yes | NULL | BPF tracking | Stage history |
| ProcessId | uniqueidentifier | Yes | NULL | FK validation | BPF instance |
| StageId | uniqueidentifier | Yes | NULL | FK validation | Current stage |

### State & Lifecycle

**Lead State Transitions**:

| StateCode | StateName | StatusCode | StatusName | Valid Transitions |
|-----------|-----------|------------|------------|-------------------|
| 0 | Open | 1 | New | → Qualified |
| 0 | Open | 2 | Attempted to Contact | → Qualified, Disqualified |
| 0 | Open | 3 | Contacted | → Qualified, Disqualified |
| 0 | Open | 4 | Open | → Qualified, Disqualified |
| 1 | Qualified | 5 | Qualified | → Won (Create Opportunity) |
| 1 | Qualified | 6 | Disqualified | → Disqualified State |
| 2 | Disqualified | 7 | Lost | Terminal |
| 2 | Disqualified | 8 | Cannot Contact | Terminal |
| 2 | Disqualified | 9 | Not Interested | Terminal |

### Validation Explanation

**Required Fields**:
- LeadId: Auto-generated
- Subject: Required for lead identification
- OwnerId: Required for security
- StateCode: Defaults to 0 (Open)

**Business Rules**:
- Lead qualification requires: Contact information, Company name, Estimated value
- Qualification creates new Opportunity with OriginatingLeadId reference
- Disqualification reasons tracked for analytics

### Relationship Explanation

**Parent Entity**:
- Campaign (CampaignId) - Marketing campaign source
- Account (ParentAccountId) - Converted to account
- Contact (ParentContactId) - Converted to contact
- Opportunity (QualifyingOpportunityId) - Created opportunity

**Child Entities**:
- Opportunity - Created during qualification
- Account - Created upon conversion
- Contact - Created upon conversion

**N:N Junction Tables**:
- AccountLeads - Associated accounts
- ContactLeads - Associated contacts
- LeadCompetitors - Competitor tracking
- LeadProduct - Product interest

---

## ENTITY 4: OPPORTUNITY

### Business Purpose

The Opportunity entity represents a qualified sales opportunity that is being actively pursued. It is the central entity in the sales pipeline, tracking potential revenue, sales stages, customer requirements, and competitive positioning. Opportunities are created from qualified leads or directly from existing customer accounts and represent measurable sales potential.

### Table Information

- **Table Name**: OpportunityBase
- **Object Type Code**: 3
- **Primary Key**: OpportunityId (uniqueidentifier)
- **Ownership Model**: User or Team ownership
- **Business Unit**: Required

### Primary Key Definition

```sql
[OpportunityId] [uniqueidentifier] NOT NULL
CONSTRAINT [PK_OpportunityBase] PRIMARY KEY CLUSTERED ([OpportunityId])
```

### Foreign Keys

| Foreign Key Column | Referenced Table | Constraint Name | Behavior |
|-------------------|------------------|-----------------|----------|
| OwningBusinessUnit | BusinessUnitBase | business_unit_opportunities | Restrict Delete |
| OwnerId | OwnerBase | owner_opportunitys | Restrict Delete |
| CustomerId | AccountBase/ContactBase | polymorphic | RemoveLink |
| CustomerIdType | int | N/A | Polymorphic type |
| PriceLevelId | PriceLevelBase | price_level_opportunities | RemoveLink |
| TransactionCurrencyId | TransactionCurrencyBase | transactioncurrency_opportunity | RemoveLink |
| OriginatingLeadId | LeadBase | opportunity_originating_lead | RemoveLink |
| CampaignId | CampaignBase | campaign_opportunities | RemoveLink |
| ParentAccountId | AccountBase | opportunity_parent_account | RemoveLink |
| ParentContactId | ContactBase | opportunity_parent_contact | RemoveLink |

### Field Analysis Table

| Field Name | Data Type | Nullable | Default | Validation | Business Meaning |
|------------|-----------|---------|---------|------------|-----------------|
| OpportunityId | uniqueidentifier | No | NEWSEQUENTIALID() | Required | Unique identifier |
| Name | nvarchar(200) | No | None | Required | Opportunity name |
| OwningBusinessUnit | uniqueidentifier | Yes | NULL | FK validation | Security boundary |
| OwnerId | uniqueidentifier | No | None | FK validation | Record owner |
| OwnerIdType | int | No | None | 0=User, 1=Team | Owner type |
| StateCode | int | No | 0 | State machine | Pipeline state |
| StatusCode | int | Yes | NULL | State-dependent | Status reason |
| CustomerId | uniqueidentifier | Yes | NULL | Polymorphic FK | Customer (Account/Contact) |
| CustomerIdType | int | Yes | NULL | 1=Account, 2=Contact | Customer type |
| CustomerIdName | nvarchar(200) | Yes | NULL | Denormalized | Customer name |
| CustomerIdYomiName | nvarchar(200) | Yes | NULL | Denormalized | Phonetic name |
| PriceLevelId | uniqueidentifier | Yes | NULL | FK validation | Price list |
| TransactionCurrencyId | uniqueidentifier | Yes | NULL | FK validation | Currency |
| ExchangeRate | decimal(23,10) | Yes | NULL | > 0 | Exchange rate |
| EstimatedValue | money | Yes | NULL | >= 0 | Projected revenue |
| EstimatedValue_Base | money | Yes | NULL | >= 0 | Base currency value |
| ActualValue | money | Yes | NULL | >= 0 | Closed revenue |
| ActualValue_Base | money | Yes | NULL | >= 0 | Base currency actual |
| CloseProbability | int | Yes | NULL | 0-100 | Win probability % |
| EstimatedCloseDate | datetime | Yes | NULL | Future date | Expected close |
| ActualCloseDate | datetime | Yes | NULL | Valid date | Actual close date |
| StepId | uniqueidentifier | Yes | NULL | FK validation | Sales process step |
| StepName | nvarchar(100) | Yes | NULL | Max 100 | Step display name |
| SalesStageCode | int | Yes | NULL | Picklist | Sales stage |
| SalesStage | nvarchar(100) | Yes | NULL | Stage name | |
| IsRevenueSystemCalculated | bit | Yes | 1 | 0/1 | Revenue calculation method |
| TotalAmount | money | Yes | NULL | Calculated | Total including tax/freight |
| TotalAmount_Base | money | Yes | NULL | Calculated | Base currency |
| DiscountAmount | money | Yes | NULL | >= 0 | Order-level discount |
| DiscountPercentage | decimal(5,2) | Yes | NULL | 0-100 | Discount % |
| FreightAmount | money | Yes | NULL | >= 0 | Shipping cost |
| TotalTax | money | Yes | NULL | Calculated | Tax amount |
| TotalLineItemAmount | money | Yes | NULL | Sum of lines | Subtotal |
| TotalLineItemDiscountAmount | money | Yes | NULL | Sum of line discounts | |
| TotalDiscountAmount | money | Yes | NULL | Combined discount | |
| TotalAmountLessFreight | money | Yes | NULL | Calculated | Total minus freight |
| VersionNumber | timestamp | Yes | NULL | Auto-increment | Concurrency |
| TraversedPath | nvarchar(1250) | Yes | NULL | BPF tracking | Stage history |
| ProcessId | uniqueidentifier | Yes | NULL | FK validation | BPF instance |
| StageId | uniqueidentifier | Yes | NULL | FK validation | Current stage |

### State & Lifecycle

**Opportunity State Transitions**:

| StateCode | StateName | StatusCode | StatusName | Valid Transitions |
|-----------|-----------|------------|------------|-------------------|
| 0 | Open | 1 | New | → In Progress |
| 0 | Open | 2 | In Progress | → Won, On Hold, Lost |
| 0 | Open | 3 | On Hold | → In Progress, Won, Lost |
| 1 | Won | 4 | Won | Terminal - Create Order |
| 2 | Lost | 5 | Lost | Terminal |
| 2 | Lost | 6 | Cancelled | Terminal |
| 2 | Lost | 7 | Out-Sold | Terminal |

### Financial Calculation Logic

**Line Item Aggregation**:
```
TotalLineItemAmount = SUM(Quantity × UnitPrice) for all OpportunityProducts
TotalLineItemDiscountAmount = SUM(Quantity × UnitPrice × LineDiscount%) 
TotalDiscountAmount = TotalLineItemDiscountAmount + OrderDiscountAmount
Subtotal = TotalLineItemAmount - TotalLineItemDiscountAmount
TotalAmount = Subtotal - DiscountAmount + FreightAmount + TotalTax
EstimatedValue_Base = EstimatedValue × ExchangeRate
```

### Validation Explanation

**Required Fields**:
- OpportunityId: Auto-generated
- Name: Required for identification
- CustomerId: Required for opportunity (polymorphic)
- OwnerId: Required for security
- StateCode: Defaults to 0 (Open)

**Financial Precision**:
- Money type uses 4 decimal places for currency amounts
- ExchangeRate decimal(23,10) provides high precision for conversion
- Base currency fields (_Base) preserve organization's reporting currency values

---

## ENTITY 5: QUOTE

### Business Purpose

The Quote entity represents a formal sales quotation sent to customers detailing products, pricing, terms, and conditions. Quotes are typically created from Opportunities and serve as the formal proposal in the sales process. They support revision tracking through RevisionNumber and有效期 management through EffectiveFrom/EffectiveTo dates.

### Table Information

- **Table Name**: QuoteBase
- **Object Type Code**: 1084
- **Primary Key**: QuoteId (uniqueidentifier)
- **Ownership Model**: User or Team ownership

### Primary Key Definition

```sql
[QuoteId] [uniqueidentifier] NOT NULL
CONSTRAINT [PK_QuoteBase] PRIMARY KEY CLUSTERED ([QuoteId])
```

### Foreign Keys

| Foreign Key Column | Referenced Table | Constraint Name | Behavior |
|-------------------|------------------|-----------------|----------|
| OwningBusinessUnit | BusinessUnitBase | business_unit_quotes | Restrict |
| OwnerId | OwnerBase | owner_quotes | Restrict |
| CustomerId | AccountBase/ContactBase | polymorphic | RemoveLink |
| CustomerIdType | int | N/A | Polymorphic |
| PriceLevelId | PriceLevelBase | price_level_quotes | RemoveLink |
| TransactionCurrencyId | TransactionCurrencyBase | transactioncurrency_quote | RemoveLink |
| OpportunityId | OpportunityBase | opportunity_quotes | RemoveLink |

### Field Analysis Table

| Field Name | Data Type | Nullable | Default | Validation | Business Meaning |
|------------|-----------|---------|---------|------------|-----------------|
| QuoteId | uniqueidentifier | No | NEWSEQUENTIALID() | Required | Unique identifier |
| QuoteNumber | nvarchar(20) | Yes | NULL | Auto-generated | System quote number |
| RevisionNumber | int | No | 0 | >= 0 | Version number |
| Name | nvarchar(200) | No | None | Required | Quote name |
| OwningBusinessUnit | uniqueidentifier | Yes | NULL | FK | Security |
| OwnerId | uniqueidentifier | No | None | FK | Owner |
| StateCode | int | No | 0 | State machine | Quote state |
| StatusCode | int | Yes | NULL | State-dep | Status |
| CustomerId | uniqueidentifier | Yes | NULL | Polymorphic | Customer |
| CustomerIdType | int | Yes | NULL | Type | Customer type |
| PriceLevelId | uniqueidentifier | Yes | NULL | FK | Price list |
| TransactionCurrencyId | uniqueidentifier | Yes | NULL | FK | Currency |
| ExchangeRate | decimal(23,10) | Yes | NULL | > 0 | Exchange rate |
| TotalAmount | money | Yes | NULL | Calculated | Total including tax |
| TotalAmount_Base | money | Yes | NULL | Calculated | Base currency |
| DiscountAmount | money | Yes | NULL | >= 0 | Order discount |
| DiscountPercentage | decimal(5,2) | Yes | NULL | 0-100 | Discount % |
| FreightAmount | money | Yes | NULL | >= 0 | Shipping |
| TotalTax | money | Yes | NULL | Calculated | Tax |
| TotalLineItemAmount | money | Yes | NULL | Sum | Lines subtotal |
| TotalLineItemDiscountAmount | money | Yes | NULL | Sum | Line discounts |
| TotalDiscountAmount | money | Yes | NULL | Combined | Total discount |
| TotalAmountLessFreight | money | Yes | NULL | Calculated | Net total |
| EffectiveFrom | datetime | Yes | NULL | Valid date | Start date |
| EffectiveTo | datetime | Yes | NULL | >= EffectiveFrom | End date |
| ExpiresOn | datetime | Yes | NULL | Valid date | Expiration |
| RequestDeliveryBy | datetime | Yes | NULL | Valid date | Requested delivery |
| ClosedOn | datetime | Yes | NULL | Date closed | Close date |
| ShippingMethodCode | int | Yes | NULL | Picklist | Shipping method |
| PaymentTermsCode | int | Yes | NULL | Picklist | Payment terms |
| WillCall | bit | Yes | 0 | 0/1 | Pickup flag |
| ShipTo_Name | nvarchar(200) | Yes | NULL | Max 200 | Ship-to name |
| ShipTo_Address | nvarchar(250) | Yes | NULL | Address | Ship-to address |
| ShipTo_City | nvarchar(80) | Yes | NULL | City | Ship-to city |
| ShipTo_StateOrProvince | nvarchar(50) | Yes | NULL | State | Ship-to state |
| ShipTo_PostalCode | nvarchar(40) | Yes | NULL | Postal | Ship-to postal |
| ShipTo_Country | nvarchar(80) | Yes | NULL | Country | Ship-to country |
| BillTo_Name | nvarchar(200) | Yes | NULL | Bill-to name | |
| BillTo_Address | nvarchar(250) | Yes | NULL | Bill-to address | |
| BillTo_City | nvarchar(80) | Yes | NULL | Bill-to city | |
| BillTo_StateOrProvince | nvarchar(50) | Yes | NULL | Bill-to state | |
| BillTo_PostalCode | nvarchar(40) | Yes | NULL | Bill-to postal | |
| BillTo_Country | nvarchar(80) | Yes | NULL | Bill-to country | |
| VersionNumber | timestamp | Yes | NULL | Auto | Concurrency |
| ProcessId | uniqueidentifier | Yes | NULL | FK | BPF |
| StageId | uniqueidentifier | Yes | NULL | FK | Stage |
| TraversedPath | nvarchar(1250) | Yes | NULL | BPF | Stage history |

### State & Lifecycle

| StateCode | StateName | StatusCode | StatusName |
|-----------|-----------|------------|------------|
| 0 | Open | 1 | In Progress |
| 0 | Open | 2 | In Progress |
| 1 | Won | 3 | Won |
| 1 | Won | 4 | Accepted |
| 1 | Won | 5 | Revised |
| 2 | Lost | 6 | Lost |
| 2 | Lost | 7 | Declined |
| 2 | Lost | 8 | Cancelled |

### Validation Explanation

**Quote Validation Rules**:
- QuoteNumber: Auto-generated via organization settings
- EffectiveTo must be >= EffectiveFrom
- ExpiresOn should be in future for active quotes
- TotalAmount calculation: Sum of line items + freight + tax - discounts

---

## ENTITY 6: SALESORDER

### Business Purpose

The SalesOrder entity represents a confirmed customer purchase order. It is created when a Quote is accepted or directly from an Opportunity. Sales Orders track fulfillment, shipping, and backend ERP integration for order processing. They maintain links to the originating Quote and Opportunity for complete sales traceability.

### Table Information

- **Table Name**: SalesOrderBase
- **Object Type Code**: 1088
- **Primary Key**: SalesOrderId (uniqueidentifier)

### Primary Key Definition

```sql
[SalesOrderId] [uniqueidentifier] NOT NULL
CONSTRAINT [PK_SalesOrderBase] PRIMARY KEY CLUSTERED ([SalesOrderId])
CONSTRAINT [AK1_SalesOrderBase] UNIQUE NONCLUSTERED ([SalesOrderId])
```

### Foreign Keys

| Foreign Key Column | Referenced Table | Constraint Name | Behavior |
|-------------------|------------------|-----------------|----------|
| OwningBusinessUnit | BusinessUnitBase | business_unit_orders | Restrict |
| OwnerId | OwnerBase | owner_salesorders | Restrict |
| CustomerId | AccountBase/ContactBase | polymorphic | RemoveLink |
| CustomerIdType | int | N/A | Polymorphic |
| PriceLevelId | PriceLevelBase | price_level_orders | RemoveLink |
| TransactionCurrencyId | TransactionCurrencyBase | transactioncurrency_order | RemoveLink |
| OpportunityId | OpportunityBase | opportunity_orders | RemoveLink |
| QuoteId | OpportunityBase | quote_orders | RemoveLink |

### Field Analysis Table

| Field Name | Data Type | Nullable | Default | Validation | Business Meaning |
|------------|-----------|---------|---------|------------|-----------------|
| SalesOrderId | uniqueidentifier | No | NEWSEQUENTIALID() | Required | Unique ID |
| OrderNumber | nvarchar(20) | Yes | NULL | Auto-generated | System order # |
| OwningBusinessUnit | uniqueidentifier | Yes | NULL | FK | Security |
| OwnerId | uniqueidentifier | No | None | FK | Owner |
| OwnerIdType | int | No | None | Owner type | |
| StateCode | int | No | 0 | State machine | Order state |
| StatusCode | int | Yes | NULL | State-dep | Status |
| CustomerId | uniqueidentifier | Yes | NULL | Polymorphic | Customer |
| CustomerIdType | int | Yes | NULL | 1=Account, 2=Contact | Type |
| PriceLevelId | uniqueidentifier | Yes | NULL | FK | Price list |
| TransactionCurrencyId | uniqueidentifier | Yes | NULL | FK | Currency |
| ExchangeRate | decimal(23,10) | Yes | NULL | > 0 | Rate |
| TotalAmount | money | Yes | NULL | Calculated | Total |
| TotalAmount_Base | money | Yes | NULL | Calculated | Base |
| DiscountAmount | money | Yes | NULL | >= 0 | Discount |
| DiscountPercentage | decimal(5,2) | Yes | NULL | 0-100 | % |
| FreightAmount | money | Yes | NULL | >= 0 | Shipping |
| TotalTax | money | Yes | NULL | Calculated | Tax |
| TotalLineItemAmount | money | Yes | NULL | Sum | Lines |
| TotalLineItemDiscountAmount | money | Yes | NULL | Sum | Line disc |
| TotalDiscountAmount | money | Yes | NULL | Combined | Total disc |
| TotalAmountLessFreight | money | Yes | NULL | Calculated | Net |
| OpportunityId | uniqueidentifier | Yes | NULL | FK | Source opp |
| QuoteId | uniqueidentifier | Yes | NULL | FK | Source quote |
| RequestDeliveryBy | datetime | Yes | NULL | Valid date | Requested |
| SubmitStatus | int | Yes | NULL | ERP status | Integration |
| SubmitStatusDescription | nvarchar(200) | Yes | NULL | Status text | |
| SubmitDate | datetime | Yes | NULL | Date submitted | |
| LastBackofficeSubmit | datetime | Yes | NULL | Last submit | |
| ShippingMethodCode | int | Yes | NULL | Picklist | Ship method |
| PaymentTermsCode | int | Yes | NULL | Picklist | Terms |
| ShipTo_Name | nvarchar(200) | Yes | NULL | Ship-to | |
| ShipTo_Address | nvarchar(250) | Yes | NULL | Address | |
| ShipTo_City | nvarchar(80) | Yes | NULL | City | |
| ShipTo_StateOrProvince | nvarchar(50) | Yes | NULL | State | |
| ShipTo_PostalCode | nvarchar(40) | Yes | NULL | Postal | |
| ShipTo_Country | nvarchar(80) | Yes | NULL | Country | |
| BillTo_Name | nvarchar(200) | Yes | NULL | Bill-to | |
| BillTo_Address | nvarchar(250) | Yes | NULL | Address | |
| BillTo_City | nvarchar(80) | Yes | NULL | City | |
| BillTo_StateOrProvince | nvarchar(50) | Yes | NULL | State | |
| BillTo_PostalCode | nvarchar(40) | Yes | NULL | Postal | |
| BillTo_Country | nvarchar(80) | Yes | NULL | Country | |
| VersionNumber | timestamp | Yes | NULL | Auto | Concurrency |
| ProcessId | uniqueidentifier | Yes | NULL | FK | BPF |
| StageId | uniqueidentifier | Yes | NULL | FK | Stage |

### State & Lifecycle

| StateCode | StateName | StatusCode | StatusName |
|-----------|-----------|------------|------------|
| 0 | Active | 1 | New |
| 0 | Active | 2 | Pending |
| 0 | Active | 3 | Submitted |
| 0 | Active | 4 | Cancelled |
| 1 | Fulfilled | 5 | Fulfilled |
| 2 | Invoiced | 6 | Invoiced |
| 3 | Paid | 7 | Paid |
| 4 | Cancelled | 8 | Cancelled |

### Backend Integration

**SubmitStatus Values**:
| Value | Meaning |
|-------|---------|
| 0 | None - Not submitted |
| 1 | Submitted - Sent to ERP |
| 2 | Accepted - Accepted by ERP |
| 3 | Rejected - Rejected by ERP |

---

## ENTITY 7: INVOICE

### Business Purpose

The Invoice entity represents a billing document requesting payment from customers. Invoices are generated from fulfilled Sales Orders and track payment terms, due dates, and payment status. They represent the revenue recognition point in the sales cycle and integrate with financial systems for accounting.

### Table Information

- **Table Name**: InvoiceBase
- **Object Type Code**: 1090
- **Primary Key**: InvoiceId (uniqueidentifier)

### Primary Key Definition

```sql
[InvoiceId] [uniqueidentifier] NOT NULL
CONSTRAINT [PK_InvoiceBase] PRIMARY KEY CLUSTERED ([InvoiceId])
```

### Foreign Keys

| Foreign Key Column | Referenced Table | Constraint Name | Behavior |
|-------------------|------------------|-----------------|----------|
| OwningBusinessUnit | BusinessUnitBase | business_unit_invoices | Restrict |
| OwnerId | OwnerBase | owner_invoices | Restrict |
| CustomerId | AccountBase/ContactBase | polymorphic | RemoveLink |
| CustomerIdType | int | N/A | Polymorphic |
| PriceLevelId | PriceLevelBase | price_level_invoices | RemoveLink |
| TransactionCurrencyId | TransactionCurrencyBase | transactioncurrency_invoice | RemoveLink |
| SalesOrderId | SalesOrderBase | order_invoices | RemoveLink |
| OpportunityId | OpportunityBase | opportunity_invoices | RemoveLink |

### Field Analysis Table

| Field Name | Data Type | Nullable | Default | Validation | Business Meaning |
|------------|-----------|---------|---------|------------|-----------------|
| InvoiceId | uniqueidentifier | No | NEWSEQUENTIALID() | Required | Unique ID |
| InvoiceNumber | nvarchar(20) | Yes | NULL | Auto-generated | Invoice # |
| OwningBusinessUnit | uniqueidentifier | Yes | NULL | FK | Security |
| OwnerId | uniqueidentifier | No | None | FK | Owner |
| OwnerIdType | int | No | None | Owner type | |
| StateCode | int | No | 0 | State machine | Invoice state |
| StatusCode | int | Yes | NULL | State-dep | Status |
| CustomerId | uniqueidentifier | Yes | NULL | Polymorphic | Customer |
| CustomerIdType | int | Yes | NULL | 1=Account, 2=Contact | Type |
| PriceLevelId | uniqueidentifier | Yes | NULL | FK | Price list |
| TransactionCurrencyId | uniqueidentifier | Yes | NULL | FK | Currency |
| ExchangeRate | decimal(23,10) | Yes | NULL | > 0 | Rate |
| TotalAmount | money | Yes | NULL | Calculated | Invoice total |
| TotalAmount_Base | money | Yes | NULL | Calculated | Base |
| DiscountAmount | money | Yes | NULL | >= 0 | Discount |
| DiscountPercentage | decimal(5,2) | Yes | NULL | 0-100 | % |
| FreightAmount | money | Yes | NULL | >= 0 | Shipping |
| TotalTax | money | Yes | NULL | Calculated | Tax |
| TotalLineItemAmount | money | Yes | NULL | Sum | Lines |
| TotalLineItemDiscountAmount | money | Yes | NULL | Sum | Line disc |
| TotalDiscountAmount | money | Yes | NULL | Combined | Total disc |
| TotalAmountLessFreight | money | Yes | NULL | Calculated | Net |
| SalesOrderId | uniqueidentifier | Yes | NULL | FK | Source order |
| OpportunityId | uniqueidentifier | Yes | NULL | FK | Source opp |
| PaymentTermsCode | int | Yes | NULL | Picklist | Terms |
| DueDate | datetime | Yes | NULL | Valid date | Payment due |
| DateSent | datetime | Yes | NULL | Date sent | |
| PaidOn | datetime | Yes | NULL | Date paid | |
| ShippingMethodCode | int | Yes | NULL | Picklist | Ship method |
| ShipTo_Name | nvarchar(200) | Yes | NULL | Ship-to | |
| ShipTo_Address | nvarchar(250) | Yes | NULL | Address | |
| ShipTo_City | nvarchar(80) | Yes | NULL | City | |
| ShipTo_StateOrProvince | nvarchar(50) | Yes | NULL | State | |
| ShipTo_PostalCode | nvarchar(40) | Yes | NULL | Postal | |
| ShipTo_Country | nvarchar(80) | Yes | NULL | Country | |
| BillTo_Name | nvarchar(200) | Yes | NULL | Bill-to | |
| BillTo_Address | nvarchar(250) | Yes | NULL | Address | |
| BillTo_City | nvarchar(80) | Yes | NULL | City | |
| BillTo_StateOrProvince | nvarchar(50) | Yes | NULL | State | |
| BillTo_PostalCode | nvarchar(40) | Yes | NULL | Postal | |
| BillTo_Country | nvarchar(80) | Yes | NULL | Country | |
| VersionNumber | timestamp | Yes | NULL | Auto | Concurrency |
| ProcessId | uniqueidentifier | Yes | NULL | FK | BPF |
| StageId | uniqueidentifier | Yes | NULL | FK | Stage |

### State & Lifecycle

| StateCode | StateName | StatusCode | StatusName |
|-----------|-----------|------------|------------|
| 0 | Active | 1 | New |
| 0 | Active | 2 | Partial |
| 0 | Active | 3 | Complete |
| 1 | Paid | 4 | Paid |
| 1 | Paid | 5 | Partial |
| 2 | Cancelled | 6 | Cancelled |

---

## ENTITY 8: OPPORTUNITYPRODUCT (Line Items)

### Business Purpose

The OpportunityProduct entity represents individual products or services sold within an opportunity. Each line item includes product pricing, quantity, discounts, and tax calculations. The aggregate of all opportunity products determines the total opportunity value.

### Table Information

- **Table Name**: OpportunityProductBase
- **Object Type Code**: 1003
- **Primary Key**: OpportunityProductId (uniqueidentifier)

### Foreign Keys

| Foreign Key Column | Referenced Table | Behavior |
|-------------------|------------------|----------|
| OpportunityId | OpportunityBase | Cascade |
| ProductId | ProductBase | RemoveLink |
| UoMId | UoMBase | RemoveLink |
| PricingErrorCode | Picklist | N/A |
| SalesUnitId | UoMScheduleBase | RemoveLink |

### Field Analysis Table

| Field Name | Data Type | Nullable | Validation | Business Meaning |
|------------|-----------|---------|------------|-----------------|
| OpportunityProductId | uniqueidentifier | No | Required, Unique | Line ID |
| OpportunityId | uniqueidentifier | No | FK Required | Parent opportunity |
| ProductId | uniqueidentifier | Yes | FK | Product reference |
| IsProductOverridden | bit | Yes | 0/1 | Custom price |
| ProductName | nvarchar(200) | Yes | Max 200 | Display name |
| UnitPrice | money | Yes | >= 0 | Price per unit |
| ExtendedAmount | money | Yes | Calculated | Qty × Price |
| Quantity | decimal(5,2) | No | > 0 | Quantity ordered |
| UoMId | uniqueidentifier | Yes | FK | Unit of measure |
| UnitScheduleId | uniqueidentifier | Yes | FK | Pricing schedule |
| LineItemDiscount | money | Yes | >= 0 | Line discount |
| LineItemDiscountPercentage | decimal(5,2) | Yes | 0-100 | Discount % |
| Tax | money | Yes | Calculated | Tax amount |
| VolDiscount | money | Yes | Volume discount | |
| ManualDiscountAmount | money | Yes | Manual discount | |
| Amount | money | Yes | Calculated | Net amount | |
| BaseAmount | money | Yes | Base currency | |
| PriceLevelId | uniqueidentifier | Yes | FK | Price list |
| QuantityBackOrder | decimal(5,2) | Yes | Backordered qty | |
| QuantityCancelled | decimal(5,2) | Yes | Cancelled qty | |
| QuantityShipped | decimal(5,2) | Yes | Shipped qty | |
| VersionNumber | timestamp | Yes | Auto | Concurrency |

---

## ENTITY 9: QUOTEDETAIL (Quote Line Items)

### Business Purpose

QuoteDetail represents individual line items within a quote, mirroring the opportunity product structure but specific to the quote context. Each detail line references a product, quantity, pricing, and calculates extended amounts.

### Table Information

- **Table Name**: QuoteDetailBase
- **Object Type Code**: 1085
- **Primary Key**: QuoteDetailId (uniqueidentifier)

### Foreign Keys

| Foreign Key Column | Referenced Table | Behavior |
|-------------------|------------------|----------|
| QuoteId | QuoteBase | Cascade |
| ProductId | ProductBase | RemoveLink |
| UoMId | UoMBase | RemoveLink |

### Field Analysis Table

| Field Name | Data Type | Nullable | Validation | Business Meaning |
|------------|-----------|---------|------------|-----------------|
| QuoteDetailId | uniqueidentifier | No | Required | Line ID |
| QuoteId | uniqueidentifier | No | FK Required | Parent quote |
| ProductId | uniqueidentifier | Yes | FK | Product |
| IsProductOverridden | bit | Yes | Custom pricing flag | |
| ProductName | nvarchar(200) | Yes | Display name | |
| UnitPrice | money | Yes | Unit price | |
| ExtendedAmount | money | Yes | Qty × Price | |
| Quantity | decimal(5,2) | No | > 0 | Qty |
| UoMId | uniqueidentifier | Yes | FK | Unit |
| LineItemDiscount | money | Yes | Line discount | |
| LineItemDiscountPercentage | decimal(5,2) | Yes | 0-100 | Discount % |
| Tax | money | Yes | Tax | |
| Description | nvarchar(max) | Yes | Line description | |
| QuoteDetailId | uniqueidentifier | Yes | Parent bundle | |
| VersionNumber | timestamp | Yes | Concurrency | |

---

## ENTITY 10: ORDERDETAIL (Sales Order Line Items)

### Business Purpose

OrderDetail represents line items on a sales order, containing product, quantity, pricing, and fulfillment tracking information.

### Table Information

- **Table Name**: SalesOrderDetailBase
- **Object Type Code**: 1089
- **Primary Key**: SalesOrderDetailId (uniqueidentifier)

### Foreign Keys

| Foreign Key Column | Referenced Table | Behavior |
|-------------------|------------------|----------|
| SalesOrderId | SalesOrderBase | Cascade |
| ProductId | ProductBase | RemoveLink |
| UoMId | UoMBase | RemoveLink |
| SalesRepId | SystemUserBase | RemoveLink |

---

## ENTITY 11: INVOICEDETAIL (Invoice Line Items)

### Business Purpose

InvoiceDetail represents line items on an invoice, tracking products, quantities, pricing, and billing information.

### Table Information

- **Table Name**: InvoiceDetailBase
- **Object Type Code**: 1091
- **Primary Key**: InvoiceDetailId (uniqueidentifier)

### Foreign Keys

| Foreign Key Column | Referenced Table | Behavior |
|-------------------|------------------|----------|
| InvoiceId | InvoiceBase | Cascade |
| ProductId | ProductBase | RemoveLink |
| UoMId | UoMBase | RemoveLink |
| ProductAssociationId | ProductAssociationBase | RemoveLink |
| SalesRepId | SystemUserBase | RemoveLink |

---

## ENTITY 12: PRODUCT

### Business Purpose

The Product entity represents items or services available for sale. Products are organized in a hierarchy and linked to price lists for pricing.

### Table Information

- **Table Name**: ProductBase
- **Object Type Code**: 1024
- **Primary Key**: ProductId (uniqueidentifier)

### Foreign Keys

| Foreign Key Column | Referenced Table | Behavior |
|-------------------|------------------|----------|
| DefaultUoMId | UoMBase | RemoveLink |
| DefaultUoMScheduleId | UoMScheduleBase | RemoveLink |
| PriceLevelId | PriceLevelBase | RemoveLink |
| TransactionCurrencyId | TransactionCurrencyBase | RemoveLink |

### Field Analysis Table

| Field Name | Data Type | Nullable | Validation | Business Meaning |
|------------|-----------|---------|------------|-----------------|
| ProductId | uniqueidentifier | No | Unique | Product ID |
| Name | nvarchar(200) | No | Required | Product name |
| ProductNumber | nvarchar(100) | Yes | Unique | SKU/Part # |
| StateCode | int | No | State | Product state |
| StatusCode | int | Yes | Status | Status reason |
| ProductTypeCode | int | Yes | Picklist | Type (Product/Service) |
| QuantityDecimal | decimal(5,2) | Yes | Decimal precision | |
| DefaultUoMId | uniqueidentifier | Yes | FK | Default unit |
| Price | money | Yes | Default price | |
| Cost | money | Yes | Cost amount | |
| StandardCost | money | Yes | Standard cost | |
| CurrentCost | money | Yes | Current cost | |
| PriceLevelId | uniqueidentifier | Yes | FK | Default price list |
| Amount | money | Yes | List price | |
| VendorId | uniqueidentifier | Yes | Primary vendor | |
| VendorName | nvarchar(100) | Yes | Vendor name | |
| ProductUrl | nvarchar(200) | Yes | Product URL | |
| Description | nvarchar(max) | Yes | Description | |
| HierarchyId | uniqueidentifier | Yes | Product hierarchy | |
| ParentProductId | uniqueidentifier | Yes | Parent product | |
| QuantityOnHand | int | Yes | Inventory qty | |
| VersionNumber | timestamp | Yes | Concurrency | |

---

## ENTITY 13: PRICELIST (PriceLevel)

### Business Purpose

The PriceLevel (PriceList) entity defines pricing structures for products, containing effective dates and currency information.

### Table Information

- **Table Name**: PriceLevelBase
- **Object Type Code**: 1022
- **Primary Key**: PriceLevelId (uniqueidentifier)

### Foreign Keys

| Foreign Key Column | Referenced Table | Behavior |
|-------------------|------------------|----------|
| TransactionCurrencyId | TransactionCurrencyBase | RemoveLink |

### Field Analysis Table

| Field Name | Data Type | Nullable | Validation | Business Meaning |
|------------|-----------|---------|------------|-----------------|
| PriceLevelId | uniqueidentifier | No | Unique | Price list ID |
| Name | nvarchar(100) | No | Required | List name |
| StateCode | int | No | State | List state |
| StatusCode | int | Yes | Status | Status |
| BeginDate | datetime | Yes | Valid date | Start date |
| EndDate | datetime | Yes | >= BeginDate | End date |
| TransactionCurrencyId | uniqueidentifier | Yes | FK | Currency |
| IsDefault | bit | Yes | Default flag | |
| VersionNumber | timestamp | Yes | Concurrency | |

---

## ENTITY 14: ACTIVITYPOINTER

### Business Purpose

The ActivityPointer entity is the base entity for all activity types (Email, Phone Call, Task, Appointment, etc.). It tracks communications and interactions with customers.

### Table Information

- **Table Name**: ActivityPointerBase
- **Object Type Code**: 4200
- **Primary Key**: ActivityId (uniqueidentifier)

### Foreign Keys

| Foreign Key Column | Referenced Table | Behavior |
|-------------------|------------------|----------|
| OwningBusinessUnit | BusinessUnitBase | Restrict |
| OwnerId | OwnerBase | Restrict |
| TransactionCurrencyId | TransactionCurrencyBase | RemoveLink |
| RegardingObjectId | Various | Polymorphic |
| RegardingObjectTypeCode | int | N/A |
| ServiceId | ServiceBase | RemoveLink |
| SiteId | SiteBase | RemoveLink |
| OriginatingActivityId | ActivityPointerBase | Self-ref |

### Field Analysis Table

| Field Name | Data Type | Nullable | Validation | Business Meaning |
|------------|-----------|---------|------------|-----------------|
| ActivityId | uniqueidentifier | No | Unique | Activity ID |
| ActivityTypeCode | nvarchar(100) | No | Required | Activity type |
| OwningBusinessUnit | uniqueidentifier | Yes | FK | Security BU |
| OwnerId | uniqueidentifier | No | FK | Owner |
| OwnerIdType | int | No | Owner type | |
| StateCode | int | No | State | Activity state |
| StatusCode | int | Yes | Status | Status |
| Subject | nvarchar(200) | Yes | Max 200 | Subject |
| Description | nvarchar(max) | Yes | Description | |
| ScheduledStart | datetime | Yes | Start time | |
| ScheduledEnd | datetime | Yes | End time | |
| ActualStart | datetime | Yes | Actual start | |
| ActualEnd | datetime | Yes | Actual end | |
| PriorityCode | int | Yes | Picklist | Priority |
| PercentComplete | int | Yes | 0-100 | % complete |
| IsBilled | bit | Yes | Billed flag | |
| IsWorkflowCreated | bit | Yes | Workflow-created | |
| DirectionCode | bit | Yes | 0=In, 1=Out | Direction |
| ScheduledDurationMinutes | int | Yes | Duration | |
| ActualDurationMinutes | int | Yes | Actual duration | |
| RegardingObjectId | uniqueidentifier | Yes | Polymorphic | Related record |
| RegardingObjectTypeCode | int | Yes | Type code | |
| TransactionCurrencyId | uniqueidentifier | Yes | FK | Currency |
| ExchangeRate | decimal(23,10) | Yes | Exchange rate | |
| TimeZoneRuleVersionNumber | int | Yes | TZ version | |
| VersionNumber | timestamp | Yes | Concurrency | |

### State & Lifecycle

| StateCode | StateName | Meaning |
|-----------|-----------|---------|
| 0 | Open | Activity pending |
| 1 | Completed | Activity done |
| 2 | Cancelled | Activity cancelled |

---

## ENTITY 15: ACTIVITYPARTY

### Business Purpose

ActivityParty tracks participants in activities, including senders, recipients, attendees, and organizers. This enables multi-party communications.

### Table Information

- **Table Name**: ActivityPartyBase
- **Object Type Code**: 135
- **Primary Key**: Composite (ActivityId, PartyId, ParticipationTypeMask)

### Field Analysis Table

| Field Name | Data Type | Nullable | Validation | Business Meaning |
|------------|-----------|---------|------------|-----------------|
| ActivityId | uniqueidentifier | No | FK Required | Activity |
| PartyId | uniqueidentifier | Yes | FK | Participant |
| ParticipationTypeMask | int | No | Picklist | Role type |
| AddressUsed | nvarchar(200) | Yes | Email/Phone | Address used |
| AddressUsedEmailColumn | int | Yes | Email column | |
| DoNotSend | bit | Yes | 0/1 | Opt-out |

### ParticipationTypeMask Values

| Value | Meaning |
|-------|---------|
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

---

## ENTITY 16: ANNOTATION (Notes/Attachments)

### Business Purpose

Annotation stores notes and attachments associated with any CRM record. Provides rich text notes and file attachments.

### Table Information

- **Table Name**: AnnotationBase
- **Object Type Code**: 1000
- **Primary Key**: AnnotationId (uniqueidentifier)

### Foreign Keys

| Foreign Key Column | Referenced Table | Behavior |
|-------------------|------------------|----------|
| OwningBusinessUnit | BusinessUnitBase | Restrict |
| OwnerId | OwnerBase | Restrict |
| ObjectId | Various | Polymorphic |
| ObjectTypeCode | int | N/A |

### Field Analysis Table

| Field Name | Data Type | Nullable | Validation | Business Meaning |
|------------|-----------|---------|------------|-----------------|
| AnnotationId | uniqueidentifier | No | Unique | Note ID |
| Subject | nvarchar(200) | Yes | Subject | Note subject |
| NoteText | nvarchar(max) | Yes | Text content | Note body |
| FileName | nvarchar(255) | Yes | File name | Attachment |
| FileSize | int | Yes | Size bytes | File size |
| MimeType | nvarchar(50) | Yes | MIME type | File type |
| Body | varbinary(max) | Yes | Binary | File content |
| DocumentBody | nvarchar(max) | Yes | Base64 | Encoded file |
| IsDocument | bit | Yes | Document flag | |
| LangId | int | Yes | Language | |
| OwningBusinessUnit | uniqueidentifier | Yes | FK | Security |
| OwnerId | uniqueidentifier | Yes | FK | Owner |
| ObjectId | uniqueidentifier | Yes | Polymorphic | Regarding |
| ObjectTypeCode | int | Yes | Type code | |
| CreatedOn | datetime | Yes | Created | |
| CreatedBy | uniqueidentifier | Yes | Creator | |
| ModifiedOn | datetime | Yes | Modified | |
| ModifiedBy | uniqueidentifier | Yes | Modifier | |
| VersionNumber | timestamp | Yes | Concurrency | |

---

## ENTITY 17: SYSTEMUSER

### Business Purpose

SystemUser represents individual users who access the CRM system. Users are assigned security roles and own records within the CRM.

### Table Information

- **Table Name**: SystemUserBase
- **Object Type Code**: 8
- **Primary Key**: SystemUserId (uniqueidentifier)

### Foreign Keys

| Foreign Key Column | Referenced Table | Behavior |
|-------------------|------------------|----------|
| BusinessUnitId | BusinessUnitBase | Restrict |
| CalendarId | CalendarBase | RemoveLink |
| TransactionCurrencyId | TransactionCurrencyBase | RemoveLink |
| TerritoryId | TerritoryBase | RemoveLink |

### Field Analysis Table

| Field Name | Data Type | Nullable | Validation | Business Meaning |
|------------|-----------|---------|------------|-----------------|
| SystemUserId | uniqueidentifier | No | Unique | User ID |
| DomainName | nvarchar(400) | Yes | Domain\User | AD username |
| FirstName | nvarchar(50) | Yes | First name | |
| LastName | nvarchar(50) | Yes | Last name | |
| FullName | nvarchar(100) | Yes | Display name | |
| YomiFullName | nvarchar(100) | Yes | Phonetic | |
| BusinessUnitId | uniqueidentifier | Yes | FK | Primary BU |
| Title | nvarchar(100) | Yes | Job title | |
| EmailAddress1 | nvarchar(100) | Yes | Email | |
| IsDisabled | bit | Yes | Disabled flag | |
| UserLicenseType | int | Yes | License type | |
| AccessMode | int | Yes | Access mode | |
| IncomingEmailDeliveryMethod | int | Yes | Email method | |
| OutgoingEmailDeliveryMethod | int | Yes | Email method | |
| CalendarId | uniqueidentifier | Yes | FK | Calendar |
| TimeZoneCode | int | Yes | Timezone | |
| UserGroupId | uniqueidentifier | Yes | Group | |
| PositionId | uniqueidentifier | Yes | Position | |
| TerritoryId | uniqueidentifier | Yes | Territory | |
| CreatedOn | datetime | Yes | Created | |
| ModifiedOn | datetime | Yes | Modified | |
| VersionNumber | timestamp | Yes | Concurrency | |

---

## ENTITY 18: BUSINESSUNIT

### Business Purpose

BusinessUnit represents organizational divisions within the company structure. It is the primary security boundary in CRM, with users and records assigned to business units.

### Table Information

- **Table Name**: BusinessUnitBase
- **Object Type Code**: 101
- **Primary Key**: BusinessUnitId (uniqueidentifier)

### Foreign Keys

| Foreign Key Column | Referenced Table | Behavior |
|-------------------|------------------|----------|
| ParentBusinessUnitId | BusinessUnitBase | Self-ref |
| OrganizationId | OrganizationBase | Restrict |
| CalendarId | CalendarBase | RemoveLink |
| TransactionCurrencyId | TransactionCurrencyBase | RemoveLink |

### Field Analysis Table

| Field Name | Data Type | Nullable | Validation | Business Meaning |
|------------|-----------|---------|------------|-----------------|
| BusinessUnitId | uniqueidentifier | No | Unique | BU ID |
| Name | nvarchar(100) | No | Required | BU name |
| ParentBusinessUnitId | uniqueidentifier | Yes | Self-FK | Parent BU |
| OrganizationId | uniqueidentifier | No | FK | Root org |
| DivisionName | nvarchar(100) | Yes | Division | |
| Address1_Line1 | nvarchar(250) | Yes | Address | |
| Address1_City | nvarchar(80) | Yes | City | |
| Address1_StateOrProvince | nvarchar(50) | Yes | State | |
| Address1_PostalCode | nvarchar(40) | Yes | Postal | |
| Address1_Country | nvarchar(80) | Yes | Country | |
| Address1_Telephone1 | nvarchar(50) | Yes | Phone | |
| CalendarId | uniqueidentifier | Yes | FK | Work calendar |
| TransactionCurrencyId | uniqueidentifier | Yes | FK | Currency |
| VersionNumber | timestamp | Yes | Concurrency | |

---

## ENTITY 19: OWNER

### Business Purpose

OwnerBase is a polymorphic table serving as the single join point for record ownership. It stores references to SystemUser (user ownership) and Team (team ownership) records.

### Table Information

- **Table Name**: OwnerBase
- **Object Type Code**: N/A (polymorphic)
- **Primary Key**: OwnerId (uniqueidentifier)

### Field Analysis Table

| Field Name | Data Type | Nullable | Validation | Business Meaning |
|------------|-----------|---------|------------|-----------------|
| OwnerId | uniqueidentifier | No | Unique | Owner ID |
| OwnerIdType | int | No | 0/1/2 | Owner type |
| Name | nvarchar(100) | Yes | Owner name | |

### OwnerIdType Values

| Value | Meaning |
|-------|---------|
| 0 | User |
| 1 | Team |
| 2 | Organization |

---

## ENTITY 20: TRANSACTIONCURRENCY

### Business Purpose

TransactionCurrency stores currency definitions used for multi-currency support, including exchange rates and precision settings.

### Table Information

- **Table Name**: TransactionCurrencyBase
- **Object Type Code**: 9105
- **Primary Key**: TransactionCurrencyId (uniqueidentifier)

### Field Analysis Table

| Field Name | Data Type | Nullable | Validation | Business Meaning |
|------------|-----------|---------|------------|-----------------|
| TransactionCurrencyId | uniqueidentifier | No | Unique | Currency ID |
| ISOCurrencyCode | nvarchar(3) | No | Required | ISO code (USD) |
| CurrencySymbol | nvarchar(5) | Yes | Symbol ($) |
| CurrencyName | nvarchar(100) | Yes | Name | |
| Precision | int | Yes | Decimal places | |
| IsPrimaryCurrency | bit | Yes | Primary flag | |
| IsBaseCurrency | bit | Yes | Base flag | |
| ExchangeRate | decimal(23,10) | Yes | Default rate | |

---

# SECTION 3 – Relationship Architecture

## 3.1 One-to-Many (1:N) Relationships

The database implements comprehensive one-to-many relationships enabling hierarchical data organization:

**Account Hierarchy**:
- Account (ParentAccountId) → Account (self-referential)
- Account → Contact
- Account → Opportunity
- Account → Quote
- Account → SalesOrder
- Account → Invoice
- Account → Contract

**Opportunity Hierarchy**:
- Opportunity → OpportunityProduct (line items)
- Opportunity → Quote
- Opportunity → SalesOrder
- Opportunity → Invoice

**Contact Hierarchy**:
- Contact → Opportunity
- Contact → Lead

## 3.2 Many-to-Many (N:N) Relationships

Junction tables enable flexible many-to-many associations:

| Junction Table | Entity 1 | Entity 2 |
|---------------|----------|----------|
| AccountLeads | Account | Lead |
| ContactLeads | Contact | Lead |
| ContactQuotes | Contact | Quote |
| ContactOrders | Contact | SalesOrder |
| ContactInvoices | Contact | Invoice |
| LeadCompetitors | Lead | Competitor |
| LeadProduct | Lead | Product |

## 3.3 Polymorphic Relationships

Several entities use polymorphic lookups allowing a single column to reference multiple entity types:

**Opportunity.CustomerId**: Can reference Account or Contact
**Quote.CustomerId**: Can reference Account or Contact  
**SalesOrder.CustomerId**: Can reference Account or Contact
**Invoice.CustomerId**: Can reference Account or Contact

Resolution requires CustomerIdType column:
- CustomerIdType = 1: Account
- CustomerIdType = 2: Contact

## 3.4 Referential Integrity Enforcement

Foreign key constraints enforce referential integrity with various cascade behaviors:

| Behavior | Usage | Example |
|----------|-------|---------|
| Cascade | Delete child with parent | OpportunityProduct → Opportunity |
| Restricted | Prevent if children exist | Account → Contact |
| RemoveLink | Remove relationship only | Quote → Opportunity |
| None | No cascade | TransactionCurrency references |

---

# SECTION 4 – CRM Sales Flow Mapping

## 4.1 Complete Sales Pipeline

The following flow demonstrates data movement through the CRM sales process:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         LEAD → OPPORTUNITY                               │
├─────────────────────────────────────────────────────────────────────────┤
│  1. Lead created with qualification info                               │
│  2. Sales rep qualifies lead (StateCode → 1)                         │
│  3. System creates:                                                    │
│     - New Account (if new company)                                     │
│     - New Contact (if new person)                                      │
│     - New Opportunity (CustomerId → Account/Contact)                   │
│     - Lead.QualifyingOpportunityId → Opportunity.OpportunityId           │
│     - Opportunity.OriginatingLeadId → Lead.LeadId                      │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                    OPPORTUNITY → QUOTE                                  │
├─────────────────────────────────────────────────────────────────────────┤
│  1. Sales rep creates quote from opportunity                         │
│  2. System copies:                                                    │
│     - Quote.CustomerId ← Opportunity.CustomerId                        │
│     - Quote.PriceLevelId ← Opportunity.PriceLevelId                    │
│     - Quote.TransactionCurrencyId ← Opportunity.TransactionCurrencyId   │
│     - Quote.OpportunityId ← Opportunity.OpportunityId                  │
│     - Quote line items from OpportunityProducts                        │
│  3. Quote totals calculated from line items                            │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                       QUOTE → SALES ORDER                              │
├─────────────────────────────────────────────────────────────────────────┤
│  1. Customer accepts quote                                            │
│  2. Quote state changes to Won (StateCode → 1)                        │
│  3. System creates Sales Order:                                        │
│     - Order.CustomerId ← Quote.CustomerId                             │
│     - Order.PriceLevelId ← Quote.PriceLevelId                         │
│     - Order.QuoteId ← Quote.QuoteId                                   │
│     - Order.OpportunityId ← Quote.OpportunityId                       │
│     - Order line items from QuoteDetails                              │
│  4. Order totals copied from quote                                    │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                     SALES ORDER → INVOICE                              │
├─────────────────────────────────────────────────────────────────────────┤
│  1. Order fulfilled (StateCode → 1, Fulfilled)                      │
│  2. System creates Invoice:                                           │
│     - Invoice.CustomerId ← Order.CustomerId                          │
│     - Invoice.SalesOrderId ← Order.SalesOrderId                       │
│     - Invoice.OpportunityId ← Order.OpportunityId                      │
│     - Invoice line items from OrderDetails                            │
│  3. Invoice totals copied from order                                 │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                       INVOICE → PAYMENT                                │
├─────────────────────────────────────────────────────────────────────────┤
│  1. Customer payment received                                          │
│  2. Invoice state changes to Paid (StateCode → 1, StatusCode → 4)   │
│  3. Revenue recognized:                                                │
│     - Opportunity.ActualValue = won revenue                            │
│     - Opportunity.ActualCloseDate = close date                         │
│     - Opportunity.StateCode → 1 (Won)                                 │
└─────────────────────────────────────────────────────────────────────────┘
```

---

# SECTION 5 – Data Integrity & Validation Architecture

## 5.1 Database-Level Validation

**Primary Key Constraints**:
- All entities use uniqueidentifier (GUID) as primary key
- Auto-generated via NEWSEQUENTIALID() default constraint
- Clustered primary key index with FILLFACTOR = 80

**Foreign Key Constraints**:
- With CHECK clause for data integrity
- Named convention: [source]_[destination]
- Support cascade behaviors: Cascade, Restricted, RemoveLink, None

**Default Constraints**:
- StateCode defaults to 0 (Active/Open)
- OwnerIdType defaults to appropriate value
- Various date fields default to GETUTCDATE()

## 5.2 Application-Level Validation Assumptions

**Required Field Validation**:
- Application layer enforces NOT NULL on fields like Name, Subject
- Business rules validate required fields before save

**Picklist Validation**:
- Values stored in StringMapBase
- Application enforces valid option set values

**Format Validation**:
- Email format validation
- Phone number format validation
- URL format validation

## 5.3 Financial Calculation Integrity

**Multi-Currency Architecture**:
- TransactionCurrencyId links all monetary fields
- ExchangeRate captured at transaction time
- Base currency (_Base) fields preserve reporting accuracy

**Calculation Formulae**:
```
Line Amount = Quantity × UnitPrice
Line Discount = Line Amount × LineDiscount%
Net Line = Line Amount - Line Discount
Subtotal = SUM(Net Line) + Manual Discount
Tax = Calculated on net amount
Total = Subtotal + Freight + Tax
Base Amount = Transaction Amount × Exchange Rate
```

## 5.4 Concurrency Control

**Optimistic Concurrency**:
- VersionNumber (timestamp) column on all entities
- Auto-incremented on every UPDATE
- Application compares originalVersionNumber in WHERE clause
- If @@ROWCOUNT = 0, concurrency conflict detected

```sql
UPDATE AccountBase
SET Name = @Name, ModifiedOn = GETUTCDATE()
WHERE AccountId = @AccountId 
  AND VersionNumber = @OriginalVersionNumber
```

---

# SECTION 6 – Expert Review

## 6.1 Architecture Strengths

1. **Comprehensive Entity Coverage**: Complete sales lifecycle entities with proper lineage tracking
2. **Standard Patterns**: Follows Microsoft Dynamics 365 conventions exactly
3. **Multi-Currency Support**: Proper base currency preservation for reporting
4. **Polymorphic Relationships**: Flexible customer modeling (Account/Contact)
5. **Audit Trail**: Full CreatedOn/ModifiedOn tracking with VersionNumber
6. **Security Model**: Business unit hierarchy, ownership, roles, teams, field security
7. **State Machines**: Proper StateCode/StatusCode lifecycle enforcement

## 6.2 Identified Risks

1. **GUID Primary Keys**: While globally unique, they cause page fragmentation; FILLFACTOR 80 helps
2. **Polymorphic FKs**: Application must enforce referential integrity; DB constraints limited
3. **Complex Aggregations**: Rollup calculations in application layer may cause performance issues at scale
4. **Limited Check Constraints**: Business validation relies heavily on application layer
5. **Large Base Tables**: With 300+ tables, certain tables may require partitioning for very large deployments

## 6.3 Missing Components

1. **Partitioning**: Not implemented; needed for very large data volumes
2. **Temporal Tables**: Not used; historical tracking via Audit only
3. **Computed Columns**: Denormalized Name fields could be computed for automatic updates

## 6.4 Improvement Suggestions

1. **Index Optimization**: Consider filtered indexes for common queries
2. **Compression**: Enable page compression on large tables
3. **Archive Strategy**: Implement data archival for historical records
4. **Query Store**: Enable for performance monitoring
5. **Columnstore**: Consider for large analytical queries on transactional data

---

*Document prepared by Senior Microsoft Dynamics 365 CRM Architect & Dataverse Database Expert*
*This documentation provides complete enterprise-grade analysis suitable for CRM solution architects and database engineers*
