# Chapter 6: Power Automate Actions - Code Examples

## Overview
This document contains all Power Automate action configurations, workflow patterns, and code examples from Chapter 6: Actions in Power Automate for Workflows.

---

## Employee Onboarding Workflow

### Complete Flow Implementation

**Description:** Comprehensive employee onboarding automation across HR, IT, and Facilities

**Business Requirements:**
- Create employee accounts
- Set up equipment and system access
- Prepare workspace
- Schedule orientation
- Track progress

---

### Step 1: Trigger Configuration

```javascript
Trigger: When an item is created (SharePoint)
  Site Address: [Your SharePoint site]
  List Name: "New Employees"
  
Trigger Output Fields:
  - FirstName
  - LastName
  - Email
  - StartDate
  - Department
  - ManagerEmail
  - JobTitle
```

---

### Step 2: Initialize Variables

**Description:** Track onboarding progress across departments

```javascript
// Variable 1: HR Tasks Status
Initialize Variable:
  Name: hrTasksComplete
  Type: Boolean
  Value: false

// Variable 2: IT Tasks Status
Initialize Variable:
  Name: itTasksComplete
  Type: Boolean
  Value: false

// Variable 3: Facilities Tasks Status
Initialize Variable:
  Name: facilitiesTasksComplete
  Type: Boolean
  Value: false

// Variable 4: Onboarding Status
Initialize Variable:
  Name: onboardingStatus
  Type: String
  Value: "In Progress"

// Variable 5: Error Messages
Initialize Variable:
  Name: errorMessages
  Type: Array
  Value: []
```

---

### Step 3: HR Scope Actions

**Description:** Handle all HR-related onboarding tasks

```javascript
Scope: HR_Onboarding

  Action 1: Create User Account
    Connector: Office 365 Users
    Action: Create user
    
    Configuration:
      User Principal Name: @{concat(
        toLower(triggerBody()?['FirstName']),
        '.',
        toLower(triggerBody()?['LastName']),
        '@company.com'
      )}
      Display Name: @{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}
      Mail Nickname: @{concat(
        toLower(triggerBody()?['FirstName']),
        toLower(triggerBody()?['LastName'])
      )}
      Account Enabled: true
      Password Profile:
        Password: @{guid()} // Temporary password
        Force Change Password: true
      Job Title: @{triggerBody()?['JobTitle']}
      Department: @{triggerBody()?['Department']}
      Manager: @{triggerBody()?['ManagerEmail']}

  Action 2: Add to Security Groups
    Connector: Azure AD
    Action: Add member to group
    
    Apply to each: @variables('requiredGroups')
      Group ID: @{item()?['groupId']}
      Member ID: @{body('Create_User')?['id']}
      
    Required Groups Array:
      - All Employees
      - Department specific group
      - Location specific group

  Action 3: Create Employee Record
    Connector: SharePoint
    Action: Create item
    
    List: Employee Directory
    Fields:
      Employee ID: @{body('Create_User')?['employeeId']}
      Full Name: @{body('Create_User')?['displayName']}
      Email: @{body('Create_User')?['userPrincipalName']}
      Department: @{triggerBody()?['Department']}
      Start Date: @{triggerBody()?['StartDate']}
      Manager: @{triggerBody()?['ManagerEmail']}
      Status: Active

  Action 4: Set HR Tasks Complete
    Set Variable:
      Name: hrTasksComplete
      Value: true

  // Error Handling
  Configure Run After (on HR Scope):
    Run after: Has failed
    
    Actions:
      Append to array:
        Name: errorMessages
        Value: "HR onboarding failed: @{body('HR_Scope')?['error']}"
```

---

### Step 4: IT Scope Actions (Parallel Branch)

**Description:** Run IT setup tasks in parallel with HR tasks

