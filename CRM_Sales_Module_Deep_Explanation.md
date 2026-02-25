is scr# CRM Sales Module
## Data Structure and Data Flow Explanation Document

---

## 1. Introduction

### What is this CRM Sales Module?

This CRM Sales Module is a database system that manages the complete sales process - from the first time we meet a potential customer until we receive payment. It helps our sales team track:

- **Who** our customers are (companies and contacts)
- **What** they are interested in buying (products and services)
- **Where** we are in the sales process (lead, opportunity, quote, order, invoice)
- **Who** is responsible for each sales opportunity
- **How much** we expect to sell (financial tracking)

In simple terms, this system makes sure no customer falls through the cracks and every sale is properly tracked from start to finish.

---

## 2. Overall System Overview

### How the System is Organized

The CRM Sales Module is organized into four main groups:

#### Core Business Entities
These are the main tables that track customers and sales:

| Entity | Purpose |
|--------|---------|
| **Account** | Represents companies/businesses we sell to |
| **Contact** | Represents individual people at those companies |
| **Lead** | A potential customer who has shown interest |
| **Opportunity** | A qualified sales deal we're actively working on |
| **Quote** | A formal price proposal sent to the customer |
| **SalesOrder** | A confirmed purchase order from the customer |
| **Invoice** | A bill sent to the customer requesting payment |

#### Transaction Entities (Line Items)
These track the products/services in each transaction:

| Entity | Purpose |
|--------|---------|
| **OpportunityProduct** | Products added to an opportunity |
| **QuoteDetail** | Products added to a quote |
| **OrderDetail** | Products added to a sales order |
| **InvoiceDetail** | Products added to an invoice |

#### Supporting Entities
These provide reference data and pricing:

| Entity | Purpose |
|--------|---------|
| **Product** | The products and services we sell |
| **PriceList** | Pricing rules and price lists |
| **TransactionCurrency** | Different currencies for international sales |

#### Security Structure
These control who can see and do what:

| Entity | Purpose |
|--------|---------|
| **SystemUser** | People who use the system |
| **BusinessUnit** | Organizational departments/teams |
| **Role** | Permissions and access levels |

---

## 3. Data Structure Explanation

This section explains each table in detail.

---

### CORE ENTITY: ACCOUNT

#### Purpose
The Account table stores information about companies or businesses we sell to. It is the parent entity for most customer-related data.

#### Important Fields

| Field | Data Type | Description |
|-------|-----------|-------------|
| AccountId | Unique ID | Primary key - unique identifier for each account |
| Name | Text (160 chars) | Company name - **REQUIRED FIELD** |
| AccountNumber | Text (20 chars) | Our internal reference number |
| IndustryCode | Number | Type of industry (e.g., Manufacturing, IT, Healthcare) |
| Revenue | Money | Annual revenue of the company |
| NumberOfEmployees | Number | Size of the company |
| CreditLimit | Money | Maximum credit we extend to this customer |
| CreditOnHold | Yes/No | If customer has payment issues |
| Telephone1 | Text | Main phone number |
| EmailAddress1 | Text | Main email address |
| WebsiteUrl | Text | Company website |

#### Primary Key
```
AccountId (UniqueIdentifier) - Auto-generated unique ID
```

#### Foreign Keys

| Field | References | Meaning |
|-------|------------|---------|
| OwnerId | SystemUser | Who owns/responsible for this account |
| OwningBusinessUnit | BusinessUnit | Which department owns this record |
| ParentAccountId | Account | For company hierarchies (parent-subsidiary) |
| PrimaryContactId | Contact | Main contact person at this company |
| PriceLevelId | PriceList | Default price list for this customer |
| TransactionCurrencyId | Currency | Currency used for transactions |

#### Status Tracking
- **StateCode**: 0 = Active, 1 = Inactive
- **StatusCode**: 1 = Active, 2 = Inactive

#### Audit Fields
- **CreatedOn**: When the record was created
- **CreatedBy**: Who created it
- **ModifiedOn**: When it was last changed
- **ModifiedBy**: Who last changed it
- **VersionNumber**: Counter for detecting changes (concurrency control)

#### How It Connects
- One Account can have **many Contacts**
- One Account can have **many Opportunities**
- One Account can have **many Quotes**
- One Account can have **many SalesOrders**
- One Account can have **many Invoices**

