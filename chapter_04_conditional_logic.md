# Chapter 4: Conditional Logic Patterns and Code Examples

## Overview
This document contains conditional logic patterns, expressions, and pseudo-code examples from Chapter 4: Working with Conditional Logic.

---

## Basic Conditional Logic Patterns

### 1. Simple If-Then-Else Structure

**Pattern:**
```
IF condition is met
  THEN perform action A
  ELSE perform action B
```

**Example: Document Approval**
```
Trigger: New document uploaded

IF document size > 5MB
  THEN send to manager for approval
  ELSE auto-approve and notify team
```

---

## Expression Examples

### Date Comparison Expressions

**Check if due date has passed:**
```javascript
@less(item()?['DueDate'], utcNow())
```

**Check if date is within 7 days:**
```javascript
@less(
  sub(
    ticks(item()?['DueDate']),
    ticks(utcNow())
  ),
  6048000000000
)
```

### String Evaluation Expressions

**Check email content and length:**
```javascript
@and(
  contains(triggerBody()?['Subject'], 'urgent'),
  greater(length(triggerBody()?['Body']), 100)
)
```

**Multiple string conditions:**
```javascript
@or(
  contains(item()?['Status'], 'pending'),
  contains(item()?['Status'], 'review'),
  contains(item()?['Status'], 'waiting')
)
```

---

## Error Handling Patterns

### Try-Catch-Finally Pattern

**Complete Error Handling Workflow:**
```
TRY:
  Process file
  
  IF processing successful:
    Update record status to "Complete"
    Send success notification
  ELSE:
    Log error details
    Send failure notification
    
CATCH (unexpected errors):
  Log exception
  Alert administrator
  
FINALLY:
  Cleanup temporary files
  Update audit log
```

**Power Automate Implementation Structure:**
```
Main Action: Process File
├─ Configure run after: is successful
│  └─ Update status to Complete
│  └─ Send success notification
│
├─ Configure run after: has failed
│  └─ Log error
│  └─ Send failure notification
│
└─ Configure run after: is successful, has failed, has timed out
   └─ Cleanup action (always runs)
```

### Condition-Based Error Handling

**HTTP Status Code Handling:**
```javascript
Switch on: @outputs('HTTP_Request')?['statusCode']

Case 200 (Success):
  - Process response data
  - Update records
  - Send confirmation

Case 404 (Not Found):
  - Log "Resource not found"
  - Retry with alternative endpoint
  - Notify user

Case 500+ (Server Error):
  - Log error details
  - Implement retry with backoff
  - Alert system admin

Default:
  - Generic error handling
  - Log unexpected status
  - Send alert
```

**Expression for Status Code Check:**
```javascript
@equals(outputs('HTTP_Request')?['statusCode'], 200)
```

---

## Real-World Application Patterns

### Document Management System

**Nested Conditional Logic:**
```
Trigger: New document uploaded

IF document type = "Contract":
  
  IF contract value > $100,000:
    Route to: Legal Department
    Priority: High
    Approval: Legal team + CFO
  
  ELSE IF contract value > $10,000:
    Route to: Department Manager
    Priority: Medium
    Approval: Manager
  
  ELSE:
    Route to: Team Leader
    Priority: Low
    Approval: Team Leader

ELSE IF document type = "Invoice":
  Route to: Accounting
  
ELSE:
  Route to: General inbox
```

**Power Automate Expression:**
```javascript
@if(
  equals(triggerBody()?['DocumentType'], 'Contract'),
  if(
    greater(triggerBody()?['Value'], 100000),
    'Legal Department',
    if(
      greater(triggerBody()?['Value'], 10000),
      'Department Manager',
      'Team Leader'
    )
  ),
  'General Inbox'
)
```

### Customer Service Automation

**Priority-Based Routing:**
```
Trigger: New support ticket

IF customer tier = "Premium":
  
  IF issue type = "System Outage":
    - Create P1 incident
    - Notify support manager immediately
    - Start SLA timer (15 min response)
    - Assign to: Senior engineer on-call
  
  ELSE:
    - Create P2 ticket
    - Assign to: Premium support team
    - Start SLA timer (1 hour response)

ELSE:
  - Create standard ticket
  - Route to: General support queue
  - Standard SLA (4 hours)
```

