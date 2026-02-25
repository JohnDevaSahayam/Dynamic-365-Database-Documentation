# Microsoft Dynamics 365 CRM Sales Module
## Enterprise Mermaid Diagrams - Data Structure & Data Flow

**Document Classification**: Enterprise Architecture Visual Documentation  
**Author**: Senior Microsoft Dynamics 365 CRM Solution Architect & Dataverse Database Expert  
**Database**: CRM_DB (SQL Server)  
**Date**: January 2026

---

## SECTION 1 – Entity Relationship Diagram (ERD)

### 1.1 Core Sales Entities ERD

```mermaid
erDiagram
    %% ==================== CORE SALES ENTITIES ====================
    
    ACCOUNT {
        uuid AccountId PK
        nvarchar AccountNumber
        nvarchar Name
        uuid OwningBusinessUnit FK
        uuid OwnerId FK
        int StateCode
        int StatusCode
        uuid ParentAccountId FK
        uuid PrimaryContactId FK
        uuid MasterId FK
        int IndustryCode
        money Revenue
        int NumberOfEmployees
        timestamp VersionNumber
    }
    
    CONTACT {
        uuid ContactId PK
        nvarchar FirstName
        nvarchar LastName
        nvarchar FullName
        nvarchar EmailAddress1
        nvarchar Telephone1
        uuid OwningBusinessUnit FK
        uuid OwnerId FK
        uuid ParentCustomerId FK
        int ParentCustomerIdType
        uuid MasterId FK
        int StateCode
        int StatusCode
        timestamp VersionNumber
    }
    
    LEAD {
        uuid LeadId PK
        nvarchar Subject
        nvarchar FirstName
        nvarchar LastName
        nvarchar CompanyName
        nvarchar EmailAddress
        nvarchar Telephone
        uuid OwningBusinessUnit FK
        uuid OwnerId FK
        int StateCode
        int StatusCode
        uuid CampaignId FK
        uuid QualifyingOpportunityId FK
        uuid ParentAccountId FK
        uuid ParentContactId FK
        uuid OriginatingLeadId FK
        int IndustryCode
        int LeadSourceCode
        nvarchar TraversedPath
        timestamp VersionNumber
    }
    
    OPPORTUNITY {
        uuid OpportunityId PK
        nvarchar Name
        uuid OwningBusinessUnit FK
        uuid OwnerId FK
        int StateCode
        int StatusCode
        uuid CustomerId
        int CustomerIdType
        uuid PriceLevelId FK
        uuid TransactionCurrencyId FK
        money EstimatedValue
        money EstimatedValue_Base
        money ActualValue
        money TotalAmount
        int CloseProbability
        datetime EstimatedCloseDate
        datetime ActualCloseDate
        int SalesStageCode
        nvarchar SalesStage
        uuid OriginatingLeadId FK
        uuid CampaignId FK
        nvarchar TraversedPath
        timestamp VersionNumber
    }
    
    QUOTE {
        uuid QuoteId PK
        nvarchar QuoteNumber
        int RevisionNumber
        nvarchar Name
        uuid OwningBusinessUnit FK
        uuid OwnerId FK
        int StateCode
        int StatusCode
        uuid CustomerId
        int CustomerIdType
        uuid PriceLevelId FK
        uuid TransactionCurrencyId FK
        uuid OpportunityId FK
        money TotalAmount
        money DiscountAmount
        money FreightAmount
        money TotalTax
        datetime EffectiveFrom
        datetime EffectiveTo
        datetime ExpiresOn
        timestamp VersionNumber
    }
    
    SALESORDER {
        uuid SalesOrderId PK
        nvarchar OrderNumber
        uuid OwningBusinessUnit FK
        uuid OwnerId FK
        int StateCode
        int StatusCode
        uuid CustomerId
        int CustomerIdType
        uuid PriceLevelId FK
        uuid TransactionCurrencyId FK
        uuid OpportunityId FK
        uuid QuoteId FK
        money TotalAmount
        money DiscountAmount
        money FreightAmount
        money TotalTax
        datetime RequestDeliveryBy
        int SubmitStatus
        timestamp VersionNumber
    }
    
    INVOICE {
        uuid InvoiceId PK
        nvarchar InvoiceNumber
        uuid OwningBusinessUnit FK
        uuid OwnerId FK
        int StateCode
        int StatusCode
        uuid CustomerId
        int CustomerIdType
        uuid PriceLevelId FK
        uuid TransactionCurrencyId FK
        uuid SalesOrderId FK
        uuid OpportunityId FK
        money TotalAmount
        money TotalTax
        datetime DueDate
        int PaymentTermsCode
        timestamp VersionNumber
    }
    
    %% ==================== PRODUCT & PRICING ENTITIES ====================
    
    PRODUCT {
        uuid ProductId PK
        nvarchar Name
        nvarchar ProductNumber
        int StateCode
        int StatusCode
        uuid DefaultUoMId FK
        uuid PriceLevelId FK
        money QuantityDecimal
        int ProductTypeCode
        timestamp VersionNumber
    }
    
    PRICELIST {
        uuid PriceLevelId PK
        nvarchar Name
        int StateCode
        int StatusCode
        uuid TransactionCurrencyId FK
        datetime BeginDate
        datetime EndDate
        bit IsDefault
        timestamp VersionNumber
    }
    
    PRICELISTITEM {
        uuid ProductPriceLevelId PK
        uuid PriceLevelId FK
        uuid ProductId FK
        money Amount
        decimal Percentage
        money FloorPrice
        money CeilingPrice
        timestamp VersionNumber
    }
    
    UOM {
        uuid UoMId PK
        nvarchar Name
        uuid UoMScheduleId FK
        timestamp VersionNumber
    }
    
    UOMSCHEDULE {
        uuid UoMScheduleId PK
        nvarchar Name
        timestamp VersionNumber
    }
    
    DISCOUNTTYPE {
        uuid DiscountTypeId PK
        nvarchar Name
        int Type
        uuid TransactionCurrencyId FK
        timestamp VersionNumber
    }
    
    DISCOUNT {
        uuid DiscountId PK
        uuid DiscountTypeId FK
        int LowQuantity
        int HighQuantity
        decimal Percentage
        money Amount
        uuid TransactionCurrencyId FK
        timestamp VersionNumber
    }
    
    TRANSACTIONCURRENCY {
        uuid TransactionCurrencyId PK
        nvarchar ISOCurrencyCode
        nvarchar CurrencyName
        string CurrencySymbol
        int Precision
        timestamp VersionNumber
    }
    
    %% ==================== LINE ITEM ENTITIES ====================
    
    OPPORTUNITYPRODUCT {
        uuid OpportunityProductId PK
        uuid OpportunityId FK
        uuid ProductId FK
        uuid UoMId FK
        int Quantity
        money UnitPrice
        money ExtendedAmount
        money LineItemDiscount
        decimal LineItemDiscountPercentage
        money Tax
        timestamp VersionNumber
    }
    
    QUOTEDETAIL {
        uuid QuoteDetailId PK
        uuid QuoteId FK
        uuid ProductId FK
        uuid UoMId FK
        int Quantity
        money UnitPrice
        money ExtendedAmount
        money LineItemDiscount
        decimal LineItemDiscountPercentage
        money Tax
        timestamp VersionNumber
    }
    
    ORDERDETAIL {
        uuid SalesOrderDetailId PK
        uuid SalesOrderId FK
        uuid ProductId FK
        uuid UoMId FK
        int Quantity
        money UnitPrice
        money ExtendedAmount
        money LineItemDiscount
        decimal LineItemDiscountPercentage
        money Tax
        timestamp VersionNumber
    }
    
    INVOICEDETAIL {
        uuid InvoiceDetailId PK
        uuid InvoiceId FK
        uuid ProductId FK
        uuid UoMId FK
        int Quantity
        money UnitPrice
        money ExtendedAmount
        money LineItemDiscount
        decimal LineItemDiscountPercentage
        money Tax
        timestamp VersionNumber
    }
    
    %% ==================== RELATIONSHIPS ====================
    
    ACCOUNT ||--o{ CONTACT : "has_primary"
    ACCOUNT ||--o{ OPPORTUNITY : "generates"
    ACCOUNT ||--o{ QUOTE : "creates"
    ACCOUNT ||--o{ SALESORDER : "orders"
    ACCOUNT ||--o{ INVOICE : "billed"
    ACCOUNT ||--o{ ACCOUNT : "parent_child"
    ACCOUNT ||--o{ LEAD : "originates"
    ACCOUNT }o--|| PRICELIST : "uses"
    
    CONTACT ||--o{ OPPORTUNITY : "associated"
    CONTACT ||--o{ LEAD : "qualifies"
    CONTACT--o{ QU ||OTE : "receives"
    CONTACT ||--o{ SALESORDER : "purchases"
    CONTACT ||--o{ INVOICE : "receives"
    CONTACT }o--|| ACCOUNT : "belongs_to"
    
    LEAD ||--o{ LEAD : "self_referential"
    LEAD }o--|| CAMPAIGN : "source"
    LEAD ||--o{ OPPORTUNITY : "qualifies_to"
    LEAD }o--|| ACCOUNT : "becomes"
    LEAD }o--|| CONTACT : "becomes"
    
    OPPORTUNITY ||--o{ OPPORTUNITYPRODUCT : "contains"
    OPPORTUNITY ||--o{ QUOTE : "generates"
    OPPORTUNITY ||--o{ SALESORDER : "creates"
    OPPORTUNITY ||--o{ INVOICE : "creates"
    OPPORTUNITY }o--|| LEAD : "originates"
    OPPORTUNITY }o--|| ACCOUNT : "customer"
    OPPORTUNITY }o--|| CONTACT : "customer"
    OPPORTUNITY }o--|| PRICELIST : "uses"
    
    QUOTE ||--o{ QUOTEDETAIL : "contains"
    QUOTE }o--|| OPPORTUNITY : "from"
    QUOTE ||--o{ SALESORDER : "converts_to"
    
    SALESORDER ||--o{ ORDERDETAIL : "contains"
    SALESORDER }o--|| QUOTE : "from"
    SALESORDER ||--o{ INVOICE : "invoices"
    
    INVOICE ||--o{ INVOICEDETAIL : "contains"
    INVOICE }o--|| SALESORDER : "from"
    
    PRODUCT ||--o{ OPPORTUNITYPRODUCT : "sold_in"
    PRODUCT ||--o{ QUOTEDETAIL : "quoted"
    PRODUCT ||--o{ ORDERDETAIL : "ordered"
    PRODUCT ||--o{ INVOICEDETAIL : "invoiced"
    PRODUCT }o--|| PRICELIST : "priced_in"
    PRODUCT }o--|| UOM : "measured_in"
    
    PRICELIST ||--o{ PRICELISTITEM : "contains"
    
    UOMSCHEDULE ||--o{ UOM : "contains"
    
    DISCOUNTTYPE ||--o{ DISCOUNT : "defines"
```