---

### CORE ENTITY: CONTACT

#### Purpose
The Contact table stores information about individual people - decision makers, buyers, and stakeholders at our customer companies.

#### Important Fields

| Field | Data Type | Description |
|-------|-----------|-------------|
| ContactId | Unique ID | Primary key |
| FirstName | Text (50 chars) | First name |
| LastName | Text (50 chars) | Last name - **REQUIRED FIELD** |
| FullName | Text (160 chars) | Computed full name (First + Last) |
| EmailAddress1 | Text | Primary email |
| Telephone1 | Text | Primary phone |
| MobilePhone | Text | Mobile number |
| JobTitle | Text | Position/role at company |
| Department | Text | Department they work in |

#### Primary Key
```
ContactId (UniqueIdentifier) - Auto-generated unique ID
```

#### Foreign Keys

| Field | References | Meaning |
|-------|------------|---------|
| OwnerId | SystemUser | Who owns this contact |
| OwningBusinessUnit | BusinessUnit | Department ownership |
| ParentCustomerId | Account OR Contact | Which company they belong to |
| ParentCustomerIdType | Number | 1 = Account, 2 = Contact (polymorphic) |

#### Status Tracking
- **StateCode**: 0 = Active, 1 = Inactive
- **StatusCode**: 1 = Active, 2 = Inactive

#### Audit Fields
- CreatedOn, CreatedBy, ModifiedOn, ModifiedBy, VersionNumber

#### How It Connects
- One Contact belongs to **one Account** (or another Contact)
- One Contact can have **many Opportunities**
- One Contact can have **many Quotes**
- One Contact can have **many Orders**
- One Contact can have **many Invoices**

---

### CORE ENTITY: LEAD

#### Purpose
A Lead is a potential customer who has shown interest but hasn't been qualified yet. It's the starting point of our sales process.

#### Important Fields

| Field | Data Type | Description |
|-------|-----------|-------------|
| LeadId | Unique ID | Primary key |
| Subject | Text (200 chars) | Brief description - **REQUIRED FIELD** |
| FirstName | Text | Lead's first name |
| LastName | Text | Lead's last name |
| CompanyName | Text | Company they work for |
| EmailAddress | Text | Email address |
| Telephone | Text | Phone number |
| IndustryCode | Number | Industry they work in |
| LeadSourceCode | Number | Where did this lead come from? (Campaign, Referral, Website, etc.) |
| RatingCode | Number | How promising is this lead? (Hot, Warm, Cold) |
| BudgetAmount | Money | Their budget for this purchase |
| EstimatedAmount | Money | Our estimated value of this deal |
| PurchaseTimeframe | Number | When are they planning to buy? |

#### Primary Key
```
LeadId (UniqueIdentifier) - Auto-generated unique ID
```

#### Foreign Keys

| Field | References | Meaning |
|-------|------------|---------|
| OwnerId | SystemUser | Who is working this lead |
| OwningBusinessUnit | Department |
| CampaignId | Campaign | Which marketing campaign generated this lead |
| ParentAccountId | Account | If converted to existing account |
| ParentContactId | Contact | If converted to existing contact |
| QualifyingOpportunityId | Opportunity | If an opportunity was created |

#### Status Tracking

**StateCode: Open (0)**
| StatusCode | Meaning |
|------------|---------|
| 1 | New - Just created |
| 2 | Attempted to Contact - Tried reaching them |
| 3 | Contacted - Had initial conversation |
| 4 | Open - Still working on it |

**StateCode: Qualified (1)**
| StatusCode | Meaning |
|------------|---------|
| 5 | Qualified - Ready to become opportunity |

**StateCode: Disqualified (2)**
| StatusCode | Meaning |
|------------|---------|
| 7 | Lost - Not interested anymore |
| 8 | Cannot Contact - Can't reach them |
| 9 | Not Interested - Said no |

#### Audit Fields
- CreatedOn, CreatedBy, ModifiedOn, ModifiedBy, VersionNumber

#### How It Connects
- One Lead can convert to **one Account**
- One Lead can convert to **one Contact**
- One Lead can create **one Opportunity**

---

### CORE ENTITY: OPPORTUNITY

