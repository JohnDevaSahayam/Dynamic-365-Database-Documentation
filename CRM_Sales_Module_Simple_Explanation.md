# CRM Sales Module - Simple Explanation Document

**For**: Team Lead and Manager Review  
**Date**: January 2026

---

This document explains our CRM Sales Module database in simple, easy-to-understand terms.

---

## 1. System Overview

### What is this CRM System?

Our CRM (Customer Relationship Management) system is a database that helps our sales team manage the complete sales process - from first contact with a potential customer all the way through getting paid.

**In simple terms:**
- It tracks who our customers are (Accounts, Contacts)
- It tracks potential customers (Leads)
- It manages the sales process (Opportunities, Quotes, Orders, Invoices)
- It stores product information and pricing
- It controls who can see and edit data (Security)

The system follows a standard sales flow:

```
Lead → Qualification → Opportunity → Quote → Order → Invoice → Close
```

---

## 2. Data Structure Explanation

### Main Business Entities

Our CRM has 7 main entities that handle the sales process:

| Entity | Purpose | What it Represents |
|--------|---------|-------------------|
| **Account** | Customer Company | A business/organization we sell to |
| **Contact** | Individual Person | A person at a company (decision maker, buyer) |
| **Lead** | Potential Customer | Someone interested in our products but not yet qualified |
| **Opportunity** | Active Sale | A qualified sales opportunity we're working on |
| **Quote** | Price Proposal | Formal pricing we sent to customer |
| **SalesOrder** | Customer Order | Confirmed purchase order |
| **Invoice** | Payment Request | Bill sent to customer for payment |

### Supporting Entities

These support the main entities:

| Entity | Purpose |
|--------|---------|
| **Product** | Items we sell (products and services) |
| **PriceList** | Pricing rules for products |
| **OpportunityProduct** | Products added to an opportunity (line items) |
| **QuoteDetail** | Products added to a quote (line items) |
| **OrderDetail** | Products added to an order (line items) |
| **InvoiceDetail** | Products added to an invoice (line items) |

### Security Entities

These control who can access what:

| Entity | Purpose |
|--------|---------|
| **User** | People who use the system |
| **Role** | Defines what users can do (permissions) |
| **BusinessUnit** | Organizes users into teams/departments |

### What Each Main Entity Contains

**ACCOUNT (Companies)**
- Company name, address, phone, website
- Industry classification
- Revenue, employee count
- Credit limit
- Primary contact person

**CONTACT (People)**
- First name, last name, email, phone
- Job title, department
- Associated company (Account)

**LEAD (Potential Customer)**
- Name, company, contact information
- Interest level (rating)
- Source of lead (campaign, referral, etc.)
- Estimated budget

**OPPORTUNITY (Active Sale)**
- Customer (Account or Contact)
- Estimated value (how much we might sell)
- Probability of winning
- Expected close date
- Sales stage (New → In Progress → Won/Lost)

**QUOTE (Price Proposal)**
- Customer and pricing
- Products and quantities
- Discounts and total amount
- Valid until date

**SALES ORDER (Confirmed Order)**
- Customer and order details
- Products and quantities
- Shipping information
- Status (New → Fulfilled → Invoiced → Paid)

**INVOICE (Bill)**
- Customer and amount due
- Payment terms
- Due date
- Payment status (Paid/Unpaid)

---

## 3. Relationship Explanation

### How Tables Connect

Tables are connected using **relationships**. Think of it like a family tree - some things "belong to" other things.

### One-to-Many Relationships

One record in a parent table can have many related records in a child table.

**Example:**
- One **Account** can have many **Contacts**
- One **Opportunity** can have many **OpportunityProducts**
- One **Quote** can have many **QuoteDetails** (line items)

```
Account (1) ──────→ Many Contacts
Opportunity (1) ──────→ Many OpportunityProducts
Quote (1) ──────→ Many QuoteDetails
```

### Why Use Relationships?

Relationships help us:
1. **Organize data** - Connect related records together
2. **Find data quickly** - Link information across tables
3. **Maintain accuracy** - Ensure we don't delete data that's being used

### Foreign Keys - The Connection Point

A **Foreign Key** is a field in one table that points to a record in another table.

**Example:**
- Contact table has a field called `ParentAccountId`
- This points to which Account the Contact belongs to
- This is how we know which company a person works for

---

## 4. Data Flow Explanation

This is how a typical sale moves through our system:

### Step 1: LEAD (New Potential Customer)
```
What's created: A Lead record
What happens: Sales rep enters prospect information
Data captured: Name, company, phone, email, interest level
Status: Open, Attempted to Contact, Contacted
```