```javascript
Parallel Branch: IT_Setup

Scope: IT_Onboarding

  Action 1: Create Equipment Request
    Connector: SharePoint
    Action: Create item
    
    List: Equipment Requests
    Fields:
      Employee Name: @{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}
      Equipment Type: Standard Workstation
      Items Requested:
        - Laptop: @{triggerBody()?['LaptopModel']}
        - Monitor: Dual Display
        - Keyboard: Standard
        - Mouse: Standard
        - Headset: Yes
        - Dock: Yes
      Delivery Location: @{triggerBody()?['OfficeLocation']}
      Required By: @{triggerBody()?['StartDate']}
      Status: Pending
      Priority: @{if(
        less(
          sub(
            ticks(triggerBody()?['StartDate']),
            ticks(utcNow())
          ),
          6048000000000  // 7 days in ticks
        ),
        'High',
        'Normal'
      )}

  Action 2: Create System Access Request
    Connector: SharePoint
    Action: Create item
    
    List: Access Requests
    Fields:
      Employee: @{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}
      Email: @{body('Create_User')?['userPrincipalName']}
      Department: @{triggerBody()?['Department']}
      Access Type: New Hire
      Systems Required:
        - Email: Yes
        - VPN: Yes
        - File Shares: Department specific
        - CRM: @{if(
          or(
            equals(triggerBody()?['Department'], 'Sales'),
            equals(triggerBody()?['Department'], 'Support')
          ),
          'Yes',
          'No'
        )}
        - ERP: @{if(
          or(
            equals(triggerBody()?['Department'], 'Finance'),
            equals(triggerBody()?['Department'], 'Operations')
          ),
          'Yes',
          'No'
        )}
      Start Date: @{triggerBody()?['StartDate']}
      Manager Approval: Pending

  Action 3: Create Microsoft Teams Account
    Connector: Microsoft Teams
    Action: Add user to team
    
    Team ID: @{variables('departmentTeamId')}
    User ID: @{body('Create_User')?['id']}
    Role: Member

  Action 4: Send Welcome Email
    Connector: Office 365 Outlook
    Action: Send an email (V2)
    
    To: @{body('Create_User')?['userPrincipalName']}
    Subject: Welcome to the Company!
    Body: |
      <html>
      <body>
        <h2>Welcome @{triggerBody()?['FirstName']}!</h2>
        
        <p>We're excited to have you join our team.</p>
        
        <h3>Your Account Details:</h3>
        <ul>
          <li>Email: @{body('Create_User')?['userPrincipalName']}</li>
          <li>Start Date: @{formatDateTime(triggerBody()?['StartDate'], 'MMMM dd, yyyy')}</li>
          <li>Department: @{triggerBody()?['Department']}</li>
          <li>Manager: @{triggerBody()?['ManagerEmail']}</li>
        </ul>
        
        <h3>First Day Information:</h3>
        <p>Report to reception at 9:00 AM on your start date.</p>
        <p>Please bring a valid government-issued ID.</p>
        
        <p>Your temporary password will be sent separately.</p>
        
        <p>Best regards,<br>HR Team</p>
      </body>
      </html>
    Importance: High

  Action 5: Set IT Tasks Complete
    Set Variable:
      Name: itTasksComplete
      Value: true
```

---

### Step 5: Facilities Scope Actions

**Description:** Workspace preparation tasks

```javascript
Scope: Facilities_Setup

  Action 1: Create Workspace Request
    Connector: SharePoint
    Action: Create item
    
    List: Workspace Requests
    Fields:
      Employee Name: @{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}
      Location: @{triggerBody()?['OfficeLocation']}
      Workspace Type: @{if(
        contains(triggerBody()?['JobTitle'], 'Manager'),
        'Office',
        'Cubicle'
      )}
      Setup Requirements:
        - Desk: Standard
        - Chair: Ergonomic
        - File Cabinet: Yes
        - Whiteboard: @{if(
          contains(triggerBody()?['JobTitle'], 'Manager'),
          'Yes',
          'No'
        )}
        - Name Plate: Yes
      Required By: @{triggerBody()?['StartDate']}
      Status: Pending
      Assignment: @{triggerBody()?['WorkspaceNumber']}

  Action 2: Create Badge Request
    Connector: SharePoint
    Action: Create item
    
    List: Badge Requests
    Fields:
      Employee Name: @{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}
      Employee ID: @{body('Create_User')?['employeeId']}
      Department: @{triggerBody()?['Department']}
      Access Levels:
        - Building: Main Entrance
        - Floor: @{triggerBody()?['FloorNumber']}
        - Parking: @{if(
          triggerBody()?['ParkingRequired'],
          'Assigned Spot',
          'None'
        )}
      Badge Type: Employee
      Issue Date: @{triggerBody()?['StartDate']}
      Photo Required: Yes

  Action 3: Schedule Office Tour
    Connector: Office 365 Outlook
    Action: Create event
    
    Calendar: Facilities Calendar
    Subject: Office Tour - @{concat(
      triggerBody()?['FirstName'],
      ' ',
      triggerBody()?['LastName']
    )}
    Start Time: @{addHours(
      triggerBody()?['StartDate'],
      9.5
    )} // 9:30 AM on start date
    End Time: @{addHours(
      triggerBody()?['StartDate'],
      10
    )} // 10:00 AM
    Location: Reception
    Required Attendees: @{concat(
      body('Create_User')?['userPrincipalName'],
      ';',
      'facilities@company.com'
    )}
    Body: |
      Office tour for new employee.
      
      Employee: @{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}
      Department: @{triggerBody()?['Department']}
      Workspace: @{triggerBody()?['WorkspaceNumber']}

  Action 4: Set Facilities Tasks Complete
    Set Variable:
      Name: facilitiesTasksComplete
      Value: true
```