### 1.2 Security & Organization Entities ERD

```mermaid
erDiagram
    %% ==================== SECURITY ENTITIES ====================
    
    SYSTEMUSER {
        uuid SystemUserId PK
        nvarchar FullName
        nvarchar FirstName
        nvarchar LastName
        nvarchar DomainName
        nvarchar BusinessUnitIdName
        uuid BusinessUnitId FK
        int UserLicenseType
        bit IsDisabled
        int AccessMode
        uuid CalendarId FK
        nvarchar TimeZoneCode
        timestamp VersionNumber
    }
    
    BUSINESSUNIT {
        uuid BusinessUnitId PK
        nvarchar Name
        uuid ParentBusinessUnitId FK
        uuid OrganizationId FK
        uuid CalendarId FK
        uuid TransactionCurrencyId FK
        timestamp VersionNumber
    }
    
    TEAM {
        uuid TeamId PK
        nvarchar Name
        uuid BusinessUnitId FK
        int TeamType
        timestamp VersionNumber
    }
    
    ROLE {
        uuid RoleId PK
        nvarchar Name
        uuid BusinessUnitId FK
        uuid RoleTemplateId FK
        timestamp VersionNumber
    }
    
    USERROLE {
        uuid SystemUserId PK
        uuid RoleId PK
    }
    
    TEAMMEMBERSHIP {
        uuid SystemUserId PK
        uuid TeamId PK
    }
    
    OWNER {
        uuid OwnerId PK
        int OwnerIdType
        nvarchar Name
    }
    
    PRIVILEGE {
        uuid PrivilegeId PK
        nvarchar Name
        int PrivilegeType
        int CanBeBasic
        int CanBeLocal
        int CanBeDeep
        int CanBeGlobal
    }
    
    ROLEPRIVILEGE {
        uuid RoleId PK
        uuid PrivilegeId PK
        int PrivilegeDepthMask
    }
    
    ORGANIZATION {
        uuid OrganizationId PK
        nvarchar Name
        nvarchar UniqueName
        int StateCode
        int StatusCode
        nvarchar BaseCurrencyIdName
        uuid BaseCurrencyId FK
        timestamp VersionNumber
    }
    
    FIELDSECURITYPROFILE {
        uuid FieldSecurityProfileId PK
        nvarchar Name
        timestamp VersionNumber
    }
    
    FIELDPERMISSION {
        uuid FieldPermissionId PK
        uuid FieldSecurityProfileId FK
        nvarchar EntityName
        nvarchar AttributeName
        bit CanRead
        bit CanCreate
        bit CanUpdate
    }
    
    %% ==================== RELATIONSHIPS ====================
    
    BUSINESSUNIT ||--o{ BUSINESSUNIT : "parent_child"
    BUSINESSUNIT }o--|| BUSINESSUNIT : "parent"
    BUSINESSUNIT }o--|| ORGANIZATION : "belongs_to"
    BUSINESSUNIT }o--|| TRANSACTIONCURRENCY : "uses"
    
    SYSTEMUSER }o--|| BUSINESSUNIT : "belongs_to"
    SYSTEMUSER }o--|| OWNER : "owned_by"
    SYSTEMUSER ||--o{ USERROLE : "assigned"
    SYSTEMUSER ||--o{ TEAMMEMBERSHIP : "member_of"
    SYSTEMUSER }o--|| FIELDSECURITYPROFILE : "secured_by"
    
    TEAM }o--|| BUSINESSUNIT : "belongs_to"
    TEAM ||--o{ TEAMMEMBERSHIP : "has_members"
    
    ROLE }o--|| BUSINESSUNIT : "assigned_to"
    ROLE ||--o{ USERROLE : "granted_to"
    ROLE ||--o{ ROLEPRIVILEGE : "contains"
    
    PRIVILEGE ||--o{ ROLEPRILEGE : "assigned_via"
    
    %% Polymorphic relationship for ownership
    ACCOUNT }o--|| OWNER : "owned_by"
    CONTACT }o--|| OWNER : "owned_by"
    LEAD }o--|| OWNER : "owned_by"
    OPPORTUNITY }o--|| OWNER : "owned_by"
    QUOTE }o--|| OWNER : "owned_by"
    SALESORDER }o--|| OWNER : "owned_by"
    INVOICE }o--|| OWNER : "owned_by"
```

### 1.3 Activity & Communication Entities ERD

