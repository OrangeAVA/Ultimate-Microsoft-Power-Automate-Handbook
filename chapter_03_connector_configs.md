# Chapter 3: Power Automate Connector Configurations

## Overview
This document contains connector configurations and integration patterns from Chapter 3: Expanding Your Workflow with Connectors. This chapter focuses on using various connectors to build cross-platform workflows.

---

## Connector Types Overview

### Standard Connectors
- Available with any Office 365 license
- No additional cost
- Common services: SharePoint, Outlook, Teams

### Premium Connectors  
- Require Premium license
- Enterprise services: Salesforce, DocuSign, Adobe
- Enable advanced automation scenarios

### Custom Connectors
- Created for specific needs
- Connect to internal systems or unique APIs
- Require development knowledge

---

## Microsoft 365 Connector Examples

### 1. Outlook Connector - Automatic Email Processing

**Flow Name:** Automated Email Processing

**Trigger:** When a new email arrives (Outlook)

**Filter Configuration:**
- Contains specific keywords in subject or body
- Example keywords: "urgent", "invoice", "approval needed"

**Actions:**

**Action 1: Create Task in Planner**
- Connector: Microsoft Planner
- Task Title: Extract from email subject
- Description: Email body content
- Assigned To: Based on email category
- Due Date: Calculate based on priority

**Action 2: Save Attachment to SharePoint**
- Connector: SharePoint
- Site: Document Management Site
- Library: Email Attachments
- Folder: Organized by date or category
- File Name: Original attachment name
- Metadata: Add email sender, date received

**Configuration Notes:**
- Use connection references for environment portability
- Set up filter conditions to reduce noise
- Implement error handling for missing attachments

---

### 2. Teams Connector - Automated Team Notifications

**Flow Name:** File Modification Notifications

**Trigger:** When a file is modified (SharePoint)

**Configuration:**
- Site Address: Team SharePoint site
- Document Library: Shared Documents
- Include Subfolders: Yes

**Actions:**

**Action: Post Message to Teams Channel**
- Connector: Microsoft Teams
- Team: Select target team
- Channel: File Updates or relevant channel

**Message Template:**
```
📄 **File Updated**

**File Name:** {FileName}
**Modified By:** {ModifiedBy}
**Modified Date:** {Modified}
**Location:** {FilePath}

[View File]({FileLink})

---
This is an automated notification from Power Automate.
```

**Dynamic Content Used:**
- `{FileName}` - Name of modified file
- `{ModifiedBy}` - User who made changes
- `{Modified}` - Timestamp of modification
- `{FilePath}` - Full path to file
- `{FileLink}` - Direct URL to file

---

### 3. SharePoint Connector - Document Processing

**Flow Name:** New Document Processing

**Trigger:** New file in document library (SharePoint)

**Trigger Configuration:**
- Site Address: Corporate SharePoint
- Folder: Document Library root
- Include Subfolders: Yes

**Actions:**

**Action 1: Extract File Properties**
- Get file properties using file identifier
- Extract metadata: Title, Author, Created date, File type

**Action 2: Update SharePoint List**
- Connector: SharePoint
- Site Address: Same as trigger
- List Name: Document Tracking
- Action: Create item

**List Item Fields:**
```
Document Name: {Name}
File Type: {File Type}
Created By: {Created By}
Created Date: {Created}
File Size: {Size}
File URL: {Link to Item}
Processing Status: "Indexed"
Processing Date: utcNow()
```

---

## Data Connector Examples

### 4. Excel Connector - Automated Reporting

**Flow Name:** Daily Excel Report Generation

**Trigger:** Recurrence (Scheduled)
- Frequency: Daily
- Time: 8:00 AM
- Time Zone: Local

**Actions:**

**Action 1: Get Rows from Excel**
- Connector: Excel Online (Business)
- File: Sales_Data.xlsx
- Location: SharePoint or OneDrive
- Table: SalesTable

**Action 2: Process Data Using Expressions**

**Calculate Daily Total:**
```
sum(body('List_rows_present_in_a_table')?['value'], 'Amount')
```

**Filter High-Value Sales:**
```
filter(body('List_rows_present_in_a_table')?['value'], item()?['Amount'] > 10000)
```