---

### Step 6: Condition - Check All Tasks Complete

**Description:** Verify all departments completed their tasks

```javascript
Condition: All_Tasks_Complete
  
  Expression:
    @and(
      and(
        variables('hrTasksComplete'),
        variables('itTasksComplete')
      ),
      variables('facilitiesTasksComplete')
    )

  If Yes (All Complete):
    
    Action 1: Update Onboarding Status
      Set Variable:
        Name: onboardingStatus
        Value: "Complete"
    
    Action 2: Create Completion Summary
      Compose: OnboardingSummary
      {
        "employeeName": "@{concat(
          triggerBody()?['FirstName'],
          ' ',
          triggerBody()?['LastName']
        )}",
        "employeeEmail": "@{body('Create_User')?['userPrincipalName']}",
        "startDate": "@{triggerBody()?['StartDate']}",
        "onboardingCompleted": "@{utcNow()}",
        "tasksCompleted": {
          "hrTasks": true,
          "itTasks": true,
          "facilitiesTasks": true
        },
        "accountsCreated": {
          "azureAD": "@{body('Create_User')?['id']}",
          "email": "@{body('Create_User')?['userPrincipalName']}",
          "teams": "Added"
        },
        "equipmentRequested": true,
        "workspaceAssigned": "@{triggerBody()?['WorkspaceNumber']}"
      }
    
    Action 3: Send Completion Email to Manager
      Connector: Office 365 Outlook
      Action: Send an email (V2)
      
      To: @{triggerBody()?['ManagerEmail']}
      CC: hr@company.com
      Subject: Onboarding Complete - @{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}
      Body: |
        <html>
        <body>
          <h2>Employee Onboarding Complete</h2>
          
          <p>All onboarding tasks have been completed for your new team member.</p>
          
          <h3>Employee Information:</h3>
          <table>
            <tr><td><strong>Name:</strong></td><td>@{concat(
              triggerBody()?['FirstName'],
              ' ',
              triggerBody()?['LastName']
            )}</td></tr>
            <tr><td><strong>Email:</strong></td><td>@{body('Create_User')?['userPrincipalName']}</td></tr>
            <tr><td><strong>Start Date:</strong></td><td>@{formatDateTime(
              triggerBody()?['StartDate'],
              'MMMM dd, yyyy'
            )}</td></tr>
            <tr><td><strong>Department:</strong></td><td>@{triggerBody()?['Department']}</td></tr>
            <tr><td><strong>Job Title:</strong></td><td>@{triggerBody()?['JobTitle']}</td></tr>
            <tr><td><strong>Workspace:</strong></td><td>@{triggerBody()?['WorkspaceNumber']}</td></tr>
          </table>
          
          <h3>Completed Tasks:</h3>
          <ul>
            <li>✓ User account created</li>
            <li>✓ Security groups assigned</li>
            <li>✓ Equipment requested</li>
            <li>✓ System access configured</li>
            <li>✓ Workspace prepared</li>
            <li>✓ Access badge requested</li>
            <li>✓ Welcome email sent</li>
          </ul>
          
          <p>Please reach out to your new team member to welcome them.</p>
          
          <p>Best regards,<br>HR Team</p>
        </body>
        </html>
      Importance: High
    
    Action 4: Update SharePoint Record
      Connector: SharePoint
      Action: Update item
      
      Site Address: [Your SharePoint site]
      List: New Employees
      ID: @{triggerBody()?['ID']}
      Fields:
        Onboarding Status: Complete
        Completion Date: @{utcNow()}
        HR Tasks: Complete
        IT Tasks: Complete
        Facilities Tasks: Complete
    
    Action 5: Create Teams Welcome Post
      Connector: Microsoft Teams
      Action: Post message in a chat or channel
      
      Post as: Flow bot
      Post in: Channel
      Team: @{variables('departmentTeamId')}
      Channel: General
      Message: |
        👋 Welcome to the team, @{triggerBody()?['FirstName']}!
        
        We're excited to have you join us in @{triggerBody()?['Department']}.
        
        Feel free to introduce yourself and don't hesitate to ask questions!

  If No (Incomplete):
    
    Action 1: Identify Incomplete Tasks
      Compose: IncompleteTasks
      {
        "employeeName": "@{concat(
          triggerBody()?['FirstName'],
          ' ',
          triggerBody()?['LastName']
        )}",
        "startDate": "@{triggerBody()?['StartDate']}",
        "tasksStatus": {
          "hrComplete": @{variables('hrTasksComplete')},
          "itComplete": @{variables('itTasksComplete')},
          "facilitiesComplete": @{variables('facilitiesTasksComplete')}
        },
        "errors": @{variables('errorMessages')}
      }
    
    Action 2: Send Reminder Email
      Connector: Office 365 Outlook
      Action: Send an email (V2)
      
      To: @{if(
        not(variables('hrTasksComplete')),
        'hr@company.com',
        if(
          not(variables('itTasksComplete')),
          'it@company.com',
          'facilities@company.com'
        )
      )}
      CC: onboarding@company.com
      Subject: Action Required - Incomplete Onboarding Tasks
      Body: |
        <html>
        <body>
          <h2>⚠️ Incomplete Onboarding Tasks</h2>
          
          <p>The following onboarding tasks are incomplete:</p>
          
          <ul>
            @{if(
              not(variables('hrTasksComplete')),
              '<li>❌ HR Tasks</li>',
              '<li>✓ HR Tasks</li>'
            )}
            @{if(
              not(variables('itTasksComplete')),
              '<li>❌ IT Tasks</li>',
              '<li>✓ IT Tasks</li>'
            )}
            @{if(
              not(variables('facilitiesTasksComplete')),
              '<li>❌ Facilities Tasks</li>',
              '<li>✓ Facilities Tasks</li>'
            )}
          </ul>
          
          <h3>Employee Information:</h3>
          <p>Name: @{concat(
            triggerBody()?['FirstName'],
            ' ',
            triggerBody()?['LastName']
          )}</p>
          <p>Start Date: @{formatDateTime(
            triggerBody()?['StartDate'],
            'MMMM dd, yyyy'
          )}</p>
          
          <p>Please complete these tasks before the employee's start date.</p>
        </body>
        </html>
      Importance: High
    
    Action 3: Create Follow-up Task
      Connector: Microsoft Planner
      Action: Create a task
      
      Group ID: HR Team
      Plan ID: Onboarding Plan
      Title: Complete Onboarding - @{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}
      Bucket: Pending
      Priority: @{if(
        less(
          sub(
            ticks(triggerBody()?['StartDate']),
            ticks(utcNow())
          ),
          2592000000000  // 3 days
        ),
        'Urgent',
        'Important'
      )}
      Due Date: @{addDays(utcNow(), 1)}
      Description: |
        Incomplete onboarding tasks need attention.
        
        Employee: @{concat(
          triggerBody()?['FirstName'],
          ' ',
          triggerBody()?['LastName']
        )}
        Start Date: @{formatDateTime(
          triggerBody()?['StartDate'],
          'MMMM dd, yyyy'
        )}
        
        Incomplete Tasks:
        @{if(
          not(variables('hrTasksComplete')),
          '- HR Tasks\n',
          ''
        )}@{if(
          not(variables('itTasksComplete')),
          '- IT Tasks\n',
          ''
        )}@{if(
          not(variables('facilitiesTasksComplete')),
          '- Facilities Tasks\n',
          ''
        )}
```