```mermaid
erDiagram
    %% ==================== ACTIVITY ENTITIES ====================
    
    ACTIVITYPOINTER {
        uuid ActivityId PK
        nvarchar ActivityTypeCode
        uuid OwningBusinessUnit FK
        uuid OwnerId FK
        int StateCode
        int StatusCode
        nvarchar Subject
        nvarchar Description
        datetime ScheduledStart
        datetime ScheduledEnd
        datetime ActualStart
        datetime ActualEnd
        int PriorityCode
        int DirectionCode
        uuid RegardingObjectId
        int RegardingObjectTypeCode
        uuid ServiceId FK
        uuid SiteId FK
        timestamp VersionNumber
    }
    
    TASK {
        uuid ActivityId PK
        nvarchar Subject
        nvarchar Description
        int StatusCode
        int PriorityCode
        datetime ScheduledEnd
        bit IsBilled
    }
    
    EMAIL {
        uuid ActivityId PK
        nvarchar Subject
        nvarchar Description
        nvarchar Sender
        nvarchar ToRecipients
        nvarchar CcRecipients
        bit IsBilled
        int StatusCode
        nvarchar EmailHash
    }
    
    PHONECALL {
        uuid ActivityId PK
        nvarchar Subject
        nvarchar Description
        nvarchar PhoneNumber
        int DirectionCode
        int StatusCode
    }
    
    APPOINTMENT {
        uuid ActivityId PK
        nvarchar Subject
        nvarchar Description
        datetime ScheduledStart
        datetime ScheduledEnd
        int StatusCode
        int PriorityCode
    }
    
    LETTER {
        uuid ActivityId PK
        nvarchar Subject
        nvarchar Description
        nvarchar Address
        int StatusCode
    }
    
    FAX {
        uuid ActivityId PK
        nvarchar Subject
        nvarchar Description
        nvarchar FaxNumber
        int DirectionCode
        int StatusCode
    }
    
    ANNOTATION {
        uuid AnnotationId PK
        nvarchar Subject
        nvarchar NoteText
        nvarchar FileName
        varbinary FileAttachment
        uuid OwningBusinessUnit FK
        uuid OwnerId FK
        uuid ObjectId FK
        int ObjectTypeCode
        timestamp VersionNumber
    }
    
    ACTIVITYPARTY {
        uuid ActivityId PK
        uuid PartyId FK
        int ParticipationTypeMask
        nvarchar AddressUsed
        bit DoNotSend
    }
    
    %% ==================== RELATIONSHIPS ====================
    
    TASK |o--|| ACTIVITYPOINTER : "is_a"
    EMAIL |o--|| ACTIVITYPOINTER : "is_a"
    PHONECALL |o--|| ACTIVITYPOINTER : "is_a"
    APPOINTMENT |o--|| ACTIVITYPOINTER : "is_a"
    LETTER |o--|| ACTIVITYPOINTER : "is_a"
    FAX |o--|| ACTIVITYPOINTER : "is_a"
    
    ACTIVITYPOINTER ||--o{ ACTIVITYPARTY : "has_participants"
    ACTIVITYPOINTER }o--|| ACCOUNT : "regarding"
    ACTIVITYPOINTER }o--|| CONTACT : "regarding"
    ACTIVITYPOINTER }o--|| LEAD : "regarding"
    ACTIVITYPOINTER }o--|| OPPORTUNITY : "regarding"
    
    ANNOTATION }o--|| ACCOUNT : "annotated"
    ANNOTATION }o--|| CONTACT : "annotated"
    ANNOTATION }o--|| LEAD : "annotated"
    ANNOTATION }o--|| OPPORTUNITY : "annotated"
    ANNOTATION }o--|| QUOTE : "annotated"
    ANNOTATION }o--|| SALESORDER : "annotated"
    ANNOTATION }o--|| INVOICE : "annotated"
    ANNOTATION }o--|| OWNER : "owned_by"
```

---

## SECTION 2 – Database Table Structure Flow

### 2.1 Core Entity Class Diagram