**Format Date Range:**
```
formatDateTime(addDays(utcNow(), -1), 'yyyy-MM-dd')
```

**Action 3: Update Dashboard**
- Create new Excel row with summary data
- Update Power BI dataset (if applicable)
- Send email report to stakeholders

**Excel Connector Actions Available:**
- Add a row into a table
- Delete a row
- Get a row  
- Update a row
- List rows present in a table
- Get tables
- Create table
- Get worksheets
- Create worksheet
- Run script
- Run script from SharePoint library

---

### 5. SQL Connector - Database Operations

**Flow Name:** Database Synchronization

**Trigger:** When a record is modified (SQL Server)

**Connection Configuration:**
- Server Name: sql.company.com
- Database Name: CustomerDB
- Authentication: SQL Server Authentication or Windows Authentication

**Trigger Configuration:**
- Table Name: Customers
- Column to monitor: ModifiedDate

**Actions:**

**Action 1: Query Related Data**
```sql
SELECT 
    c.CustomerID,
    c.CustomerName,
    c.Email,
    o.OrderCount,
    o.TotalValue
FROM Customers c
LEFT JOIN (
    SELECT 
        CustomerID,
        COUNT(*) as OrderCount,
        SUM(OrderTotal) as TotalValue
    FROM Orders
    WHERE OrderDate >= DATEADD(month, -3, GETDATE())
    GROUP BY CustomerID
) o ON c.CustomerID = o.CustomerID
WHERE c.CustomerID = @{triggerBody()?['CustomerID']}
```

**Action 2: Update Connected Systems**
- Update CRM with latest customer data
- Sync to SharePoint customer list
- Send notification if high-value customer

---

## Premium Connector Examples

### 6. Salesforce Connector - Lead Management

**Flow Name:** New Lead Processing

**Trigger:** When a record is created (Salesforce)
- Object Type: Lead
- Include Fields: FirstName, LastName, Email, Company, Phone

**Actions:**

**Action 1: Create Contact in Microsoft 365**
- Connector: Office 365 Users
- Given Name: {FirstName}
- Surname: {LastName}
- Email: {Email}
- Company: {Company}
- Business Phone: {Phone}

**Action 2: Schedule Follow-up Tasks**
- Connector: Microsoft Planner
- Plan: Sales Follow-ups
- Bucket: New Leads
- Task Title: `Follow up with {FirstName} {LastName} from {Company}`
- Description: Lead details and context
- Due Date: 2 days from now
- Assigned To: Territory sales rep

**Action 3: Send Notification to Sales Team**
- Connector: Microsoft Teams
- Team: Sales Team
- Channel: New Leads

**Message Content:**
```
🎯 **New Lead Alert**

**Lead Information:**
- Name: {FirstName} {LastName}
- Company: {Company}
- Email: {Email}
- Phone: {Phone}
- Lead Source: {LeadSource}

**Next Steps:**
✅ Contact created in Office 365
✅ Follow-up task assigned to {AssignedTo}
✅ Initial email sent

[View Lead in Salesforce]({SalesforceRecordURL})
```

**Salesforce Connector Actions Available:**
- Create record
- Get records  
- Delete record
- Get a Record by External ID
- Create a job
- Close or abort a job
- Delete a job
- Get all jobs
- Get job info
- Execute SOSL search query
- Execute a SOQL query

---

### 7. DocuSign Connector - Contract Processing

**Flow Name:** Contract Signature Processing

**Trigger:** When an envelope status changes (DocuSign)
- Status: Completed

**Actions:**

**Action 1: Save Signed Document to SharePoint**
- Connector: SharePoint
- Site: Legal Documents
- Library: Signed Contracts
- Folder: Organized by year/month
- File Name: `{EnvelopeName}_{CompletedDate}.pdf`

**Document Metadata:**
```
Contract Name: {EnvelopeName}
Signed Date: {CompletedDateTime}
Signers: {RecipientsNames}
Contract Type: {CustomField_ContractType}
Contract Value: {CustomField_Value}
Status: Fully Executed
```

**Action 2: Update Contract Tracking System**
- Connector: SharePoint or SQL
- Update contract status to "Executed"
- Record completion date
- Store document reference

