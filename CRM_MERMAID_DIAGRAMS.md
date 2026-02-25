# Microsoft Dynamics 365 CRM - Complete Mermaid Diagrams

## Database Schema Overview

**Database:** Microsoft Dynamics 365 CRM (On-Premise)  
**Total Tables:** 300+  
**Primary Key Pattern:** uniqueidentifier (GUID)  
**Ownership Pattern:** OwnerId + OwnerIdType (8=User, 9=Team)

---

## 1. COMPLETE ER DIAGRAM - CORE TABLES

```mermaid
erDiagram
    %% ==================== ORGANIZATION HIERARCHY ====================
    ORGANIZATION_BASE ||--o{ BUSINESS_UNIT_BASE : OrganizationId
    ORGANIZATION_BASE ||--o{ TRANSACTION_CURRENCY_BASE : Currency
    ORGANIZATION_BASE ||--o{ CALENDAR_BASE : DefaultCalendar
    
    BUSINESS_UNIT_BASE ||--o{ BUSINESS_UNIT_BASE : ParentBusinessUnitId
    BUSINESS_UNIT_BASE ||--o{ SYSTEM_USER_BASE : BusinessUnitId
    BUSINESS_UNIT_BASE ||--o{ TEAM_BASE : BusinessUnitId
    BUSINESS_UNIT_BASE ||--o{ ACCOUNT_BASE : OwningBusinessUnit
    BUSINESS_UNIT_BASE ||--o{ CONTACT_BASE : OwningBusinessUnit

    %% ==================== USER SECURITY ====================
    SYSTEM_USER_BASE ||--o{ TEAM_MEMBERSHIP : SystemUserId
    SYSTEM_USER_BASE ||--o{ SYSTEM_USER_ROLES : SystemUserId
    SYSTEM_USER_BASE ||--o{ SYSTEM_USER_MANAGER_MAP : SystemUserId
    SYSTEM_USER_BASE ||--o{ USER_SETTINGS_BASE : SystemUserId
    SYSTEM_USER_BASE ||--|{ OWNER_BASE : OwnerId
    
    TEAM_BASE ||--o{ TEAM_MEMBERSHIP : TeamId
    TEAM_BASE ||--o{ TEAM_ROLES : TeamId
    TEAM_BASE ||--|{ OWNER_BASE : OwnerId
    
    ROLE_BASE ||--o{ ROLE_PRIVILEGES_BASE : RoleId
    ROLE_BASE ||--o{ SYSTEM_USER_ROLES : RoleId
    ROLE_BASE ||--o{ TEAM_ROLES : RoleId
    
    PRIVILEGE_BASE ||--o{ ROLE_PRIVILEGES_BASE : PrivilegeId

    %% ==================== CUSTOMER ENTITIES ====================
    ACCOUNT_BASE ||--o{ CONTACT_BASE : ParentContactId
    ACCOUNT_BASE ||--o{ ACCOUNT_BASE : ParentAccountId
    ACCOUNT_BASE ||--o{ CUSTOMER_ADDRESS_BASE : ParentId
    
    CONTACT_BASE ||--o{ CONTACT_BASE : ParentContactId
    
    LEAD_BASE ||--o{ ACCOUNT_BASE : ParentAccountId
    LEAD_BASE ||--o{ CONTACT_BASE : ParentContactId
    LEAD_BASE ||--o{ LEAD_ADDRESS_BASE : ParentId
    LEAD_BASE ||--o{ OPPORTUNITY_BASE : OriginatingLeadId

    %% ==================== OPPORTUNITY PIPELINE ====================
    OPPORTUNITY_BASE ||--o{ QUOTE_BASE : OpportunityId
    OPPORTUNITY_BASE ||--o{ OPPORTUNITY_PRODUCT_BASE : OpportunityId
    OPPORTUNITY_BASE ||--o{ OPPORTUNITY_COMPETITORS : OpportunityId
    
    QUOTE_BASE ||--o{ QUOTE_DETAIL_BASE : QuoteId
    QUOTE_BASE ||--o{ SALES_ORDER_BASE : QuoteId
    
    SALES_ORDER_BASE ||--o{ SALES_ORDER_DETAIL_BASE : SalesOrderId
    SALES_ORDER_BASE ||--o{ INVOICE_BASE : SalesOrderId
    
    INVOICE_BASE ||--o{ INVOICE_DETAIL_BASE : InvoiceId

    %% ==================== PRODUCTS & PRICING ====================
    PRODUCT_BASE ||--o{ PRODUCT_PRICE_LEVEL_BASE : ProductId
    PRODUCT_BASE ||--o{ OPPORTUNITY_PRODUCT_BASE : ProductId
    PRODUCT_BASE ||--o{ QUOTE_DETAIL_BASE : ProductId
    PRODUCT_BASE ||--o{ SALES_ORDER_DETAIL_BASE : ProductId
    PRODUCT_BASE ||--o{ INVOICE_DETAIL_BASE : ProductId
    
    PRICE_LEVEL_BASE ||--o{ PRODUCT_PRICE_LEVEL_BASE : PriceLevelId
    
    UOM_SCHEDULE_BASE ||--o{ UOM_BASE : UoMScheduleId
    
    DISCOUNT_TYPE_BASE ||--o{ DISCOUNT_BASE : DiscountTypeId

    %% ==================== ACTIVITIES ====================
    ACTIVITY_POINTER_BASE ||--o{ ACTIVITY_PARTY_BASE : ActivityId
    ACTIVITY_POINTER_BASE ||--|{ EMAIL_BASE : ActivityId
    ACTIVITY_POINTER_BASE ||--|{ PHONE_CALL_BASE : ActivityId
    ACTIVITY_POINTER_BASE ||--|{ TASK_BASE : ActivityId
    ACTIVITY_POINTER_BASE ||--|{ APPOINTMENT_BASE : ActivityId
    ACTIVITY_POINTER_BASE ||--|{ LETTER_BASE : ActivityId
    ACTIVITY_POINTER_BASE ||--|{ FAX_BASE : ActivityId
    ACTIVITY_POINTER_BASE ||--|{ SERVICE_APPOINTMENT_BASE : ActivityId
    
    ACCOUNT_BASE ||--o{ ACTIVITY_POINTER_BASE : RegardingObjectId
    CONTACT_BASE ||--o{ ACTIVITY_POINTER_BASE : RegardingObjectId
    LEAD_BASE ||--o{ ACTIVITY_POINTER_BASE : RegardingObjectId
    OPPORTUNITY_BASE ||--o{ ACTIVITY_POINTER_BASE : RegardingObjectId
    
    ACCOUNT_BASE ||--o{ ACTIVITY_PARTY_BASE : PartyId
    CONTACT_BASE ||--o{ ACTIVITY_PARTY_BASE : PartyId
    LEAD_BASE ||--o{ ACTIVITY_PARTY_BASE : PartyId
    OPPORTUNITY_BASE ||--o{ ACTIVITY_PARTY_BASE : PartyId

    %% ==================== CONNECTION RELATIONSHIPS ====================
    CONNECTION_ROLE_BASE ||--o{ CONNECTION_BASE : Record1RoleId
    CONNECTION_ROLE_BASE ||--o{ CONNECTION_BASE : Record2RoleId
    
    ACCOUNT_BASE ||--o{ CONNECTION_BASE : Record1Id
    ACCOUNT_BASE ||--o{ CONNECTION_BASE : Record2Id
    CONTACT_BASE ||--o{ CONNECTION_BASE : Record1Id
    CONTACT_BASE ||--o{ CONNECTION_BASE : Record2Id

    %% ==================== RECORD SHARING ====================
    PRINCIPAL_OBJECT_ACCESS ||--o{ SYSTEM_USER_BASE : PrincipalId
    PRINCIPAL_OBJECT_ACCESS ||--o{ TEAM_BASE : PrincipalId
    PRINCIPAL_OBJECT_ACCESS ||--o{ ACCOUNT_BASE : ObjectId
    PRINCIPAL_OBJECT_ACCESS ||--o{ CONTACT_BASE : ObjectId
    PRINCIPAL_OBJECT_ACCESS ||--o{ LEAD_BASE : ObjectId
    PRINCIPAL_OBJECT_ACCESS ||--o{ OPPORTUNITY_BASE : ObjectId

    %% ==================== AUDIT ====================
    SYSTEM_USER_BASE ||--o{ AUDIT_BASE : UserId
    ACCOUNT_BASE ||--o{ AUDIT_BASE : ObjectId
    CONTACT_BASE ||--o{ AUDIT_BASE : ObjectId
    LEAD_BASE ||--o{ AUDIT_BASE : ObjectId
    OPPORTUNITY_BASE ||--o{ AUDIT_BASE : ObjectId

    %% ==================== QUEUES ====================
    QUEUE_BASE ||--o{ QUEUE_ITEM_BASE : QueueId
    QUEUE_BASE ||--o{ QUEUE_MEMBERSHIP : QueueId
    SYSTEM_USER_BASE ||--o{ QUEUE_MEMBERSHIP : SystemUserId
    
    QUEUE_ITEM_BASE ||--o{ ACCOUNT_BASE : ObjectId
    QUEUE_ITEM_BASE ||--o{ CONTACT_BASE : ObjectId
    QUEUE_ITEM_BASE ||--o{ LEAD_BASE : ObjectId
    QUEUE_ITEM_BASE ||--o{ OPPORTUNITY_BASE : ObjectId
    QUEUE_ITEM_BASE ||--o{ ACTIVITY_POINTER_BASE : ObjectId

    %% ==================== TERRITORY ====================
    TERRITORY_BASE ||--o{ ACCOUNT_BASE : TerritoryId
    TERRITORY_BASE ||--o{ SYSTEM_USER_BASE : TerritoryId

    %% ==================== CONTRACTS & ENTITLEMENTS ====================
    CONTRACT_TEMPLATE_BASE ||--o{ CONTRACT_BASE : ContractTemplateId
    CONTRACT_BASE ||--o{ CONTRACT_DETAIL_BASE : ContractId
    
    ENTITLEMENT_TEMPLATE_BASE ||--o{ ENTITLEMENT_BASE : EntitlementTemplateId
    
    INCIDENT_BASE ||--o{ CONTRACT_BASE : ContractId

    %% ==================== MARKETING ====================
    CAMPAIGN_BASE ||--o{ CAMPAIGN_ITEM_BASE : CampaignId
    CAMPAIGN_BASE ||--o{ CAMPAIGN_RESPONSE_BASE : CampaignId
    CAMPAIGN_BASE ||--o{ LIST_BASE : CampaignId
    
    LIST_BASE ||--o{ LIST_MEMBER_BASE : ListId
    ACCOUNT_BASE ||--o{ LIST_MEMBER_BASE : EntityId
    CONTACT_BASE ||--o{ LIST_MEMBER_BASE : EntityId
    LEAD_BASE ||--o{ LIST_MEMBER_BASE : EntityId
```