---

### Error Handling Configuration

**Description:** Comprehensive error handling for each scope

```javascript
// HR Scope Error Handler
Configure Run After: HR_Onboarding
  Run after conditions:
    - Has failed
    - Has timed out
  
  Actions:
    Compose: HR_Error_Details
    {
      "scope": "HR Onboarding",
      "employee": "@{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}",
      "error": "@{body('HR_Onboarding')?['error']?['message']}",
      "timestamp": "@{utcNow()}",
      "flowRun": "@{workflow().run.id}"
    }
    
    Send Email: HR_Error_Notification
      To: hr-admin@company.com; it-admin@company.com
      Subject: HR Onboarding Failed - @{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}
      Body: |
        HR onboarding tasks failed.
        
        Error Details:
        @{outputs('HR_Error_Details')}
        
        Please investigate and resolve immediately.
      Importance: High

// IT Scope Error Handler  
Configure Run After: IT_Onboarding
  Run after conditions:
    - Has failed
    - Has timed out
  
  Actions:
    Compose: IT_Error_Details
    {
      "scope": "IT Setup",
      "employee": "@{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}",
      "error": "@{body('IT_Onboarding')?['error']?['message']}",
      "timestamp": "@{utcNow()}",
      "flowRun": "@{workflow().run.id}"
    }
    
    Send Email: IT_Error_Notification
      To: it-admin@company.com
      Subject: IT Setup Failed - @{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}
      Body: |
        IT setup tasks failed.
        
        Error Details:
        @{outputs('IT_Error_Details')}
        
        Manual intervention required.

// Facilities Scope Error Handler
Configure Run After: Facilities_Setup
  Run after conditions:
    - Has failed
    - Has timed out
  
  Actions:
    Compose: Facilities_Error_Details
    {
      "scope": "Facilities Setup",
      "employee": "@{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}",
      "error": "@{body('Facilities_Setup')?['error']?['message']}",
      "timestamp": "@{utcNow()}",
      "flowRun": "@{workflow().run.id}"
    }
    
    Send Email: Facilities_Error_Notification
      To: facilities@company.com
      Subject: Facilities Setup Failed - @{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}
      Body: |
        Facilities setup tasks failed.
        
        Error Details:
        @{outputs('Facilities_Error_Details')}
```

