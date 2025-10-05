# Copilot Prompt: Data Validation and Transformation

## Context
Automate validation, transformation, and conditional processing of SharePoint submissions.

## Prompt Example
"I need a flow that triggers when a new item is added to a SharePoint list called 'Customer Submissions.' The flow should validate that required fields (Name, Email, Phone) are not empty, format the phone number to a standard format (XXX-XXX-XXXX), check if the email domain is from an approved list stored in another SharePoint list called 'Approved Domains,' and then create a new record in Dataverse if all validations pass. If any validation fails, send an email to the submission reviewer with details of the issues."

## Copilot-Generated Flow Outline
- Trigger: "When an item is created" (SharePoint)
- Actions: "Condition" for required fields
- Action: "Compose" for phone formatting
- Action: "Get items" for approved domains
- Action: "Filter array" for domain check
- Condition: If domain is approved, "Create a new row" in Dataverse
- Else: "Compose" error messages and "Send an email" to reviewer

## Comments
- Copilot generates validation logic, data transformation, and error notification paths.
- Review expressions and error message formatting.