---

## 2. LEAD MANAGEMENT - STATE MACHINE

```mermaid
stateDiagram-v2
    [*] --> NewLead : Create Lead
    NewLead --> Qualified : Qualify Lead
    NewLead --> Disqualified : Disqualify
    Qualified --> Developed : Develop
    Developed --> Proposed : Propose
    Proposed --> Won : Close Won
    Proposed --> Lost : Close Lost
    Won --> [*] : Convert to Account
    Lost --> [*] : Archive
    Disqualified --> [*] : Archive
    
    note right of NewLead: StateCode=0\nStatusCode=0
    note right of Qualified: StateCode=1\nStatusCode=3
    note right of Disqualified: StateCode=2\nStatusCode=5
    note right of Won: StateCode=1\nStatusCode=3
    note right of Lost: StateCode=2\nStatusCode=4
```

---

## 3. OPPORTUNITY PIPELINE - STATE MACHINE

```mermaid
stateDiagram-v2
    [*] --> OpenQualify : Create Opportunity
    OpenQualify --> OpenDevelop : Move to Develop
    OpenDevelop --> OpenProposepose
    Open : Move to ProPropose --> Won : Close Won
    OpenPropose --> Lost : Close Lost
    Won --> [*] : Create Sales Order
    Lost --> [*] : Archive
    OpenQualify --> Lost : Close Lost
    OpenDevelop --> Lost : Close Lost
    
    note right of OpenQualify: StateCode=0\nStepName=Qualify
    note right of OpenDevelop: StateCode=0\nStepName=Develop
    note right of OpenPropose: StateCode=0\nStepName=Propose
    note right of Won: StateCode=1\nStatusCode=3
    note right of Lost: StateCode=2\nStatusCode=4
```