#### Purpose
An Opportunity represents a qualified sales deal we're actively pursuing. This is where we track the progress of a sale, estimated value, and probability of winning.

#### Important Fields

| Field | Data Type | Description |
|-------|-----------|-------------|
| OpportunityId | Unique ID | Primary key |
| Name | Text (200 chars) | Name for this opportunity - **REQUIRED FIELD** |
| EstimatedValue | Money | How much we think we'll sell (projected revenue) |
| ActualValue | Money | Actual revenue when closed |
| CloseProbability | Number (0-100) | Chance of winning (%) |
| EstimatedCloseDate | Date | When we expect to close |
| ActualCloseDate | Date | When we actually closed |
| SalesStageCode | Number | Which stage in the sales process |
| StepName | Text | Current step name |

#### Primary Key
```
OpportunityId (UniqueIdentifier) - Auto-generated unique ID
```

#### Foreign Keys

| Field | References | Meaning |
|-------|------------|---------|
| OwnerId | SystemUser | Sales rep responsible |
| OwningBusinessUnit | Department |
| CustomerId | Account OR Contact | Who we're selling to |
| CustomerIdType | Number | 1 = Account, 2 = Contact |
| PriceLevelId | PriceList | Pricing to use |
| TransactionCurrencyId | Currency | Currency for the deal |
| OriginatingLeadId | Lead | If came from a lead conversion |

#### Financial Fields (Auto-Calculated)
- **TotalAmount**: Total including tax and freight
- **TotalLineItemAmount**: Sum of all products
- **TotalDiscountAmount**: Total discount applied
- **FreightAmount**: Shipping costs
- **TotalTax**: Tax amount

#### Status Tracking

**StateCode: Open (0)** - Still working on it
| StatusCode | Meaning |
|------------|---------|
| 1 | New |
| 2 | In Progress - Actively being worked |
| 3 | On Hold - Temporarily paused |

**StateCode: Won (1)** - We got the sale!
| StatusCode | Meaning |
|------------|---------|
| 4 | Won |

**StateCode: Lost (2)** - We lost the sale
| StatusCode | Meaning |
|------------|---------|
| 5 | Lost |
| 6 | Cancelled |
| 7 | Out-Sold - Competitor won |

#### Audit Fields
- CreatedOn, CreatedBy, ModifiedOn, ModifiedBy, VersionNumber, TraversedPath

#### How It Connects
- One Opportunity belongs to **one Customer** (Account or Contact)
- One Opportunity can have **many OpportunityProducts** (line items)
- One Opportunity can create **many Quotes**
- One Opportunity can create **one SalesOrder**
- One Opportunity can create **one Invoice**

---

### CORE ENTITY: QUOTE

#### Purpose
A Quote is a formal price proposal we send to the customer. It includes products, quantities, pricing, discounts, and validity dates.

#### Important Fields

| Field | Data Type | Description |
|-------|-----------|-------------|
| QuoteId | Unique ID | Primary key |
| QuoteNumber | Text (20 chars) | Auto-generated quote number |
| RevisionNumber | Number | Version number (increments on revision) |
| Name | Text (200 chars) | Quote name - **REQUIRED FIELD** |
| TotalAmount | Money | Total value of quote |
| DiscountAmount | Money | Order-level discount |
| DiscountPercentage | Percent | Discount percentage |
| FreightAmount | Money | Shipping cost |
| TotalTax | Money | Tax amount |
| EffectiveFrom | Date | Quote start date |
| EffectiveTo | Date | Quote end date |
| ExpiresOn | Date | When quote expires |

#### Primary Key
```
QuoteId (UniqueIdentifier) - Auto-generated unique ID
```

#### Foreign Keys

| Field | References | Meaning |
|-------|------------|---------|
| OwnerId | SystemUser | Who created/owns the quote |
| CustomerId | Account OR Contact | Customer we're quoting |
| CustomerIdType | Number | 1 = Account, 2 = Contact |
| PriceLevelId | PriceList | Price list used |
| TransactionCurrencyId | Currency |
| OpportunityId | Opportunity | Which opportunity this is for |

#### Status Tracking

**StateCode: Open (0)**
| StatusCode | Meaning |
|------------|---------|
| 1 | In Progress - Being prepared |
| 2 | In Progress |

