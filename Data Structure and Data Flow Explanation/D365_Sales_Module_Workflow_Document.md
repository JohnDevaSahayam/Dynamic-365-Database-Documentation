# Microsoft Dynamics 365 – Sales Module Complete Workflow

## Step-by-Step Guide for Beginners

---

## 1. WHAT IS SALES WORKFLOW?

### Understanding Sales Process Simply

A **Sales Workflow** is simply the step-by-step journey a customer takes from first knowing about your product to finally buying it.

**In Simple Terms:**

Think of selling a car. The process goes like this:

1. You meet someone interested in cars (Lead)
2. You learn what they need (Qualify)
3. You show them cars they're interested in (Opportunity)
4. You give them a price quote (Quote)
5. They agree to buy (Order)
6. You deliver the car and send invoice (Invoice)
7. They pay you (Payment)

That's exactly what Dynamics 365 Sales Module does - it tracks this entire journey automatically!

### Why Is This Important?

**Without a Workflow:**
- Sales reps lose track of customers
- Nobody knows which deals are close to winning
- Quotes get lost or forgotten
- Customers fall through the cracks

**With Dynamics 365 Workflow:**
- Every step is recorded
- Everyone sees the same information
- Follow-ups never missed
- Managers can help where needed
- Reports show exactly how sales are going

---

## 2. COMPLETE SALES PROCESS FLOW (STEP-BY-STEP)

This section explains each step of the sales process in detail. We'll cover what happens in business terms and what happens in the database.

---

### STEP 1: ACCOUNT CREATION

**What Happens in Business:**

Before you can sell anything, you need to know WHO you're selling to. The first step is creating a company record.

**Example:**
Your company wants to sell software to "ABC Corporation." You create ABC Corporation as an Account in Dynamics 365.

**Business Purpose:**

- Store company information (name, address, phone, website)
- Create a central place for all company data
- Enable multiple contacts to link to one company
- Track all interactions with that company

**Entity Used:** Account Table

**What Data is Stored:**

| Field | Example Value |
|-------|---------------|
| Company Name | ABC Corporation |
| Account Number | CUST-001 |
| Phone | 555-123-4567 |
| Website | www.abccorp.com |
| Address | 123 Business Ave, New York |
| Industry | Technology |
| Owner | John Smith (Sales Rep) |

**Technical Note:**
A new record is created in the **AccountBase** table with a unique **AccountId**.

---

### STEP 2: CONTACT CREATION

**What Happens in Business:**

Now you add the specific people you work with at that company. A company has many employees, but you need to know WHO to talk to.

**Example:**
At ABC Corporation, you add:
- John Doe (CEO)
- Jane Smith (Purchasing Manager)
- Mike Johnson (IT Director)

**Business Purpose:**

- Store individual person details
- Track who does what at each company
- Know who to contact for different issues
- Personalize communication

**Entity Used:** Contact Table

**What Data is Stored:**

| Field | Example Value |
|-------|---------------|
| First Name | Jane |
| Last Name | Smith |
| Email | jane@abccorp.com |
| Phone | 555-987-6543 |
| Job Title | Purchasing Manager |
| Company | ABC Corporation |

**Technical Note:**
Contacts are linked to Accounts through the **ParentCustomerId** field. One Account can have many Contacts.

---

### STEP 3: LEAD CREATION

**What Happens in Business:**

A Lead is someone who has shown INTEREST in your product but hasn't been verified yet. They might have:

- Filled out a form on your website
- Downloaded a whitepaper
- Met you at a trade show
- Been recommended by someone

**Example:**
Someone fills out a "Contact Us" form on your website. Their info automatically becomes a Lead in Dynamics 365.

**Business Purpose:**

- Capture initial interest from potential customers
- Track where leads come from (source)
- Store preliminary information
- Start the sales conversation

**Entity Used:** Lead Table

**What Data is Stored:**

