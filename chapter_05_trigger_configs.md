# Chapter 5: Trigger Configurations and Patterns

## Overview
This document contains trigger configurations, monitoring patterns, and technical examples from Chapter 5: Understanding and Setting Triggers.

---

## Trigger Configuration Examples

### PDF File Tracker Flow

**Complete Flow Configuration:**

**Trigger:** When a file is created (OneDrive for Business)
- Folder: Select monitored folder
- File type filter: PDF only

**Trigger Condition Expression:**
```javascript
@equals(triggerOutputs()?['body/Name'], '.pdf')
```

**Actions:**

**1. Get File Content**
```javascript
Connector: OneDrive for Business
Action: Get file content
File Identifier: @triggerOutputs()?['body/{Identifier}']
```

**2. Create SharePoint List Item**
```javascript
Connector: SharePoint
Action: Create item
Site Address: [Your SharePoint site]
List Name: Document Tracking

Fields:
  Title: @triggerOutputs()?['body/Name']
  FileName: @concat(
    formatDateTime(utcNow(), 'yyyyMMdd'),
    '_',
    triggerOutputs()?['body/Name']
  )
  FileSize: @triggerOutputs()?['body/Size']
  SubmissionDate: @utcNow()
  Status: "New"
```

**3. Dynamic Status Based on File Size**
```javascript
Switch(@triggerOutputs()?['body/Size'])
  Case: @less(triggerOutputs()?['body/Size'], 1000000)
    -> Status: 'Quick Processing'
  Case: @and(
    greaterOrEquals(triggerOutputs()?['body/Size'], 1000000),
    less(triggerOutputs()?['body/Size'], 5000000)
  )
    -> Status: 'Standard Processing'
  Default:
    -> Status: 'Extended Processing'
```

---

## Automated Trigger Examples

### High-Priority Email Processing

**Trigger Configuration:**
```javascript
Trigger: When a new email arrives (V3)
Folder: Inbox
Include Attachments: Yes

Advanced Condition:
@and(
  contains(triggerBody()?['importance'], 'high'),
  or(
    contains(triggerBody()?['from'], 'vip@company.com'),
    contains(triggerBody()?['subject'], 'URGENT')
  )
)
```

**Actions:**
```javascript
1. Parse Email Content
   Subject: @triggerBody()?['subject']
   From: @triggerBody()?['from']
   Body: @triggerBody()?['body']
   Importance: @triggerBody()?['importance']

2. Create Task in Planner
   Task Title: @concat('Email: ', triggerBody()?['subject'])
   Due Date: @addDays(utcNow(), 1)
   Priority: High
   Assigned To: Manager

3. Send Teams Notification
   Channel: Support Team
   Message: |
     🚨 High Priority Email Received
     
     From: @{triggerBody()?['from']}
     Subject: @{triggerBody()?['subject']}
     Received: @{formatDateTime(triggerBody()?['receivedDateTime'], 'yyyy-MM-dd HH:mm')}
     
     Action Required: Response needed within 4 hours
```

---

## Scheduled Trigger Examples

### Inventory Management System

**Recurrence Configuration:**
```javascript
Trigger: Recurrence
Frequency: Daily
Interval: 1
Time: 02:00 AM
Time Zone: (UTC-08:00) Pacific Time (US & Canada)
Start Time: 2025-01-01T02:00:00Z
```

**Workflow Logic:**
```javascript
1. Get All Products (SharePoint/SQL)
   Query: SELECT * FROM Products WHERE Active = 1

2. For Each Product:
   Current Stock: @item()?['CurrentStock']
   Minimum Stock: @item()?['MinimumStock']
   
   If @less(item()?['CurrentStock'], item()?['MinimumStock']):
     Calculate Order Quantity:
       @sub(item()?['OptimalStock'], item()?['CurrentStock'])
     
     Create Purchase Order
     Send Alert to Procurement:
       Product: @{item()?['ProductName']}
       Current: @{item()?['CurrentStock']}
       Needed: @{variables('orderQuantity')}

3. Send Daily Summary Email
   Products Below Minimum: @{variables('lowStockCount')}
   Total Orders Created: @{variables('ordersCreated')}
   Total Value: $@{variables('totalOrderValue')}
```

### Monthly Reconciliation Flow

**Recurrence Configuration:**
```javascript
Trigger: Recurrence
Frequency: Month
Interval: 1
On these days: 1 (First day of month)
Time: 06:00 AM
Time Zone: Eastern Standard Time

Advanced:
  Start Time: First day of current month
  End Time: Never
```

