# Microsoft Dynamics 365 – Sales Module Data Structure

## Complete Beginner Guide

---

## 1. INTRODUCTION TO DYNAMICS 365 CRM (BEGINNER EXPLANATION)

### What is Microsoft Dynamics 365?

Microsoft Dynamics 365 is a **software system** that helps companies manage their day-to-day business activities. Think of it as a **digital notebook** that stores everything about your customers, sales, and service interactions.

**In Simple Terms:**
- It's a computer program running on cloud servers
- Multiple users can access it at the same time
- It replaces messy spreadsheets and paper files
- Everything is stored in one central place

### What is the Sales Module?

The **Sales Module** is one part of Dynamics 365 specifically designed to help sales teams. It tracks:

- Companies you sell to
- People you contact
- Sales deals in progress
- Quotes you send to customers
- Orders and invoices
- Products you sell

**Think of it as a sales team's command center.**

### Why Do Companies Use It?

**Benefits for Businesses:**

1. **No More Lost Information**
   - All customer data in one place
   - Everyone sees the same information

2. **Track Every Sales Deal**
   - Know exactly where each deal stands
   - Never miss a follow-up

3. **Work Together Better**
   - Sales team sees same customer history
   - Managers can see team performance

4. **Save Time**
   - Automatic calculations
   - Ready-made reports
   - Less data entry

5. **Make Better Decisions**
   - See which products sell best
   - Know customer buying patterns

**Example:**
Before: A salesperson writes customer info on sticky notes
After: Everyone enters info into Dynamics 365, accessible from anywhere

---

## 2. CORE SALES ENTITIES (TABLES) EXPLANATION

In Dynamics 365, data is stored in **tables** (also called "entities"). Each table holds a specific type of information.

Below are the main tables in the Sales Module.

---

### 2.1 ACCOUNT (Company/Customer)

**Entity Name:** Account

**What it Stores:**
The Account table holds information about **companies or businesses** you work with.

**Simple Explanation:**
This is where you store company information. If you sell to businesses (B2B), this is your main customer table.

**Why It Is Important:**
- Every other sales record usually links to an Account
- It represents your business relationships
- All contacts and opportunities connect here

**Primary Key:** AccountId (a unique number that identifies each company)

**Important Fields:**

| Field Name | What It Stores | Example |
|------------|----------------|---------|
| Name | Company name | ABC Corporation |
| AccountNumber | Your internal customer ID | CUST-001 |
| PrimaryContactId | Main person to contact | John Smith |
| WebsiteUrl | Company website | www.abc.com |
| Telephone1 | Main phone number | 555-123-4567 |
| Address | Company address | 123 Main Street |

**Connected To:**

- **Contact** (One Account → Many Contacts)
- **Opportunity** (One Account → Many Sales Deals)
- **Quote, Order, Invoice** (Through Opportunities)
- **Activity** (Emails, Calls, Meetings)

---

### 2.2 CONTACT (Individual Person)

**Entity Name:** Contact

**What it Stores:**
The Contact table stores information about **individual people**.

**Simple Explanation:**
These are the actual people you talk to. A contact works at a company (Account).

**Why It Is Important:**
- You communicate with people, not companies
- Each contact has their own email, phone, preferences
- Sales happen through people

**Primary Key:** ContactId (unique identifier)

**Important Fields:**

| Field Name | What It Stores | Example |
|------------|----------------|---------|
| FirstName | Person's first name | John |
| LastName | Person's last name | Smith |
| Email | Email address | john@abccorp.com |
| Phone | Phone number | 555-123-4567 |
| JobTitle | Their role | Sales Manager |
| ParentCustomerId | Which company they work for | ABC Corporation |

**Connected To:**

- **Account** (Many Contacts → One Account)
- **Opportunity** (Can sell to a contact directly)
- **Activity** (All communications)

---

### 2.3 LEAD (Potential Customer)

**Entity Name:** Lead

**What it Stores:**
A Lead represents a **potential customer** who has shown interest but hasn't been qualified yet.

**Simple Explanation:**
Someone might have filled out a web form, or you met them at a trade show. They're interested, but you're not sure if they're a real opportunity yet.

**Why It Is Important:**
- First step in the sales process
- Captures initial interest
- Can be converted to an Opportunity when qualified

**Primary Key:** LeadId (unique identifier)

**Important Fields:**

| Field Name | What It Stores | Example |
|------------|----------------|---------|
| FullName | Person's name | John Smith |
| CompanyName | Company they're from | ABC Corp |
| Email | Email address | john@abccorp.com |
| LeadSource | Where they came from | Website, Trade Show |
| Subject | What they're interested in | Product Demo Request |
| StatusCode | Current status | New, Contacted, Qualified |