| Field | Example Value |
|-------|---------------|
| Full Name | Robert Williams |
| Company | Williams Tech Solutions |
| Email | robert@williamstech.com |
| Phone | 555-111-2222 |
| Lead Source | Website Form |
| Interest | Enterprise Software |
| Status | New |

**Technical Note:**
Lead is stored in **LeadBase** table with **LeadId**. Initially, it's not connected to an Account or Contact yet.

---

### STEP 4: LEAD QUALIFICATION

**What Happens in Business:**

Now you determine if this Lead is worth pursuing. You ask questions like:

- Do they have a real need for our product?
- Do they have budget?
- Can they make buying decisions?
- Is the timing right?

**Example:**
You call Robert Williams. He confirms his company needs software, has $50,000 budget, and wants to buy within 3 months. He's a QUALIFIED lead!

**Business Purpose:**

- Filter out unqualified prospects
- Focus on real opportunities
- Ensure sales team spends time wisely
- Move promising leads forward

**What Changes:**

| Before | After |
|--------|-------|
| Lead Status: New | Lead Status: Qualified |
| Not connected to Account | Can create Account if new |
| Just interest | Potential deal identified |

**Technical Note:**
The Lead's **StatusCode** changes from "New" to "Qualified." When qualified, the Lead can be **converted** to an Opportunity.

---

### STEP 5: OPPORTUNITY CREATION

**What Happens in Business:**

When a Lead is qualified, you create an **Opportunity** - this is a REAL sales deal you're working on. You expect to win this business!

**Example:**
You create "Williams Tech - Q2 2026 Deal" as an Opportunity worth $50,000.

**Business Purpose:**

- Track active sales deals
- Estimate potential revenue
- See your sales pipeline
- Measure sales progress

**Entity Used:** Opportunity Table

**What Data is Stored:**

| Field | Example Value |
|-------|---------------|
| Opportunity Name | Williams Tech - Q2 2026 Deal |
| Customer | Williams Tech Solutions |
| Estimated Value | $50,000 |
| Expected Close Date | June 30, 2026 |
| Sales Stage | Qualify |
| Probability | 10% |
| Status | In Progress |

**Technical Note:**
Opportunity is stored in **OpportunityBase** table with **OpportunityId**. It links to the Account or Contact through **CustomerId**.

---

### STEP 6: OPPORTUNITY DEVELOPMENT

**What Happens in Business:**

This is where the real sales work happens. You:

- Meet with the customer
- Understand their needs
- Demonstrate your product
- Build the business case
- Get internal approvals

**Example:**
You have 3 meetings with Robert Williams. You show product demos, discuss pricing options, and prepare a custom proposal.

**Business Purpose:**

- Move deal forward through sales stages
- Build relationship with customer
- Understand customer requirements
- Increase chance of winning

**Sales Stages (Typical):**

| Stage | What Happens | Probability |
|-------|--------------|-------------|
| Qualify | Verify it's a real opportunity | 10% |
| Develop | Understand needs deeply | 25% |
| Propose | Send quotes and proposals | 50% |
| Close | Final negotiations | 75% |

**Technical Note:**
As the opportunity develops, the **StepName** (sales stage) changes, and **EstimatedValue** may be adjusted based on discussions.

---

### STEP 7: QUOTE CREATION

**What Happens in Business:**

When you're ready to present pricing, you create a **Quote** - a formal document showing exactly what you'll provide and the price.

**Example:**
You create a quote for Williams Tech showing:
- Enterprise Software License: $35,000
- Implementation Services: $10,000
- Annual Support: $5,000
- Total: $50,000

**Business Purpose:**

- Present formal pricing to customer
- Show products and quantities
- Set validity period (when quote expires)
- Start formal decision process

**Entity Used:** Quote Table

**What Data is Stored:**

| Field | Example Value |
|-------|---------------|
| Quote Name | Quote for Williams Tech - March 2026 |
| Opportunity | Williams Tech - Q2 2026 Deal |
| Customer | Williams Tech Solutions |
| Total Amount | $50,000 |
| Valid From | March 1, 2026 |
| Valid To | March 31, 2026 |
| Status | Draft |