```mermaid
classDiagram
    %% ==================== ACCOUNT ENTITY ====================
    class AccountBase {
        +uuid AccountId PK
        +nvarchar AccountNumber
        +nvarchar Name
        +uuid OwningBusinessUnit FK
        +uuid OwnerId FK
        +int OwnerIdType
        +int StateCode
        +int StatusCode
        +uuid ParentAccountId FK
        +uuid PrimaryContactId FK
        +uuid MasterId FK
        +int IndustryCode
        +int BusinessTypeCode
        +money Revenue
        +int NumberOfEmployees
        +nvarchar Telephone1
        +nvarchar EmailAddress1
        +nvarchar WebsiteUrl
        +int AccountRatingCode
        +bit CreditOnHold
        +uuid TerritoryId FK
        +uuid DefaultPriceLevelId FK
        +datetime CreatedOn
        +uuid CreatedBy FK
        +datetime ModifiedOn
        +uuid ModifiedBy FK
        +datetime CreatedOnBehalfBy
        +datetime ModifiedOnBehalfBy
        +timestamp VersionNumber
        +int ImportSequenceNumber
        +datetime OverriddenCreatedOn
        +int UTCConversionTimeZoneCode
        +int TimeZoneRuleVersionNumber
        +uuid ProcessId
        +uuid StageId
        +nvarchar TraversedPath
    }
    
    %% ==================== LEAD ENTITY ====================
    class LeadBase {
        +uuid LeadId PK
        +nvarchar Subject
        +nvarchar FirstName
        +nvarchar LastName
        +nvarchar CompanyName
        +nvarchar EmailAddress
        +nvarchar Telephone
        +uuid OwningBusinessUnit FK
        +uuid OwnerId FK
        +int OwnerIdType
        +int StateCode
        +int StatusCode
        +int IndustryCode
        +int LeadSourceCode
        +int RatingCode
        +uuid CampaignId FK
        +uuid QualifyingOpportunityId FK
        +uuid ParentAccountId FK
        +uuid ParentContactId FK
        +uuid OriginatingLeadId FK
        +bit DoNotPhone
        +bit DoNotEmail
        +bit DoNotPostalMail
        +datetime CreatedOn
        +uuid CreatedBy FK
        +datetime ModifiedOn
        +uuid ModifiedBy FK
        +timestamp VersionNumber
        +nvarchar TraversedPath
        +uuid ProcessId
        +uuid StageId
    }
    
    %% ==================== OPPORTUNITY ENTITY ====================
    class OpportunityBase {
        +uuid OpportunityId PK
        +nvarchar Name
        +uuid OwningBusinessUnit FK
        +uuid OwnerId FK
        +int OwnerIdType
        +int StateCode
        +int StatusCode
        +uuid CustomerId
        +int CustomerIdType
        +nvarchar CustomerIdName
        +uuid PriceLevelId FK
        +uuid TransactionCurrencyId FK
        +money EstimatedValue
        +money EstimatedValue_Base
        +money ActualValue
        +money ActualValue_Base
        +int CloseProbability
        +datetime EstimatedCloseDate
        +datetime ActualCloseDate
        +int SalesStageCode
        +nvarchar SalesStage
        +uuid CampaignId FK
        +uuid OriginatingLeadId FK
        +money TotalAmount
        +money TotalAmount_Base
        +money DiscountAmount
        +money DiscountPercentage
        +money FreightAmount
        +money TotalTax
        +money TotalLineItemAmount
        +money TotalDiscountAmount
        +money TotalAmountLessFreight
        +datetime CreatedOn
        +uuid CreatedBy FK
        +datetime ModifiedOn
        +uuid ModifiedBy FK
        +timestamp VersionNumber
        +nvarchar TraversedPath
        +uuid ProcessId
        +uuid StageId
    }
    
    %% ==================== QUOTE ENTITY ====================
    class QuoteBase {
        +uuid QuoteId PK
        +nvarchar QuoteNumber
        +int RevisionNumber
        +nvarchar Name
        +uuid OwningBusinessUnit FK
        +uuid OwnerId FK
        +int OwnerIdType
        +int StateCode
        +int StatusCode
        +uuid CustomerId
        +int CustomerIdType
        +uuid PriceLevelId FK
        +uuid TransactionCurrencyId FK
        +uuid OpportunityId FK
        +money TotalAmount
        +money TotalAmount_Base
        +money DiscountAmount
        +money DiscountPercentage
        +money FreightAmount
        +money TotalTax
        +money TotalLineItemAmount
        +money TotalDiscountAmount
        +money TotalAmountLessFreight
        +datetime EffectiveFrom
        +datetime EffectiveTo
        +datetime ExpiresOn
        +datetime RequestDeliveryBy
        +datetime ClosedOn
        +int ShippingMethodCode
        +int PaymentTermsCode
        +datetime CreatedOn
        +timestamp VersionNumber
    }
    
    %% ==================== SALES ORDER ENTITY ====================
    class SalesOrderBase {
        +uuid SalesOrderId PK
        +nvarchar OrderNumber
        +uuid OwningBusinessUnit FK
        +uuid OwnerId FK
        +int OwnerIdType
        +int StateCode
        +int StatusCode
        +uuid CustomerId
        +int CustomerIdType
        +uuid PriceLevelId FK
        +uuid TransactionCurrencyId FK
        +uuid OpportunityId FK
        +uuid QuoteId FK
        +money TotalAmount
        +money TotalAmount_Base
        +money DiscountAmount
        +money FreightAmount
        +money TotalTax
        +datetime RequestDeliveryBy
        +int SubmitStatus
        +datetime SubmitDate
        +datetime LastBackofficeSubmit
        +datetime CreatedOn
        +timestamp VersionNumber
    }
    
    %% ==================== INVOICE ENTITY ====================
    class InvoiceBase {
        +uuid InvoiceId PK
        +nvarchar InvoiceNumber
        +uuid OwningBusinessUnit FK
        +uuid OwnerId FK
        +int OwnerIdType
        +int StateCode
        +int StatusCode
        +uuid CustomerId
        +int CustomerIdType
        +uuid PriceLevelId FK
        +uuid TransactionCurrencyId FK
        +uuid SalesOrderId FK
        +uuid OpportunityId FK
        +money TotalAmount
        +money TotalAmount_Base
        +money DiscountAmount
        +money FreightAmount
        +money TotalTax
        +int PaymentTermsCode
        +datetime DueDate
        +datetime DateSent
        +datetime PaidOn
        +datetime CreatedOn
        +timestamp VersionNumber
    }
    
    %% ==================== SYSTEM ATTRIBUTES MIXIN ====================
    class SystemAttributes {
        <<mixin>>
        +datetime CreatedOn
        +uuid CreatedBy
        +datetime ModifiedOn
        +uuid ModifiedBy
        +datetime CreatedOnBehalfBy
        +datetime ModifiedOnBehalfBy
        +timestamp VersionNumber
    }
    
    class OwnershipAttributes {
        <<mixin>>
        +uuid OwnerId
        +int OwnerIdType
        +uuid OwningBusinessUnit
    }
    
    class StateManagementAttributes {
        <<mixin>>
        +int StateCode
        +int StatusCode
    }
    
    class AuditAttributes {
        <<mixin>>
        +int ImportSequenceNumber
        +datetime OverriddenCreatedOn
        +int UTCConversionTimeZoneCode
        +int TimeZoneRuleVersionNumber
    }
    
    class BPFAttributes {
        <<mixin>>
        +uuid ProcessId
        +uuid StageId
        +nvarchar TraversedPath
    }
    
    %% Apply mixins
    AccountBase ..|> SystemAttributes
    AccountBase ..|> OwnershipAttributes
    AccountBase ..|> StateManagementAttributes
    AccountBase ..|> AuditAttributes
    AccountBase ..|> BPFAttributes
    
    LeadBase ..|> SystemAttributes
    LeadBase ..|> OwnershipAttributes
    LeadBase ..|> StateManagementAttributes
    LeadBase ..|> AuditAttributes
    LeadBase ..|> BPFAttributes
    
    OpportunityBase ..|> SystemAttributes
    OpportunityBase ..|> OwnershipAttributes
    OpportunityBase ..|> StateManagementAttributes
    OpportunityBase ..|> AuditAttributes
    OpportunityBase ..|> BPFAttributes
    
    QuoteBase ..|> SystemAttributes
    QuoteBase ..|> OwnershipAttributes
    QuoteBase ..|> StateManagementAttributes
    QuoteBase ..|> AuditAttributes
    
    SalesOrderBase ..|> SystemAttributes
    SalesOrderBase ..|> OwnershipAttributes
    SalesOrderBase ..|> StateManagementAttributes
    SalesOrderBase ..|> AuditAttributes
    
    InvoiceBase ..|> SystemAttributes
    InvoiceBase ..|> OwnershipAttributes
    InvoiceBase ..|> StateManagementAttributes
    InvoiceBase ..|> AuditAttributes
```

### 2.2 StateCode & StatusCode Structure

```mermaid
classDiagram
    class StateStatusModel {
        <<enumeration>>
    }
    
    note for StateStatusModel "STATE CODE & STATUS CODE ARCHITECTURE"
    
    class LeadStateMachine {
        <<enumeration>>
        +STATE_OPEN = 0
        +STATE_QUALIFIED = 1
        +STATE_DISQUALIFIED = 2
    }
    
    class LeadStatusCodes {
        <<enumeration>>
        +NEW = 1
        +ATTEMPTED_CONTACT = 2
        +CONTACTED = 3
        +OPEN = 4
        +QUALIFIED = 5
        +DISQUALIFIED = 6
        +LOST = 7
        +CANNOT_CONTACT = 8
        +NOT_INTERESTED = 9
    }
    
    class OpportunityStateMachine {
        <<enumeration>>
        +STATE_OPEN = 0
        +STATE_WON = 1
        +STATE_LOST = 2
    }
    
    class OpportunityStatusCodes {
        <<enumeration>>
        +NEW = 1
        +IN_PROGRESS = 2
        +ON_HOLD = 3
        +WON = 4
        +LOST = 5
        +CANCELLED = 6
        +OUT_SOLD = 7
    }
    
    class SalesOrderStateMachine {
        <<enumeration>>
        +STATE_ACTIVE = 0
        +STATE_FULFILLED = 1
        +STATE_INVOICED = 2
        +STATE_PAID = 3
        +STATE_CANCELLED = 4
    }
    
    class InvoiceStateMachine {
        <<enumeration>>
        +STATE_ACTIVE = 0
        +STATE_PAID = 1
        +STATE_CANCELLED = 2
    }
    
    StateStatusModel --> LeadStateMachine
    StateStatusModel --> LeadStatusCodes
    StateStatusModel --> OpportunityStateMachine
    StateStatusModel --> OpportunityStatusCodes
    StateStatusModel --> SalesOrderStateMachine
    StateStatusModel --> InvoiceStateMachine
```

---

## SECTION 3 – CRM Business Process Flow

### 3.1 Lead to Cash Pipeline Flow

