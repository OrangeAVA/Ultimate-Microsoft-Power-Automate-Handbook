# Chapter 2: Power Automate Flow Definitions

## Overview
This document contains the detailed Power Automate flow example from Chapter 2: Creating Your First Flow. This chapter provides a step-by-step walkthrough of building a SharePoint file notification system.

---

## Main Example: SharePoint File Notification System

**Flow Name:** SharePoint File Notification

**Description:** Automated email notification system that alerts users when new files are added to a specific SharePoint folder

**Flow Type:** Automated Cloud Flow

---

### Step 1: Trigger Configuration

**Trigger:** When a file is created in a folder (SharePoint)

**Configuration Parameters:**
- **Site Address:** [Select from dropdown or enter SharePoint site URL]
  - Must have appropriate permissions to access the site
  - Example: `https://yourdomain.sharepoint.com/sites/ProjectSite`

- **Folder Id:** [Browse and select specific folder to monitor]
  - Click folder icon to navigate site structure
  - Select target folder path
  - Example: `/Shared Documents/Project Files`

- **Include Subfolders:** No
  - Set to "Yes" if monitoring entire folder structure is needed
  - "No" provides more predictable behavior for most scenarios

**Dynamic Content Available from Trigger:**
- File name
- File size
- File path
- Creation time
- File identifier
- File creator (if available)
- Link to item
- File metadata

---

### Step 2: Email Notification Action (Basic)

**Action:** Send an email (V2) - Office 365 Outlook

**Configuration:**

**To Field:**
- Enter specific email address
- OR use dynamic content for file creator
- OR use distribution list for team notifications
- Example: `projectteam@yourdomain.com`

**Subject Field:**
```
New file added to SharePoint: {File name}
```
- Uses dynamic content: `{File name}` from trigger
- Alternative: `New file added to [Folder Name]: [File Name]`

**Body Field:**
```
Hello,

A new file has been added to the SharePoint project folder.

File Details:
- File Name: {File name}
- File Size: {File size} bytes
- Created: {Created}
- Created By: {Created By}

You can access the file directly here: {Link to item}

This is an automated notification from Power Automate.
```

**Dynamic Content Used:**
- `{File name}` - Name of the uploaded file
- `{File size}` - Size in bytes
- `{Created}` - Timestamp of file creation
- `{Created By}` - User who uploaded the file
- `{Link to item}` - Direct URL to the file

**HTML Formatting (Optional):**
```html
<h3>New File Added to SharePoint</h3>
<p>A new file has been added to the project folder.</p>

<h4>File Details:</h4>
<ul>
  <li><strong>File Name:</strong> {File name}</li>
  <li><strong>File Size:</strong> {File size} bytes</li>
  <li><strong>Created:</strong> {Created}</li>
  <li><strong>Created By:</strong> {Created By}</li>
</ul>

<p><a href="{Link to item}">Click here to view the file</a></p>

<p><em>This is an automated notification.</em></p>
```

---

### Step 3: Add Conditional Logic (Enhanced Version)

**Action:** Condition

**Purpose:** Only send detailed notifications for files over a certain size threshold

**Condition Configuration:**

**First Value:** File size (from dynamic content)

**Operator:** is greater than

**Second Value:** 5000000 (represents 5MB in bytes)

**File Size Reference:**
- 1 MB = 1,000,000 bytes
- 5 MB = 5,000,000 bytes
- 10 MB = 10,000,000 bytes

---

### Step 4a: If Yes Branch (Large Files)

**Condition Met:** File size > 5MB

**Action:** Send detailed email notification

**Email Configuration:**

**To:** `projectmanager@yourdomain.com`

**Subject:**
```
LARGE FILE ALERT: {File name} added to SharePoint
```

**Body:**
```
Attention,

A large file has been uploaded to the SharePoint project folder and may require review.

File Details:
- File Name: {File name}
- File Size: {File size} bytes (Approximately {Expression: div(File size, 1000000)} MB)
- Created: {Created}
- Created By: {Created By}
- File Location: {Folder path}

Direct Link: {Link to item}

Please review this file to ensure:
1. It meets project requirements
2. It's in the correct location
3. Appropriate team members have been notified

This is an automated notification for files exceeding 5MB.
```