**Expression for Customer Tier Check:**
```javascript
@and(
  equals(triggerBody()?['CustomerTier'], 'Premium'),
  equals(triggerBody()?['IssueType'], 'System Outage')
)
```

---

## Performance Optimization Patterns

### Condition Ordering for Performance

**Inefficient (checks rare cases first):**
```
IF rare_case:
  handle_rare
ELSE IF special_case:
  handle_special
ELSE IF common_case:
  handle_common
```

**Optimized (checks common cases first):**
```
IF common_case:
  handle_common
ELSE IF special_case:
  handle_special  
ELSE IF rare_case:
  handle_rare
```

**Performance Impact:**
```
Inefficient approach:
- 100 executions
- Rare case: 5 times (5% hit first check)
- Special case: 15 times (15% check 2 conditions)
- Common case: 80 times (80% check 3 conditions)
- Total condition checks: 275

Optimized approach:
- 100 executions
- Common case: 80 times (80% hit first check)
- Special case: 15 times (15% check 2 conditions)
- Rare case: 5 times (5% check 3 conditions)
- Total condition checks: 135

Reduction: 51% fewer condition checks
```

### Reducing Complexity with Compound Conditions

**Complex Nested Structure (Avoid):**
```
IF condition1:
  IF condition2:
    IF condition3:
      action_A
    ELSE:
      action_B
  ELSE:
    action_C
ELSE:
  action_D
```

**Simplified with Compound Operators:**
```
IF condition1 AND condition2 AND condition3:
  action_A
ELSE IF condition1 AND condition2:
  action_B
ELSE IF condition1:
  action_C
ELSE:
  action_D
```

**Power Automate Expression:**
```javascript
@and(
  equals(item()?['Status'], 'Active'),
  greater(item()?['Amount'], 1000),
  less(item()?['DueDate'], addDays(utcNow(), 7))
)
```

### Switch Statement Pattern

**Instead of multiple IF-ELSE:**
```
IF status = "New":
  handle_new
ELSE IF status = "InProgress":
  handle_in_progress
ELSE IF status = "Review":
  handle_review
ELSE IF status = "Complete":
  handle_complete
```

**Use Switch:**
```
SWITCH status:
  CASE "New":
    handle_new
  CASE "InProgress":
    handle_in_progress
  CASE "Review":
    handle_review
  CASE "Complete":
    handle_complete
  DEFAULT:
    handle_unknown
```

---

## API Consumption Optimization

### Inefficient Pattern (Avoid)

**Nested Loops - High API Usage:**
```
For each customer (100 customers):
  Get customer details           // 100 API calls
  Get customer orders            // 100 API calls
  Get order items               // 100 API calls
  Update customer record        // 100 API calls

Total: 400 API calls for 100 customers
```

**Expression showing the problem:**
```javascript
Apply to each: @body('Get_customers')?['value']
  ├─ Get customer details: @item()?['customerId']
  ├─ Get customer orders: @item()?['customerId']
  ├─ Get order items: @body('Get_orders')?['orderId']
  └─ Update customer: @item()?['customerId']
```

### Optimized Pattern

**Batch Processing - Reduced API Usage:**
```
Get all customers (1 API call)
Get all orders (1 API call)
Get all items (1 API call)

Process in memory:
  For each customer:
    Filter orders for this customer
    Filter items for these orders
    Calculate totals
    
Batch update customers (1 API call for all)

Total: 4 API calls for 100 customers
Reduction: 99% fewer API calls
```

**Implementation:**
```javascript
// Get all data first
@body('Get_all_customers')?['value']
@body('Get_all_orders')?['value']

// Filter in memory
@filter(
  body('Get_all_orders')?['value'],
  equals(item()?['customerId'], variables('currentCustomerId'))
)

// Batch update
Array of updates: [
  { "customerId": "1", "totalOrders": 10 },
  { "customerId": "2", "totalOrders": 5 }
]
```

### Redundant Data Retrieval (Avoid)

**Inefficient:**
```
For each item in list:
  Get lookup data (same data retrieved 100 times)
  Process with lookup data
```

**Optimized with Caching:**
```
Get lookup data once
Store in variable

For each item in list:
  Use cached lookup data
  Process with lookup data
```

