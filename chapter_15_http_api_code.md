# Chapter 15: HTTP Requests and API Integration - Code Examples

## Overview
This document contains all HTTP request configurations, JSON schemas, API patterns, and code examples from Chapter 15: HTTP Requests and API Integration.

---

## HTTP Request Configurations

### 1. Basic GET Request - E-commerce API

**Description:** Retrieve products from an e-commerce API

**HTTP Configuration:**
```
Method: GET
URI: https://api.example-store.com/products

Headers:
  Content-Type: application/json
  Accept: application/json
```

**Power Automate Expression:**
```javascript
// Access response
@body('HTTP_request')?['products']
```

---

### 2. GET Request with Query Parameters

**Description:** Retrieve filtered and sorted products

**HTTP Configuration:**
```
Method: GET
URI: https://api.example-store.com/products

Query Parameters:
  category: electronics
  sort: price
  order: asc
```

**Full URI:**
```
https://api.example-store.com/products?category=electronics&sort=price&order=asc
```

**Power Automate Dynamic Expression:**
```javascript
@{concat(
  'https://api.example-store.com/products?',
  'category=', variables('categoryFilter'),
  '&sort=', variables('sortField'),
  '&order=', variables('sortOrder')
)}
```

---

### 3. POST Request - Create Product

**Description:** Create a new product in the system

**HTTP Configuration:**
```
Method: POST
URI: https://api.example-store.com/products

Headers:
  Content-Type: application/json
  Authorization: Bearer {token}

Body:
{
  "name": "Wireless Mouse",
  "description": "Ergonomic wireless mouse with precision tracking",
  "price": 29.99,
  "category": "electronics",
  "inStock": true,
  "quantity": 150
}
```

**Dynamic Body with Form Data:**
```json
{
  "name": "@{triggerBody()?['productName']}",
  "description": "@{triggerBody()?['productDescription']}",
  "price": @{float(triggerBody()?['productPrice'])},
  "category": "@{triggerBody()?['productCategory']}",
  "inStock": true,
  "quantity": @{int(triggerBody()?['productQuantity'])}
}
```

---

### 4. Processing API Response with For Each Loop

**Description:** Iterate through product array from API response

**Power Automate Configuration:**
```javascript
// For Each Loop Input
@body('Get_Products')?['products']

// Inside Loop - Access Properties
Product Name: @{items('Apply_to_each')?['name']}
Product Price: @{items('Apply_to_each')?['price']}
Stock Level: @{items('Apply_to_each')?['stock']}
```

**Access Specific Item:**
```javascript
// Get first product
@body('HTTP_request')?['products'][0]

// Get first product name
@body('HTTP_request')?['products'][0]?['name']
```

---

## Authentication Patterns

### 1. API Key in Header

**Description:** Include API key in request header

**HTTP Configuration:**
```
Headers:
  X-API-Key: your_api_key_here
  Content-Type: application/json
```

**Dynamic API Key from Variable:**
```
Headers:
  X-API-Key: @{variables('apiKey')}
  Content-Type: application/json
```

---

### 2. API Key in Query Parameter

**Description:** Pass API key as URL parameter

**URI:**
```
https://api.example.com/data?apikey=@{variables('apiKey')}&format=json
```

---

### 3. Bearer Token Authentication

**Description:** Use bearer token for authentication

**HTTP Configuration:**
```
Headers:
  Authorization: Bearer @{variables('accessToken')}
  Content-Type: application/json
```

**Complete OAuth Flow:**
```javascript
// Step 1: Obtain Token
Method: POST
URI: https://auth.example.com/oauth/token

Body:
{
  "grant_type": "client_credentials",
  "client_id": "@{variables('clientId')}",
  "client_secret": "@{variables('clientSecret')}",
  "scope": "read write"
}

// Step 2: Extract Token
@body('Get_Token')?['access_token']

// Step 3: Use Token in Subsequent Requests
Headers:
  Authorization: Bearer @{body('Get_Token')?['access_token']}
```

---

### 4. Basic Authentication

**Description:** Use username and password for authentication