**Technical Note:**
Quote is stored in **QuoteBase** table with **QuoteId**. It links to Opportunity through **OpportunityId**.

---

### STEP 8: ORDER CREATION

**What Happens in Business:**

The customer ACCEPTED your quote! They want to buy. You create an **Order** - this is a binding agreement.

**Example:**
Robert Williams calls and says "We want to proceed!" You create Order #1001.

**Business Purpose:**

- Confirm the sale
- Trigger fulfillment process
- Create a binding agreement
- Start delivery/process

**Entity Used:** SalesOrder (Order) Table

**What Data is Stored:**

| Field | Example Value |
|-------|---------------|
| Order Number | ORD-1001 |
| Quote | Quote for Williams Tech |
| Opportunity | Williams Tech - Q2 2026 Deal |
| Customer | Williams Tech Solutions |
| Order Total | $50,000 |
| Order Date | April 1, 2026 |
| Status | Submitted |

**Technical Note:**
Order is stored in **SalesOrderBase** table with **OrderId**. It typically originates from a **Quote** (one Quote → one Order).

---

### STEP 9: INVOICE GENERATION

**What Happens in Business:**

Now you need to get paid! You create an **Invoice** - a formal request for payment.

**Example:**
You create Invoice #INV-001 for Williams Tech with payment terms Net 30 (pay within 30 days).

**Business Purpose:**

- Request payment from customer
- Track money owed
- Start accounts receivable process
- Record completed sale

**Entity Used:** Invoice Table

**What Data is Stored:**

| Field | Example Value |
|-------|---------------|
| Invoice Number | INV-001 |
| Order | ORD-1001 |
| Customer | Williams Tech Solutions |
| Invoice Amount | $50,000 |
| Invoice Date | April 1, 2026 |
| Due Date | May 1, 2026 |
| Status | Pending Payment |

**Technical Note:**
Invoice is stored in **InvoiceBase** table with **InvoiceId**. Each Order typically creates one Invoice.

---

### STEP 10: DEAL CLOSURE (WON / LOST)

**What Happens in Business:**

The final step is closing the deal. There are two outcomes:

**Option A: WON (Success!)**
- Customer paid the invoice
- Deal is officially closed as "Won"
- Relationship moves to ongoing account management

**Example:**
Williams Tech pays the $50,000 invoice on April 15, 2026. You mark the Opportunity as "Won."

**Option B: LOST (Not Successful)**
- Customer chose a competitor
- Customer decided not to buy
- Timing didn't work out

**Example:**
Williams Tech says they're going with a competitor. You mark the Opportunity as "Lost" and note the reason.

**Business Purpose:**

- Close the loop on sales deals
- Record final outcome
- Analyze wins and losses
- Plan improvements

**Technical Note:**
When an Opportunity is Won:
- **StateCode** changes to 1 (Won)
- **StatusCode** changes to 3 (Won)

When an Opportunity is Lost:
- **StateCode** changes to 2 (Lost)
- **StatusCode** changes to 4 (Lost)

---

## 3. DATA FLOW BETWEEN TABLES

This section explains how data moves from one table to another during the sales process.

---

### Complete Data Flow Diagram

```
LEAD
  │
  │ (When Qualified)
  ↓
OPPORTUNITY ──────────────→ ACCOUNT (if new company)
  │                             │
  │                             ↓
  │                        CONTACT (if new person)
  │                             │
  ├─────────────────────────────┘
  │
  │ (When Ready to Quote)
  ↓
QUOTE
  │
  │ (When Customer Accepts)
  ↓
ORDER
  │
  │ (When Order Confirmed)
  ↓
INVOICE
  │
  │ (When Paid)
  ↓
CLOSED (Won)
```

---

### Specific Data Flow Scenarios