**Expression:**
```javascript
// Initialize cache
@{outputs('Get_lookup_data')?['body']?['value']}

// Use in loop
@variables('lookupCache')
```

### Intelligent Polling

**Inefficient (Constant Polling):**
```
Every 5 minutes:
  Check for new items
  Process items
  
Daily API calls: 288 (24 hours × 12 checks per hour)
```

**Optimized (Webhook + Fallback):**
```
Primary: Use webhook trigger (0 polling API calls)

Fallback: Poll only during business hours
  Monday-Friday, 8 AM - 6 PM
  Every 30 minutes
  
Daily API calls: 20 (5 days × 10 hours × 2 checks per hour ÷ 5)
Reduction: 93% fewer API calls
```

---

## Monitoring and Management

### Usage Tracking Implementation

**Track API Calls:**
```javascript
// Initialize counter at flow start
Initialize variable: apiCallCount = 0

// Increment after each API action
Set variable: apiCallCount = @add(variables('apiCallCount'), 1)

// Log at flow end
Compose: {
  "flowName": "@{workflow().name}",
  "runId": "@{workflow().run.id}",
  "totalAPICalls": "@{variables('apiCallCount')}",
  "timestamp": "@{utcNow()}"
}
```

### Alert Configuration

**API Limit Alert:**
```javascript
IF @greater(variables('apiCallCount'), 1000):
  Send alert: {
    "subject": "API Usage Alert",
    "message": "Flow @{workflow().name} exceeded 1000 API calls",
    "apiCount": "@{variables('apiCallCount')}",
    "threshold": 1000
  }
```

---

## Cost-Effective Design Patterns

### Intelligent Batching

**Process in Batches:**
```javascript
// Initialize
@{body('Get_items')?['value']}

// Chunk into batches of 100
@chunk(variables('allItems'), 100)

// Process each batch
Apply to each: @variables('batches')
  Process batch of 100 items
  Single API call per batch
```

### Conditional Execution

**Skip Unnecessary Processing:**
```javascript
// Check if processing needed
@and(
  greater(length(body('Get_items')?['value']), 0),
  equals(triggerBody()?['status'], 'ready')
)

// Only process if condition met
IF condition is true:
  Perform expensive operations
ELSE:
  Skip and log
```

---

## Compose Action Patterns

### Basic Compose for Debugging

**Capture Values:**
```javascript
// Compose action: Debug_Values
{
  "triggerType": "@{triggerBody()?['type']}",
  "itemCount": "@{length(body('Get_items')?['value'])}",
  "timestamp": "@{utcNow()}",
  "runId": "@{workflow().run.id}"
}
```

### Complex Expression Testing

**Test Multiple Conditions:**
```javascript
// Compose action: Condition_Test
{
  "condition1": "@{equals(variables('status'), 'active')}",
  "condition2": "@{greater(variables('amount'), 1000)}",
  "combinedResult": "@{and(
    equals(variables('status'), 'active'),
    greater(variables('amount'), 1000)
  )}",
  "evaluationTime": "@{utcNow()}"
}
```

### Data Transformation Checkpoints

**Track Transformations:**
```javascript
// Compose: Original_Data
@{triggerBody()}

// Compose: After_Filter
@{filter(body('Get_items'), item => equals(item.status, 'active'))}

// Compose: After_Transform
@{select(
  variables('filteredItems'),
  {
    'id': item.id,
    'name': item.name,
    'total': mul(item.quantity, item.price)
  }
)}
```

### Array Manipulation Verification

**Compose for Array Operations:**
```javascript
{
  "originalArray": "@{variables('inputArray')}",
  "filteredArray": "@{filter(
    variables('inputArray'),
    item => equals(item.status, 'active')
  )}",
  "arrayLength": "@{length(variables('inputArray'))}",
  "filteredLength": "@{length(variables('filteredArray'))}"
}
```

---

## Variable Management Patterns

### Initialize Variable for Complex Objects

**Complex Data Structure:**
```javascript
// Initialize variable: ProcessingState
{
  "startTime": "@{utcNow()}",
  "itemsProcessed": 0,
  "errors": [],
  "status": "running",
  "metadata": {
    "flowName": "@{workflow().name}",
    "runId": "@{workflow().run.id}"
  }
}
```