---

## 4. SALES PROCESS FLOW

```mermaid
flowchart TB
    subgraph LEAD_QUALIFY ["Lead Qualification"]
        LC1[Lead Created] --> LC2{Validate Lead}
        LC2 -->|Valid| LC3{Has Customer Info?}
        LC3 -->|New| LC4[Create Account]
        LC3 -->|Existing| LC5[Select Account]
        LC4 --> LC6[Create Contact]
        LC5 --> LC6
        LC6 --> LC7[Create Opportunity]
        LC7 --> LC8[Update Lead Status]
    end
    
    subgraph OPPORTUNITY ["Opportunity Management"]
        OM1[Opportunity Created] --> OM2{Define Sales Stage}
        OM2 --> OM3[Add Products]
        OM3 --> OM4[Create Quote]
        OM4 --> OM5{Revise Quote?}
        OM5 -->|Yes| OM4
        OM5 --> OM6{Customer Accepts?}
        OM6 -->|Yes| OM7[Create Sales Order]
        OM6 -->|No| OM8[Lost Opportunity]
    end
    
    subgraph ORDER_TO_CASH ["Order to Cash"]
        OTC1[Order Created] --> OTC2{Fulfill Order}
        OTC2 --> OTC3[Create Invoice]
        OTC3 --> OTC4{Customer Pays?}
        OTC4 -->|Yes| OTC5[Revenue Recognized]
        OTC4 -->|No| OTC6[Send Reminder]
        OTC6 --> OTC3
    end
    
    LC7 --> OM1
    OM7 --> OTC1
    
    style LC1 fill:#e3f2fd
    style LC7 fill:#c8e6c9
    style OM1 fill:#fff3e0
    style OM7 fill:#c8e6c9
    style OTC5 fill:#c8e6c9
```