**Workflow:**
```javascript
1. Get Previous Month Data
   Start Date: @addMonths(utcNow(), -1, 'yyyy-MM-01')
   End Date: @formatDateTime(utcNow(), 'yyyy-MM-01')

2. Aggregate Financial Data
   SQL Query:
   ```sql
   SELECT 
     Department,
     SUM(Revenue) as TotalRevenue,
     SUM(Expenses) as TotalExpenses,
     SUM(Revenue) - SUM(Expenses) as NetIncome
   FROM Transactions
   WHERE TransactionDate >= @{variables('startDate')}
     AND TransactionDate < @{variables('endDate')}
   GROUP BY Department
   ```

3. Generate Excel Report
   Template: Monthly_Reconciliation_Template.xlsx
   Populate Data Sheets
   Apply Formatting
   Calculate Variances

4. Distribute Report
   To: Finance Team
   CC: Management
   Attachment: Monthly_Reconciliation_@{formatDateTime(addMonths(utcNow(), -1), 'yyyy-MM')}.xlsx
```

---

## Manual Trigger Examples

### Expense Reimbursement Workflow

**Trigger Configuration:**
```javascript
Trigger: Manually trigger a flow
Inputs:
  - Expense Amount (Number, Required)
  - Expense Category (Choice: Travel, Meals, Supplies, Other)
  - Receipt Attachment (File, Required)
  - Description (Text, Required)
  - Employee ID (Text, Required)
```

**Workflow with Input Validation:**
```javascript
1. Initialize Variables
   Amount: @triggerBody()?['number']
   Category: @triggerBody()?['choice']
   EmployeeID: @triggerBody()?['text_1']

2. Validate Input
   Condition: @and(
     greater(variables('Amount'), 0),
     less(variables('Amount'), 10000),
     not(empty(variables('EmployeeID')))
   )
   
   If Invalid:
     Respond to Flow:
       Status: Rejected
       Message: "Invalid input. Amount must be between $0 and $10,000"
     Terminate

3. Get Employee Details
   Filter: EmployeeID eq '@{variables('EmployeeID')}'
   Manager: @body('Get_Employee')?['Manager']
   Department: @body('Get_Employee')?['Department']

4. Determine Approval Path
   If @less(variables('Amount'), 500):
     Auto-approve
     Notify Employee: "Approved automatically"
   
   Else If @less(variables('Amount'), 2000):
     Start Approval: Manager
     
   Else:
     Start Approval: Manager + Finance Director
     Require: All must approve

5. Process Based on Approval Result
   If Approved:
     - Create accounting entry
     - Schedule payment
     - Update expense tracking
     - Notify employee
   
   If Rejected:
     - Log rejection reason
     - Notify employee
     - Archive submission
```

---

## Custom Trigger Examples

### Manufacturing Quality Control

**Custom Trigger Logic:**
```javascript
Trigger: HTTP Request (Webhook)
Method: POST
Expected Schema:
{
  "batchId": "string",
  "productLine": "string",
  "qualityScore": number,
  "defectCount": number,
  "inspectorId": "string",
  "timestamp": "datetime"
}

Trigger Condition:
@or(
  less(triggerBody()?['qualityScore'], 95),
  greater(triggerBody()?['defectCount'], 5),
  equals(triggerBody()?['productLine'], 'Critical')
)
```

**Response Logic:**
```javascript
Parse JSON:
  BatchID: @{triggerBody()?['batchId']}
  Score: @{triggerBody()?['qualityScore']}
  Defects: @{triggerBody()?['defectCount']}

Determine Severity:
  If @less(triggerBody()?['qualityScore'], 85):
    Severity: Critical
    Actions:
      - Stop production line
      - Page quality manager immediately
      - Create incident report
      - Quarantine batch
  
  Else If @less(triggerBody()?['qualityScore'], 95):
    Severity: High
    Actions:
      - Flag for review
      - Notify supervisor
      - Schedule re-inspection
  
  Else If @greater(triggerBody()?['defectCount'], 5):
    Severity: Medium
    Actions:
      - Document issues
      - Request root cause analysis
      - Monitor next 3 batches
```

---

## API Limit Management Patterns

### Connection Limits Monitoring