**Action 3: Send Confirmation Emails**
- To: All signers
- CC: Legal team, Contract administrator
- Subject: `Contract Executed: {EnvelopeName}`

**Email Template:**
```html
<h3>Contract Successfully Executed</h3>

<p>The following contract has been fully executed:</p>

<ul>
  <li><strong>Contract:</strong> {EnvelopeName}</li>
  <li><strong>Completion Date:</strong> {CompletedDateTime}</li>
  <li><strong>Signers:</strong> {RecipientsNames}</li>
</ul>

<p>The signed document has been saved to SharePoint and is available at:</p>
<p><a href="{SharePointDocumentURL}">View Signed Contract</a></p>

<p>Thank you for your participation.</p>
```

---

## Custom Connector Development

### API Definition Template

**Step 1: Define the API**

**General Information:**
```yaml
API Name: Internal System API
Description: Custom connector for company ERP system
Host: api.company.com
Base URL: /api/v1
```

**Authentication Configuration:**
```yaml
Authentication Type: OAuth 2.0
Authorization URL: https://api.company.com/oauth/authorize
Token URL: https://api.company.com/oauth/token
Refresh URL: https://api.company.com/oauth/refresh
Scope: read write
```

**Alternative Authentication - API Key:**
```yaml
Authentication Type: API Key
Parameter Name: X-API-Key
Parameter Location: Header
```

**Step 2: Define Operations**

**Operation: Get Customer**
```yaml
Operation ID: GetCustomer
Summary: Retrieve customer information
HTTP Method: GET
URL Path: /customers/{customerId}

Parameters:
  - Name: customerId
    Type: string
    Required: true
    Location: path

Response:
  Status Code: 200
  Schema:
    {
      "customerId": "string",
      "name": "string",
      "email": "string",
      "phone": "string",
      "address": {
        "street": "string",
        "city": "string",
        "state": "string",
        "zip": "string"
      }
    }
```

**Operation: Create Order**
```yaml
Operation ID: CreateOrder
Summary: Create new order
HTTP Method: POST
URL Path: /orders

Request Body:
  {
    "customerId": "string",
    "items": [
      {
        "productId": "string",
        "quantity": number,
        "price": number
      }
    ],
    "shippingAddress": {
      "street": "string",
      "city": "string",
      "state": "string",
      "zip": "string"
    }
  }

Response:
  Status Code: 201
  Schema:
    {
      "orderId": "string",
      "orderDate": "datetime",
      "totalAmount": number,
      "status": "string"
    }
```

---

## Advanced Multi-Connector Scenarios

### 8. Customer Onboarding Automation

**Flow Name:** Complete Customer Onboarding

**Trigger:** New customer record created (CRM)

**Multi-Connector Actions:**

**1. Create SharePoint Site (SharePoint Connector)**
```
Site Design: Customer Project Template
Site Name: {CustomerName} - Project Space
Site URL: /sites/{CustomerNameSlug}
Owners: Account Manager, Project Manager
Members: Customer contacts
```

**2. Set Up Teams Channel (Teams Connector)**
```
Team: Customer Success
New Channel: {CustomerName}
Channel Type: Standard
Add Members: Account team, Customer contacts
Post Welcome Message with onboarding checklist
```

**3. Generate Welcome Documents (Word Connector)**
```
Template: Customer Welcome Package
Populate Fields:
  - Customer Name
  - Account Number
  - Primary Contact
  - Account Manager
  - Important Dates
Output Location: SharePoint customer site
```

**4. Send for Signature (DocuSign Connector)**
```
Envelope Name: Welcome Package - {CustomerName}
Documents: Terms of Service, NDA, Service Agreement
Recipients:
  - Customer Primary Contact (Signer)
  - Customer Secondary Contact (CC)
  - Account Manager (CC)
Reminder: 3 days
Expiration: 30 days
```

**5. Update Customer Record (CRM Connector)**
```
Update Fields:
  - Onboarding Status: "In Progress"
  - SharePoint Site: {SiteURL}
  - Teams Channel: {ChannelLink}
  - Documents Sent: {DocuSignEnvelopeID}
  - Onboarding Start Date: {CurrentDate}
```