---

## 5. SECURITY & ACCESS CONTROL FLOW

```mermaid
flowchart TD
    subgraph AUTHENTICATION ["Authentication"]
        A1[User Login] --> A2{Valid Credentials?}
        A2 -->|No| A3[Login Failed]
        A2 -->|Yes| A4[Get SystemUserId]
        A4 --> A5[Load User Roles]
    end
    
    subgraph RECORD_ACCESS ["Record Access Check"]
        A5 --> R1[Query Record]
        R1 --> R2{Is Owner?}
        R2 -->|Yes| GRANT[Grant Access]
        R2 -->|No| R3{Member of Owner's Team?}
        R3 -->|Yes| GRANT
        R3 -->|No| R4{Has POA Record?}
        R4 -->|Yes| GRANT
        R4 -->|No| R5{Same Business Unit?}
        R5 -->|Yes| R6{Has Privilege?}
        R5 -->|No| DENY[Access Denied]
        R6 -->|Yes| GRANT
        R6 -->|No| DENY
    end
    
    subgraph FIELD_SECURITY ["Field Level Security"]
        GRANT --> F1{Field Has Security?}
        F1 -->|Yes| F2{Current User Has Access?}
        F1 -->|No| RETURN[Return Data]
        F2 -->|Yes| RETURN
        F2 -->|No| MASK[Return Masked Value]
    end
    
    GRANT --> F1
    DENY --> ERROR[Display Error]
    
    style GRANT fill:#c8e6c9
    style DENY fill:#ffcdd2
    style RETURN fill:#c8e6c9
    style MASK fill:#fff3e0
    style ERROR fill:#ffcdd2
```

---

## 6. RECORD SHARING WORKFLOW

```mermaid
flowchart LR
    subgraph SHARE_RECORD ["Share Record"]
        S1[Select Record] --> S2[Click Share]
        S2 --> S3[Select User/Team]
        S3 --> S4{Select Access Level}
        S4 -->|Read| SL1[AccessMask=1]
        S4 -->|Write| SL2[AccessMask=3]
        S4 -->|Delete| SL3[AccessMask=7]
        S4 -->|All| SL4[AccessMask=63]
        SL1 --> S5[Click Share]
    end
    
    subgraph VALIDATE ["Security Validation"]
        S5 --> V1{Has Share Privilege?}
        V1 -->|No| ERR1[Error: No Privilege]
        V1 -->|Yes| V2{Owns Record?}
        V2 -->|No| V3{Has Assign Privilege?}
        V2 -->|Yes| V4[Insert POA Record]
        V3 -->|No| ERR2[Error: No Privilege]
        V3 -->|Yes| V4
    end
    
    subgraph UPDATE ["Update Tables"]
        V4 --> U1[Insert PrincipalObjectAccess]
        U1 --> U2[Update PrincipalEntityMap]
        U2 --> U3[Create Audit Record]
        U3 --> SUCCESS[Share Complete]
    end
    
    ERR1 --> END
    ERR2 --> END
    
    style S1 fill:#e3f2fd
    style V4 fill:#c8e6c9
    style SUCCESS fill:#c8e6c9
    style ERR1 fill:#ffcdd2
    style ERR2 fill:#ffcdd2
```

