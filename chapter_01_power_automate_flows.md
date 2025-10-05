# Chapter 1: Power Automate Flow Definitions

## Overview
This document contains Power Automate flow patterns and examples from Chapter 1: Getting Started with Power Automate.

---

## 1. Automated Cloud Flow - Email Notification

**Description:** Sends a mobile notification when a new email arrives

**Flow Type:** Automated Cloud Flow

**Trigger:**
- **Type:** When a new email arrives (V3)
- **Configuration:** 
  - Folder: Inbox
  - Conditions: Can filter by importance, sender, subject

**Actions:**
1. **Send me a mobile notification**
   - Text: "New email from {From} about {Subject}"
   - Dynamic Content: From, Subject fields from trigger

**Use Case:** Get instant mobile notifications for important emails without constantly checking inbox

---

## 2. Expense Report Processing Flow

**Description:** Automates expense report approval and processing workflow

**Flow Type:** Automated Cloud Flow

**Trigger:**
- **Event:** Expense report submitted via form

**Actions:**
1. **Parse submission data**
   - Extract: Amount, Category, Receipts, Justification

2. **Conditional Logic - Amount Check**
   - **If Amount > $500:**
     - Send approval request to manager
     - Wait for approval response
   - **If Amount ≤ $500:**
     - Auto-approve
     - Proceed to next step

3. **Update Finance System**
   - Create expense record
   - Attach receipts
   - Update employee reimbursement queue

4. **Send Notifications**
   - Email confirmation to employee
   - Teams message to finance team
   - Update dashboard

**Outcome:** Streamlined expense processing with automated approvals and notifications

---

## 3. Document Management Automation

**Description:** Automatically organizes documents based on content using AI

**Flow Type:** Automated Cloud Flow with AI Builder

**Trigger:**
- **Event:** New document uploaded to SharePoint

**Actions:**
1. **AI Content Analysis**
   - Use AI Builder to analyze document content
   - Extract: Document type, Keywords, Category

2. **Folder Structure Creation**
   - Check if category folder exists
   - Create new folder if needed

3. **Move Document**
   - Move document to appropriate category folder
   - Preserve metadata

4. **Send Confirmation**
   - Notify uploader of new location
   - Log action in tracking system

**Use Case:** Automatic document organization without manual filing

---

## 4. Meeting Follow-up Automation

**Description:** Handles post-meeting tasks automatically

**Flow Type:** Scheduled/Triggered Cloud Flow

**Trigger:**
- **Event:** Meeting ends (from Teams calendar)

**Actions:**
1. **Extract Meeting Data**
   - Get meeting notes from Teams
   - Identify attendees
   - Extract action items

2. **Create Tasks**
   - Create task in Planner for each action item
   - Assign to responsible person
   - Set due dates

3. **Send Summary Email**
   - Compile meeting summary
   - Include action items
   - Send to all attendees

4. **Schedule Follow-up**
   - If needed, schedule follow-up meeting
   - Send calendar invites

**Outcome:** Automated meeting follow-up process ensuring action items are tracked

---

## 5. Customer Support Ticket Routing

**Description:** Routes support tickets based on priority and category

**Flow Type:** Automated Cloud Flow

**Trigger:**
- **Event:** New support ticket created

**Actions:**
1. **Analyze Ticket**
   - Extract: Priority, Category, Customer tier
   - Keywords analysis for urgency

2. **Routing Logic (Nested Conditions)**
   ```
   IF Priority = "High" OR Customer = "Premium":
       IF Category = "Technical":
           Assign to: Senior Technical Team
       ELSE IF Category = "Billing":
           Assign to: Account Manager
       ELSE:
           Assign to: Customer Success Lead
   ELSE:
       IF Category = "Technical":
           Assign to: Technical Support Queue
       ELSE:
           Assign to: General Support Queue
   ```

3. **Send Notifications**
   - Notify assigned team member
   - Send acknowledgment to customer
   - Update ticket tracking system

4. **Set SLA Timer**
   - Calculate response deadline
   - Set reminder alerts
   - Track time to resolution

---

## 6. SharePoint Document Processing

**Description:** Process documents uploaded to SharePoint with size checking

**Flow Type:** Automated Cloud Flow

**Trigger:**
- **Event:** When a file is created in SharePoint

**Actions:**
1. **Get File Properties**
   - File name
   - File size
   - Uploaded by

2. **Size Check Condition**
   - **If File Size > 5MB:**
     - Send approval request to manager
     - Add comment: "Large file uploaded, requires approval"
     - Wait for response
   - **If Approved:**
     - Keep file in location
     - Send confirmation to uploader
   - **If Rejected:**
     - Move to archive
     - Notify uploader