```mermaid
flowchart TD
    %% ==================== STYLES ====================
    classDef stage fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef entity fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    classDef decision fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef process fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    classDef output fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    
    %% ==================== NODES ====================
    
    START(("Start"))
    
    subgraph LEAD_QUALIFICATION["Lead Qualification"]
        direction LR
        L1["Lead Created"]
        L2{"Qualify?"}
        L3["Lead Qualified"]
        L4["Lead Disqualified"]
    end
    
    subgraph OPPORTUNITY_PROCESS["Opportunity Management"]
        direction LR
        O1["Opportunity Created"]
        O2["Develop Proposal"]
        O3["Present Solution"]
        O4{"Close Opportunity"}
    end
    
    QUOTE_CREATE["Quote Created"]
    QUOTE_REVISE["Quote Revised"]
    QUOTE_ACCEPT["Quote Accepted"]
    QUOTE_REJECT["Quote Rejected"]
    
    ORDER_CREATE["Sales Order Created"]
    ORDER_FULFILL["Order Fulfilled"]
    ORDER_CANCEL["Order Cancelled"]
    
    INVOICE_CREATE["Invoice Created"]
    INVOICE_PAID["Invoice Paid"]
    INVOICE_CANCEL["Invoice Cancelled"]
    
    CLOSE["Close / Win"]
    LOST["Lost / No Sale"]
    
    %% ==================== EDGES ====================
    
    START --> L1
    L1 --> L2
    L2 -->|Yes| L3
    L2 -->|No| L4
    
    L3 --> O1
    
    O1 --> O2
    O2 --> O3
    O3 --> O4
    
    O4 -->|Create Quote| QUOTE_CREATE
    O4 -->|Direct Close| CLOSE
    O4 -->|Lost| LOST
    
    QUOTE_CREATE --> QUOTE_REVISE
    QUOTE_REVISE -->|Accept| QUOTE_ACCEPT
    QUOTE_REVISE -->|Reject| QUOTE_REJECT
    QUOTE_ACCEPT --> ORDER_CREATE
    QUOTE_REJECT --> LOST
    
    ORDER_CREATE --> ORDER_FULFILL
    ORDER_CREATE --> ORDER_CANCEL
    
    ORDER_FULFILL --> INVOICE_CREATE
    ORDER_CANCEL --> LOST
    
    INVOICE_CREATE --> INVOICE_PAID
    INVOICE_CREATE --> INVOICE_CANCEL
    
    INVOICE_PAID --> CLOSE
    INVOICE_CANCEL --> LOST
    
    %% ==================== CLASSIFICATIONS ====================
    class START,L1,L3,L4,CLOSE,LOST entity
    class L2,O4 decision
    class O1,O2,O3,QUOTE_CREATE,ORDER_CREATE,INVOICE_CREATE process
    class QUOTE_REVISE,ORDER_FULFILL stage
```

### 3.2 Lead State Transition Flow

```mermaid
flowchart TD
    %% ==================== LEAD STATE TRANSITIONS ====================
    
    subgraph OPEN["State: OPEN (0)"]
        direction TB
        NEW["New (1)"]
        ATTEMPTED["Attempted to Contact (2)"]
        CONTACTED["Contacted (3)"]
        OPEN_LEAD["Open (4)"]
    end
    
    subgraph QUALIFIED["State: QUALIFIED (1)"]
        direction TB
        QUALIFIED_LEAD["Qualified (5)"]
        DISQUALIFIED["Disqualified (6)"]
    end
    
    subgraph DISQUALIFIED_STATE["State: DISQUALIFIED (2)"]
        direction TB
        LOST["Lost (7)"]
        CANNOT_CONTACT["Cannot Contact (8)"]
        NOT_INTERESTED["Not Interested (9)"]
    end
    
    %% Transitions
    NEW --> ATTEMPTED
    ATTEMPTED --> CONTACTED
    CONTACTED --> OPEN_LEAD
    
    NEW --> QUALIFIED_LEAD
    ATTEMPTED --> QUALIFIED_LEAD
    CONTACTED --> QUALIFIED_LEAD
    OPEN_LEAD --> QUALIFIED_LEAD
    
    NEW --> DISQUALIFIED
    ATTEMPTED --> DISQUALIFIED
    CONTACTED --> DISQUALIFIED
    OPEN_LEAD --> DISQUALIFIED
    
    QUALIFIED_LEAD -->|Convert| OPPORTUNITY("→ Opportunity")
    DISQUALIFIED --> LOST
    DISQUALIFIED --> CANNOT_CONTACT
    DISQUALIFIED --> NOT_INTERESTED
    
    QUALIFIED_LEAD --> LOST
    
    class NEW,ATTEMPTED,CONTACTED,OPEN_LEAD fill:#e3f2fd,stroke:#1565c0
    class QUALIFIED_LEAD,DISQUALIFIED fill:#c8e6c9,stroke:#2e7d32
    class LOST,CANNOT_CONTACT,NOT_INTERESTED fill:#ffcdd2,stroke:#c62828
```

### 3.3 Opportunity State Transition Flow

```mermaid
flowchart TD
    %% ==================== OPPORTUNITY STATE TRANSITIONS ====================
    
    subgraph OPEN_OPP["State: OPEN (0)"]
        direction LR
        NEW_OPP["New (1)"]
        IN_PROGRESS["In Progress (2)"]
        ON_HOLD["On Hold (3)"]
    end
    
    WON["State: WON (1)"]
    WON_STATUS["Won (4)"]
    
    subgraph LOST_OPP["State: LOST (2)"]
        direction LR
        LOST_OPP_STATUS["Lost (5)"]
        CANCELLED["Cancelled (6)"]
        OUT_SOLD["Out-Sold (7)"]
    end
    
    %% Transitions
    NEW_OPP --> IN_PROGRESS
    IN_PROGRESS --> ON_HOLD
    IN_PROGRESS --> WON_STATUS
    ON_HOLD --> IN_PROGRESS
    ON_HOLD --> WON_STATUS
    
    NEW_OPP --> LOST_OPP_STATUS
    IN_PROGRESS --> LOST_OPP_STATUS
    ON_HOLD --> LOST_OPP_STATUS
    
    WON_STATUS --> WON
    
    LOST_OPP_STATUS --> LOST_OPP
    CANCELLED --> LOST_OPP
    OUT_SOLD --> LOST_OPP
    
    WON_STATUS -->|"Create Order"| ORDER["→ Sales Order"]
    LOST_OPP_STATUS -->|"Create"| LEAD["← Lead"]
    
    class NEW_OPP,IN_PROGRESS,ON_HOLD fill:#e3f2fd,stroke:#1565c0
    class WON_STATUS,WON fill:#c8e6c9,stroke:#2e7d32
    class LOST_OPP_STATUS,CANCELLED,OUT_SOLD,LOST_OPP fill:#ffcdd2,stroke:#c62828
```

---

## SECTION 4 – Data Flow Diagram (DFD)

### 4.1 CRM Data Flow Architecture