---

## 7. ACTIVITY MANAGEMENT FLOW

```mermaid
flowchart TB
    subgraph CREATE_ACTIVITY ["Create Activity"]
        AM1[Select Activity Type] --> AM2{Fill Required Fields}
        AM2 --> AM3[Set Owner]
        AM3 --> AM4{Add Regarding?}
        AM4 -->|Yes| AM5[Select Regarding Record]
        AM4 -->|No| AM6[Skip Regarding]
        AM5 --> AM7[Add Participants]
    end
    
    subgraph PARTICIPANTS ["Activity Parties"]
        AM6 --> AM7
        AM7 --> AM8{Create ActivityParty Records}
        AM8 --> AM9[Add Owner as Party]
        AM9 --> AM10{Regarding Party?}
        AM10 -->|Yes| AM11[Add Regarding Party]
        AM10 -->|No| AM12[Add To/Cc Parties]
        AM11 --> AM12
    end
    
    subgraph SAVE ["Save & Complete"]
        AM12 --> SAVE1[Insert ActivityPointerBase]
        SAVE1 --> SAVE2[Insert ActivityPartyBase]
        SAVE2 --> SAVE3[Update PrincipalEntityMap]
        SAVE3 --> SAVE4[Create Audit Record]
        SAVE4 --> COMPLETE[Activity Created]
    end
    
    note right of AM1: TypeCodes:\n4201=Appointment\n4202=Email\n4210=PhoneCall\n4212=Task
    
    style AM1 fill:#e3f2fd
    style SAVE1 fill:#c8e6c9
    style COMPLETE fill:#c8e6c9
```

---

## 8. DATA MODEL - OWNERSHIP

```mermaid
erDiagram
    OWNER_BASE ||--o{ ACCOUNT_BASE : OwnerId
    OWNER_BASE ||--o{ CONTACT_BASE : OwnerId
    OWNER_BASE ||--o{ LEAD_BASE : OwnerId
    OWNER_BASE ||--o{ OPPORTUNITY_BASE : OwnerId
    OWNER_BASE ||--o{ QUOTE_BASE : OwnerId
    OWNER_BASE ||--o{ SALES_ORDER_BASE : OwnerId
    OWNER_BASE ||--o{ INVOICE_BASE : OwnerId
    OWNER_BASE ||--o{ CONTRACT_BASE : OwnerId
    OWNER_BASE ||--o{ INCIDENT_BASE : OwnerId
    OWNER_BASE ||--o{ ANNOTATION_BASE : OwnerId
    OWNER_BASE ||--o{ ACTIVITY_POINTER_BASE : OwnerId
    
    OWNER_BASE {
        guid OwnerId PK
        int OwnerIdType "8=User, 9=Team"
    }
    
    SYSTEM_USER_BASE {
        guid SystemUserId PK
        guid BusinessUnitId FK
        string FullName
        string DomainName
        int StateCode
    }
    
    TEAM_BASE {
        guid TeamId PK
        guid BusinessUnitId FK
        string Name
        int TeamType
    }
    
    ACCOUNT_BASE {
        guid AccountId PK
        guid OwnerId FK
        guid OwningBusinessUnit FK
        string Name
        int StateCode
        int StatusCode
    }
```

---

## 9. CUSTOMER RELATIONSHIP MODEL

