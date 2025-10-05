# Chapter 7: Data Operations - Code Examples

## Overview
This document contains all Power Automate data operation expressions, transformation patterns, and code examples from Chapter 7: Navigating Data Operations.

---

## Variable Initialization Examples

### Initialize Multiple Variables for Data Tracking

**Description:** Set up variables for tracking different data types in a workflow

```javascript
// String Variable
Initialize Variable:
  Name: orderStatus
  Type: String
  Value: "Pending"

// Number Variable
Initialize Variable:
  Name: totalAmount
  Type: Float
  Value: 0.0

// Boolean Variable
Initialize Variable:
  Name: isProcessed
  Type: Boolean
  Value: false

// Array Variable
Initialize Variable:
  Name: productList
  Type: Array
  Value: []

// Object Variable
Initialize Variable:
  Name: customerData
  Type: Object
  Value: {
    "id": "",
    "name": "",
    "email": ""
  }
```

---

## Array Operations

### Creating and Manipulating Arrays

**Description:** Common array operations in Power Automate

```javascript
// Initialize Array with Data
Initialize Variable:
  Name: products
  Type: Array
  Value: [
    {
      "id": "P001",
      "name": "Laptop",
      "price": 1200,
      "category": "Electronics"
    },
    {
      "id": "P002",
      "name": "Mouse",
      "price": 25,
      "category": "Electronics"
    },
    {
      "id": "P003",
      "name": "Desk Chair",
      "price": 250,
      "category": "Furniture"
    }
  ]

// Append Item to Array
Append to array variable:
  Name: products
  Value: {
    "id": "P004",
    "name": "Keyboard",
    "price": 80,
    "category": "Electronics"
  }

// Process Array with Apply to Each
Apply to each:
  Select output from previous steps: @variables('products')
  
  Actions:
    Compose: Current_Product
      @{items('Apply_to_each')}
    
    Condition: Check_Price
      @greater(items('Apply_to_each')?['price'], 100)
      
      If Yes:
        Append to array: expensiveProducts
        Value: @{items('Apply_to_each')}
```

---

## Data Transformation Expressions

### String Manipulation Expressions

**Description:** Common string transformation operations

```javascript
// Concatenate Strings
@concat('Hello', ' ', 'World')
// Result: "Hello World"

// Replace Text
@replace('Hello World', 'World', 'Universe')
// Result: "Hello Universe"

// Split String into Array
@split('apple,orange,banana', ',')
// Result: ["apple", "orange", "banana"]

// Get String Length
@length('Hello World')
// Result: 11

// Extract Substring
@substring('Hello World', 0, 5)
// Result: "Hello"

// Convert to Uppercase
@toUpper('hello world')
// Result: "HELLO WORLD"

// Convert to Lowercase
@toLower('HELLO WORLD')
// Result: "hello world"

// Trim Whitespace
@trim('  Hello World  ')
// Result: "Hello World"
```

### Type Conversion Expressions

**Description:** Convert between different data types

```javascript
// String to Integer
@int('42')
// Result: 42

// String to Float
@float('3.14')
// Result: 3.14

// Number to String
@string(42)
// Result: "42"

// Parse JSON String
@json('{"name": "John", "age": 30}')
// Result: Object with properties

// Convert to JSON String
@string(variables('customerObject'))
// Result: JSON string representation
```

### Complex Name Formatting Example

**Description:** Format a name with proper capitalization

```javascript
// Format Name: First letter uppercase, rest lowercase
Compose: FormattedName
  Value: @{
    concat(
      toUpper(substring(variables('fullName'), 0, 1)),
      toLower(substring(variables('fullName'), 1, sub(length(variables('fullName')), 1)))
    )
  }

// Example: "jOHN DOE" becomes "John doe"
```

---

## Data Mapping Between Systems

### Customer Data Mapping

**Description:** Transform Salesforce customer data to internal format

