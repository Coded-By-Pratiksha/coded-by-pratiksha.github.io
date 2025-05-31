---
layout: post
title: "Send a Table of Related Records in Email using Flows"
date: 2025-05-31
cover-img: /assets/img/Dust.jpg
thumbnail-img: assets/img/lemon.jpg
share-img: /assets/img/Dust.jpg
categories: [Salesforce, Flows, Email Templates, Automation]
tags: [Dynamic Email Content, Salesforce, HTML Table in Email, Email Alert]
author: Pratiksha
---

Sometimes you need to send an email from Salesforce that includes **a table of related records**, like listing all products in a Closed Won Opportunity. But Lightning Email Templates don’t support looping through child records like `OpportunityLineItems` out-of-the-box.

In this post, you’ll learn how to dynamically build a table of related records and send it via email using **Flows**, **Text Templates**, and optional **Email Alerts**.

---

## 🏷 Use Case

When an **Opportunity is marked as Closed Won**, send an email to the Account Owner listing all the products (OpportunityLineItems) added to that opportunity in a clean HTML table format.

---

## 💼 Step-by-Step Guide

### ✅ Step 1: Create a Lightning Email Template with a Placeholder

1. Go to **Setup → Email Templates → New Email Template**
2. Name it: `Opportunity Product Table Email`
3. Subject:
   
   ```
   Opportunity {!Opportunity.Name} - Product Details
   ```
   
5. Body (HTML format):


```html
Hi {!Opportunity.Owner.Name},

Your opportunity {!Opportunity.Name} has been Closed Won. Here are the products associated with it:

{!Opportunity.ProductTableHTML__c}

Regards,  
Sales Team
```

> ⚠ The `{!Opportunity.ProductTableHTML__c}` placeholder will be filled using a Flow-generated HTML table stored in a custom Long Text Area field.

---


### 🛠 Step 2: Create a Custom Field on Opportunity

- **Field Label**: `Product Table HTML`
- **API Name**: `ProductTableHTML__c`
- **Type**: Long Text Area (32768 characters)
- Add it to the page layout (optional, for testing/debugging)

---

### 🔁 Step 3: Build the Flow to Populate the Table

#### 1. Create a **Record-Triggered Flow**

- Object: `Opportunity`
- Trigger: **When record is updated**
- Entry Condition: `StageName Equals Closed Won`
- Optimize for: **Actions and Related Records**

#### 2. Get Related Products (OpportunityLineItems)

- **Get Records**: `Get Opportunity Products`
- Object: `OpportunityLineItem`
- Filter: `OpportunityId = {!$Record.Id}`
- Store all records

#### 3. Create Table Header Text Template

- **Text Template**: `tt_TableHeader`

```html
<table border="1" cellpadding="5" cellspacing="0" style="border-collapse: collapse;">
  <thead>
    <tr>
      <th>Product Name</th>
      <th>Quantity</th>
      <th>Unit Price</th>
      <th>Total Price</th>
    </tr>
  </thead>
  <tbody>
```

#### 4. Loop Through Products and Append Rows

- Add a **Loop**: `Loop Through Products`
- Collection: `Get Opportunity Products`
- Loop Variable: `loopProduct`

Inside the loop:

- **Text Template**: `tt_Row`

```html
<tr>
  <td>{!loopProduct.PricebookEntry.Name}</td>
  <td>{!loopProduct.Quantity}</td>
  <td>{!TEXT(loopProduct.UnitPrice)}</td>
  <td>{!TEXT(loopProduct.TotalPrice)}</td>
</tr>
```

- **Text Variable**: `varTableRows` (Text, no default value)
- **Assignment**: Append each row

```text
varTableRows = varTableRows + tt_Row
```

#### 5. Add Table Footer and Combine All

- **Text Template**: `tt_FinalTable`

```html
{!tt_TableHeader}
{!varTableRows}
</tbody></table>
```

#### 6. Update Opportunity Record

- **Assignment**:  
  `$Record.ProductTableHTML__c = tt_FinalTable`

- **Update Records**:  
  Record: `$Record`  
  Fields to update: `ProductTableHTML__c`

---

### ✉️ Step 4: Send the Email

You have 2 options:

#### Option A: Use Email Alert

1. Go to **Setup → Email Alerts → New**
2. Object: `Opportunity`
3. Email Template: `Opportunity Product Table Email`
4. Recipient: Opportunity Owner

Then, add an **Action in Flow**:
- **Action Type**: Email Alert
- Choose your newly created Email Alert

> ⚠ Make sure the Opportunity record has the `ProductTableHTML__c` field populated **before** the email is sent.

#### Option B: Use Send Email Action in Flow

Use the **"Send Email"** action directly in Flow:
- To: `{!$Record.Owner.Email}`
- Subject: `"Opportunity {!$Record.Name} - Product Details"`
- Body: `{!tt_FinalTable}` (HTML formatted)

---

## ✅ Output Example

The email will look like:

**Hi Priya,**

Your opportunity **"Acme Deal"** has been Closed Won. Here are the products associated with it:

<table border="1" cellpadding="5" cellspacing="0" style="border-collapse: collapse;">
  <thead>
    <tr>
      <th>Product Name</th>
      <th>Quantity</th>
      <th>Unit Price</th>
      <th>Total Price</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>Printer</td><td>2</td><td>₹5000</td><td>₹10000</td></tr>
    <tr><td>Ink Cartridge</td><td>3</td><td>₹800</td><td>₹2400</td></tr>
  </tbody>
</table>

Regards,  
Sales Team

---

## 🧠 Summary

| Feature | Approach |
|---------|----------|
| Build Dynamic Table | Use Loop + Text Templates in Flow |
| Store Final HTML | Custom Long Text Area Field |
| Send Email | Use Email Template + Alert or Direct Send Email Action |

This pattern works for any **Parent → Child** relationship, such as:
- Account → Contacts
- Case → Case Comments
- Opportunity → OpportunityLineItems
- CustomObject → Related Items

---

### 💡 Bonus Tip
You can even extend this logic to include **totals, discount fields, or hyperlinks** in the table if needed!

---

Let me know in the comments if you'd like the Flow screenshots or reusable components!