3. **Notify in Teams**
   - Send message to designated channel
   - Include file name and uploader
   - Add link to document

---

## 7. Daily Task Summary Flow

**Description:** Sends daily email with today's tasks

**Flow Type:** Scheduled Cloud Flow

**Trigger:**
- **Schedule:** Daily at 8:00 AM

**Actions:**
1. **Get Today's Tasks**
   - Query Planner/To Do
   - Filter by: Assigned to me, Due today

2. **Format Task List**
   - Create HTML table
   - Include: Task name, Priority, Due time

3. **Send Email**
   - To: Current user
   - Subject: "Your Tasks for {Today's Date}"
   - Body: Formatted task list

---

## 8. Approval Workflow - Purchase Requests

**Description:** Multi-level approval for purchase requests over $1000

**Flow Type:** Automated Cloud Flow

**Trigger:**
- **Event:** Purchase request submitted

**Actions:**
1. **Extract Request Details**
   - Amount
   - Category
   - Requestor
   - Justification

2. **First Level Approval**
   - **If Amount > $1000:**
     - Send to direct manager
     - Wait for response
     - **If Rejected:** End flow, notify requestor
     
3. **Second Level Approval**
   - **If Amount > $5000:**
     - Send to department head
     - Wait for response
     - **If Rejected:** End flow, notify all parties

4. **Final Processing**
   - Update procurement system
   - Generate purchase order
   - Notify finance team
   - Send confirmation to requestor

---

## 9. Survey Response Processing

**Description:** Collects and processes survey responses with weekly summary

**Flow Type:** Automated + Scheduled Cloud Flow

**Trigger Part 1 (Collection):**
- **Event:** New survey response received

**Actions (Real-time):**
1. Store response in SharePoint list
2. Send thank you email to respondent

**Trigger Part 2 (Summary):**
- **Schedule:** Weekly, every Monday at 9:00 AM

**Actions (Summary):**
1. **Get All Responses** from past week
2. **Analyze Data**
   - Calculate averages
   - Identify trends
   - Flag concerns
3. **Generate Report**
   - Create Excel summary
   - Generate charts
4. **Distribute Report**
   - Email to management
   - Post to Teams channel

---

## 10. Data Sync Flow - Salesforce to SharePoint

**Description:** Syncs customer data between Salesforce and SharePoint

**Flow Type:** Scheduled Cloud Flow

**Trigger:**
- **Schedule:** Every 6 hours

**Actions:**
1. **Get Updated Records from Salesforce**
   - Filter: Modified in last 6 hours
   - Include: Customer name, Contact info, Status

2. **For Each Record:**
   - Check if exists in SharePoint
   - **If Exists:** Update existing item
   - **If New:** Create new item

3. **Error Handling**
   - Log any failed syncs
   - Send alert if errors exceed threshold
   - Retry failed items once

4. **Send Summary**
   - Records processed
   - Successful updates
   - Any errors

---

## Flow Development Best Practices (from Chapter 1)

### Naming Conventions
- Use descriptive names indicating purpose
- Include trigger or main action in name
- Use PascalCase or spaces for readability
- Include department/team if applicable
- Example: "HR - New Employee Onboarding"

### Documentation
- Provide flow description
- Add comments to complex actions
- Maintain flow inventory
- Document version changes
- Create process diagrams for complex flows

### Testing Guidelines
- Start with small test cases
- Create comprehensive test data set
- Monitor flow run history regularly
- Implement try/catch error handling
- Use Flow Checker for validation

---

## Common Trigger Types

### 1. Schedule-Based Triggers
- Daily report generation (Every weekday at 9 AM)
- Weekly data backups (Every Sunday at midnight)
- Monthly summaries (First day of month at 8 AM)

### 2. Event-Based Triggers
- When email arrives
- When file created/modified
- When item added to list
- When form submitted
- When message posted

### 3. Manual Triggers
- Button clicks
- Mobile app triggers
- SharePoint command bars
- Teams message actions

---

## Common Actions

### Data Operations
- Get items from lists
- Create/Update items
- Parse JSON
- Select/Filter arrays
- Compose data

### Communication
- Send email
- Post to Teams
- Send mobile notification
- Create Teams meeting

### Approval
- Start approval
- Wait for approval
- Custom responses

### Condition & Control
- Condition (if/else)
- Switch case
- Apply to each (loops)
- Do until
- Scope (error handling)

---

## Notes
- All flows in this chapter are conceptual examples
- No actual executable code or JSON provided in source material
- Screenshots referenced but not accessible in document text
- These patterns should be implemented using Power Automate's visual designer