---

#### SCENARIO 1: Lead Qualification

**What Happens:**
```
Lead Record (Status: Qualified)
         ↓
    System creates
         ↓
Opportunity Record (New!)
```

**Technical Flow:**
1. User clicks "Qualify Lead" button
2. Dynamics 365 creates new Opportunity
3. Copies relevant data from Lead to Opportunity
4. Links Opportunity to Account/Contact
5. Updates Lead status to "Qualified"
6. Lead can no longer be modified (becomes read-only)

**Database Changes:**
- New row in **OpportunityBase** table
- **OpportunityId** created
- **CustomerId** linked to Account or Contact
- **LeadBase** record status updated

---

#### SCENARIO 2: Quote to Order Conversion

**What Happens:**
```
Quote (Status: Won/Accepted)
         ↓
    System creates
         ↓
Order Record (New!)
```

**Technical Flow:**
1. User clicks "Create Order" from Quote
2. Dynamics 365 creates new Order
3. Copies all quote line items to order line items
4. Links Order to original Quote
5. Links Order to original Opportunity
6. Updates Quote status to "Won"

**Database Changes:**
- New row in **SalesOrderBase** table
- **OrderId** created
- **QuoteId** linked to original quote
- **OpportunityId** linked to source opportunity
- **QuoteBase** record status updated

---

#### SCENARIO 3: Order to Invoice Generation

**What Happens:**
```
Order (Status: Fulfilled/Complete)
         ↓
    System creates
         ↓
Invoice Record (New!)
```

**Technical Flow:**
1. User clicks "Create Invoice" from Order
2. Dynamics 365 creates new Invoice
3. Copies all order line items to invoice line items
4. Links Invoice to original Order
5. Updates Order status to "Invoiced"

**Database Changes:**
- New row in **InvoiceBase** table
- **InvoiceId** created
- **SalesOrderId** linked to original order
- **InvoiceBase** status set to "Pending Payment"

---

#### SCENARIO 4: Opportunity Won Closure

**What Happens:**
```
Opportunity (Status: Won)
         ↓
    System updates status
         ↓
Deal is Closed!
```

**Technical Flow:**
1. User clicks "Close as Won" button
2. System records close date
3. Updates status to "Won"
4. Optional: Creates Order automatically
5. Probability changes to 100%

**Database Changes:**
- **StateCode** changes from 0 to 1
- **StatusCode** changes to Won value
- **ActualCloseDate** set to current date
- **StepName** changes to "Closed"

---

### Parent-Child Relationship Summary

| Parent Entity | Child Entity | Relationship |
|---------------|---------------|--------------|
| Account | Contact | One Account → Many Contacts |
| Account | Opportunity | One Account → Many Opportunities |
| Lead | Opportunity | One Lead → One Opportunity (on qualify) |
| Opportunity | Quote | One Opportunity → Many Quotes |
| Quote | Order | One Quote → One Order |
| Order | Invoice | One Order → One Invoice |

---

## 4. STATUS & LIFECYCLE FLOW

Every record in Dynamics 365 goes through a **lifecycle** - it starts in one state and moves through various stages. This section explains what each status means.

---

### LEAD LIFECYCLE

**Purpose:** Track potential customers from first interest to becoming a qualified opportunity.