---

## Error Handling Patterns

### Try-Catch Pattern Implementation

**Configure Run After Settings:**

```
Main Action: Send Email
├─ Run after: is successful
└─ Error Handling Branch
    ├─ Run after: has failed, has timed out
    └─ Actions:
        ├─ Log error to SharePoint
        ├─ Send alert to admin
        └─ Retry with different parameters
```

**Retry Policy Configuration:**
```yaml
Retry Policy:
  Type: Exponential
  Count: 4
  Interval: PT5S  # 5 seconds
  Maximum Interval: PT1H  # 1 hour
  Minimum Interval: PT5S  # 5 seconds
```

**Error Logging Action:**
```
Connector: SharePoint
List: Error Log
Fields:
  Flow Name: @{workflow().name}
  Run ID: @{workflow().run.id}
  Error Message: @{body('Failed_Action')?['error']?['message']}
  Error Code: @{body('Failed_Action')?['error']?['code']}
  Timestamp: @{utcNow()}
  Action Name: Failed_Action
```

---

## Real-World Flow Examples

### 9. HR Employee Onboarding

**Flow Name:** Complete Employee Onboarding Process

**Trigger:** New employee record in HR system

**Actions Flow:**

**1. Create User Accounts (Azure AD Connector)**
```
User Principal Name: {FirstName}.{LastName}@company.com
Display Name: {FirstName} {LastName}
Job Title: {JobTitle}
Department: {Department}
Manager: {ManagerUPN}
Office Location: {Office}
```

**2. Set Up Workstation (ServiceNow Connector)**
```
Request Type: New Employee Setup
Category: Hardware Provisioning
Items:
  - Laptop: {LaptopModel}
  - Monitor: Dual monitors
  - Phone: {PhoneModel}
  - Access Card: Building access
Priority: High
Assigned To: IT Support
Due Date: {StartDate}
```

**3. Schedule Orientation (Outlook Connector)**
```
Subject: New Employee Orientation - {EmployeeName}
Start Time: {StartDate} 9:00 AM
Duration: 4 hours
Location: Conference Room A
Attendees:
  - {EmployeeEmail}
  - {ManagerEmail}
  - HR@company.com
Body: Orientation agenda and materials
```

**4. Generate Paperwork (SharePoint Connector)**
```
Documents to Generate:
  - Employment Agreement
  - Benefits Enrollment Form
  - Direct Deposit Form
  - Emergency Contact Form
  - IT Acceptable Use Policy

Location: HR Documents/{EmployeeName}
Permissions: HR team, Employee, Manager
```

---

### 10. Sales Lead to Order Process

**Flow Name:** Lead to Order Automation

**Trigger:** New lead created (Salesforce)

**Actions Flow:**

**1. Enrich Lead Data (LinkedIn Connector - Premium)**
```
Search: {LeadName} at {Company}
Retrieve:
  - Professional Summary
  - Current Role
  - Company Size
  - Industry
Update Lead Record with enriched data
```

**2. Create Opportunity (Salesforce Connector)**
```
Opportunity Name: {Company} - {Product Interest}
Account: Link to {Company} account
Contact: Link to {LeadName}
Stage: Qualification
Amount: {Estimated Value}
Close Date: {Current Date + 90 days}
Owner: {Territory Sales Rep}
```

**3. Schedule Follow-up (Teams Connector)**
```
Create Meeting:
  Subject: Initial Consultation - {Company}
  Duration: 30 minutes
  Proposed Times: 3 options in next 5 business days
  Attendees:
    - {Lead Email}
    - {Sales Rep}
  Teams Meeting Link: Auto-generated
Send Meeting Invite
```

**4. Generate Quote (PDF Connector - Premium)**
```
Template: Sales Quote Template
Data Source: Opportunity record
Populate:
  - Company Information
  - Product Details
  - Pricing
  - Terms and Conditions
Output: PDF document
Save to: SharePoint Sales Documents
```

---

## Security Best Practices

### Connection Security Configuration

**1. Managed Identity Setup (When Available)**
```yaml
Connection Type: Managed Identity
Identity: System-assigned
Permissions: Least privilege required
Scope: Specific resources only
```