### Step 2: QUALIFICATION (Evaluating the Lead)
```
What's created: Lead is reviewed and evaluated
What happens: Sales rep checks if prospect is serious
Data updated: Rating, estimated budget, purchase timeframe
Decision: Qualify or Disqualify the lead
```

### Step 3: OPPORTUNITY (Active Sales Deal)
```
What's created: Opportunity record (from qualified lead)
What happens: We start actively pursuing the sale
Data includes: Customer, estimated value, probability, expected close date
Sales stages: New → In Progress → [Won or Lost]
```

### Step 4: QUOTE (Price Proposal)
```
What's created: Quote from Opportunity
What happens: Sales rep prepares formal pricing
Data includes: Products, quantities, prices, discounts
Status: Draft → Active → Won (accepted) or Lost (declined)
```

### Step 5: SALES ORDER (Confirmed Purchase)
```
What's created: Sales Order from Quote (when won)
What happens: Customer accepted the quote
Data includes: Order details, shipping address, payment terms
Status: New → Fulfilled → Invoiced → Paid
```

### Step 6: INVOICE (Payment Request)
```
What's created: Invoice from Sales Order
What happens: Billing is generated
Data includes: Amount due, payment terms, due date
Status: New → Partial → Complete → Paid
```

### Step 7: CLOSE (Sale Complete)
```
What's created: Opportunity marked as Won
What happens: Revenue is recorded
Data updated: Actual value, actual close date
Status: Closed - Sale is complete!
```

---

## 5. Validation & Control

### Required Fields

Certain fields MUST be filled in to save a record:

| Entity | Required Fields |
|--------|----------------|
| Account | Name, Owner |
| Contact | Last Name, Owner |
| Lead | Subject, Owner |
| Opportunity | Name, Customer, Owner |
| Quote | Customer, Owner |
| Order | Customer, Owner |
| Invoice | Customer, Owner |

### Status Tracking (StateCode / StatusCode)

Every record has a status to track where it is in its lifecycle:

**StateCode** = The main state (Active/Inactive)

**StatusCode** = The specific reason for the state

**Example - Opportunity:**
| StateCode | Meaning | StatusCode | Meaning |
|-----------|---------|------------|----------|
| 0 (Open) | Still working | 1 = New, 2 = In Progress, 3 = On Hold |
| 1 (Won) | We won! | 4 = Won |
| 2 (Lost) | We lost | 5 = Lost, 6 = Cancelled |

### Audit Fields - Who Changed What & When

Every record automatically tracks:

| Field | What it Shows |
|-------|---------------|
| **CreatedOn** | When the record was first created |
| **CreatedBy** | Who created the record |
| **ModifiedOn** | When the record was last changed |
| **ModifiedBy** | Who last changed the record |
| **VersionNumber** | A counter that changes each time - helps detect conflicts |

### Data Integrity

**How we keep data accurate:**

1. **Unique IDs** - Every record gets a unique identifier (like a fingerprint)
2. **Required fields** - Can't save without essential information
3. **Valid choices** - Dropdown fields only allow predefined options
4. **Currency tracking** - Money fields store both original currency and base currency
5. **Number tracking** - Version numbers prevent two people from overwriting each other's changes

---

## 6. Short Summary (For Meeting Speaking Notes)

**Here's what you can say in a meeting:**

> "Our CRM Sales Module is a database that manages our complete sales process - from lead to payment.
>
> It has 7 main areas: Accounts (companies), Contacts (people), Leads (potential customers), Opportunities (active deals), Quotes (price proposals), Orders (confirmed sales), and Invoices (bills).
>
> The system tracks each sale as it moves through stages: Lead gets qualified, becomes an Opportunity, gets a Quote, which turns into an Order when won, then an Invoice is sent, and finally we mark it as Paid when we receive money.
>
> Every record tracks who owns it, when it was created, and when it was last modified. Status fields show where each record is in its lifecycle.
>
> The database connects all these pieces together - for example, an Invoice knows which Order it came from, which Order came from which Quote, which Quote came from which Opportunity, which Opportunity is for which Account and Contact.
>
> In simple terms: This system tracks our customers and every sale we make from first contact to getting paid."

---

## Quick Reference Card

| Stage | Entity Created | Status |
|-------|---------------|--------|
| New prospect | Lead | Open |
| Qualified | Opportunity | Open |
| Pricing sent | Quote | Active |
| Order received | Sales Order | Fulfilled |
| Bill sent | Invoice | Active |
| Payment received | Invoice | Paid |

---

**End of Document**