---

## Data Operation Examples

### 1. Compose Action - Data Formatting

**Description:** Format employee data for external system

```javascript
Compose: FormattedEmployeeData
{
  "employee": {
    "id": "@{body('Create_User')?['employeeId']}",
    "name": {
      "first": "@{triggerBody()?['FirstName']}",
      "last": "@{triggerBody()?['LastName']}",
      "display": "@{concat(
        triggerBody()?['FirstName'],
        ' ',
        triggerBody()?['LastName']
      )}"
    },
    "contact": {
      "email": "@{body('Create_User')?['userPrincipalName']}",
      "phone": "@{triggerBody()?['Phone']}",
      "address": {
        "office": "@{triggerBody()?['OfficeLocation']}",
        "desk": "@{triggerBody()?['WorkspaceNumber']}"
      }
    },
    "employment": {
      "startDate": "@{formatDateTime(
        triggerBody()?['StartDate'],
        'yyyy-MM-dd'
      )}",
      "department": "@{triggerBody()?['Department']}",
      "title": "@{triggerBody()?['JobTitle']}",
      "manager": "@{triggerBody()?['ManagerEmail']}",
      "status": "Active"
    }
  },
  "metadata": {
    "createdBy": "Power Automate",
    "createdAt": "@{utcNow()}",
    "flowRun": "@{workflow().run.id}"
  }
}
```

### 2. Select Action - Data Transformation

**Description:** Transform array of employee records

```javascript
Select: TransformEmployeeRecords
  From: @body('Get_Employees')?['value']
  
  Map:
  {
    "EmployeeID": "@{item()?['employeeId']}",
    "FullName": "@{concat(
      item()?['firstName'],
      ' ',
      item()?['lastName']
    )}",
    "Email": "@{item()?['email']}",
    "HireDate": "@{formatDateTime(
      item()?['startDate'],
      'MM/dd/yyyy'
    )}",
    "Department": "@{item()?['department']}",
    "YearsOfService": "@{div(
      sub(
        ticks(utcNow()),
        ticks(item()?['startDate'])
      ),
      315360000000000
    )}"
  }
```

---

## Control Flow Examples

### 1. Condition Action - Order Processing

**Description:** Route orders based on amount and customer type