```javascript
Compose: MappedCustomerData
{
  "customer": {
    "id": "@{variables('salesforceCustomer')?['sf_id']}",
    "fullName": "@{concat(
      variables('salesforceCustomer')?['first_name'],
      ' ',
      variables('salesforceCustomer')?['last_name']
    )}",
    "emailAddress": "@{variables('salesforceCustomer')?['email']}",
    "accountType": "@{if(
      equals(variables('salesforceCustomer')?['customer_type'], 'Premium'),
      'Gold',
      'Standard'
    )}",
    "phoneNumber": "@{variables('salesforceCustomer')?['phone']}",
    "address": {
      "street": "@{variables('salesforceCustomer')?['street']}",
      "city": "@{variables('salesforceCustomer')?['city']}",
      "state": "@{variables('salesforceCustomer')?['state']}",
      "zipCode": "@{variables('salesforceCustomer')?['zip']}"
    },
    "metadata": {
      "source": "Salesforce",
      "mappedDate": "@{utcNow()}",
      "mappedBy": "Power Automate"
    }
  }
}
```

### Field Mapping Techniques

**Description:** Various field mapping patterns

```javascript
// Direct Field Mapping
Target_Field: CustomerName
Expression: @{triggerOutputs()?['body/fullname']}

// Field Concatenation
Target_Field: FullAddress
Expression: @{concat(
  triggerOutputs()?['body/address1_line1'],
  ', ',
  triggerOutputs()?['body/address1_city'],
  ', ',
  triggerOutputs()?['body/address1_stateorprovince']
)}

// Field Splitting (Extract First Name)
First_Name: @{split(triggerOutputs()?['body/fullname'], ' ')?[0]}

// Field Splitting (Extract Last Name with Safety Check)
Last_Name: @{if(
  greater(length(split(triggerOutputs()?['body/fullname'], ' ')), 1),
  split(triggerOutputs()?['body/fullname'], ' ')?[1],
  ''
)}

// Text to Number Conversion with Null Handling
Expression: @{if(
  empty(triggerOutputs()?['body/revenue']),
  0,
  int(triggerOutputs()?['body/revenue'])
)}

// Date Format Standardization
Expression: @{formatDateTime(
  triggerOutputs()?['body/createdon'],
  'yyyy-MM-dd'
)}

// Boolean Value Mapping
Expression: @{if(
  equals(triggerOutputs()?['body/isactive'], 'Yes'),
  true,
  false
)}
```

### Conditional Value Mapping

**Description:** Map values using switch expressions

```javascript
// Priority Mapping
Compose: MappedPriority
  Value: @{switch(
    triggerOutputs()?['body/urgency'],
    'Low', 3,
    'Medium', 2,
    'High', 1,
    3  // Default value
  )}

// Status Code Mapping
Compose: StatusText
  Value: @{switch(
    triggerOutputs()?['body/statusCode'],
    '1', 'New',
    '2', 'In Progress',
    '3', 'Completed',
    '4', 'Cancelled',
    'Unknown'  // Default
  )}
```

### Null and Empty Value Handling

**Description:** Handle missing or null values safely

```javascript
// Coalesce - Use first non-null value
Expression: @{coalesce(
  triggerOutputs()?['body/email'],
  triggerOutputs()?['body/alternativeemail'],
  'no-email@company.com'
)}

// Check for Empty with Default
Expression: @{if(
  empty(triggerOutputs()?['body/description']),
  'No description provided',
  triggerOutputs()?['body/description']
)}

// Safe Property Access
Expression: @{if(
  not(equals(triggerOutputs()?['body/customer'], null)),
  triggerOutputs()?['body/customer']?['name'],
  'Unknown Customer'
)}
```

---

## JSON Operations

### Parse JSON Configuration

**Description:** Parse JSON data from API response