```
┌─────────────────────────────────────────────────────────┐
│                    LEAD LIFECYCLE                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌──────────┐     ┌──────────────┐     ┌───────────┐ │
│   │   NEW    │────→│  QUALIFIED   │────→│ WON (Won)  │ │
│   │          │     │              │     │  (Becomes  │ │
│   └──────────┘     └──────────────┘     │ Opportunity)│
│        ↑                                     └───────────┘
│        │                                           │
│        │     ┌──────────────┐                       │
│        └─────│DISQUALIFIED  │←──────────────────────┘
│              │              │
│              └──────────────┘
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Status Explanations:**

| Status | What It Means | What To Do |
|--------|--------------|------------|
| **New** | Lead just created, not contacted yet | Reach out to understand needs |
| **Contacted** | Initial contact made, learning more | Continue conversations |
| **Qualified** | Lead has potential, moving to Opportunity | Create Opportunity |
| **Disqualified** | Not a good fit right now | Keep for future, note reason |

**Technical Values:**
- **StateCode 0 (Open):** New, Contacted, Qualified
- **StateCode 2 (Canceled):** Disqualified

---

### OPPORTUNITY LIFECYCLE

**Purpose:** Track active sales deals from qualification to closure.

```
┌─────────────────────────────────────────────────────────┐
│                  OPPORTUNITY LIFECYCLE                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌──────────┐     ┌──────────┐     ┌──────────┐      │
│   │ QUALIFY  │────→│ DEVELOP  │────→│  PROPOSE  │      │
│   │ (10%)    │     │  (25%)    │     │   (50%)   │      │
│   └──────────┘     └──────────┘     └──────────┘      │
│                                            │             │
│                                            ↓             │
│   ┌──────────┐     ┌──────────┐     ┌──────────┐      │
│   │DISQUALIF │←────│   LOST   │←────│  CLOSE   │      │
│   │ (0%)     │     │  (0%)    │     │  (100%)  │      │
│   └──────────┘     └──────────┘     └──────────┘      │
│                         ↑                              │
│                         └──────────────────────────────┘
│                           (Won!)
└─────────────────────────────────────────────────────────┘
```

**Status Explanations:**

| Status | What It Means | Probability |
|--------|--------------|-------------|
| **Qualify** | Just started, verifying potential | 10% |
| **Develop** | Understanding customer needs | 25% |
| **Propose** | Sent proposal/quote to customer | 50% |
| **Close** | Final negotiations, ready to close | 75% |
| **Won** | Customer said YES! | 100% |
| **Lost** | Customer said no (or no response) | 0% |
| **On Hold** | Paused temporarily | Current % |

**Technical Values:**
- **StateCode 0 (Open):** Qualify, Develop, Propose, Close
- **StateCode 1 (Won):** Won
- **StateCode 2 (Lost):** Lost

---

### QUOTE LIFECYCLE

**Purpose:** Track price proposals from creation to acceptance.

```
┌─────────────────────────────────────────────────────────┐
│                    QUOTE LIFECYCLE                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌──────────┐     ┌──────────┐     ┌──────────┐      │
│   │  DRAFT   │────→│  ACTIVE  │────→│   WON    │      │
│   │          │     │ (Sent to │     │(Accepted)│      │
│   │          │     │ Customer)│     │          │      │
│   └──────────┘     └──────────┘     └──────────┘      │
│        ↑                                     │            │
│        │     ┌──────────┐                   │            │
│        └─────│ CLOSED   │←──────────────────┘            │
│              │(Expired/  │                                  │
│              │Cancelled) │                                  │
│              └──────────┘                                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Status Explanations:**

| Status | What It Means |
|--------|--------------|
| **Draft** | Creating quote, not sent yet |
| **Active** | Quote sent to customer, awaiting response |
| **Won** | Customer accepted quote |
| **Closed** | Quote expired or was cancelled |

**Technical Values:**
- **StateCode 0 (Open):** Draft, Active
- **StateCode 1 (Closed):** Won, Closed

---

### ORDER LIFECYCLE

**Purpose:** Track confirmed purchases from creation to fulfillment.

