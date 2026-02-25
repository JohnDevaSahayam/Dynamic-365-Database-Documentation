# CRM Sales Module
## Simple Explanation - Data Structure & Data Flow

---

## 1. What is This CRM System?

This CRM system tracks our entire sales process - from finding a potential customer to getting paid.

**In simple words:**
- It stores who our customers are
- It tracks every sale we work on
- It shows where each deal stands
- It calculates money amounts

---

## 2. Data Structure

### Main Entities (The Core Tables)

| Entity | What It Is | Key Purpose |
|--------|------------|-------------|
| **Account** | Company | A business we sell to |
| **Contact** | Person | Someone at a company |
| **Lead** | Potential Customer | Someone interested but not qualified yet |
| **Opportunity** | Active Sale | A deal we're working on |
| **Quote** | Price Proposal | Formal pricing we send to customer |
| **SalesOrder** | Confirmed Order | Customer agreed to buy |
| **Invoice** | Bill | Request for payment |

---

### Supporting Entities

| Entity | Purpose |
|--------|---------|
| **Product** | Things we sell |
| **PriceList** | Pricing rules |
| **OpportunityProduct** | Products in an opportunity |
| **QuoteDetail** | Products in a quote |
| **OrderDetail** | Products in an order |
| **InvoiceDetail** | Products in an invoice |

---

### Security Entities

| Entity | Purpose |
|--------|---------|
| **SystemUser** | People who use the system |
| **Role** | What users can do |
| **BusinessUnit** | Teams/departments |

---

## 3. Entity Details

### ACCOUNT
- **Purpose**: Stores company information
- **Required Field**: Name
- **Key Fields**: Industry, Revenue, CreditLimit, Phone, Email
- **Links to**: Contacts, Opportunities, Quotes, Orders, Invoices

### CONTACT
- **Purpose**: Stores person information
- **Required Field**: LastName
- **Key Fields**: FirstName, Email, Phone, JobTitle
- **Links to**: Account (works at which company)

### LEAD
- **Purpose**: New potential customer
- **Required Field**: Subject
- **Key Fields**: CompanyName, Email, Rating (Hot/Warm/Cold), Budget
- **Status**: Open → Qualified → Disqualified
- **Links to**: Account, Contact, Opportunity (when qualified)

### OPPORTUNITY
- **Purpose**: Active sales deal we're working on
- **Required Field**: Name, Customer
- **Key Fields**: EstimatedValue, CloseProbability, ExpectedCloseDate
- **Status**: Open → Won OR Lost
- **Links to**: Customer (Account/Contact), Products, Quotes, Order, Invoice

### QUOTE
- **Purpose**: Formal price proposal
- **Required Field**: Customer
- **Key Fields**: TotalAmount, Discount, ExpiresOn
- **Status**: Open → Won (accepted) → Lost (declined)
- **Links to**: Opportunity, Products (line items), Order (when won)

### SALES ORDER
- **Purpose**: Confirmed customer purchase
- **Required Field**: Customer
- **Key Fields**: TotalAmount, RequestDeliveryBy, Status
- **Status**: New → Fulfilled → Invoiced → Paid
- **Links to**: Quote, Products (line items), Invoice

### INVOICE
- **Purpose**: Bill to collect payment
- **Required Field**: Customer
- **Key Fields**: TotalAmount, DueDate, PaidOn
- **Status**: Active → Paid
- **Links to**: Order, Products (line items)

---

## 4. How Tables Connect

### One-to-Many Relationships

```
Account (1) ──────→ Many Contacts
Account (1) ──────→ Many Opportunities
Opportunity (1) ──→ Many Products
Quote (1) ────────→ Many Line Items
Order (1) ────────→ Many Line Items
Invoice (1) ──────→ Many Line Items
```

### Foreign Keys (The Connections)

A Foreign Key is a field that links to another table:

| Field | Links To | Example |
|-------|----------|---------|
| Contact.ParentAccountId | Account | Which company this person works for |
| Opportunity.CustomerId | Account/Contact | Who we're selling to |
| Quote.OpportunityId | Opportunity | Which deal this quote is for |
| Order.QuoteId | Quote | Which quote became this order |
| Invoice.OrderId | SalesOrder | Which order this invoice is for |

---

## 5. Data Flow (Step by Step)

### The Sales Process

```
Lead → Opportunity → Quote → Order → Invoice → Paid
```

---

### Step 1: LEAD

**What happens**: New potential customer enters system

**What's created**: Lead record

**Data captured**: Name, Company, Phone, Email, Rating

**Status**: Open (New → Contacted → Qualified)

---

### Step 2: OPPORTUNITY

**What happens**: Lead is qualified as real opportunity

**What's created**: Opportunity record

**Data added**: 
- Link to Customer (Account/Contact)
- Products they want to buy
- Estimated value

**Status**: Open (New → In Progress → Won/Lost)

---

### Step 3: QUOTE

**What happens**: We prepare formal pricing

**What's created**: Quote record

**Data copied**: 
- Customer details
- Products from Opportunity
- Pricing from PriceList
- Discounts

**Status**: Open → Won (accepted) or Lost (declined)

---

### Step 4: SALES ORDER

**What happens**: Customer accepts quote

**What's created**: Sales Order record

**Data copied**: 
- Everything from Quote
- Shipping address
- Payment terms

**Status**: New → Fulfilled → Invoiced → Paid

---

### Step 5: INVOICE

**What happens**: We create bill

**What's created**: Invoice record

**Data copied**: 
- Everything from Order
- Due date
- Payment terms

**Status**: Active → Paid

---

### Step 6: CLOSE (Sale Complete)

**What happens**: Customer pays

**What changes**:
- Invoice marked as Paid
- Opportunity marked as Won
- Actual revenue recorded

---

## 6. Status Tracking

Every record has a status to track progress:

### Opportunity Status
| StateCode | Meaning |
|-----------|---------|
| 0 (Open) | Still working |
| 1 (Won) | Sale completed |
| 2 (Lost) | Sale lost |

### Quote Status
| StateCode | Meaning |
|-----------|---------|
| 0 (Open) | Being prepared |
| 1 (Won) | Customer accepted |
| 2 (Lost) | Customer declined |

### Order Status
| StateCode | Meaning |
|-----------|---------|
| 0 (Active) | New/Pending |
| 1 (Fulfilled) | Shipped |
| 2 (Invoiced) | Invoice created |
| 3 (Paid) | Payment received |

---

## 7. Audit Fields (Tracking Changes)

Every record automatically tracks:

| Field | What It Shows |
|-------|---------------|
| CreatedOn | When record was created |
| CreatedBy | Who created it |
| ModifiedOn | When it was last changed |
| ModifiedBy | Who changed it |
| OwnerId | Who is responsible |

---

## 8. Meeting Summary (Ready to Speak)

> Module tracks our complete sales process "Our CRM Sales.
>
> We start with **Leads** (potential customers), qualify the good ones into **Opportunities** (active deals), create **Quotes** (price proposals), and when the customer accepts, we turn them into **Orders**. Then we send **Invoices** to get paid.
>
> Each table connects to the next through foreign keys - for example, an Invoice knows which Order it came from, which came from a Quote, which came from an Opportunity, which is for a specific Customer.
>
> Every record tracks who owns it, when it was created, and when it was modified. Status fields show where each deal stands in the process.
>
> In simple terms: This system makes sure we track every customer and every sale from start to finish."

---

**End of Document**