```javascript
Action: Parse JSON
  Content: @{triggerBody()}
  
  Schema: {
    "type": "object",
    "properties": {
      "id": {
        "type": "string"
      },
      "customer": {
        "type": "object",
        "properties": {
          "name": {"type": "string"},
          "email": {"type": "string"},
          "phone": {"type": "string"}
        }
      },
      "orderItems": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "productId": {"type": "string"},
            "quantity": {"type": "integer"},
            "price": {"type": "number"}
          }
        }
      },
      "total": {
        "type": "number"
      },
      "orderDate": {
        "type": "string"
      }
    }
  }

// Access Parsed Values
Customer_Name: @{body('Parse_JSON')?['customer']?['name']}
First_Item: @{body('Parse_JSON')?['orderItems']?[0]?['productId']}
Total_Amount: @{body('Parse_JSON')?['total']}
```

### Create JSON Structure

**Description:** Build JSON object for API request

```javascript
Compose: OrderJSON
{
  "order": {
    "orderId": "@{guid()}",
    "customer": {
      "customerId": "@{triggerBody()?['customerId']}",
      "name": "@{triggerBody()?['customerName']}",
      "email": "@{triggerBody()?['customerEmail']}"
    },
    "items": @{triggerBody()?['items']},
    "totals": {
      "subtotal": @{variables('subtotal')},
      "tax": @{variables('tax')},
      "shipping": @{variables('shipping')},
      "total": @{add(add(variables('subtotal'), variables('tax')), variables('shipping'))}
    },
    "orderDate": "@{utcNow()}",
    "status": "Pending"
  }
}
```

---

## XML Operations

### Parse XML Data

**Description:** Convert XML to JSON for processing

```javascript
// XML String to Parse
@xml('<customer><id>123</id><name>John Doe</name><email>john@example.com</email></customer>')

// Convert XML to JSON
Compose: XMLtoJSON
  Value: @{json(xml(triggerBody()))}

// Access Converted Values
Customer_ID: @{body('XMLtoJSON')?['customer']?['id']}
Customer_Name: @{body('XMLtoJSON')?['customer']?['name']}
```

---

## CSV Operations

### Parse CSV Data

**Description:** Process CSV file content

```javascript
// Split CSV into rows
Compose: CSVRows
  Value: @{split(triggerBody(), '\n')}

// Get headers (first row)
Compose: Headers
  Value: @{split(first(outputs('CSVRows')), ',')}

// Process data rows
Apply to each: @skip(outputs('CSVRows'), 1)
  
  Compose: RowData
    Value: @{split(item(), ',')}
  
  // Create object from row
  Compose: RowObject
  {
    "name": "@{outputs('RowData')?[0]}",
    "email": "@{outputs('RowData')?[1]}",
    "phone": "@{outputs('RowData')?[2]}"
  }
```

---

## Filtering Operations

### Filter Array - Simple Example

**Description:** Filter products above a certain price

```javascript
Action: Filter array
  From: @variables('products')
  
  Condition:
    @greater(item()?['price'], 100)

// Result: Only products with price > 100
```

### Filter with Complex Conditions

**Description:** Filter with multiple criteria

```javascript
Compose: FilteredProducts
  Value: @{filter(
    variables('products'),
    and(
      greater(item()?['price'], 50),
      less(item()?['price'], 500),
      equals(item()?['category'], 'Electronics')
    )
  )}

// Filters products where:
// - price > 50
// - price < 500  
// - category = 'Electronics'
```

---

## Sorting Operations

### Sort Array Ascending

**Description:** Sort items by price in ascending order

```javascript
Compose: SortedProducts
  Value: @{sort(variables('products'), item()?['price'])}
```

### Sort Array Descending

**Description:** Sort items by price in descending order

```javascript
Compose: SortedProductsDesc
  Value: @{reverse(sort(variables('products'), item()?['price']))}
```

---

## Grouping and Aggregation

### Group Orders by Customer

**Description:** Group order data by customer ID

