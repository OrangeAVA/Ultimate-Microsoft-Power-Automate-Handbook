# Copilot Prompt: Contract Review Document Flow

## Context
Automate contract review, AI data extraction, approval, and document movement.

## Prompt Example
"Create a flow for our contract review process. When a new contract document is uploaded to our 'Contracts Pending Review' SharePoint library, use AI Builder to extract the contract value, effective date, and client name. Create an approval task in Planner assigned to our legal team with the extracted information and a link to the document. After approval, move the document to either 'Approved Contracts' or 'Rejected Contracts' library based on the decision, and send a notification to the sales representative (stored in a column called 'Account Manager') with the approval status and any comments."

## Copilot-Generated Flow Outline
- Trigger: "When a file is created" (SharePoint)
- Action: "AI Builder - Extract information from documents"
- Actions: "Compose" for formatting extracted data
- Action: "Create a task" in Planner
- Action: "Start and wait for an approval"
- Condition: Approval result → "Copy file" and "Delete file" to move document
- Action: "Get user profile" for Account Manager
- Action: "Send an email" with status and comments

## Comments
- Copilot integrates AI, Planner, SharePoint, and Outlook.
- Review extracted fields and approval routing.