**StateCode: Won (1)** - Customer accepted!
| StatusCode | Meaning |
|------------|---------|
| 3 | Won |
| 4 | Accepted |
| 5 | Revised - Has been updated |

**StateCode: Lost (2)**
| StatusCode | Meaning |
|------------|---------|
| 6 | Lost |
| 7 | Declined - Customer said no |
| 8 | Cancelled |

#### Audit Fields
- CreatedOn, CreatedBy, ModifiedOn, ModifiedBy, VersionNumber

#### How It Connects
- One Quote belongs to **one Customer**
- One Quote belongs to **one Opportunity**
- One Quote can have **many QuoteDetails** (line items)
- One Quote can convert to **one SalesOrder**

---

### CORE ENTITY: SALESORDER

#### Purpose
A Sales Order (Order) is a confirmed purchase from the customer. It represents a binding agreement to buy.

#### Important Fields

| Field | Data Type | Description |
|-------|-----------|-------------|
| SalesOrderId | Unique ID | Primary key |
| OrderNumber | Text (20 chars) | Auto-generated order number |
| TotalAmount | Money | Total order value |
| DiscountAmount | Money | Order discount |
| FreightAmount | Money | Shipping cost |
| TotalTax | Money | Tax amount |
| RequestDeliveryBy | Date | When customer wants delivery |
| SubmitStatus | Number | ERP integration status |

#### Primary Key
```
SalesOrderId (UniqueIdentifier) - Auto-generated unique ID
```

#### Foreign Keys

| Field | References | Meaning |
|-------|------------|---------|
| OwnerId | SystemUser |
| CustomerId | Account OR Contact |
| CustomerIdType | Number |
| PriceLevelId | PriceList |
| TransactionCurrencyId | Currency |
| OpportunityId | Opportunity | Source opportunity |
| QuoteId | Quote | Source quote |

#### Status Tracking

**StateCode: Active (0)**
| StatusCode | Meaning |
|------------|---------|
| 1 | New |
| 2 | Pending - Awaiting processing |
| 3 | Submitted - Sent to fulfillment |
| 4 | Cancelled |

**StateCode: Fulfilled (1)**
| StatusCode | Meaning |
|------------|---------|
| 5 | Fulfilled - Order shipped/completed |

**StateCode: Invoiced (2)**
| StatusCode | Meaning |
|------------|---------|
| 6 | Invoiced - Invoice created |

**StateCode: Paid (3)**
| StatusCode | Meaning |
|------------|---------|
| 7 | Paid - Payment received |

**StateCode: Cancelled (4)**
| StatusCode | Meaning |
|------------|---------|
| 8 | Cancelled |

#### Audit Fields
- CreatedOn, CreatedBy, ModifiedOn, ModifiedBy, VersionNumber

#### How It Connects
- One Order belongs to **one Customer**
- One Order comes from **one Quote** (if quote was accepted)
- One Order can have **many OrderDetails** (line items)
- One Order can create **one Invoice**

---

### CORE ENTITY: INVOICE

#### Purpose
An Invoice is a formal bill sent to the customer requesting payment. It's the final step where we ask for money.

#### Important Fields

| Field | Data Type | Description |
|-------|-----------|-------------|
| InvoiceId | Unique ID | Primary key |
| InvoiceNumber | Text (20 chars) | Auto-generated invoice number |
| TotalAmount | Money | Total amount due |
| DiscountAmount | Money | Invoice discount |
| FreightAmount | Money | Shipping cost (if not paid by customer) |
| TotalTax | Money | Tax amount |
| DueDate | Date | When payment is due |
| PaymentTermsCode | Number | Payment terms (Net 30, Net 60, etc.) |
| DateSent | Date | When invoice was sent |
| PaidOn | Date | When payment was received |

#### Primary Key
```
InvoiceId (UniqueIdentifier) - Auto-generated unique ID
```

#### Foreign Keys

| Field | References | Meaning |
|-------|------------|---------|
| OwnerId | SystemUser |
| CustomerId | Account OR Contact |
| CustomerIdType | Number |
| PriceLevelId | PriceList |
| TransactionCurrencyId | Currency |
| SalesOrderId | SalesOrder | Source order |
| OpportunityId | Opportunity | Original opportunity |

#### Status Tracking