```javascript
Compose: GroupedOrders
  Value: @{
    reduce(
      variables('orders'),
      {},
      (acc, cur) => 
        setProperty(
          acc,
          cur.customerId,
          concat(
            coalesce(acc[cur.customerId], []),
            [cur]
          )
        )
    )
  }

// Result: Object with customer IDs as keys,
// each containing array of their orders
```

### Calculate Aggregates

**Description:** Calculate sum, average, count

```javascript
// Sum of Order Totals
Compose: TotalRevenue
  Value: @{sum(
    select(variables('orders'), item()?['total'])
  )}

// Average Order Value
Compose: AverageOrderValue
  Value: @{div(
    sum(select(variables('orders'), item()?['total'])),
    length(variables('orders'))
  )}

// Count of High-Value Orders
Compose: HighValueOrderCount
  Value: @{length(
    filter(
      variables('orders'),
      greater(item()?['total'], 1000)
    )
  )}
```

---

## Advanced Collection Operations

### Select - Extract and Transform Properties

**Description:** Transform array to extract specific properties

```javascript
Action: Select
  From: @variables('orders')
  
  Map:
  {
    "OrderID": "@{item()?['id']}",
    "CustomerName": "@{item()?['customer']?['name']}",
    "OrderTotal": @{item()?['total']},
    "OrderDate": "@{formatDateTime(item()?['date'], 'yyyy-MM-dd')}",
    "Status": "@{item()?['status']}"
  }
```

### Join Array Elements

**Description:** Combine array elements into string

```javascript
// Join with comma
Compose: ProductNames
  Value: @{join(
    select(variables('products'), item()?['name']),
    ', '
  )}

// Result: "Laptop, Mouse, Keyboard, Monitor"
```

---

## Date and Time Operations

### Date Calculations

**Description:** Common date manipulation operations

```javascript
// Add 30 days to current date
@addDays(utcNow(), 30)

// Subtract 7 days
@addDays(utcNow(), -7)

// Format date
@formatDateTime(utcNow(), 'yyyy-MM-dd')

// Format with time
@formatDateTime(utcNow(), 'yyyy-MM-dd HH:mm:ss')

// Get date 30 days from now
Compose: FutureDate
  Value: @{formatDateTime(
    addDays(utcNow(), 30),
    'yyyy-MM-dd'
  )}

// Calculate days between dates
Compose: DaysDifference
  Value: @{div(
    sub(
      ticks(variables('endDate')),
      ticks(variables('startDate'))
    ),
    864000000000  // Ticks in a day
  )}

// Check if date is in past
Compose: IsDatePast
  Value: @{less(
    ticks(variables('checkDate')),
    ticks(utcNow())
  )}
```

---

## Conditional Data Transformation

### Complex Conditional Logic

**Description:** Transform data based on multiple conditions

```javascript
Compose: ProcessedOrders
  Value: @{
    map(
      variables('orders'),
      {
        "orderId": item()?['id'],
        "customer": item()?['customerName'],
        "total": item()?['total'],
        "priority": if(
          greater(item()?['total'], 10000),
          'High',
          if(
            greater(item()?['total'], 5000),
            'Medium',
            'Low'
          )
        ),
        "discount": if(
          equals(item()?['customerType'], 'Premium'),
          mul(item()?['total'], 0.10),
          if(
            greater(item()?['total'], 1000),
            mul(item()?['total'], 0.05),
            0
          )
        ),
        "shippingMethod": if(
          greater(item()?['total'], 500),
          'Express',
          'Standard'
        )
      }
    )
  }
```

---

## Error Handling Patterns

### Try-Catch with Scope

**Description:** Implement error handling using Scope actions

```javascript
Scope: Try_Data_Processing
  
  Actions:
    // Main data processing actions
    Parse JSON
    Apply to each
    Update database
    
// Configure Run After - Error Handler
Scope: Catch_Error
  Configure Run After: Try_Data_Processing
    Run after: has failed, has timed out
  
  Actions:
    Compose: ErrorDetails
    {
      "error": "@{body('Try_Data_Processing')?['error']}",
      "timestamp": "@{utcNow()}",
      "flowRun": "@{workflow().run.id}"
    }
    
    Send Email: ErrorNotification
      To: admin@company.com
      Subject: Data Processing Error
      Body: @{outputs('ErrorDetails')}
```