**2. Credential Rotation Schedule**
```
Service Account Passwords: Every 90 days
API Keys: Every 180 days
OAuth Tokens: Auto-refresh enabled
Certificate-based Auth: Annual renewal
```

**3. Access Control**
```yaml
Connection Ownership: Service accounts
Sharing: Disabled by default
Approved Users: Documented list
Review Frequency: Quarterly
```

### Data Loss Prevention (DLP) Policy

**Connector Classification:**

**Business Data Group:**
- SharePoint
- OneDrive
- Exchange
- Teams
- Planner

**Non-Business Data Group:**
- Twitter
- Facebook
- Instagram
- Consumer Gmail

**Blocked Connectors:**
- File system operations
- Desktop flows (in production)
- HTTP connectors (without approval)

**Policy Enforcement:**
```
Rule: Business connectors cannot be used with non-business connectors in same flow
Action: Block flow save/activation
Alert: Send to compliance team
Audit: Log all violations
```

---

## Performance Optimization

### Pagination Implementation

**For Large Datasets:**
```yaml
Action: List items (SharePoint)
Settings:
  Top Count: 5000  # Maximum per request
  Pagination:
    Enable: Yes
    Threshold: 100000  # Total items to retrieve
```

**Processing Pattern:**
```
Do Until: No more items
├─ Get items (top 5000)
├─ Process batch
├─ Update cursor/skip token
└─ Check for more items
```

### Batch Operations

**Example: Update Multiple Records**
```yaml
Instead of: Loop through items, update one by one
Use: Batch update API

Batch Configuration:
  Items per batch: 100
  Parallel batches: 5
  Retry on failure: Yes
  Error handling: Continue on partial failure
```

### Caching Strategy

**Implement for:**
- Reference data that changes infrequently
- Lookup tables
- Configuration settings
- User profiles

**Cache Location Options:**
- SharePoint list (simple)
- Azure Table Storage (scalable)
- Flow variables (session-only)

---

## Troubleshooting Guide

### Common Issues and Solutions

**Issue 1: Authentication Failed**
```
Symptoms: 401 Unauthorized errors
Solutions:
  ✓ Verify credentials are current
  ✓ Check token expiration
  ✓ Confirm permissions are granted
  ✓ Re-authorize connection
  ✓ Check service principal access
```

**Issue 2: Timeout Errors**
```
Symptoms: 408 Request Timeout
Solutions:
  ✓ Increase timeout setting
  ✓ Reduce data payload size
  ✓ Implement pagination
  ✓ Use async patterns where available
  ✓ Check network connectivity
```

**Issue 3: Rate Limiting**
```
Symptoms: 429 Too Many Requests
Solutions:
  ✓ Implement retry with backoff
  ✓ Reduce polling frequency
  ✓ Use webhooks instead of polling
  ✓ Batch operations
  ✓ Request rate limit increase
```

**Issue 4: Data Mapping Errors**
```
Symptoms: Incorrect or missing data in target system
Solutions:
  ✓ Validate field mappings
  ✓ Check data types match
  ✓ Handle null values
  ✓ Implement data transformation
  ✓ Add validation before write
```

---

## Monitoring and Maintenance

### Connection Health Checks

**Scheduled Monitoring Flow:**
```yaml
Trigger: Recurrence (Daily at 6 AM)

Actions:
  1. Get all connections in environment
  2. For each connection:
     - Test connection
     - Check last successful use
     - Verify credentials valid
     - Check permission levels
  3. If issues found:
     - Log to monitoring list
     - Alert admin team
     - Create service ticket
```

### Usage Analytics

**Track Metrics:**
```
- Runs per connector per day
- Success rate by connector
- Average execution time
- Error frequency and types
- Peak usage times
- Cost per connector (premium)
```

**Dashboard Elements:**
- Connector health status
- Run history trends
- Error rate charts
- Performance metrics
- Cost analysis

---

## Notes

- All connector configurations shown are examples and should be customized
- Premium connectors require appropriate licensing
- Custom connector development requires API knowledge
- Security policies should be reviewed with IT/Security teams
- Monitor connector usage for optimization opportunities
- Keep connector credentials secure and rotated regularly
- Test thoroughly in non-production environments first