### Set Variable with Transformed Data

**Update Complex Variable:**
```javascript
// Set variable: ProcessingState
@{
  setProperty(
    variables('ProcessingState'),
    'itemsProcessed',
    add(
      variables('ProcessingState')?['itemsProcessed'],
      1
    )
  )
}
```

---

## Parse JSON Pattern

**Schema Definition and Usage:**
```javascript
// Parse JSON action
Input: @{body('HTTP_Request')}

Schema: {
  "type": "object",
  "properties": {
    "id": { "type": "string" },
    "name": { "type": "string" },
    "amount": { "type": "number" },
    "items": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "itemId": { "type": "string" },
          "quantity": { "type": "number" }
        }
      }
    }
  }
}

// Use parsed data
@{body('Parse_JSON')?['name']}
@{body('Parse_JSON')?['items'][0]?['quantity']}
```

---

## Debugging Strategies

### Flow Path Tracking

**Track Execution Path:**
```javascript
// Compose: Path_Checkpoint_1
{
  "checkpoint": "Before condition evaluation",
  "variables": {
    "status": "@{variables('status')}",
    "count": "@{variables('count')}"
  },
  "timestamp": "@{utcNow()}"
}

// After condition
// Compose: Path_Checkpoint_2
{
  "checkpoint": "After condition - Yes branch",
  "conditionResult": true,
  "nextAction": "Process items"
}
```

### Error Condition Monitoring

**Monitor Error States:**
```javascript
// Compose: Error_Check
{
  "actionStatus": "@{actions('Previous_Action')?['status']}",
  "errorMessage": "@{actions('Previous_Action')?['error']?['message']}",
  "errorCode": "@{actions('Previous_Action')?['error']?['code']}",
  "outputData": "@{outputs('Previous_Action')}",
  "timestamp": "@{utcNow()}"
}
```

---

## Best Practice Naming Conventions

**Descriptive Compose Names:**
```javascript
// Good naming
Compose: Debug_CustomerData_BeforeTransform
Compose: Verify_OrderTotal_Calculation
Compose: Log_ErrorState_AfterAPI

// Poor naming (avoid)
Compose: Compose_1
Compose: Data
Compose: Test
```

---

## Testing Scenarios

### Comprehensive Test Cases

**Test Case Structure:**
```
Test Scenario 1: Happy Path
  Input: Valid customer data
  Expected: Successful processing
  Verify: All conditions met
  
Test Scenario 2: Edge Case - Boundary Values
  Input: Amount exactly at threshold
  Expected: Correct routing
  Verify: Boundary condition handling
  
Test Scenario 3: Error Case - Invalid Data
  Input: Missing required fields
  Expected: Error handling triggered
  Verify: Graceful failure, logging
  
Test Scenario 4: Performance Case - High Volume
  Input: 1000+ items
  Expected: Completes within SLA
  Verify: No timeouts, proper batching
```

---

## Complete Optimization Example

### Before Optimization

**Inefficient Order Processing:**
```
API Calls: 1,500 per day

FOR EACH order (500 orders):
  Get customer data        // 500 calls
  Get product details      // 500 calls  
  Calculate total
  Update order             // 500 calls
```

### After Optimization

**Optimized Order Processing:**
```
API Calls: 50 per day (97% reduction)

Batch 1: Get all customers (1 call)
Batch 2: Get all products (1 call)

FOR EACH order batch (10 batches of 50):
  Calculate using cached data
  Batch update orders (10 calls)
  
Scheduled: Off-peak hours
Intelligent polling: Business hours only
```

**Performance Metrics:**
```
Before:
- API calls: 1,500/day
- Execution time: 45 minutes
- Cost: $150/month

After:
- API calls: 50/day
- Execution time: 5 minutes
- Cost: $5/month

Improvements:
- 97% fewer API calls
- 89% faster execution
- 97% cost reduction
```

---

## Notes

- All expressions use Power Automate expression syntax
- Patterns shown are templates - customize for your scenarios
- Always test conditional logic thoroughly before production
- Monitor API consumption to stay within limits
- Use Compose actions liberally during development
- Remove unnecessary Compose actions in production for performance