```javascript
Condition: ProcessOrder
  
  Expression:
    @and(
      greater(triggerBody()?['orderAmount'], 1000),
      equals(triggerBody()?['customerType'], 'Premium')
    )
  
  If Yes:
    // High-value premium customer order
    
    Action 1: Send to Priority Queue
      Connector: Azure Service Bus
      Action: Send message
      Queue: priority-orders
      Content: @{triggerBody()}
    
    Action 2: Notify Account Manager
      Send Email:
        To: @{triggerBody()?['accountManager']}
        Subject: High-Value Order Alert
        Body: Premium customer order over $1,000 received
    
    Action 3: Apply Discount
      Set Variable: discount
      Value: @{mul(triggerBody()?['orderAmount'], 0.10)}
  
  If No:
    // Standard order processing
    
    Action 1: Send to Standard Queue
      Connector: Azure Service Bus
      Action: Send message
      Queue: standard-orders
      Content: @{triggerBody()}
    
    Action 2: Apply Standard Discount
      Set Variable: discount
      Value: @{if(
        greater(triggerBody()?['orderAmount'], 500),
        mul(triggerBody()?['orderAmount'], 0.05),
        0
      )}
```

### 2. Switch Action - Multi-path Routing

**Description:** Route requests based on department

```javascript
Switch: RouteByDepartment
  On: @{triggerBody()?['department']}
  
  Case: "Sales"
    Action 1: Send to Sales Queue
      Create Item (SharePoint):
        List: Sales Requests
        Title: @{triggerBody()?['title']}
        Priority: @{triggerBody()?['priority']}
    
    Action 2: Notify Sales Manager
      Send Email:
        To: sales-manager@company.com
        Subject: New Sales Request
        Body: @{triggerBody()?['description']}
  
  Case: "Support"
    Action 1: Create Support Ticket
      HTTP POST:
        URI: https://support-api.company.com/tickets
        Body: @{triggerBody()}
    
    Action 2: Send Auto-reply
      Send Email:
        To: @{triggerBody()?['requesterEmail']}
        Subject: Support Ticket Created
        Body: Ticket #@{body('Create_Ticket')?['ticketId']} created
  
  Case: "IT"
    Action 1: Create IT Ticket
      Create Item (SharePoint):
        List: IT Tickets
        Category: @{triggerBody()?['category']}
        Urgency: @{triggerBody()?['urgency']}
    
    Action 2: Assign Based on Category
      Condition: @{equals(triggerBody()?['category'], 'Hardware')}
      If Yes: Assign to Hardware Team
      If No: Assign to Software Team
  
  Default:
    Action: Route to General Queue
      Send Email:
        To: admin@company.com
        Subject: Unrouted Request
        Body: Request from unknown department
```

---

## Variable Management Examples

### 1. Initialize and Track Order Status

**Description:** Track order processing state

```javascript
// Initialize variables
Initialize Variable: orderStatus
  Type: String
  Value: "Pending"

Initialize Variable: orderItems
  Type: Array
  Value: []

Initialize Variable: totalAmount
  Type: Float
  Value: 0

Initialize Variable: processedCount
  Type: Integer
  Value: 0

// Process order
Apply to each: @triggerBody()?['items']
  
  // Add to order items array
  Append to array:
    Name: orderItems
    Value: {
      "sku": "@{item()?['sku']}",
      "quantity": @{item()?['quantity']},
      "price": @{item()?['price']},
      "subtotal": @{mul(
        item()?['quantity'],
        item()?['price']
      )}
    }
  
  // Update total
  Set Variable: totalAmount
    Value: @{add(
      variables('totalAmount'),
      mul(item()?['quantity'], item()?['price'])
    )}
  
  // Increment counter
  Increment Variable: processedCount

// Update status
Set Variable: orderStatus
  Value: @{if(
    equals(variables('processedCount'), length(triggerBody()?['items'])),
    'Processed',
    'Processing'
  )}
```

### 2. Increment Variable - Batch Processing

**Description:** Process items in batches with counter

```javascript
Initialize Variable: batchSize
  Type: Integer
  Value: 100

Initialize Variable: currentBatch
  Type: Integer
  Value: 0

Initialize Variable: totalProcessed
  Type: Integer
  Value: 0

Do Until: @{equals(
  variables('totalProcessed'),
  length(body('Get_Items')?['value'])
)}
  
  // Calculate batch boundaries
  Compose: batchStart
    Value: @{mul(variables('currentBatch'), variables('batchSize'))}
  
  Compose: batchEnd
    Value: @{add(
      mul(variables('currentBatch'), variables('batchSize')),
      variables('batchSize')
    )}
  
  // Get current batch
  Compose: currentBatch