**SharePoint Connection Example:**
```javascript
// Standard Limits
Per connection: 600 requests per 60 seconds
Per user: 2,400 requests per minute

// Monitoring Implementation
Initialize Variable: requestCount = 0
Initialize Variable: lastResetTime = @utcNow()

For Each Item:
  // Check if reset needed
  If @greater(
    sub(ticks(utcNow()), ticks(variables('lastResetTime'))),
    600000000
  ): // 60 seconds in ticks
    Set requestCount = 0
    Set lastResetTime = @utcNow()
  
  // Check limit
  If @greaterOrEquals(variables('requestCount'), 500): // Safety margin
    Delay: 60 seconds
    Set requestCount = 0
    Set lastResetTime = @utcNow()
  
  // Perform action
  Get SharePoint Item
  Increment requestCount
```

### Batch Processing for High Volume

**Optimized Pattern:**
```javascript
// Instead of individual operations
FOR EACH item (10,000 items):
  Get item details        // 10,000 API calls
  Process item
  Update item            // 10,000 API calls
Total: 20,000 API calls

// Use batch processing
Batch Size: 100

FOR EACH batch (100 batches):
  Get items in batch (1 call, returns 100 items)
  Process items in memory
  Batch update items (1 call, updates 100 items)
Total: 200 API calls

Reduction: 99% fewer API calls
```

**Implementation:**
```javascript
Initialize: allItems = @body('Get_Items')?['value']
Initialize: batchSize = 100
Initialize: batchCount = @div(length(variables('allItems')), variables('batchSize'))

Apply to each: @range(0, variables('batchCount'))
  Set: currentBatch = @skip(variables('allItems'), mul(item(), variables('batchSize')))
  Set: currentBatch = @take(variables('currentBatch'), variables('batchSize'))
  
  // Process entire batch in memory
  Batch Update:
    Items: @variables('currentBatch')
    Operation: Update
```

### Intelligent Polling Pattern

**Optimized Polling:**
```javascript
// Inefficient: Constant polling
Recurrence: Every 5 minutes
Daily executions: 288
Monthly API calls: 8,640

// Optimized: Business hours only
Recurrence: Every 15 minutes
Days: Monday - Friday
Hours: 8:00 AM - 6:00 PM
Time Zone: Eastern Standard Time

Calculation:
  Days per month: 20-22 working days
  Hours per day: 10 hours
  Checks per hour: 4
  Monthly executions: 20 × 10 × 4 = 800

Reduction: 91% fewer API calls
```

---

## Error Handling Patterns

### Retry Logic with Exponential Backoff

**Configuration:**
```javascript
Action: HTTP Request to External API

Configure Run After: Is successful

Retry Policy:
  Type: Exponential
  Count: 4
  Interval: PT5S  // 5 seconds
  
Backoff Pattern:
  Attempt 1: Immediate
  Attempt 2: 5 seconds
  Attempt 3: 10 seconds  
  Attempt 4: 20 seconds
  Attempt 5: 40 seconds
  
Maximum Interval: PT1H  // 1 hour
Minimum Interval: PT5S  // 5 seconds
```

### Comprehensive Error Handling

**Critical Database Operation:**
```javascript
Scope: Try_Database_Operation
  ├─ Action: Execute SQL Query
  │   Query: UPDATE Orders SET Status = @{variables('newStatus')}
  │         WHERE OrderID = @{variables('orderId')}
  │   Timeout: 30 seconds
  │
  ├─ Configure Run After: Is successful
  │   └─ Action: Log Success
  │       Log Entry:
  │         Level: Information
  │         Message: Order @{variables('orderId')} updated successfully
  │         Timestamp: @{utcNow()}
  │
  └─ Configure Run After: Has failed, Has timed out
      └─ Scope: Error_Handling
          ├─ Action: Log Error
          │   Log Entry:
          │     Level: Error
          │     Message: @{actions('Execute_SQL_Query')?['error']?['message']}
          │     ErrorCode: @{actions('Execute_SQL_Query')?['error']?['code']}
          │     OrderID: @{variables('orderId')}
          │
          ├─ Condition: Check if transient error
          │   @or(
          │     contains(actions('Execute_SQL_Query')?['error']?['message'], 'timeout'),
          │     contains(actions('Execute_SQL_Query')?['error']?['message'], 'deadlock')
          │   )
          │   
          │   If Yes:
          │     Delay: 30 seconds
          │     Retry Operation
          │   
          │   If No:
          │     Send Alert to Admin:
          │       Subject: Critical Database Error
          │       Body: Non-recoverable error occurred
          │       OrderID: @{variables('orderId')}
          │     
          │     Create Support Ticket
          │
          └─ Action: Update Order Status
              Status: "Error - Pending Manual Review"

Finally (Configure Run After: Is successful, Has failed, Has timed out):
  └─ Action: Cleanup Resources
      - Close database connections
      - Clear temporary variables
      - Update audit log
```