**HTTP Configuration:**
```
Authentication Type: Basic
Username: your_username
Password: your_password
```

**Manual Header Construction:**
```javascript
// Base64 encode credentials
Headers:
  Authorization: Basic @{base64(concat(variables('username'), ':', variables('password')))}
```

---

### 5. OAuth 2.0 Configuration

**Description:** Complete OAuth 2.0 setup

**Configuration:**
```
Authentication Type: OAuth 2.0

Authority: https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token
Client ID: {your-client-id}
Client Secret: {your-client-secret}
Scope: https://graph.microsoft.com/.default
Grant Type: Client Credentials
```

**Token Request Body:**
```
Content-Type: application/x-www-form-urlencoded

Body:
grant_type=client_credentials
&client_id={client-id}
&client_secret={client-secret}
&scope=https://graph.microsoft.com/.default
```

---

## Response Handling Patterns

### 1. Status Code Checking

**Description:** Verify request success based on status code

**Expression:**
```javascript
// Check if successful (2xx range)
@and(
  greaterOrEquals(outputs('HTTP_action')?['statusCode'], 200),
  less(outputs('HTTP_action')?['statusCode'], 300)
)

// Check specific status
@equals(outputs('HTTP_action')?['statusCode'], 200)

// Check for creation
@equals(outputs('HTTP_action')?['statusCode'], 201)
```

**Condition Logic:**
```javascript
Condition: 
  @and(
    outputs('HTTP_action')?['statusCode'] >= 200,
    outputs('HTTP_action')?['statusCode'] < 300
  )

If Yes:
  // Process successful response
  @body('HTTP_action')

If No:
  // Handle error
  @body('HTTP_action')?['error']
```

---

### 2. Extract Error Details

**Description:** Parse error information from failed requests

**Error Response Example:**
```json
{
  "error": {
    "code": "InvalidRequest",
    "message": "The request body is missing required fields",
    "details": [
      {
        "field": "email",
        "message": "Email address is required"
      },
      {
        "field": "name",
        "message": "Name must be at least 2 characters"
      }
    ],
    "timestamp": "2023-09-15T14:22:15Z",
    "requestId": "abc-123-def-456"
  }
}
```

**Extract Error Information:**
```javascript
// Error code
@body('HTTP_action')?['error']?['code']

// Error message
@body('HTTP_action')?['error']?['message']

// Specific field error
@body('HTTP_action')?['error']?['details'][0]?['message']

// Request ID for support
@body('HTTP_action')?['error']?['requestId']
```

---

### 3. Retry Logic with Do Until Loop

**Description:** Implement custom retry mechanism

**Complete Pattern:**
```javascript
// Initialize retry counter
Initialize Variable: attempts
Type: Integer
Value: 0

// Do Until Configuration
Do Until:
  @or(
    greater(variables('attempts'), 3),
    and(
      greaterOrEquals(outputs('HTTP_Request')?['statusCode'], 200),
      less(outputs('HTTP_Request')?['statusCode'], 300)
    )
  )

Actions Inside Loop:
  1. Increment Variable: attempts
     Value: @add(variables('attempts'), 1)
  
  2. HTTP Request
     [Your API call configuration]
  
  3. Condition: Check if failed
     @or(
       less(outputs('HTTP_Request')?['statusCode'], 200),
       greaterOrEquals(outputs('HTTP_Request')?['statusCode'], 300)
     )
     
     If Yes:
       Delay:
         Interval: @{mul(variables('attempts'), 30)}
         Unit: Second
         
         // Exponential backoff calculation
         Delay: @{mul(power(2, variables('attempts')), 1000)} milliseconds
```

---

## Practical Implementation Examples

### Example 1: Daily Inventory Report

**Description:** Automated daily product inventory report