```mermaid
flowchart LR
    %% ==================== EXTERNAL ENTITIES ====================
    subgraph EXTERNAL["User Interactions"]
        UI["CRM UI / Power Apps"]
        API["Web API / Services"]
        INTEGRATION["External Integration"]
    end
    
    %% ==================== APPLICATION LAYER ====================
    subgraph APPLICATION["Application Layer"]
        VALIDATE["Data Validation"]
        SECURITY["Security Check"]
        WORKFLOW["Workflow Engine"]
        PLUGIN["Plugin Pipeline"]
        BPF["Business Process Flow"]
    end
    
    %% ==================== DATA LAYER ====================
    subgraph DATA["Data Layer"]
        BASE["Base Tables"]
        VIEWS["Logical Views"]
        AUDIT["Audit Tables"]
    end
    
    %% ==================== ENTITY OPERATIONS ====================
    
    %% CREATE FLOW
    CREATE[("Create Record")]
    VALIDATE -->|"Validate Required Fields"| SECURITY
    SECURITY -->|"Check Permissions"| WORKFLOW
    WORKFLOW -->|"Run Pre-Operation Plugins"| PLUGIN
    PLUGIN -->|"Execute BPF"| BPF
    BPF -->|"Insert to Base Table"| BASE
    BASE -->|"Create Audit Record"| AUDIT
    BASE -->|"Update Views"| VIEWS
    
    %% UPDATE FLOW
    UPDATE[("Update Record")]
    VALIDATE_UPDATE["Validate Changes"]
    CONCURRENCY["Check VersionNumber"]
    VALIDATE_UPDATE -->|"Validate Business Rules"| CONCURRENCY
    CONCURRENCY -->|"Optimistic Lock Check"| WORKFLOW
    WORKFLOW -->|"Run Pre-Update Plugins"| PLUGIN
    PLUGIN -->|"Update Base Table"| BASE
    BASE -->|"Create Audit Record"| AUDIT
    
    %% RETRIEVE FLOW
    RETRIEVE[("Retrieve Record")]
    RETRIEVE -->|"Query Logical View"| VIEWS
    VIEWS -->|"Apply Security Filter"| SECURITY
    SECURITY -->|"Return Filtered Data"| UI
    
    %% DELETE FLOW
    DELETE[("Delete Record")]
    DELETE -->|"Check Dependencies"| WORKFLOW
    WORKFLOW -->|"Cascade Delete"| BASE
    BASE -->|"Create Delete Audit"| AUDIT
    
    %% Flows from external
    UI --> CREATE
    UI --> UPDATE
    UI --> RETRIEVE
    UI --> DELETE
    
    API --> CREATE
    API --> UPDATE
    API --> RETRIEVE
    
    INTEGRATION --> CREATE
    INTEGRATION --> UPDATE
    
    %% Output to external
    VIEWS --> UI
    VIEWS --> API
    VIEWS --> INTEGRATION
    
    class UI,API,INTEGRATION fill:#e8eaf6,stroke:#3f51b5
    class VALIDATE,SECURITY,WORKFLOW,PLUGIN,BPF fill:#fff8e1,stroke:#ff8f00
    class BASE,VIEWS,AUDIT fill:#e8f5e9,stroke:#2e7d32
```

### 4.2 Record Lifecycle Data Flow

```mermaid
flowchart TD
    %% ==================== RECORD LIFECYCLE ====================
    
    START(("Record<br/>Creation"))
    
    PRE_CREATE["Pre-Create<br/>Plugins"]
    POST_CREATE["Post-Create<br/>Plugins"]
    VALIDATE_CREATE["Validate<br/>Required Fields"]
    
    ACTIVE["Active<br/>Record"]
    
    PRE_UPDATE["Pre-Update<br/>Plugins"]
    POST_UPDATE["Post-Update<br/>Plugins"]
    VALIDATE_UPDATE["Validate<br/>Business Rules"]
    
    STATE_CHANGE["State/Status<br/>Change"]
    POST_STATE["Post-State<br/>Plugins"]
    
    DEACTIVATE["Deactivate<br/>Record"]
    PRE_DELETE["Pre-Delete<br/>Plugins"]
    POST_DELETE["Post-Delete<br/>Plugins"]
    HARD_DELETE["Hard Delete"]
    
    AUDIT_CREATE["Create Audit<br/>Record"]
    AUDIT_UPDATE["Update Audit<br/>Record"]
    AUDIT_DELETE["Delete Audit<br/>Record"]
    
    %% Connections
    START --> VALIDATE_CREATE
    VALIDATE_CREATE --> PRE_CREATE
    PRE_CREATE --> ACTIVE
    ACTIVE --> POST_CREATE
    POST_CREATE --> AUDIT_CREATE
    
    ACTIVE --> VALIDATE_UPDATE
    VALIDATE_UPDATE --> PRE_UPDATE
    PRE_UPDATE --> ACTIVE
    ACTIVE --> POST_UPDATE
    POST_UPDATE --> AUDIT_UPDATE
    
    ACTIVE --> STATE_CHANGE
    STATE_CHANGE --> POST_STATE
    POST_STATE --> ACTIVE
    
    ACTIVE --> DEACTIVATE
    DEACTIVATE --> PRE_DELETE
    PRE_DELETE --> HARD_DELETE
    HARD_DELETE --> POST_DELETE
    POST_DELETE --> AUDIT_DELETE
    
    class START,ACTIVE fill:#e3f2fd,stroke:#1565c0
    class PRE_CREATE,POST_CREATE,PRE_UPDATE,POST_UPDATE,PRE_DELETE,POST_DELETE fill:#fff8e1,stroke:#ff8f00
    class VALIDATE_CREATE,VALIDATE_UPDATE,STATE_CHANGE,DEACTIVATE fill:#fce4ec,stroke:#c2185b
    class AUDIT_CREATE,AUDIT_UPDATE,AUDIT_DELETE fill:#f3e5f5,stroke:#7b1fa2
```

---

## SECTION 5 – Security Model Diagram

### 5.1 CRM Security Architecture

```mermaid
flowchart TB
    %% ==================== SECURITY ARCHITECTURE ====================
    
    subgraph ORG["Organization Level"]
        ORG1["Organization"]
    end
    
    subgraph BU_LEVEL["Business Unit Level"]
        BU1["Root Business Unit"]
        BU2["Sales BU"]
        BU3["Marketing BU"]
        BU4["Service BU"]
    end
    
    subgraph SECURITY_ROLES["Security Roles"]
        R1["Salesperson"]
        R2["Sales Manager"]
        R3["Sales VP"]
        R4["Marketing Manager"]
        R5["System Admin"]
    end
    
    subgraph TEAMS["Teams"]
        T1["Sales Team North"]
        T2["Sales Team South"]
        T3["Enterprise Team"]
    end
    
    subgraph USERS["Users"]
        U1["User A"]
        U2["User B"]
        U3["User C"]
    end
    
    subgraph RECORDS["Record Access"]
        RA1["Record Level<br/>Security"]
        RA2["Field Level<br/>Security"]
        RA3["Row Level<br/>Security"]
    end
    
    %% Hierarchy connections
    ORG1 -->|contains| BU1
    BU1 -->|contains| BU2
    BU1 -->|contains| BU3
    BU1 -->|contains| BU4
    
    %% Role assignments
    BU2 -->|assigns| R1
    BU2 -->|assigns| R2
    BU2 -->|assigns| R3
    BU3 -->|assigns| R4
    BU1 -->|assigns| R5
    
    %% Team memberships
    R1 -->|creates| T1
    R1 -->|creates| T2
    R2 -->|creates| T3
    
    %% User assignments
    R1 -->|assigned to| U1
    R2 -->|assigned to| U2
    R3 -->|assigned to| U3
    
    U1 -->|member of| T1
    U2 -->|member of| T1
    U2 -->|member of| T2
    U3 -->|member of| T3
    
    %% Record access
    USERS -->|"access via"| RA3
    TEAMS -->|"access via"| RA3
    ROLES -->|"defines"| RA2
    
    class ORG1,BU1,BU2,BU3,BU4 fill:#bbdefb,stroke:#1565c0
    class R1,R2,R3,R4,R5 fill:#c8e6c9,stroke:#2e7d32
    class T1,T2,T3 fill:#fff9c4,stroke:#f9a825
    class U1,U2,U3 fill:#f8bbd0,stroke:#c2185b
    class RA1,RA2,RA3 fill:#e1bee7,stroke:#7b1fa2
```

### 5.2 Record-Level Security Flow