**Connected To:**

- **Account** (can create new Account)
- **Contact** (can create new Contact)
- **Opportunity** (converts to when qualified)
- **Activity** (follow-up tasks)

---

### 2.4 OPPORTUNITY (Sales Deal)

**Entity Name:** Opportunity

**What it Stores:**
An Opportunity represents a **potential sales deal** that is being actively worked on.

**Simple Explanation:**
This is a real sales deal. You've qualified the lead, and now you're negotiating. You expect to win this business.

**Why It Is Important:**
- This is where revenue is tracked
- Shows pipeline of expected sales
- Managers track team performance here

**Primary Key:** OpportunityId (unique identifier)

**Important Fields:**

| Field Name | What It Stores | Example |
|------------|----------------|---------|
| Name | Opportunity name | ABC Corp - Q1 2026 Deal |
| CustomerId | Who you're selling to | ABC Corporation |
| EstimatedValue | Expected revenue | $50,000 |
| EstimatedCloseDate | When deal will close | March 31, 2026 |
| StepName | Current sales stage | Proposal, Negotiation |
| StatusCode | Deal status | In Progress, Won, Lost |

**Sales Stages (Default):**

1. Qualify - Is this a real opportunity?
2. Develop - Understand customer needs
3. Propose - Send quotes/proposals
4. Close - Finalize the deal

**Connected To:**

- **Account** (or Contact as customer)
- **Quote** (One Opportunity → Many Quotes)
- **Order** (when won)
- **OpportunityProduct** (products in this deal)

---

### 2.5 QUOTE (Sales Proposal)

**Entity Name:** Quote

**What it Stores:**
A Quote is a **formal price proposal** sent to a customer.

**Simple Explanation:**
You've talked about pricing. Now you send a document showing exactly what you'll provide and how much it costs. The customer can accept or reject it.

**Why It Is Important:**

- Formal pricing document
- Shows products and prices
- Can be converted to Order when accepted

**Primary Key:** QuoteId (unique identifier)

**Important Fields:**

| Field Name | What It Stores | Example |
|------------|----------------|---------|
| Name | Quote name | Quote for ABC Corp - March 2026 |
| OpportunityId | Which deal this is for | ABC Corp Deal |
| CustomerId | Who receives the quote | ABC Corporation |
| TotalAmount | Total price | $50,000 |
| EffectiveFrom | Quote valid from | March 1, 2026 |
| EffectiveTo | Quote valid until | March 31, 2026 |
| StatusCode | Quote status | Draft, Active, Won, Closed |

**Connected To:**

- **Opportunity** (Many Quotes → One Opportunity)
- **QuoteProduct** (products in the quote)
- **Order** (converts to when accepted)

---

### 2.6 ORDER (Customer Purchase)

**Entity Name:** SalesOrder (often called Order)

**What it Stores:**
An Order represents a **confirmed purchase** from a customer.

**Simple Explanation:**
The customer accepted your quote! Now you have a binding agreement. You need to fulfill this order.

**Why It Is Important:**
- This is a confirmed sale
- Triggers fulfillment process
- Basis for invoicing

**Primary Key:** OrderId (unique identifier)

**Important Fields:**

| Field Name | What It Stores | Example |
|------------|----------------|---------|
| Name | Order name | Order #1001 |
| OpportunityId | Original deal | ABC Corp Deal |
| CustomerId | Who ordered | ABC Corporation |
| TotalAmount | Order total | $50,000 |
| DateFulfilled | When order fulfilled | April 15, 2026 |
| StatusCode | Order status | Submitted, Fulfilled, Invoiced |

**Connected To:**

- **Opportunity** (came from)
- **Quote** (converted from)
- **Invoice** (generated from)
- **OrderProduct** (products ordered)

---

### 2.7 INVOICE (Payment Request)

**Entity Name:** Invoice

**What it Stores:**
An Invoice is a **request for payment** sent to the customer.

**Simple Explanation:**
"You owe us money." The invoice shows what was purchased and the payment amount and terms.

**Why It Is Important:**
- Tracks money owed to company
- Part of accounting process
- Shows completed sales

**Primary Key:** InvoiceId (unique identifier)

**Important Fields:**

| Field Name | What It Stores | Example |
|------------|----------------|---------|
| Name | Invoice number | INV-1001 |
| OrderId | Related order | Order #1001 |
| CustomerId | Who pays | ABC Corporation |
| TotalAmount | Amount due | $50,000 |
| InvoiceDate | Date issued | April 20, 2026 |
| DueDate | Payment deadline | May 20, 2026 |
| StatusCode | Invoice status | Paid, Overdue, Canceled |