### Null Value Checking

**Description:** Check for null or empty values before processing

```javascript
Condition: CheckData
  Expression: @and(
    not(equals(triggerBody(), null)),
    not(empty(triggerBody()?['requiredField'])),
    not(equals(triggerBody()?['requiredField'], ''))
  )
  
  If Yes:
    // Process data
    Compose: ProcessedData
      Value: @{triggerBody()}
  
  If No:
    // Handle missing data
    Compose: ErrorMessage
      Value: "Required data is missing or invalid"
    
    Send Email: ValidationError
      Subject: Data Validation Failed
      Body: @{outputs('ErrorMessage')}
```

### Retry Logic Implementation

**Description:** Implement retry mechanism for failed operations

```javascript
Initialize Variable: retryCount
  Type: Integer
  Value: 0

Initialize Variable: maxRetries
  Type: Integer
  Value: 3

Initialize Variable: success
  Type: Boolean
  Value: false

Do Until: @or(
  variables('success'),
  greaterOrEquals(variables('retryCount'), variables('maxRetries'))
)
  
  Actions:
    // Attempt operation
    HTTP: CallExternalAPI
      Method: POST
      URI: https://api.example.com/data
      Body: @{variables('requestData')}
    
    // Check if successful
    Condition: CheckSuccess
      @equals(outputs('CallExternalAPI')['statusCode'], 200)
      
      If Yes:
        Set Variable: success
          Value: true
      
      If No:
        // Increment retry count
        Increment Variable: retryCount
        
        // Wait before retry (exponential backoff)
        Delay:
          Count: @{mul(power(2, variables('retryCount')), 5)}
          Unit: Second
        
        Compose: RetryMessage
          Value: "Retry attempt @{variables('retryCount')} of @{variables('maxRetries')}"
```

---

## Performance Optimization Patterns

### Batch Processing

**Description:** Process data in batches to improve performance

```javascript
Initialize Variable: batchSize
  Type: Integer
  Value: 100

Initialize Variable: totalItems
  Type: Integer
  Value: @{length(variables('allItems'))}

Initialize Variable: currentBatch
  Type: Integer
  Value: 0

Do Until: @greaterOrEquals(
  mul(variables('currentBatch'), variables('batchSize')),
  variables('totalItems')
)
  
  // Calculate batch boundaries
  Compose: batchStart
    Value: @{mul(variables('currentBatch'), variables('batchSize'))}
  
  Compose: batchEnd
    Value: @{min(
      add(
        mul(variables('currentBatch'), variables('batchSize')),
        variables('batchSize')
      ),
      variables('totalItems')
    )}
  
  // Get current batch
  Compose: currentBatchItems
    Value: @{skip(
      take(
        variables('allItems'),
        outputs('batchEnd')
      ),
      outputs('batchStart')
    )}
  
  // Process batch
  Apply to each: @outputs('currentBatchItems')
    // Process each item in batch
    
  // Increment batch counter
  Increment Variable: currentBatch
```

### Pagination Strategy

**Description:** Handle paginated API responses

```javascript
Initialize Variable: pageNumber
  Type: Integer
  Value: 1

Initialize Variable: hasMorePages
  Type: Boolean
  Value: true

Initialize Variable: allResults
  Type: Array
  Value: []

Do Until: @equals(variables('hasMorePages'), false)
  
  // Get current page
  HTTP: GetPage
    Method: GET
    URI: @{concat(
      'https://api.example.com/data?page=',
      variables('pageNumber'),
      '&pageSize=100'
    )}
  
  // Parse response
  Parse JSON: PageResponse
    Content: @{body('GetPage')}
  
  // Append results
  Append to array variable: allResults
    Value: @{body('PageResponse')?['items']}
  
  // Check if more pages exist
  Set Variable: hasMorePages
    Value: @{body('PageResponse')?['hasNextPage']}
  
  // Increment page number
  Increment Variable: pageNumber
  
  // Add delay to respect rate limits
  Delay:
    Count: 1
    Unit: Second
```