```mermaid
erDiagram
    ACCOUNT_BASE ||--o{ CONTACT_BASE : PrimaryContact
    ACCOUNT_BASE ||--o{ ACCOUNT_BASE : ParentAccount
    ACCOUNT_BASE ||--o{ OPPORTUNITY_BASE : CustomerId
    ACCOUNT_BASE ||--o{ QUOTE_BASE : CustomerId
    ACCOUNT_BASE ||--o{ SALES_ORDER_BASE : CustomerId
    ACCOUNT_BASE ||--o{ INVOICE_BASE : CustomerId
    ACCOUNT_BASE ||--o{ CONTRACT_BASE : CustomerId
    ACCOUNT_BASE ||--o{ INCIDENT_BASE : CustomerId
    
    CONTACT_BASE ||--o{ OPPORTUNITY_BASE : CustomerId
    CONTACT_BASE ||--o{ QUOTE_BASE : CustomerId
    CONTACT_BASE ||--o{ SALES_ORDER_BASE : CustomerId
    CONTACT_BASE ||--o{ INVOICE_BASE : CustomerId
    CONTACT_BASE ||--o{ CONTRACT_BASE : CustomerId
    CONTACT_BASE ||--o{ INCIDENT_BASE : CustomerId
    
    LEAD_BASE ||--o{ ACCOUNT_BASE : ConvertedToAccount
    LEAD_BASE ||--o{ CONTACT_BASE : ConvertedToContact
    LEAD_BASE ||--o{ OPPORTUNITY_BASE : OriginatingLeadId
    
    OPPORTUNITY_BASE {
        guid OpportunityId PK
        guid CustomerId FK "Account or Contact"
        int CustomerIdType "1=Account, 2=Contact"
        guid OriginatingLeadId FK
        string Name
        money EstimatedValue
        int StateCode
        string StepName
    }
    
    QUOTE_BASE {
        guid QuoteId PK
        guid CustomerId FK
        int CustomerIdType
        guid OpportunityId FK
        money TotalAmount
        int StatusCode
    }
```

---

## 10. AUDIT TRACKING FLOW

```mermaid
flowchart TD
    subgraph TRIGGER ["Audit Trigger"]
        AT1[User Modifies Record] --> AT2[Platform Intercepts]
        AT2 --> AT3[Capture Old Values]
        AT3 --> AT4[Perform UPDATE]
    end
    
    subgraph CREATE_AUDIT ["Create Audit"]
        AT4 --> A1[Determine Changed Fields]
        A1 --> A2[Generate AttributeMask]
        A2 --> A3[Get User Context]
        A3 --> A4[Insert AuditBase Record]
    end
    
    subgraph STORE_DETAILS ["Store Details"]
        A4 --> D1{Detailed Tracking Enabled?}
        D1 -->|Yes| D2[Insert AuditDetail Values]
        D1 -->|No| D3[Skip Details]
        D2 --> COMPLETE
        D3 --> COMPLETE
    end
    
    COMPLETE[Complete]
    
    note right of AT1: Operation Types:\n1=Create\n2=Update\n3=Delete
    
    style AT1 fill:#e3f2fd
    style A4 fill:#c8e6c9
    style COMPLETE fill:#c8e6c9
```

---

## 11. SEQUENCE: LEAD QUALIFICATION

```mermaid
sequenceDiagram
    participant U as User
    participant S as CRM Service
    participant L as LeadBase
    participant A as AccountBase
    participant C as ContactBase
    participant O as OpportunityBase
    participant POA as PrincipalObjectAccess
    participant AU as AuditBase
    
    U->>S: Qualify Lead
    S->>L: Validate Lead State
    L-->>S: Return StateCode=0
    
    alt Valid State
        S->>A: Check Existing Account
        A-->>S: Return Result
        
        alt Create New Account
            S->>A: INSERT AccountBase
            A-->>S: Return AccountId
        end
        
        S->>C: Check Existing Contact
        C-->>S: Return Result
        
        alt Create New Contact
            S->>C: INSERT ContactBase
            C-->>S: Return ContactId
        end
        
        S->>O: INSERT OpportunityBase
        O->>O: Set CustomerId, CustomerIdType
        O->>O: Set OriginatingLeadId
        O-->>S: Return OpportunityId
        
        S->>L: UPDATE Lead StateCode=1
        L-->>S: Confirm
        
        S->>POA: Update PrincipalEntityMap
        POA-->>S: Confirm
        
        S->>AU: INSERT Audit Record
        AU-->>S: Confirm
        
        S->>U: Return Success
    else Invalid State
        S->>U: Return Error
    end
```

---

## 12. SEQUENCE: OPPORTUNITY CLOSE (WON)