**Connected To:**

- **Order** (One Invoice → One Order)
- **Account/Contact** (customer)

---

### 2.8 PRODUCT (Items You Sell)

**Entity Name:** Product

**What it Stores:**
Products are the **items or services** your company sells.

**Simple Explanation:**
This is your product catalog. Every item you sell - whether it's a physical product or a service - is stored here with its price.

**Why It Is Important:**
- Used in Quotes, Orders, Invoices
- Central pricing information
- Inventory tracking (if enabled)

**Primary Key:** ProductId (unique identifier)

**Important Fields:**

| Field Name | What It Stores | Example |
|------------|----------------|---------|
| Name | Product name | Enterprise Software License |
| ProductNumber | SKU/Product code | PROD-001 |
| Price | Base price | $1,000 |
| Unit | Measurement unit | Per User, Per Year |
| Description | Product details | Annual license fee |
| StatusCode | Product status | Active, Retired |

**Connected To:**

- **QuoteProduct, OrderProduct, InvoiceProduct** (line items)
- **PriceList** (pricing tiers)

---

### 2.9 ACTIVITY (Communication Records)

**Entity Name:** Activity (or ActivityPointer)

**What it Stores:**
Activities track **all communications** with customers - emails, calls, meetings, tasks.

**Simple Explanation:**
Every time you talk to a customer, you create an Activity record. This keeps a history of all interactions.

**Why It Is Important:**
- Complete communication history
- Follow-up reminders
- Audit trail
- Team visibility

**Types of Activities:**

| Activity Type | What It Is |
|--------------|------------|
| Email | Written correspondence |
| Phone Call | Telephone conversations |
| Task | To-do items |
| Appointment | Scheduled meetings |
| Letter | Physical mail |
| Fax | Fax communications |

**Primary Key:** ActivityId (unique identifier)

**Important Fields:**

| Field Name | What It Stores | Example |
|------------|----------------|---------|
| Subject | What the activity is about | Follow up on proposal |
| Description | Details | Discuss pricing options |
| StateCode | Open/Completed/Canceled | Open |
| StatusCode | Specific status | In Progress |
| ScheduledStart | When it starts | March 15, 2026 10:00 AM |
| ScheduledEnd | When it ends | March 15, 2026 11:00 AM |
| RegardingObjectId | What it's about | ABC Corporation |

**Connected To:**

- **Account** (communication with company)
- **Contact** (communication with person)
- **Opportunity** (deal-related activities)
- **Lead** (lead-related activities)

---

## 3. RELATIONSHIP STRUCTURE

Relationships explain **how tables connect** to each other. Understanding this helps you know how data flows.

---

### Basic Relationship Types

#### One-to-Many (1:N)
One record links to many records.

**Example:** One Account → Many Contacts

```
Account: ABC Corporation
    ├── Contact: John Smith
    ├── Contact: Jane Doe
    └── Contact: Bob Johnson
```

#### Many-to-One (N:1)
Many records point back to one record.

**Example:** Many Contacts → One Account

```
Contact: John Smith → Account: ABC Corporation
Contact: Jane Doe → Account: ABC Corporation
Contact: Bob Johnson → Account: ABC Corporation
```

#### Many-to-Many (N:N)
Many records link to many records.

**Example:** Many Products → Many Price Lists

---

### Key Sales Relationships

#### 1. Account → Many Contacts
**Rule:** One company can have many people.

```
ABC Corporation
    ├── John Smith (Sales Manager)
    ├── Jane Doe (CEO)
    └── Bob Johnson (Purchasing)
```

**Why?** One company has multiple employees you communicate with.

---

#### 2. Account → Many Opportunities
**Rule:** One company can have multiple sales deals.

```
ABC Corporation
    ├── Opportunity: Q1 Software License
    └── Opportunity: Q3 Training Services
```

**Why?** You can sell multiple products or services to the same customer over time.

---

#### 3. Opportunity → Many Quotes
**Rule:** One sales deal can have multiple quotes.

```
ABC Corporation Deal
    ├── Quote: Standard Pricing
    ├── Quote: Premium Pricing
    └── Quote: Enterprise Pricing
```

**Why?** You might send different pricing options to the customer.

---

#### 4. Quote → One Order
**Rule:** One quote typically becomes one order.

```
Quote #101 (Standard Pricing)
    └── Order #201
```