---

## Secure Data Handling

### Secure Connection String

**Description:** Use environment variables for sensitive data

```javascript
// Store connection string securely
// In environment variables: ConnectionString

// Use in flow
Compose: SecureConnection
  Value: @{environment().ConnectionString}

// Never hardcode:
// ❌ BAD: "Server=myserver;Database=mydb;User=sa;Password=P@ssw0rd"
// ✅ GOOD: @{environment().DatabaseConnectionString}
```

### Data Masking for Logging

**Description:** Mask sensitive data in logs

```javascript
Compose: MaskedCustomerData
{
  "customerId": "@{variables('customer')?['id']}",
  "name": "@{variables('customer')?['name']}",
  "email": "@{concat(
    substring(variables('customer')?['email'], 0, 3),
    '***@',
    split(variables('customer')?['email'], '@')?[1]
  )}",
  "phone": "@{concat(
    '***-***-',
    substring(variables('customer')?['phone'], 7, 4)
  )}",
  "creditCard": "****-****-****-@{substring(
    variables('customer')?['creditCard'],
    12,
    4
  )}"
}

// Result shows partially masked data for security
```

---

## Data Source Integration Examples

### SharePoint Operations

**Description:** Common SharePoint data operations

```javascript
// Get Items with Filter
Action: Get items (SharePoint)
  Site Address: https://company.sharepoint.com/sites/team
  List Name: Customers
  Filter Query: Status eq 'Active' and Created gt '@{formatDateTime(addDays(utcNow(), -30), 'yyyy-MM-dd')}'
  Select Query: ID,Title,Email,CustomerType
  Top Count: 500

// Create Item
Action: Create item (SharePoint)
  Site Address: https://company.sharepoint.com/sites/team
  List Name: Orders
  Title: @{triggerBody()?['orderNumber']}
  Customer: @{triggerBody()?['customerId']}
  Amount: @{triggerBody()?['total']}
  OrderDate: @{utcNow()}

// Update Item
Action: Update item (SharePoint)
  Site Address: https://company.sharepoint.com/sites/team
  List Name: Orders
  Id: @{triggerBody()?['orderId']}
  Status: "Processed"
  ProcessedDate: @{utcNow()}
```

### Dataverse (Common Data Service) Operations

**Description:** Work with Dataverse entities

```javascript
// List Records with Filter
Action: List records (Dataverse)
  Environment: [Your environment]
  Table name: Accounts
  Filter Query: contains(name, '@{variables('searchTerm')}')
  Select columns: accountid,name,emailaddress1,telephone1
  Row count: 100

// Create Record
Action: Add a new row (Dataverse)
  Environment: [Your environment]
  Table name: Contacts
  First Name: @{triggerBody()?['firstName']}
  Last Name: @{triggerBody()?['lastName']}
  Email: @{triggerBody()?['email']}
  Company Name: @{triggerBody()?['company']}

// Update Record
Action: Update a row (Dataverse)
  Environment: [Your environment]
  Table name: Accounts
  Row ID: @{triggerBody()?['accountId']}
  Annual Revenue: @{variables('newRevenue')}
  Last Updated: @{utcNow()}
```

### SQL Server Operations

**Description:** Execute SQL queries safely