**Additional Actions (Optional):**
- Log to SharePoint list for tracking
- Send Teams notification to project channel
- Create approval request for manager

---

### Step 4b: If No Branch (Small Files)

**Condition Not Met:** File size ≤ 5MB

**Action:** Send basic email notification OR log only

**Option 1: Basic Email**

**To:** `projectteam@yourdomain.com`

**Subject:**
```
File added: {File name}
```

**Body:**
```
A new file has been added to SharePoint.

File: {File name}
Size: {File size} bytes
Link: {Link to item}

This is an automated notification.
```

**Option 2: Log to SharePoint List (No Email)**

**Action:** Create item (SharePoint)

**List:** File Activity Log

**Fields:**
- File Name: `{File name}`
- File Size: `{File size}`
- Created Date: `{Created}`
- Created By: `{Created By}`
- File Link: `{Link to item}`
- Notification Sent: No

---

## Advanced Configuration Options

### Error Handling Configuration

**Timeout Settings:**
- Email action timeout: 120 seconds (default)
- Increase for large attachments or network issues

**Configure Run After:**
- Set on email actions
- Options: Run after previous step succeeds, fails, is skipped, or times out
- For non-critical notifications: Run even if previous step fails

**Retry Policy:**
- Email sending: Automatic retry with exponential backoff
- Configure: Up to 4 retry attempts
- Interval: 5, 10, 20, 40 seconds

---

### Multiple Connector Integration (Extended Version)

**After Email Notification, Add:**

**1. Post to Microsoft Teams Channel**

**Action:** Post message in a chat or channel (Teams)

**Configuration:**
- Team: Project Team
- Channel: File Updates
- Message:
```
📄 New file uploaded to SharePoint

**File:** {File name}
**Size:** {File size} bytes
**Uploaded by:** {Created By}

[View File]({Link to item})
```

**2. Log to Database (Optional)**

**Action:** Insert row (SQL Server or Dataverse)

**Purpose:** Track all file uploads for reporting

**Fields:**
- FileName: `{File name}`
- FileSize: `{File size}`
- UploadDate: `{Created}`
- UploadedBy: `{Created By}`
- FilePath: `{Folder path}`

**3. Create Planner Task (If Required)**

**Action:** Create a task (Planner)

**Condition:** Only for certain file types or large files

**Configuration:**
- Plan: Project Plan
- Bucket: File Reviews
- Title: `Review uploaded file: {File name}`
- Assigned To: Project Manager
- Due Date: 2 days from now

---

### Parallel Branch Implementation

**Purpose:** Perform multiple actions simultaneously for better performance

**Configuration:**

After trigger, create parallel branches for:

**Branch 1:** Send email notification
**Branch 2:** Post to Teams
**Branch 3:** Log to database
**Branch 4:** Update SharePoint metadata

All branches execute concurrently, reducing total flow execution time.

**Note:** Use carefully with actions that modify the same data

---

## Expression Examples for Advanced Logic

### Date Formatting

**Format Creation Date:**
```
formatDateTime(triggerBody()?['Created'], 'MM/dd/yyyy hh:mm tt')
```
**Output:** 10/05/2025 02:30 PM

### File Size Conversion

**Convert Bytes to MB:**
```
div(triggerBody()?['Size'], 1000000)
```

**Convert with Rounding:**
```
concat(string(div(triggerBody()?['Size'], 1000000)), ' MB')
```

### String Manipulation

**Extract File Extension:**
```
substring(triggerBody()?['Name'], lastIndexOf(triggerBody()?['Name'], '.'))
```

**Create Custom File Path:**
```
concat('https://yourdomain.sharepoint.com', triggerBody()?['{Path}'], '/', triggerBody()?['{FilenameWithExtension}'])
```

### Conditional Logic in Expressions

**Determine File Type Category:**
```
if(
  or(
    endsWith(triggerBody()?['Name'], '.pdf'),
    endsWith(triggerBody()?['Name'], '.docx')
  ),
  'Document',
  'Other'
)
```