**Complete Flow:**
```javascript
Trigger: Recurrence
  Frequency: Daily
  Time: 08:00 AM

Action 1: HTTP Request
  Method: GET
  URI: https://api.example-store.com/products
  Headers:
    X-API-Key: @{variables('apiKey')}
    Accept: application/json

Action 2: Parse JSON
  Content: @body('HTTP_Request')
  Schema: {
    "type": "object",
    "properties": {
      "products": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "id": {"type": "string"},
            "name": {"type": "string"},
            "stock": {"type": "integer"},
            "reorderLevel": {"type": "integer"}
          }
        }
      }
    }
  }

Action 3: Filter Array
  From: @body('Parse_JSON')?['products']
  Where: @less(item()?['stock'], item()?['reorderLevel'])

Action 4: Create HTML Table
  From: @body('Filter_array')
  Columns: Automatic

Action 5: Send Email
  To: inventory@company.com
  Subject: Low Stock Alert - @{formatDateTime(utcNow(), 'yyyy-MM-dd')}
  Body: |
    Low Stock Items Report
    
    The following items are below reorder levels:
    
    @{body('Create_HTML_table')}
```

---

### Example 2: Support Ticket Creation

**Description:** Create support ticket from form submission

**Complete Flow:**
```javascript
Trigger: When a new response is submitted (Microsoft Forms)

Action 1: Get Response Details
  Response ID: @triggerBody()?['resourceData']?['responseId']

Action 2: HTTP Request - Create Ticket
  Method: POST
  URI: https://support-api.company.com/tickets
  Headers:
    Authorization: Bearer @{variables('supportAPIToken')}
    Content-Type: application/json
  
  Body:
  {
    "title": "@{body('Get_response_details')?['r_subject']}",
    "description": "@{body('Get_response_details')?['r_description']}",
    "priority": "@{body('Get_response_details')?['r_priority']}",
    "category": "@{body('Get_response_details')?['r_category']}",
    "requestedBy": {
      "name": "@{body('Get_response_details')?['responder_name']}",
      "email": "@{body('Get_response_details')?['responder_email']}"
    },
    "customFields": {
      "source": "Microsoft Forms",
      "formId": "@{triggerBody()?['resourceData']?['formId']}",
      "submissionDate": "@{utcNow()}"
    }
  }

Action 3: Condition - Check Success
  @equals(outputs('HTTP_Request')?['statusCode'], 201)
  
  If Yes:
    Parse JSON: @body('HTTP_Request')
    
    Send Email - Confirmation:
      To: @{body('Get_response_details')?['responder_email']}
      Subject: Ticket Created - #@{body('Parse_JSON')?['ticketId']}
      Body: |
        Your support ticket has been created successfully.
        
        Ticket ID: @{body('Parse_JSON')?['ticketId']}
        Subject: @{body('Get_response_details')?['r_subject']}
        Priority: @{body('Get_response_details')?['r_priority']}
        
        Expected Response Time: @{body('Parse_JSON')?['sla']?['responseTime']}
        
        Track your ticket: @{body('Parse_JSON')?['trackingUrl']}
  
  If No:
    Send Email - Error:
      To: support-admin@company.com
      Subject: Failed to Create Ticket
      Body: |
        Failed to create support ticket from form submission.
        
        Status Code: @{outputs('HTTP_Request')?['statusCode']}
        Error: @{body('HTTP_Request')?['error']?['message']}
        
        Form Response ID: @{triggerBody()?['resourceData']?['responseId']}
```

---

### Example 3: Pagination Implementation

**Description:** Retrieve all customers using pagination