```javascript
// Parameterized Query (Prevents SQL Injection)
Action: Execute a SQL query (SQL Server)
  Server name: sql.company.com
  Database name: SalesDB
  
  Query:
  SELECT 
    OrderID,
    CustomerName,
    OrderDate,
    TotalAmount,
    Status
  FROM Orders
  WHERE 
    CustomerID = @{parameters('customerId')}
    AND OrderDate >= @{parameters('startDate')}
    AND Status IN ('Pending', 'Processing')
  ORDER BY OrderDate DESC

// Insert with Parameters
Action: Execute a SQL query (SQL Server)
  Query:
  INSERT INTO Customers (
    FirstName,
    LastName,
    Email,
    Phone,
    CreatedDate
  ) VALUES (
    @{parameters('firstName')},
    @{parameters('lastName')},
    @{parameters('email')},
    @{parameters('phone')},
    GETDATE()
  )
```

---

## Real-World Use Case: Customer Data Sync

### Complete Synchronization Flow

**Description:** Daily customer data synchronization between CRM and Marketing

```javascript
Trigger: Recurrence
  Frequency: Day
  Interval: 1
  Start time: 02:00 AM

// Step 1: Get Updated Customers
Action: List records (Dynamics 365)
  Entity name: contacts
  Filter Query: modifiedon gt @{formatDateTime(addDays(utcNow(), -1), 'yyyy-MM-ddTHH:mm:ssZ')}
  
// Step 2: Transform Data
Action: Select
  From: @body('List_records')?['value']
  
  Map:
  {
    "email": "@{item()?['emailaddress1']}",
    "firstName": "@{item()?['firstname']}",
    "lastName": "@{item()?['lastname']}",
    "phone": "@{item()?['telephone1']}",
    "company": "@{item()?['parentcustomerid@OData.Community.Display.V1.FormattedValue']}",
    "status": "@{if(
      equals(item()?['statecode'], 0),
      'active',
      'inactive'
    )}",
    "tags": [
      "@{if(
        equals(item()?['preferredcontactmethodcode'], 1),
        'email-preferred',
        'phone-preferred'
      )}"
    ],
    "customFields": {
      "crmId": "@{item()?['contactid']}",
      "lastModified": "@{item()?['modifiedon']}"
    }
  }

// Step 3: Validate Data
Apply to each: @body('Select')
  
  Condition: ValidateEmail
    @and(
      not(empty(item()?['email'])),
      contains(item()?['email'], '@')
    )
    
    If Yes:
      // Step 4: Upload to Marketing Platform
      HTTP: UpdateMailchimp
        Method: PUT
        URI: https://api.mailchimp.com/3.0/lists/@{parameters('listId')}/members/@{toLower(item()?['email'])}
        Headers:
          Authorization: Bearer @{parameters('mailchimpApiKey')}
          Content-Type: application/json
        Body: {
          "email_address": "@{item()?['email']}",
          "status": "@{item()?['status']}",
          "merge_fields": {
            "FNAME": "@{item()?['firstName']}",
            "LNAME": "@{item()?['lastName']}",
            "PHONE": "@{item()?['phone']}",
            "COMPANY": "@{item()?['company']}"
          },
          "tags": @{item()?['tags']}
        }
      
      // Log Success
      Append to array: successLog
        Value: @{item()?['email']}
    
    If No:
      // Log Validation Failure
      Append to array: validationErrors
        Value: {
          "email": "@{item()?['email']}",
          "reason": "Invalid email format"
        }

// Step 5: Send Summary Report
Send Email:
  To: admin@company.com
  Subject: Daily Customer Sync Complete
  Body: |
    Customer synchronization completed.
    
    Successfully synced: @{length(variables('successLog'))} customers
    Validation errors: @{length(variables('validationErrors'))} customers
    
    @{if(
      greater(length(variables('validationErrors')), 0),
      concat('Errors: ', json(string(variables('validationErrors')))),
      ''
    )}
```

---

## Notes

- All expressions use Power Automate syntax
- Replace placeholder values with actual data
- Test all expressions in development before production
- Use secure storage for sensitive values
- Monitor performance with large datasets
- Implement proper error handling
- Document complex transformations
- Version control important flows