**Priority Based on Size:**
```
if(
  greater(triggerBody()?['Size'], 10000000),
  'High',
  if(
    greater(triggerBody()?['Size'], 1000000),
    'Medium',
    'Low'
  )
)
```

---

## Testing Procedures

### Test Scenario 1: Basic Functionality
1. Upload a small file (< 5MB) to monitored folder
2. Verify email notification received
3. Check notification contains correct file details
4. Confirm link to file works correctly

### Test Scenario 2: Large File Handling
1. Upload a large file (> 5MB) to monitored folder
2. Verify detailed notification sent
3. Confirm manager receives notification
4. Check file size calculation is accurate

### Test Scenario 3: Edge Cases
1. Upload file exactly at 5MB threshold
2. Upload file with special characters in name
3. Upload multiple files simultaneously
4. Upload file then immediately delete it

### Test Scenario 4: Error Scenarios
1. Test with expired SharePoint connection
2. Test with invalid email addresses
3. Test with SharePoint permissions issues
4. Test during SharePoint maintenance

---

## Monitoring Configuration

### Key Metrics to Track

**Performance Metrics:**
- Average execution time
- Success rate percentage
- Number of runs per day
- Peak usage times

**Error Metrics:**
- Failed runs count
- Timeout occurrences
- Connection failures
- Invalid data errors

### Alert Configuration

**Create Monitoring Flow:**

**Trigger:** Recurrence (Daily at 9 AM)

**Actions:**
1. Get flow runs from last 24 hours
2. Count failed runs
3. If failures > threshold:
   - Send alert email to admin
   - Post to Teams admin channel
   - Create incident ticket

---

## Best Practices Applied

### Naming Conventions
✅ **Good:** "SharePoint File Notification - Project Folder"
❌ **Bad:** "Flow1" or "My Flow"

### Documentation
- Add description to flow
- Comment complex conditions
- Document expected behavior
- Note any dependencies

### Security
- Use service accounts for authentication
- Implement least privilege access
- Secure sensitive data in email bodies
- Regular connection credential rotation

### Performance
- Minimize external API calls
- Use parallel branches appropriately
- Implement appropriate timeouts
- Batch operations when possible

### Maintenance
- Regular review of flow runs
- Update documentation with changes
- Test after system updates
- Archive unused flows

---

## Flow Activation Checklist

Before activating the flow:

- [ ] All connections authenticated
- [ ] SharePoint site and folder paths verified
- [ ] Email recipients confirmed
- [ ] Test cases completed successfully
- [ ] Error handling configured
- [ ] Monitoring alerts set up
- [ ] Documentation completed
- [ ] Stakeholders notified
- [ ] Support plan in place
- [ ] Rollback procedure documented

---

## Common Issues and Solutions

### Issue 1: Flow Not Triggering
**Symptoms:** New files added but no notification sent

**Solutions:**
- Verify SharePoint connection is active
- Check folder path is correct
- Confirm flow is turned on
- Review trigger configuration
- Check for throttling limits

### Issue 2: Missing Dynamic Content
**Symptoms:** Email shows blank fields or {placeholders}

**Solutions:**
- Refresh dynamic content list
- Verify field names haven't changed
- Check SharePoint permissions
- Re-select dynamic content
- Test with different file types

### Issue 3: Performance Issues
**Symptoms:** Flow takes too long to execute

**Solutions:**
- Reduce number of actions
- Implement parallel branches
- Optimize database queries
- Check network connectivity
- Review connector throttling

### Issue 4: Email Delivery Failures
**Symptoms:** Flow succeeds but emails not received

**Solutions:**
- Check spam/junk folders
- Verify email addresses
- Confirm Office 365 license
- Review email sending limits
- Check email server connectivity

---

## Notes

- All configurations shown are examples and should be customized for your environment
- File size thresholds can be adjusted based on organizational needs
- Additional actions can be added based on specific requirements
- Screenshots referenced in source document are not accessible in text format
- This flow serves as a foundation for more complex document management automation