```mermaid
flowchart TD
    %% ==================== RECORD ACCESS DECISION ====================
    
    REQUEST["Record Access<br/>Request"]
    
    %% Ownership Check
    OWNER_CHECK{"User is<br/>Owner?"}
    OWNER_YES["Grant Access"]
    
    %% BU Check
    BU_CHECK{"User's BU<br/>matches?"}
    BU_YES["Grant Access"]
    
    %% Team Check
    TEAM_CHECK{"User in Team<br/>with Access?"}
    TEAM_YES["Grant Access"]
    
    %% Role Check
    ROLE_CHECK{"Role has<br/>Privilege?"}
    ROLE_YES["Grant Access"]
    
    %% Share Check
    SHARE_CHECK{"Record Shared<br/>to User/Team?"}
    SHARE_YES["Grant Access"]
    
    DENY["Deny Access"]
    
    %% Flow
    REQUEST --> OWNER_CHECK
    OWNER_CHECK -->|Yes| OWNER_YES
    OWNER_CHECK -->|No| BU_CHECK
    
    BU_CHECK -->|Yes| BU_YES
    BU_CHECK -->|No| TEAM_CHECK
    
    TEAM_CHECK -->|Yes| TEAM_YES
    TEAM_CHECK -->|No| ROLE_CHECK
    
    ROLE_CHECK -->|Yes| ROLE_YES
    ROLE_CHECK -->|No| SHARE_CHECK
    
    SHARE_CHECK -->|Yes| SHARE_YES
    SHARE_CHECK -->|No| DENY
    
    class REQUEST fill:#e8eaf6,stroke:#3f51b5
    class OWNER_YES,BU_YES,TEAM_YES,ROLE_YES,SHARE_YES fill:#c8e6c9,stroke:#2e7d32
    class DENY fill:#ffcdd2,stroke:#c62828
```

---

## SECTION 6 – Activity & Timeline Relationship Diagram

### 6.1 Activity Party Model

```mermaid
flowchart TB
    %% ==================== ACTIVITY RELATIONSHIPS ====================
    
    subgraph ACTIVITIES["Activities"]
        APPT["Appointment"]
        EMAIL["Email"]
        PHONE["Phone Call"]
        TASK["Task"]
        LETTER["Letter"]
        FAX["Fax"]
        SOCIAL["Social Activity"]
        SERVICE["Service Appointment"]
    end
    
    subgraph PARTICIPANTS["Activity Parties"]
        SENDER["Sender (1)"]
        TO["To Recipient (2)"]
        CC["CC (3)"]
        BCC["BCC (4)"]
        REQUIRED["Required Attendee (5)"]
        OPTIONAL["Optional Attendee (6)"]
        ORGANIZER["Organizer (7)"]
        REGARDING["Regarding (8)"]
    end
    
    subgraph REGARDING_ENTITIES["Regarding Entities"]
        ACCOUNT_ACT["Account"]
        CONTACT_ACT["Contact"]
        LEAD_ACT["Lead"]
        OPP_ACT["Opportunity"]
        CASE_ACT["Case"]
        QUOTE_ACT["Quote"]
        ORDER_ACT["Sales Order"]
        INVOICE_ACT["Invoice"]
    end
    
    %% Activity to Participant
    APPT -->|has| SENDER
    APPT -->|has| TO
    APPT -->|has| CC
    APPT -->|has| REQUIRED
    APPT -->|has| OPTIONAL
    APPT -->|has| ORGANIZER
    
    EMAIL -->|from| SENDER
    EMAIL -->|to| TO
    EMAIL -->|cc| CC
    EMAIL -->|bcc| BCC
    
    PHONE -->|from| SENDER
    PHONE -->|to| TO
    
    TASK -->|assigned_to| REGARDING
    TASK -->|has| SENDER
    
    %% Participant to Regarding
    SENDER --> ACCOUNT_ACT
    SENDER --> CONTACT_ACT
    SENDER --> LEAD_ACT
    
    TO --> ACCOUNT_ACT
    TO --> CONTACT_ACT
    TO --> LEAD_ACT
    TO --> OPP_ACT
    
    REGARDING --> ACCOUNT_ACT
    REGARDING --> CONTACT_ACT
    REGARDING --> LEAD_ACT
    REGARDING --> OPP_ACT
    REGARDING --> CASE_ACT
    REGARDING --> QUOTE_ACT
    REGARDING --> ORDER_ACT
    REGARDING --> INVOICE_ACT
    
    class APPT,EMAIL,PHONE,TASK,LETTER,FAX,SOCIAL,SERVICE fill:#e3f2fd,stroke:#1565c0
    class SENDER,TO,CC,BCC,REQUIRED,OPTIONAL,ORGANIZER,REGARDING fill:#fff8e1,stroke:#ff8f00
    class ACCOUNT_ACT,CONTACT_ACT,LEAD_ACT,OPP_ACT,CASE_ACT,QUOTE_ACT,ORDER_ACT,INVOICE_ACT fill:#c8e6c9,stroke:#2e7d32
```

### 6.2 Timeline Integration Flow

```mermaid
flowchart LR
    %% ==================== TIMELINE ARCHITECTION ====================
    
    ACCOUNT_TL["Account<br/>Timeline"]
    CONTACT_TL["Contact<br/>Timeline"]
    LEAD_TL["Lead<br/>Timeline"]
    OPPORTUNITY_TL["Opportunity<br/>Timeline"]
    
    TIMELINE_AGG["Timeline<br/>Aggregation"]
    
    ACTIVITY_STREAM["Activity<br/>Stream"]
    POSTS["Posts &<br/>Notes"]
    DOCUMENTS["Documents"]
    CONNECTORS["Connectors"]
    
    TIMELINE_VIEW["Timeline<br/>View Control"]
    
    %% Relationships
    ACCOUNT_TL -->|"regarding<br/>AccountId"| TIMELINE_AGG
    CONTACT_TL -->|"regarding<br/>ContactId"| TIMELINE_AGG
    LEAD_TL -->|"regarding<br/>LeadId"| TIMELINE_AGG
    OPPORTUNITY_TL -->|"regarding<br/>OpportunityId"| TIMELINE_AGG
    
    TIMELINE_AGG -->|"sort by<br/>date"| ACTIVITY_STREAM
    TIMELINE_AGG -->|"sort by<br/>date"| POSTS
    TIMELINE_AGG -->|"sort by<br/>date"| DOCUMENTS
    TIMELINE_AGG -->|"sort by<br/>date"| CONNECTORS
    
    ACTIVITY_STREAM -->|"display"| TIMELINE_VIEW
    POSTS -->|"display"| TIMELINE_VIEW
    DOCUMENTS -->|"display"| TIMELINE_VIEW
    CONNECTORS -->|"display"| TIMELINE_VIEW
    
    class ACCOUNT_TL,CONTACT_TL,LEAD_TL,OPPORTUNITY_TL fill:#bbdefb,stroke:#1565c0
    class TIMELINE_AGG fill:#fff9c4,stroke:#f9a825
    class ACTIVITY_STREAM,POSTS,DOCUMENTS,CONNECTORS fill:#c8e6c9,stroke:#2e7d32
    class TIMELINE_VIEW fill:#f8bbd0,stroke:#c2185b
```

---

## SECTION 7 – Financial Calculation Flow

### 7.1 Pricing Calculation Flow