**StateCode: Active (0)** - Awaiting payment
| StatusCode | Meaning |
|------------|---------|
| 1 | New |
| 2 | Partial - Partially paid |
| 3 | Complete - Ready for final payment |

**StateCode: Paid (1)** - Payment received!
| StatusCode | Meaning |
|------------|---------|
| 4 | Paid |
| 5 | Partial - Some payment received |

**StateCode: Cancelled (2)**
| StatusCode | Meaning |
|------------|---------|
| 6 | Cancelled |

#### Audit Fields
- CreatedOn, CreatedBy, ModifiedOn, ModifiedBy, VersionNumber

#### How It Connects
- One Invoice comes from **one SalesOrder**
- One Invoice can have **many InvoiceDetails** (line items)
- One Invoice traces back to the original **Opportunity**

---

### SUPPORTING ENTITY: PRODUCT

#### Purpose
The Product table contains all products and services we sell. It's our product catalog.

#### Important Fields

| Field | Data Type | Description |
|-------|-----------|-------------|
| ProductId | Unique ID | Primary key |
| Name | Text (200 chars) | Product name - **REQUIRED** |
| ProductNumber | Text (100 chars) | SKU or part number |
| ProductTypeCode | Number | Is it a Product or Service? |
| Price | Money | Default selling price |
| Cost | Money | Cost to us |
| StandardCost | Money | Standard cost for calculations |
| QuantityOnHand | Number | Inventory quantity |
| Description | Text | Product details |

#### Primary Key
```
ProductId (UniqueIdentifier)
```

#### Foreign Keys

| Field | References | Meaning |
|-------|------------|---------|
| DefaultUoMId | UnitOfMeasure | Default selling unit |
| PriceLevelId | PriceList | Default price list |

#### Status Tracking
- **StateCode**: 0 = Active, 1 = Inactive

#### How It Connects
- One Product appears in **many OpportunityProducts**
- One Product appears in **many QuoteDetails**
- One Product appears in **many OrderDetails**
- One Product appears in **many InvoiceDetails**

---

### SUPPORTING ENTITY: PRICELIST (PriceLevel)

#### Purpose
A PriceList defines the pricing for products. Different customers can have different price lists (e.g., premium customers get better prices).

#### Important Fields

| Field | Data Type | Description |
|-------|-----------|-------------|
| PriceLevelId | Unique ID | Primary key |
| Name | Text (100 chars) | Price list name - **REQUIRED** |
| BeginDate | Date | Start date |
| EndDate | Date | End date |
| IsDefault | Yes/No | Is this the default price list? |

#### Primary Key
```
PriceLevelId (UniqueIdentifier)
```

#### Foreign Keys

| Field | References | Meaning |
|-------|------------|---------|
| TransactionCurrencyId | Currency | Currency for this price list |

---

### SUPPORTING ENTITY: TRANSACTIONCURRENCY

#### Purpose
Stores currency information for multi-currency support. Allows us to sell in different currencies.

#### Important Fields

| Field | Data Type | Description |
|-------|-----------|-------------|
| TransactionCurrencyId | Unique ID | Primary key |
| ISOCurrencyCode | Text (3 chars) | ISO code (USD, EUR, GBP) |
| CurrencySymbol | Text | Symbol ($, €, £) |
| CurrencyName | Text | Full name |
| Precision | Number | Decimal places (usually 2) |
| ExchangeRate | Number | Rate to base currency |

---

### LINE ITEM ENTITIES

These tables store individual products within each transaction:

#### OpportunityProduct
- Links products to an Opportunity
- Contains: Product, Quantity, UnitPrice, Discount, Tax

#### QuoteDetail
- Links products to a Quote
- Contains: Product, Quantity, UnitPrice, Discount, Tax

#### OrderDetail
- Links products to a Sales Order
- Contains: Product, Quantity, UnitPrice, Discount, Tax, Shipping info

#### InvoiceDetail
- Links products to an Invoice
- Contains: Product, Quantity, UnitPrice, Discount, Tax

**Key Point**: Each Line Item table has a Foreign Key to its parent (OpportunityId, QuoteId, SalesOrderId, InvoiceId) - this is how we know which document the product belongs to.

---

### SECURITY ENTITIES

#### SystemUser
Represents people who use the CRM system.