---

## Performance Optimization Patterns

### Large Dataset Processing

**Optimized Approach:**
```javascript
Trigger: Recurrence (Off-peak hours: 2:00 AM)

1. Get Record Count
   Query: SELECT COUNT(*) as TotalRecords FROM LargeTable
   
2. Calculate Batches
   Batch Size: 1000
   Total Batches: @div(body('Get_Count')?['TotalRecords'], 1000)

3. Initialize Tracking
   Processed: 0
   Failed: 0
   StartTime: @utcNow()

4. Process in Batches
   Apply to each: @range(0, variables('TotalBatches'))
   
   Parallel Processing:
     Degree of Parallelism: 5
     
   For each batch:
     Get Records:
       Query: |
         SELECT * FROM LargeTable
         ORDER BY ID
         OFFSET @{mul(item(), 1000)} ROWS
         FETCH NEXT 1000 ROWS ONLY
     
     Process Records (in memory):
       Transform data
       Apply business rules
       Validate
     
     Batch Insert to Target:
       Single API call per 1000 records
     
     Update Progress:
       Increment Processed by batch size
       Log completion percentage

5. Performance Metrics
   Total Time: @{sub(ticks(utcNow()), ticks(variables('StartTime')))}
   Records Processed: @{variables('Processed')}
   Records Failed: @{variables('Failed')}
   Success Rate: @{div(mul(variables('Processed'), 100), body('Get_Count')?['TotalRecords'])}%
```

### Real-Time Notification Optimization

**Efficient Pattern:**
```javascript
Trigger: When an item is modified (SharePoint)
Trigger Conditions:
  @or(
    equals(triggerBody()?['Status'], 'Critical'),
    equals(triggerBody()?['Priority'], 'High')
  )

1. Initialize: notificationsSent = 0
2. Initialize: notificationCache = []

3. Check Recent Notifications
   // Prevent notification spam
   Condition: @not(
     contains(
       variables('notificationCache'),
       concat(
         triggerBody()?['ID'],
         '_',
         formatDateTime(utcNow(), 'yyyy-MM-dd-HH')
       )
     )
   )

4. If not recently notified:
   
   Parallel Branch 1: Teams Notification
     Post to channel: Alerts
     Message: Critical update on @{triggerBody()?['Title']}
     
   Parallel Branch 2: Email Notification
     To: On-call team
     Subject: Critical Item Updated
     Priority: High
     
   Parallel Branch 3: SMS via Twilio
     To: Manager phone number
     Message: Critical alert - check Teams
   
   Add to Cache:
     @{concat(
       triggerBody()?['ID'],
       '_',
       formatDateTime(utcNow(), 'yyyy-MM-dd-HH')
     )}
   
   Increment: notificationsSent

5. Cache Cleanup (hourly)
   If @equals(minute(utcNow()), 0):
     Clear cache
     Reset counter
```

---

## Custom Monitoring Solutions

### API Usage Tracking

**Implementation:**
```javascript
// At flow initialization
Initialize Variable: apiCallLog = []
Initialize Variable: totalCalls = 0

// Before each API action
Compose: Pre_API_Log
{
  "action": "@{actions('Current_Action')?['name']}",
  "timestamp": "@{utcNow()}",
  "endpoint": "@{actions('Current_Action')?['inputs']?['uri']}"
}

// After each API action
Append to array: apiCallLog
Value: @outputs('Pre_API_Log')

Increment: totalCalls

// Check threshold
If @greaterOrEquals(variables('totalCalls'), 900): // 90% of limit
  Send Alert:
    Subject: API Usage Warning
    Message: |
      Flow: @{workflow().name}
      Current Usage: @{variables('totalCalls')}
      Threshold: 1000
      Time: @{utcNow()}

// At flow completion
Create SharePoint Item:
  List: API Usage Log
  FlowName: @{workflow().name}
  RunID: @{workflow().run.id}
  TotalCalls: @{variables('totalCalls')}
  CallDetails: @{json(string(variables('apiCallLog')))}
  Duration: @{sub(ticks(utcNow()), ticks(variables('startTime')))}
```

---

## Integration Patterns

### Customer Support Case Management

