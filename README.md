# Customer Restock Notification System

**Automated retention workflow** - Notify customers when out-of-stock items are back in stock.

## Overview

Workflow that:
1. Monitors when product is restocked (qty increases from low)
2. Finds customers who bought that product (last 5 orders)
3. Sends personalized Klaviyo email: "Item you wanted is back in stock!"
4. Logs notification for analytics

- **Status:** ✅ Designed & Tested
- **Tech Stack:** Zapier, Shopify API, Klaviyo, Google Sheets
- **Trigger:** Product restocked (qty increases significantly)
- **Recipients:** Customers with recent purchases of that item
- **Expected Conversion:** 5-10% (restock = missed sales recovery)

## The Problem

**Lost revenue from stockouts:**
- Customer wants product A
- Product A is out of stock
- Customer forgets / buys from competitor
- Product A is restocked 2 weeks later
- Customer never notified = lost sale

**Cost:** Estimated 5-10% lost revenue per restock event

## The Solution

**Automated "it's back!" notification:**
- Detect when product is restocked
- Find interested customers (those who bought before)
- Send personalized Klaviyo email
- Result: Recover lost sales from stockout period

**Expected:** 5-10% of notified customers convert = +revenue

## Architecture
Shopify: Product Updated (qty increases) ↓ [DETECT: was product out-of-stock recently?] ↓ [IF YES]: Product just got restocked ↓ [LOOKUP: Customers who bought this item] ├─ Query Shopify orders ├─ Filter: this product in last 5 orders ├─ Extract: customer emails └─ Exclude: customers who bought after restock ↓ [FOR EACH customer]: ├─ Send Klaviyo email: "Back in stock!" ├─ Log in Google Sheets: notification_sent └─ Track: open rate, click rate, conversion

## How It Works

### Trigger: Product Restocked
- Event: Product Updated (Shopify)
- Condition: Quantity was < 5, now > 10 (significant increase)
- Data: product_id, title, qty_before, qty_now

### Step 1: Get Product Info
Product ID: 12345
Title: Happy Cat Minkas - 10kg
Previous Qty: 2 (out of stock)
Current Qty: 45 (restocked)
Status: RESTOCKED

### Step 2: Query Shopify Orders
GET /orders
FILTER:
  - products.product_id = 12345
  - created_date >= 6 months ago
  - status = fulfilled
SORT: created_date DESC
LIMIT: 100 orders

### Step 3: Extract Customers
For each order:
  - Get customer.email
  - Get order.date
  - Exclude if customer bought AFTER restock date
  - Deduplicate

Result: 20-50 unique customer emails

### Step 4: Send Klaviyo Email
Template: Restock Notification
Recipients: Customers from Step 3
Personalization:
  - Customer name
  - Product name
  - "Order before it sells out" CTA

### Step 5: Log in Google Sheets
Sheet: Restock Notifications
Columns:
  - Timestamp
  - Product
  - Customers Notified
  - Status

## Klaviyo Integration

### Email Template

Subject: "Happy Cat is back in stock! 🐱"

Body:
Hi {{first_name}},

Good news! Happy Cat Minkas - 10kg is back in stock.

We noticed you bought this before. Order again before it sells out!

[Button: Buy Now] → product page

We appreciate your loyalty!

Best regards,
Happy4Pets Team

### Tracking

Klaviyo tracks:
- Open rate (estimated: 20-30%)
- Click rate (estimated: 5-10%)
- Conversion rate (estimated: 2-5%)

## Test Scenario

Timeline:
Mar 15: Happy Cat 10kg sells out (qty = 0)
        Customers A, B, C, D, E all bought before

Apr 1: Supplier delivers 50 units
       Qty = 50 (RESTOCK detected!)

Apr 1, 9:05 AM: Zapier triggers
       - Looks up customers A-E
       - Sends Klaviyo email to all 5
       - Logs to Google Sheets

Apr 1-5: Tracking
       - Open: 3/5 (60%)
       - Click: 2/5 (40%)
       - Convert: 1/5 (20%) = Customer A buys 2 units

Result: +€80 revenue from automated notification

## Expected Performance

- Customers notified per restock: 20-50
- Email open rate: 20-30%
- Click rate: 5-10%
- Conversion rate: 2-5%
- Revenue per restock: €50-200

## Cost & ROI

- Development time: 15-20 hours
- Klaviyo integration: Requires API setup
- Ongoing cost: Klaviyo email (~€0.05-0.10 per email)
- Maintenance: 30 min/month
- Expected ROI: €50-200 per restock = 10-50x ROI

## Status

✅ **Designed & Ready** (Awaiting Klaviyo API setup)

Dependency: Klaviyo account + API key configuration

Last updated: April 27, 2026