| Field | Description |
|-------|-------------|
| SystemUserId | Primary key |
| FullName | User's display name |
| DomainName | Windows username |
| BusinessUnitId | Which department |
| IsDisabled | Is user active? |

#### Role
Defines permissions (what users can do).

| Field | Description |
|-------|-------------|
| RoleId | Primary key |
| Name | Role name (Salesperson, Manager, Admin) |
| BusinessUnitId | Which department |

#### BusinessUnit
Organizes users into departments/teams. This is the main security boundary.

| Field | Description |
|-------|-------------|
| BusinessUnitId | Primary key |
| Name | Department name |
| ParentBusinessUnitId | Parent department (hierarchy) |

---

## 4. Relationship Explanation

### One-to-Many Relationships

One record in a parent table can have multiple related records in a child table.

**Examples:**

```
Account (1) ──────────────→ Many Contacts
    └── One company can have many people working there

Account (1) ──────────────→ Many Opportunities
    └── One company can have multiple sales deals

Opportunity (1) ──────────→ Many OpportunityProducts
    └── One sales deal can include many products

Quote (1) ────────────────→ Many QuoteDetails
    └── One quote can list many products

SalesOrder (1) ───────────→ Many OrderDetails
    └── One order can contain many products
```

### Parent-Child Structure

The system uses a hierarchical structure:

```
Account
    └── Contact (belongs to Account)
    └── Opportunity (for Account)
    └── Quote (for Account)
    └── Order (for Account)
    └── Invoice (for Account)

Lead
    └── Opportunity (created from Lead)

Opportunity
    └── Quote (created from Opportunity)
    └── Order (created from Opportunity)
    └── Invoice (created from Opportunity)
    └── OpportunityProducts (items in Opportunity)

Quote
    └── Order (created when Quote is accepted)
    └── QuoteDetails (items in Quote)

Order
    └── Invoice (created from Order)
    └── OrderDetails (items in Order)

Invoice
    └── InvoiceDetails (items in Invoice)
```

### How Foreign Keys Maintain Data Integrity

Foreign Keys ensure data stays connected and valid:

1. **Referencing Existing Records**: You can't create a Contact for an Account that doesn't exist
2. **Preventing Orphaned Records**: Can't delete an Account if it has Contacts (unless configured to cascade)
3. **Maintaining Relationships**: When you delete a parent, you can choose what happens to children (delete them too, or just remove the link)

**Example:**
```
Contact.ParentAccountId → Account.AccountId
    This "link" tells us which company this person works for
```

### How Line Items Connect to Parent Documents

Line items use a simple but powerful pattern:

```
OpportunityProduct table has:
    - OpportunityProductId (unique ID)
    - OpportunityId (Foreign Key to parent Opportunity)
    - ProductId (Foreign Key to Product)
    - Quantity
    - UnitPrice
    
The OpportunityId is the "glue" that connects
each line item to its parent opportunity.
```

This pattern is repeated for:
- QuoteDetail → Quote
- OrderDetail → SalesOrder
- InvoiceDetail → Invoice

---

## 5. Data Flow Explanation (Step-by-Step)

This explains how a sale moves through our system from start to finish.

---

### STEP 1: LEAD CREATION

**What Happens:**
A new potential customer enters the system as a Lead.

**What Gets Created:**
- New **Lead** record

**Data Captured:**
- Name, company, phone, email
- Source of lead (campaign, referral, website)
- Initial interest level
- Budget information (if known)

**Status:**
- StateCode = 0 (Open)
- StatusCode = 1 (New)

**How It Connects:**
- Lead can reference a Marketing Campaign (how we found them)
- Lead can reference the Account/Contact they're associated with

---

### STEP 2: LEAD QUALIFICATION

**What Happens:**
Sales rep evaluates the lead to see if they're a serious prospect.

**Data Reviewed/Updated:**
- Rating (Hot/Warm/Cold)
- Budget amount
- Purchase timeframe
- Decision maker status
- Company size fit

**Decision Made:**

**QUALIFY** (Lead → Opportunity)
- Lead.StateCode changes to 1 (Qualified)
- Creates new **Opportunity**
- Links Opportunity to Lead (OriginatingLeadId)
- Links Opportunity to Account/Contact

**DISQUALIFY** (Lead → Closed)
- Lead.StateCode changes to 2 (Disqualified)
- Records reason (Lost, Cannot Contact, Not Interested)
- No further action