**Multi-Service Integration:**
```javascript
Trigger: When a new form response is submitted (Microsoft Forms)

1. Parse Form Response
   CustomerName: @{body('Get_response_details')?['responder_name']}
   IssueType: @{body('Get_response_details')?['r_issue_type']}
   Priority: @{body('Get_response_details')?['r_priority']}
   Description: @{body('Get_response_details')?['r_description']}

2. Create SharePoint Case
   Connector: SharePoint
   Site: Support Portal
   List: Cases
   Fields:
     Title: @{variables('CustomerName')} - @{variables('IssueType')}
     Status: New
     Priority: @{variables('Priority')}
     Description: @{variables('Description')}
     CreatedDate: @{utcNow()}
     AssignedTo: (Determined by rules)

3. Store in Dataverse
   Connector: Dataverse
   Table: Cases
   Columns:
     Case Number: Auto-generated
     Customer Name: @{variables('CustomerName')}
     Case Type: @{variables('IssueType')}
     Status: Open
     Created On: @{utcNow()}

4. Notify Team via Teams
   Connector: Microsoft Teams
   Team: Support Team
   Channel: New Cases
   Message: |
     📋 **New Support Case**
     
     **Customer:** @{variables('CustomerName')}
     **Type:** @{variables('IssueType')}
     **Priority:** @{variables('Priority')}
     
     **Description:**
     @{variables('Description')}
     
     **Case ID:** @{body('Create_SP_Item')?['ID']}
     
     [View in SharePoint](@{body('Create_SP_Item')?['{Link}']})

5. Send Confirmation Email
   Connector: Office 365 Outlook
   To: @{body('Get_response_details')?['responder_email']}
   Subject: Case Confirmation - @{body('Create_SP_Item')?['ID']}
   Body: |
     Dear @{variables('CustomerName')},
     
     Your support case has been created:
     Case ID: @{body('Create_SP_Item')?['ID']}
     Priority: @{variables('Priority')}
     
     Our team will respond within:
     - High Priority: 4 hours
     - Medium Priority: 24 hours
     - Low Priority: 48 hours
```

### Sales Process Automation

**CRM Integration:**
```javascript
Trigger: When a record is created (Salesforce - Lead)

1. Enrich Lead Data
   Connector: LinkedIn Sales Navigator API
   Search: @{triggerBody()?['Name']} at @{triggerBody()?['Company']}
   
   Extract:
     Job Title: @{body('LinkedIn_Search')?['title']}
     Company Size: @{body('LinkedIn_Search')?['companySize']}
     Industry: @{body('LinkedIn_Search')?['industry']}

2. Score Lead
   Calculate Lead Score:
     @add(
       if(equals(body('LinkedIn_Search')?['seniorityLevel'], 'Executive'), 30, 0),
       if(greater(body('LinkedIn_Search')?['companySize'], 500), 20, 0),
       if(contains(triggerBody()?['Source'], 'Referral'), 25, 0),
       if(greater(triggerBody()?['EstimatedValue'], 50000), 25, 0)
     )

3. Create Dynamics 365 Opportunity
   If @greater(variables('leadScore'), 50):
     Connector: Dynamics 365
     Action: Create record
     Entity: Opportunity
     Fields:
       Name: @{triggerBody()?['Company']} Opportunity
       Account: Link to company
       Contact: Link to lead
       Estimated Value: @{triggerBody()?['EstimatedValue']}
       Probability: @{div(variables('leadScore'), 100)}
       Expected Close Date: @{addMonths(utcNow(), 3)}

4. Assign to Sales Rep
   Get Territory: Based on company location
   Get Rep: From territory mapping
   
   Update Salesforce Lead:
     Owner: @{body('Get_Rep')?['SalesforceID']}
     Status: Qualified
     Lead Score: @{variables('leadScore')}

5. Schedule Follow-up
   Connector: Outlook
   Action: Create event
   Subject: Initial Call - @{triggerBody()?['Company']}
   Start Time: Next available slot in rep's calendar
   Duration: 30 minutes
   Attendees:
     - @{body('Get_Rep')?['Email']}
     - @{triggerBody()?['Email']}
```

---

## Notes

- All expressions use Power Automate syntax
- Replace placeholder values with actual environment-specific data
- Test thoroughly in development before production deployment
- Monitor API consumption to stay within limits
- Document all custom configurations
- Keep trigger logic as simple as possible for maintainability
- Use environment variables for configuration values
- Implement proper error handling and logging
- Regular review and optimization of trigger performance