**Why?** When customer accepts a quote, it becomes an order.

---

#### 5. Order → One Invoice
**Rule:** One order typically creates one invoice.

```
Order #201
    └── Invoice #301
```

**Why?** Invoice is the payment request for the order.

---

### Simple Visual Relationship Map

```
ACCOUNT (Company)
    │
    ├──→ CONTACT (People)
    │
    ├──→ OPPORTUNITY (Sales Deals)
    │        │
    │        ├──→ QUOTE (Price Proposals)
    │        │        │
    │        │        └──→ ORDER (Confirmed Purchase)
    │        │                 │
    │        │                 └──→ INVOICE (Payment Request)
    │        │
    │        └──→ OPPORTUNITY PRODUCT (Products in Deal)
    │
    └──→ ACTIVITY (All Communications)
```

---

## 4. DATA HIERARCHY STRUCTURE

The hierarchy shows the **flow of data** from top to bottom. This is how sales naturally progress.

---

### Complete Sales Hierarchy

```
ACCOUNT (Company)
    │
    ├── LEAD (Potential Interest)
    │        │
    │        └── When Qualified ↓
    │
    ├── OPPORTUNITY (Real Sales Deal)
    │        │
    │        ├── QUOTE #1
    │        │        │
    │        │        └── ORDER #1
    │        │                │
    │        │                └── INVOICE #1 (Paid)
    │        │
    │        ├── QUOTE #2
    │        │
    │        └── OPPORTUNITY PRODUCTS (Items being sold)
    │
    ├── CONTACT (People at Company)
    │
    └── ACTIVITY (History of All Interactions)
             ├── Emails
             ├── Phone Calls
             ├── Tasks
             └── Meetings
```

---

### How Data Flows (Step by Step)

**Step 1: Create Account**
```
Start: Create ABC Corporation in Account table
```

**Step 2: Add Contacts**
```
Add John Smith, Jane Doe to Contact table
Link them to ABC Corporation
```

**Step 3: Create Lead (Optional)**
```
Lead: Someone showed interest
Later: Qualify and convert to Opportunity
```

**Step 4: Create Opportunity**
```
Opportunity: ABC Corp - 2026 Deal
Value: $50,000
Stage: Qualify
```

**Step 5: Add Products**
```
Opportunity Products:
- Enterprise License
- Training Services
- Support Package
```

**Step 6: Create Quote**
```
Quote: Send pricing to customer
Include all products and prices
```

**Step 7: Customer Accepts Quote**
```
Quote Status: Won
Convert to Order
```

**Step 8: Create Order**
```
Order: Customer purchase confirmed
Trigger fulfillment
```

**Step 9: Create Invoice**
```
Invoice: Send to customer for payment
Track payment status
```

**Step 10: Record Activities**
```
Throughout: Log all emails, calls, meetings
Complete history for future reference
```

---

### Dependency Explanation

**Why does this order matter?**

1. **You can't have an Order without an Opportunity**
   - Order needs to come from a qualified deal

2. **You can't have an Invoice without an Order**
   - Invoice is payment for a confirmed order

3. **You can't have an Opportunity without an Account or Contact**
   - Something must own the opportunity

4. **Activities can link to ANY level**
   - Email about Quote? Links to Quote
   - Call about Account? Links to Account
   - Meeting about Opportunity? Links to Opportunity

---

## 5. DATABASE DESIGN LOGIC

This section explains **how the database is organized** and why it's designed this way.

---

### 5.1 Table Types

Dynamics 365 organizes tables into three main categories:

---

#### MASTER TABLES (Reference Data)

**What are they?**
Master tables store the **main entities** - the core data your business operates on.

**Purpose:**

- Store fundamental business objects
- Data changes infrequently
- Foundation for transactions

**Examples in Sales Module:**

| Table | What It Stores |
|-------|----------------|
| Account | Companies you do business with |
| Contact | Individual people |
| Product | Items you sell |
| PriceList | Pricing information |

**Key Characteristic:**
These tables exist independently. Other tables reference them.

---

#### TRANSACTION TABLES (Operational Data)

**What are they?**
Transaction tables store **day-to-day business activities** - the actual sales actions.

**Purpose:**

- Capture ongoing business transactions
- High frequency of inserts/updates
- Represents real business events

**Examples in Sales Module:**

| Table | What It Stores |
|-------|----------------|
| Opportunity | Active sales deals |
| Quote | Price proposals |
| Order | Confirmed purchases |
| Invoice | Payment requests |

**Key Characteristic:**
These drive revenue. They come and go as deals open and close.

---