---

### STEP 3: OPPORTUNITY MANAGEMENT

**What Happens:**
We actively work the qualified opportunity.

**Key Activities:**
1. Add products the customer is interested in (OpportunityProducts)
2. Update estimated value based on products
3. Set probability of winning
4. Track expected close date
5. Move through sales stages

**Sales Stages:**
```
New → In Progress → [Decision Point]
                ↓
          Won or Lost
```

**Data Flow:**
- **Enter Products**: Add records to **OpportunityProduct** table
- **Calculate Total**: System sums up (Quantity × UnitPrice) - Discounts + Tax
- **Update EstimatedValue**: Automatically calculated from line items
- **Track Progress**: SalesStageCode changes as we advance

**Status Tracking:**
- StateCode = 0 (Open) while working
- StateCode = 1 (Won) if customer agrees to buy
- StateCode = 2 (Lost) if sale is lost

---

### STEP 4: QUOTE CREATION

**What Happens:**
We prepare a formal price proposal for the customer.

**What Gets Created:**
- New **Quote** record

**Data Copied from Opportunity:**
- Customer (Account/Contact)
- Price List
- Currency
- All OpportunityProducts → QuoteDetails (line items)
- Pricing, discounts, totals

**Quote Validation:**
- RevisionNumber starts at 0
- EffectiveFrom = today
- EffectiveTo = expiration date
- ExpiresOn = last valid date

**What Sales Rep Does:**
- Add/modify products
- Apply discounts (line and order level)
- Set pricing
- Define payment terms
- Set shipping method
- Send to customer

**Status:**
- StateCode = 0 (Open) - Being prepared
- StateCode = 1 (Won) - Customer accepted
- StateCode = 2 (Lost) - Customer declined

---

### STEP 5: QUOTE ACCEPTANCE → SALES ORDER

**What Happens:**
Customer accepts the quote and agrees to buy.

**What Gets Created:**
- New **SalesOrder** record

**Data Copied from Quote:**
- Customer information
- Price List, Currency
- All QuoteDetails → OrderDetails
- Pricing, discounts, totals
- Shipping address
- Payment terms

**What Changes:**
- Quote.StateCode → 1 (Won)
- Quote.StatusCode → 3 (Won) or 4 (Accepted)
- New Order gets StateCode = 0 (Active)

**Order Status Progression:**
```
New → Pending → Submitted → Fulfilled → Invoiced → Paid
```

---

### STEP 6: ORDER FULFILLMENT

**What Happens:**
We process and fulfill the order (ship products or deliver services).

**What Changes:**
- Order.StatusCode → 3 (Submitted)
- Track shipping information
- Update inventory (if integrated with inventory system)

**When Complete:**
- Order.StateCode → 1 (Fulfilled)
- StatusCode → 5 (Fulfilled)

---

### STEP 7: INVOICE CREATION

**What Happens:**
We create a bill to request payment.

**What Gets Created:**
- New **Invoice** record

**Data Copied from Order:**
- Customer information
- All OrderDetails → InvoiceDetails
- Pricing, discounts, totals
- Payment terms
- Due date calculation

**Invoice Status:**
- StateCode = 0 (Active) - Awaiting payment
- StateCode = 1 (Paid) - Payment received
- StateCode = 2 (Cancelled)

---

### STEP 8: PAYMENT & CLOSE

**What Happens:**
Customer pays the invoice.

**What Changes:**
1. **Invoice gets marked Paid**
   - Invoice.StateCode → 1 (Paid)
   - Invoice.PaidOn = today's date

2. **Opportunity gets marked Won**
   - Opportunity.StateCode → 1 (Won)
   - Opportunity.StatusCode → 4 (Won)
   - Opportunity.ActualValue = final revenue
   - Opportunity.ActualCloseDate = close date

**Complete Flow Summary:**

```
Lead (Open)
    ↓ [Qualify]
Lead (Qualified) + Opportunity (Open)
    ↓ [Add Products]
Opportunity with Products
    ↓ [Create Quote]
Quote (Open)
    ↓ [Customer Accepts]
Quote (Won) + Sales Order (Active)
    ↓ [Fulfill]
Sales Order (Fulfilled)
    ↓ [Create Invoice]
Invoice (Active)
    ↓ [Payment Received]
Invoice (Paid) + Opportunity (Won)
    ✓ SALE COMPLETE!
```