```mermaid
flowchart TD
    %% ==================== PRICING ENGINE ====================
    
    START(("Pricing<br/>Calculation"))
    
    %% Product Pricing
    PRODUCT["Product<br/>Selection"]
    PRICELIST["Price List<br/>Lookup"]
    GET_PRICE["Get Unit<br/>Price"]
    
    %% Line Item
    LINE_QTY["Enter<br/>Quantity"]
    LINE_PRICE["Unit Price"]
    LINE_CALC["Calculate<br/>Line Amount"]
    LINE_AMOUNT["Line Amount<br/>(Qty × Price)"]
    
    %% Discounts
    LINE_DISCOUNT["Line Discount<br/>%"]
    DISCOUNT_CALC["Calculate<br/>Discount"]
    DISCOUNT_AMT["Discount<br/>Amount"]
    NET_LINE["Net Line<br/>(Amount - Discount)"]
    
    %% Extended Pricing
    TOTAL_LINES["Sum All<br/>Lines"]
    SUBTOTAL["Subtotal"]
    
    ORDER_DISCOUNT["Order Discount<br/>% or Amount"]
    ORDER_DISCOUNT_CALC["Calculate Order<br/>Discount"]
    ORDER_DISCOUNT_AMT["Discount Amount"]
    NET_SUBTOTAL["Net Subtotal"]
    
    %% Additional Costs
    FREIGHT["Freight<br/>Amount"]
    TAX_CALC["Calculate<br/>Tax"]
    TAX_AMT["Tax Amount"]
    
    %% Final Total
    TOTAL_CALC["Calculate<br/>Total"]
    TOTAL_AMOUNT["Total Amount"]
    
    %% Base Currency
    TRANSACTION_CURRENCY["Transaction<br/>Currency"]
    EXCHANGE_RATE["Exchange<br/>Rate"]
    CONVERT["Convert to<br/>Base Currency"]
    BASE_AMOUNT["Base Currency<br/>Amount"]
    
    %% Connections
    START --> PRODUCT
    PRODUCT --> PRICELIST
    PRICELIST --> GET_PRICE
    
    GET_PRICE --> LINE_PRICE
    LINE_QTY --> LINE_CALC
    LINE_PRICE --> LINE_CALC
    LINE_CALC --> LINE_AMOUNT
    
    LINE_AMOUNT --> LINE_DISCOUNT
    LINE_DISCOUNT --> DISCOUNT_CALC
    DISCOUNT_CALC --> DISCOUNT_AMT
    DISCOUNT_AMT --> NET_LINE
    
    NET_LINE --> TOTAL_LINES
    TOTAL_LINES --> SUBTOTAL
    
    SUBTOTAL --> ORDER_DISCOUNT
    ORDER_DISCOUNT --> ORDER_DISCOUNT_CALC
    ORDER_DISCOUNT_CALC --> ORDER_DISCOUNT_AMT
    ORDER_DISCOUNT_AMT --> NET_SUBTOTAL
    
    NET_SUBTOTAL --> FREIGHT
    FREIGHT --> TAX_CALC
    TAX_CALC --> TAX_AMT
    
    TAX_AMT --> TOTAL_CALC
    TOTAL_CALC --> TOTAL_AMOUNT
    
    TOTAL_AMOUNT --> TRANSACTION_CURRENCY
    TRANSACTION_CURRENCY --> EXCHANGE_RATE
    EXCHANGE_RATE --> CONVERT
    CONVERT --> BASE_AMOUNT
    
    class START,TOTAL_AMOUNT,BASE_AMOUNT fill:#e3f2fd,stroke:#1565c0
    class PRODUCT,PRICELIST,GET_PRICE fill:#c8e6c9,stroke:#2e7d32
    class LINE_QTY,LINE_PRICE,LINE_CALC fill:#fff8e1,stroke:#ff8f00
    class DISCOUNT_AMT,ORDER_DISCOUNT_AMT,TAX_AMT fill:#fce4ec,stroke:#c2185b
    class SUBTOTAL,NET_SUBTOTAL fill:#f3e5f5,stroke:#7b1fa2
```

### 7.2 Rollup Calculation Flow

```mermaid
flowchart TD
    %% ==================== OPPORTUNITY ROLLUP ====================
    
    OPP_HEADER["Opportunity<br/>Header"]
    
    subgraph LINE_ITEMS["Opportunity Products"]
        LP1["Line Item 1"]
        LP2["Line Item 2"]
        LP3["Line Item N"]
    end
    
    CALCULATE["Calculate<br/>Totals"]
    EXTENDED_AMT["Extended Amount<br/>(Qty × Price)"]
    LINE_DISC["Line Discount"]
    LINE_TAX["Tax"]
    
    AGGREGATE["Aggregate to<br/>Opportunity"]
    
    TOTAL_LINE_AMT["Total Line<br/>Amount"]
    TOTAL_DISC["Total Discount"]
    TOTAL_TAX["Total Tax"]
    
    OTHER_CHARGES["Freight &<br/>Other Charges"]
    
    OPP_TOTAL["Opportunity<br/>Total Amount"]
    
    OPP_UPDATE["Update<br/>Opportunity"]
    
    OPPORTUNITY["Opportunity<br/>Entity"]
    
    %% Flow
    OPP_HEADER --> LINE_ITEMS
    
    LP1 --> CALCULATE
    LP2 --> CALCULATE
    LP3 --> CALCULATE
    
    CALCULATE --> EXTENDED_AMT
    EXTENDED_AMT --> LINE_DISC
    LINE_DISC --> LINE_TAX
    
    LINE_ITEMS -->|"all lines"| AGGREGATE
    AGGREGATE -->|"sum| TOTAL_LINE_AMT
    AGGREGATE -->|"sum| TOTAL_DISC
    AGGREGATE -->|"sum| TOTAL_TAX
    
    TOTAL_LINE_AMT --> OTHER_CHARGES
    TOTAL_DISC --> OTHER_CHARGES
    TOTAL_TAX --> OTHER_CHARGES
    
    OTHER_CHARGES --> OPP_TOTAL
    
    OPP_TOTAL --> OPP_UPDATE
    OPP_UPDATE --> OPPORTUNITY
    
    class OPP_HEADER,OPPORTUNITY fill:#e3f2fd,stroke:#1565c0
    class LP1,LP2,LP3,CALCULATE fill:#c8e6c9,stroke:#2e7d32
    class TOTAL_LINE_AMT,TOTAL_DISC,TOTAL_TAX,OPP_TOTAL fill:#fff8e1,stroke:#ff8f00
    class AGGREGATE,OPP_UPDATE fill:#fce4ec,stroke:#c2185b
```

### 7.3 Multi-Currency Conversion Flow

```mermaid
flowchart LR
    %% ==================== CURRENCY CONVERSION ====================
    
    TRANSACTION["Transaction<br/>Currency"]
    TRANSACTION_AMT["Amount:<br/>10,000 EUR"]
    
    EXCHANGE_RATE["Exchange Rate<br/>Table Lookup"]
    RATE_VALUE["Rate: 1.09"]
    
    BASE_CURRENCY["Base Currency<br/>USD"]
    BASE_AMOUNT["Base Amount:<br/>10,900 USD"]
    
    HISTORICAL["Historical Rate<br/>Storage"]
    
    REPORTING["Reporting<br/>In Base Currency"]
    
    %% Flow
    TRANSACTION -->|"user enters"| TRANSACTION_AMT
    TRANSACTION_AMT -->|"gets| EXCHANGE_RATE
    EXCHANGE_RATE -->|"returns| RATE_VALUE
    RATE_VALUE -->|"applies| BASE_CURRENCY
    BASE_CURRENCY -->|"calculates| BASE_AMOUNT
    
    RATE_VALUE -->|"stores for| HISTORICAL
    HISTORICAL -->|"used for| REPORTING
    
    class TRANSACTION,TRANSACTION_AMT fill:#e3f2fd,stroke:#1565c0
    class EXCHANGE_RATE,RATE_VALUE fill:#c8e6c9,stroke:#2e7d32
    class BASE_CURRENCY,BASE_AMOUNT fill:#fff8e1,stroke:#ff8f00
    class HISTORICAL,REPORTING fill:#f3e5f5,stroke:#7b1fa2
```

---

*Document generated by Senior Microsoft Dynamics 365 CRM Solution Architect & Dataverse Database Expert*
*All diagrams follow Enterprise Dynamics 365 / Dataverse architecture patterns*