#### ACTIVITY TABLES (Interaction Data)

**What are they?**
Activity tables track **communications and interactions** with customers.

**Purpose:**

- Log all customer touchpoints
- Maintain history
- Enable follow-ups

**Examples:**

| Table | What It Stores |
|-------|----------------|
| Email | Email messages |
| PhoneCall | Phone call records |
| Task | To-do items |
| Appointment | Meetings |

**Key Characteristic:**
Activities can link to any other table - very flexible.

---

### 5.2 Status Columns Explained

Every transaction table has two important status columns:

---

#### StateCode (Overall Status)

**What it means:**
The general state of the record - is it active or finished?

**Values:**

| StateCode | Meaning | Example |
|-----------|---------|---------|
| 0 | Open/Active | Opportunity still being worked |
| 1 | Completed/Won | Deal closed successfully |
| 2 | Lost/Canceled | Deal was lost or cancelled |

---

#### StatusCode (Specific Status)

**What it means:**
The detailed status within each state.

**Examples for Opportunity:**

| StateCode | StatusCode | Meaning |
|-----------|------------|---------|
| 0 (Open) | 1 | In Progress - actively working |
| 0 (Open) | 2 | On Hold - paused temporarily |
| 1 (Won) | 3 | Won - customer said yes! |
| 2 (Lost) | 4 | Lost - customer said no |

---

#### Why Two Status Columns?

**StateCode:** Quick filtering
- Show me all open opportunities
- Show me all completed orders

**StatusCode:** Detailed tracking
- Why is this on hold?
- What stage is this deal in?

---

### 5.3 Date Tracking Columns

Dynamics 365 automatically tracks **when things happen**.

---

#### Common Date Columns

| Column Name | What It Tracks |
|-------------|----------------|
| CreatedOn | When record was first created |
| ModifiedOn | Last time record was changed |
| ScheduledStart | When something should start |
| ScheduledEnd | When something should end |
| ActualStart | When it actually started |
| ActualEnd | When it actually ended |

**Example:**

```
Task: Call ABC Corp
ScheduledStart: March 15, 2026 10:00 AM
ActualStart: March 15, 2026 10:15 AM
ScheduledEnd: March 15, 2026 10:30 AM
ActualEnd: March 15, 2026 10:25 AM
```

---

### 5.4 Ownership and Security

Every record has an **Owner** - the person responsible for it.

---

#### OwnerId Column

**What it does:**

- Links to SystemUser (person) or Team
- Determines who can see/edit the record
- Enables security and assignment

**Why it matters:**

- Sales reps see their own accounts
- Managers see their team's data
- Protects sensitive customer info

---

## 6. FINAL SUMMARY FOR TEAM LEAD

### 10-Line Executive Summary

---

**Microsoft Dynamics 365 Sales Module** is a customer relationship management system that helps businesses track companies, contacts, and sales deals from start to finish.

**The database has 9 core tables:** Account (companies), Contact (people), Lead (potential customers), Opportunity (active deals), Quote (price proposals), Order (confirmed sales), Invoice (payment requests), Product (items sold), and Activity (communications).

**Data flows in a clear hierarchy:** Account → Lead → Opportunity → Quote → Order → Invoice, with Contacts and Activities connected at every level.

**The system uses Master tables** (Account, Contact, Product) for core data, **Transaction tables** (Opportunity, Quote, Order, Invoice) for sales activities, and **Activity tables** for all customer communications.

**Every transaction record has two status columns:** StateCode (Open/Completed/Canceled) and StatusCode (specific progress like In Progress, Won, or Lost).

**The design follows industry best practices** with clear relationships, automatic date tracking, and ownership security built into every record.

**This is enterprise-grade architecture** proven across millions of implementations worldwide - scalable, maintainable, and ready for production use.

---

## Quick Reference Card

| Entity | Purpose | Connects To |
|--------|---------|-------------|
| Account | Company | Contact, Opportunity, Activity |
| Contact | Person | Account, Opportunity, Activity |
| Lead | Potential customer | Account, Contact, Opportunity |
| Opportunity | Sales deal | Account, Quote, Order, Product |
| Quote | Price proposal | Opportunity, Order |
| Order | Confirmed sale | Quote, Invoice |
| Invoice | Payment request | Order |
| Product | Item for sale | Opportunity, Quote, Order, Invoice |
| Activity | Communication | Any entity |

---

**Document Prepared For:** Team Lead Review
**Framework:** Microsoft Dynamics 365 Sales Module
**Complexity Level:** Beginner-Friendly
**Status:** Meeting Ready

---

*End of Document*