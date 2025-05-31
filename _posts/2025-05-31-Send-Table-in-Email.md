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


## 🔖 Use Case

When an **Opportunity is marked as Closed Won**, send an email to the Account Owner listing all the products (OpportunityLineItems) added to that opportunity in a clean HTML table format.


## 🧰 Step-by-Step Guide

### ✔️ Step 1: Create a Lightning Email Template with a Placeholder

1. Go to **Setup → Email Templates → New Email Template**

2. Name it: `Opportunity Product Details Template`

3. Subject: `Opportunity Notification with Product Detail`

4. Body (HTML format):

```html
Hi {!Opportunity.Owner.Name},

Your opportunity {!Opportunity.Name} has been Closed Won. Here are the products associated with it:

{!Opportunity.ProductTable__c}

Regards,  
Sales Team

```

> ⚠ The `{!Opportunity.ProductTable__c}` placeholder will be filled using a Flow-generated HTML table stored in a custom Rich Text Area field.

![EmailTemplate](/assets/img/EmailTemplate.png)

### 🧱 Step 2: Create a Custom Field on Opportunity

- **Field Label**: `Product Table`  
- **API Name**: `ProductTable__c`  
- **Type**: Rich Text Area (32768 characters)  
- Add it to the page layout (optional, for testing/debugging)

![customField](/assets/img/customField.png)

### 🔁 Step 3: Build the Flow to Populate the Table

#### 🔹 1. Create a **Record-Triggered Flow**

- Object: `Opportunity`  
- Trigger: **When record is updated**  
- Entry Condition: `StageName Equals Closed Won`  
- Optimize for: **Actions and Related Records**

#### 🔹 2. Get Related Products (OpportunityLineItems)

- **Get Records**: `Get Opportunity Products`  
- Object: `OpportunityLineItem`  
- Filter: `OpportunityId = {!$Record.Id}`  
- Store all records
- Automatically store all fields

#### 🔹 3. Create Table Content Text Variable

- **Text Variable**: `TableContent`

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
![TableContent](/assets/img/TableContent.png)

#### 🔹 4. Loop Through Products and Append Rows

- Add a **Loop**: `Loop Through Products`  
- Collection: `Get Opportunity Products`  
- Loop Variable: `Loop_Through_Products`

Inside the loop:

- **Assignment**: Assign Table Rows 
- **Text Variable**: `TableContent` Add below value

```html
<tr>
  <td>{!Loop_Through_Products.ProductCode}</td>
  <td>{!Loop_Through_Products.Quantity}</td>
  <td>{!Loop_Through_Products.UnitPrice}</td>
  <td>{!Loop_Through_Products.TotalPrice}</td>
</tr>
```

#### 🔹 5. Add Table Footer

- **Assignment**: `TableContent` Add below value

```html
</tbody></table>
```

#### 🔹 6. Update Opportunity Record

- **Assignment**:  
  `$Record.ProductTable__c = TableContent`

- **Update Records**:  
  Record: `$Record`  
  Fields to update: `ProductTable__c`



### 📬 Step 4: Send the Email

You have 2 options:

#### 🅰️ Option A: Use Email Alert

1. Go to **Setup → Email Alerts → New**  
2. Object: `Opportunity`  
3. Email Template: `Opportunity Product Details Alert`  
4. Recipient: Opportunity Owner  

Then, add an **Action in Flow**:  
- **Action Type**: Email Alert  
- Choose your newly created Email Alert

> ⚠ Make sure the Opportunity record has the `ProductTable__c` field populated **before** the email is sent.

#### 🅱️ Option B: Use Send Email Action in Flow

Use the **"Send Email"** action directly in Flow:  
- To: `{!$Record.Owner.Email}`  
- Subject: `"Opportunity Notification with Product Details"`  
- Body: `{!ProductTable__c}` (HTML formatted)

![Flow](/assets/img/Flow.png)


## ✔️ Output Example

The email will look like:

**Hi ABC,**

Your opportunity **"Acme Deal"** has been Closed Won. Here are the products associated with it:

| Product Name | Quantity | Unit Price | Total Price |
| :------ |:--- | :--- | :--- |
| GC1060 | 1 | 100,000 | 100,000 |
| GC1020 | 1 | 5,000 | 5,000 |
| IN7080 | 1 | 85,000 | 5,000 |
| IN7040 | 1 | 20,000 | 20,000 |

Regards,  
Sales Team


## 🧠 Summary

| Feature | Approach |
|--------|----------|
| Build Dynamic Table | Use Loop + Text Variable in Flow |
| Store Final HTML | Custom Rich Text Area Field |
| Send Email | Use Email Template + Alert or Direct Send Email Action |

This pattern works for any **Parent → Child** relationship, such as:  
- Account → Contacts  
- Case → Case Comments  
- Opportunity → OpportunityLineItems  
- CustomObject → Related Items


### 💡 Bonus Tip

You can even extend this logic to include **totals, discount fields, or hyperlinks** in the table if needed!


Let me know if you face similar requirement and how did you achieve it, Thank You!