**Complete Flow:**
```javascript
Action 1: Initialize Variables
  pageNumber:
    Type: Integer
    Value: 1
  
  hasMorePages:
    Type: Boolean
    Value: true
  
  allCustomers:
    Type: Array
    Value: []
  
  pageSize:
    Type: Integer
    Value: 100

Action 2: Do Until Loop
  Condition: @equals(variables('hasMorePages'), false)
  
  Actions Inside:
    1. HTTP Request - Get Page
       Method: GET
       URI: @{concat(
         'https://api.company.com/customers?',
         'page=', variables('pageNumber'),
         '&pageSize=', variables('pageSize')
       )}
       Headers:
         Authorization: Bearer @{variables('apiToken')}
    
    2. Parse JSON
       Content: @body('HTTP_Request')
       Schema: {
         "type": "object",
         "properties": {
           "customers": {"type": "array"},
           "pagination": {
             "type": "object",
             "properties": {
               "currentPage": {"type": "integer"},
               "totalPages": {"type": "integer"},
               "hasNextPage": {"type": "boolean"}
             }
           }
         }
       }
    
    3. Append to Array
       Name: allCustomers
       Value: @concat(
         variables('allCustomers'),
         body('Parse_JSON')?['customers']
       )
    
    4. Set Variable - hasMorePages
       Name: hasMorePages
       Value: @body('Parse_JSON')?['pagination']?['hasNextPage']
    
    5. Condition - Check if more pages
       @equals(body('Parse_JSON')?['pagination']?['hasNextPage'], true)
       
       If Yes:
         Set Variable - pageNumber
           Value: @add(variables('pageNumber'), 1)

Action 3: Apply to Each Customer
  From: @variables('allCustomers')
  
  Actions:
    Create Item (SharePoint):
      Customer ID: @{items('Apply_to_each')?['id']}
      Name: @{items('Apply_to_each')?['name']}
      Email: @{items('Apply_to_each')?['email']}
      Company: @{items('Apply_to_each')?['company']}
      Imported: @{utcNow()}
```

---

### Example 4: Webhook Receiver

**Description:** Receive and process inventory updates via webhook

**Complete Flow:**
```javascript
Trigger: When a HTTP request is received
  Method: POST
  Relative Path: /inventory-updates
  
  Request Body JSON Schema:
  {
    "type": "object",
    "properties": {
      "eventType": {"type": "string"},
      "timestamp": {"type": "string"},
      "supplierId": {"type": "string"},
      "products": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "sku": {"type": "string"},
            "name": {"type": "string"},
            "quantity": {"type": "integer"},
            "price": {"type": "number"},
            "availableDate": {"type": "string"}
          },
          "required": ["sku", "quantity"]
        }
      }
    },
    "required": ["eventType", "products"]
  }

Action 1: Parse JSON
  Content: @triggerBody()

Action 2: Apply to Each Product
  From: @body('Parse_JSON')?['products']
  
  Actions:
    1. HTTP Request - Update Inventory
       Method: PATCH
       URI: https://inventory.company.com/api/products/@{items('Apply_to_each')?['sku']}
       Headers:
         Authorization: Bearer @{variables('inventoryAPIToken')}
         Content-Type: application/json
       Body:
       {
         "quantity": @{items('Apply_to_each')?['quantity']},
         "price": @{items('Apply_to_each')?['price']},
         "lastUpdated": "@{utcNow()}",
         "supplier": "@{body('Parse_JSON')?['supplierId']}"
       }
    
    2. Condition - Check Stock Level
       @less(items('Apply_to_each')?['quantity'], 10)
       
       If Yes:
         Send Email Alert:
           To: purchasing@company.com
           Subject: Low Stock Alert - @{items('Apply_to_each')?['name']}
           Body: |
             Low stock detected:
             
             Product: @{items('Apply_to_each')?['name']}
             SKU: @{items('Apply_to_each')?['sku']}
             Current Quantity: @{items('Apply_to_each')?['quantity']}
             Supplier: @{body('Parse_JSON')?['supplierId']}

Action 3: Response
  Status Code: 200
  Headers:
    Content-Type: application/json
  Body:
  {
    "status": "success",
    "message": "Inventory updated successfully",
    "productsProcessed": @{length(body('Parse_JSON')?['products'])},
    "timestamp": "@{utcNow()}"
  }
```

---

## Real-World Challenge Scenarios

### Scenario 1: Multi-System Data Synchronization

**Description:** Synchronize product data across ERP, e-commerce, and warehouse systems

**ERP Data Extraction:**
```javascript
Method: GET
URI: https://erp-api.company.com/products/changes

Query Parameters:
  lastSync: @{variables('LastSyncTimestamp')}
  fields: id,name,description,price,stock,category

Headers:
  Authorization: OAuth 2.0 (configured)
  Content-Type: application/json

// Process Response
Parse JSON: @body('ERP_Request')

// Transform for E-commerce Platform
Compose: EcommercePay