```mermaid
sequenceDiagram
    participant U as User
    participant S as CRM Service
    participant O as OpportunityBase
    participant Q as QuoteBase
    participant SO as SalesOrderBase
    participant I as InvoiceBase
    participant A as ActivityPointerBase
    participant AU as AuditBase
    
    U->>S: Close Opportunity (Won)
    S->>O: Validate State
    O-->>S: Return StateCode=0
    
    alt Can Close
        S->>O: UPDATE StateCode=1, StatusCode=3
        O->>O: Set ActualValue, ActualCloseDate
        O-->>S: Confirm
        
        S->>Q: SELECT Quotes
        Q-->>S: Return Quotes
        
        loop For Each Quote
            S->>Q: UPDATE StateCode=1
        end
        
        S->>SO: INSERT SalesOrderBase
        SO->>SO: Copy from Quote
        SO-->>S: Return SalesOrderId
        
        loop For Each OrderDetail
            S->>SO: INSERT SalesOrderDetailBase
        end
        
        S->>I: INSERT InvoiceBase
        I->>I: Copy from SalesOrder
        I-->>S: Return InvoiceId
        
        S->>A: INSERT OpportunityClose Activity
        A-->>S: Confirm
        
        S->>AU: INSERT Audit Records
        AU-->>S: Confirm
        
        S->>U: Return Success
    else Cannot Close
        S->>U: Return Error
    end
```

---

## 13. SYSTEM USER & TEAM RELATIONSHIPS

```mermaid
erDiagram
    SYSTEM_USER_BASE ||--o{ SYSTEM_USER_ROLES : SystemUserId
    SYSTEM_USER_BASE ||--o{ TEAM_MEMBERSHIP : SystemUserId
    SYSTEM_USER_BASE ||--o{ USER_SETTINGS_BASE : SystemUserId
    SYSTEM_USER_BASE ||--o{ SYSTEM_USER_MANAGER_MAP : SystemUserId
    SYSTEM_USER_BASE ||--o{ QUEUE_MEMBERSHIP : SystemUserId
    
    TEAM_BASE ||--o{ TEAM_MEMBERSHIP : TeamId
    TEAM_BASE ||--o{ TEAM_ROLES : TeamId
    
    ROLE_BASE ||--o{ SYSTEM_USER_ROLES : RoleId
    ROLE_BASE ||--o{ TEAM_ROLES : RoleId
    
    ROLE_BASE ||--o{ ROLE_PRIVILEGES_BASE : RoleId
    PRIVILEGE_BASE ||--o{ ROLE_PRIVILEGES_BASE : PrivilegeId
    
    SYSTEM_USER_ROLES {
        guid SystemUserId PK,FK
        guid RoleId PK,FK
    }
    
    TEAM_MEMBERSHIP {
        guid TeamId PK,FK
        guid SystemUserId PK,FK
    }
    
    TEAM_ROLES {
        guid TeamId PK,FK
        guid RoleId PK,FK
    }
    
    ROLE_PRIVILEGES_BASE {
        guid RolePrivilegeId PK
        guid RoleId FK
        guid PrivilegeId FK
        int PrivilegeDepth
    }
```

---

## 14. INHERITED SECURITY MODEL

```mermaid
flowchart LR
    subgraph PRIVILEGES ["Privilege Definition"]
        P1[PrivilegeBase] --> P2[Read]
        P1 --> P3[Write]
        P1 --> P4[Delete]
        P1 --> P5[Append]
        P1 --> P6[AppendTo]
        P1 --> P7[Share]
        P1 --> P8[Assign]
    end
    
    subgraph ROLE_ASSIGNMENT ["Role Assignment"]
        P2 --> R1[RolePrivileges]
        P3 --> R1
        P4 --> R1
        P5 --> R1
        P6 --> R1
        P7 --> R1
        P8 --> R1
        
        R1 --> R2[SystemUserRoles]
        R1 --> R3[TeamRoles]
    end
    
    subgraph USER_CONTEXT ["User Context"]
        R2 --> U1[SystemUserBase]
        R3 --> T1[TeamBase]
        T1 --> TM[TeamMembership]
        TM --> U1
    end
    
    subgraph RECORD_ACCESS ["Record Access"]
        U1 --> RA1[Owner Check]
        RA1 --> RA2[POA Check]
        RA2 --> RA3[BU Check]
    end
    
    note right of P1: PrivilegeId uniqueidentifier\nName string\nAccessRight integer
```

---

*All diagrams are compatible with https://mermaid.live*  
*Generated for Microsoft Dynamics 365 CRM Database*