---

## 6. Validation & Data Integrity

### Required Fields

Each entity has required fields that MUST be filled to save a record:

| Entity | Required Fields |
|--------|----------------|
| Account | Name, OwnerId |
| Contact | LastName, OwnerId |
| Lead | Subject, OwnerId |
| Opportunity | Name, CustomerId, OwnerId |
| Quote | CustomerId, OwnerId |
| SalesOrder | CustomerId, OwnerId |
| Invoice | CustomerId, OwnerId |
| Product | Name |

### Database Constraints

**Primary Key Constraint:**
- Every table has a uniqueidentifier as Primary Key
- Auto-generated using NEWSEQUENTIALID()
- Guarantees each record is unique

**Foreign Key Constraints:**
- Ensure relationships are valid
- Example: Can't create a Contact with a non-existent AccountId

**Cascade Behaviors:**

| Behavior | What Happens |
|----------|--------------|
| Cascade | Delete children when parent is deleted |
| Restricted | Prevent deleting parent if children exist |
| RemoveLink | Just remove the link, don't delete child |

### Financial Validation

**Money Fields:**
- Use "money" data type (4 decimal precision)
- Always store positive values (>= 0)
- Can store negative for credits/adjustments

**Multi-Currency:**
- Each transaction stores both:
  - TransactionCurrencyId → Original currency amount
  - Amount_Base → Converted to base currency
- ExchangeRate captured at transaction time

**Calculations (Automatic):**

```
Line Amount = Quantity × Unit Price
Line Discount = Line Amount × Discount %
Net Line = Line Amount - Line Discount

Subtotal = Sum of all Net Lines
Order Discount = Subtotal × Discount %
Net Subtotal = Subtotal - Order Discount

Tax = Net Subtotal × Tax Rate
Freight = Shipping cost

Total Amount = Net Subtotal + Tax + Freight
```

### Status Management

**StateCode vs StatusCode:**

- **StateCode**: The main bucket (Active/Inactive, Open/Won/Lost)
- **StatusCode**: The specific reason within that state

This two-level system allows for:
1. Quick filtering (show all Active records)
2. Detailed tracking (show why it's Active)

### Audit Tracking

Every record automatically tracks:

| Field | What It Shows |
|-------|---------------|
| **CreatedOn** | Exact timestamp of record creation |
| **CreatedBy** | User who created it |
| **ModifiedOn** | Last time record was changed |
| **ModifiedBy** | User who last changed it |
| **VersionNumber** | Counter that increments on each change |

**Why This Matters:**
- Track who did what and when
- Report on customer activity
- Investigate issues
- Comply with regulations

### Concurrency Handling

**Problem:** Two users edit the same record at the same time. Who wins?

**Solution: Optimistic Concurrency**

1. When user A opens a record, they get the current VersionNumber
2. When user A saves, system checks if VersionNumber still matches
3. If someone else changed it (VersionNumber is different), warn user A
4. User A must refresh and re-apply their changes

**This prevents:**
- Lost updates
- Data corruption
- Accidental overwrites

---

## 7. Simple Meeting Summary

*(3-5 minutes explanation)*

---

> "The CRM Sales Module is our system for managing the complete sales process from first contact to getting paid.
>
> It has seven main areas. First, we have **Accounts** (companies we sell to) and **Contacts** (people at those companies). Then we track potential customers as **Leads**. When a lead looks promising, we qualify them and create an **Opportunity** - that's our active sales deal.
>
> On each opportunity, we add the products the customer wants (that's stored in **OpportunityProducts**). When we're ready to propose a price, we create a **Quote**. Once the customer agrees, that quote becomes a **Sales Order**, and we send them an **Invoice** to collect payment.
>
> Everything connects through foreign keys - for example, an Invoice knows which Order it came from, which Order came from which Quote, which Quote came from which Opportunity, and which customer we're selling to.
>
> Every record tracks who owns it, when it was created, and when it was last modified. We also track the status at every stage so management can see where each deal stands.
>
> In simple terms: This system makes sure we don't miss any customer and every sale is properly tracked from start to finish."

---

**End of Document**