```
┌─────────────────────────────────────────────────────────┐
│                    ORDER LIFECYCLE                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌────────────┐     ┌────────────┐     ┌──────────┐  │
│   │  SUBMITTED │────→│  FULFILLED │────→│ INVOICED │  │
│   │            │     │ (Delivered)│     │          │  │
│   └────────────┘     └────────────┘     └──────────┘  │
│                                                  │       │
│                                            ┌─────┘       │
│                                            ↓             │
│                                      ┌──────────┐        │
│                                      │  CLOSED  │        │
│                                      │          │        │
│                                      └──────────┘        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Status Explanations:**

| Status | What It Means |
|--------|--------------|
| **Submitted** | Order created, processing begins |
| **Fulfilled** | Products delivered / services done |
| **Invoiced** | Invoice sent to customer |
| **Closed** | Order complete, no further action |

**Technical Values:**
- **StateCode 0 (Active):** Submitted, Fulfilled, Invoiced
- **StateCode 1 (Closed):** Closed

---

### INVOICE LIFECYCLE

**Purpose:** Track payment requests from creation to payment.

```
┌─────────────────────────────────────────────────────────┐
│                   INVOICE LIFECYCLE                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌──────────┐     ┌──────────┐     ┌──────────┐      │
│   │ PENDING  │────→│   PAID   │────→│  CLOSED  │      │
│   │ PAYMENT  │     │          │     │          │      │
│   └──────────┘     └──────────┘     └──────────┘      │
│        ↑                                                   │
│        │     ┌──────────┐                                 │
│        └─────│ CANCELLED│←───────────────────────────────┘
│              │          │                                 │
│              └──────────┘                                 │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Status Explanations:**

| Status | What It Means |
|--------|--------------|
| **Pending Payment** | Invoice sent, waiting for payment |
| **Paid** | Customer paid the invoice |
| **Closed** | Invoice processed, complete |
| **Cancelled** | Invoice cancelled |

**Technical Values:**
- **StateCode 0 (Active):** Pending Payment, Paid
- **StateCode 1 (Closed):** Closed
- **StateCode 2 (Canceled):** Cancelled

---

## 5. COMPLETE WORKFLOW SUMMARY FOR TEAM LEAD

### 12-Line Executive Summary

---

**The Dynamics 365 Sales Module** tracks every customer interaction from first interest to final payment.

**The Sales Process Has 10 Main Steps:** (1) Create Account (company), (2) Add Contact (person), (3) Generate Lead (interest), (4) Qualify Lead (verify potential), (5) Create Opportunity (real deal), (6) Develop Opportunity (build relationship), (7) Create Quote (send pricing), (8) Create Order (customer accepts), (9) Generate Invoice (request payment), (10) Close Deal (Won or Lost).

**Data Flows Logically:** Lead converts to Opportunity, Opportunity generates Quote, Quote becomes Order, Order creates Invoice - each step building on the previous one.

**Every Record Has Lifecycle Status:** Leads go from New → Qualified → Won/Lost, Opportunities progress from Qualify → Develop → Propose → Close → Won/Lost, and Quotes move from Draft → Active → Won/Closed.

**The Database Reflects This Flow:** AccountBase stores companies, ContactBase stores people, LeadBase captures interest, OpportunityBase tracks deals, QuoteBase holds pricing, SalesOrderBase records orders, and InvoiceBase manages payments.

**This Systematic Approach Ensures:** No customer falls through the cracks, sales teams know exactly where each deal stands, managers can forecast revenue accurately, and the entire organization sees a single source of truth for customer data.

**The Result:** A repeatable, measurable, and scalable sales process that drives revenue growth.

---

## Quick Reference: Status Code Values

| Entity | StateCode 0 (Open) | StateCode 1 (Won/Complete) | StateCode 2 (Lost/Cancel) |
|--------|-------------------|--------------------------|--------------------------|
| Lead | New, Contacted, Qualified | - | Disqualified |
| Opportunity | In Progress, On Hold | Won | Lost |
| Quote | Draft, Active | Won | Closed |
| Order | Submitted, Fulfilled, Invoiced | - | Closed |
| Invoice | Pending Payment, Paid | Closed | Cancelled |

---

**Document Prepared For:** Team Lead Review
**Framework:** Microsoft Dynamics 365 Sales Module
**Complexity Level:** Beginner-Friendly
**Status:** Meeting Ready

---

*End of Document*