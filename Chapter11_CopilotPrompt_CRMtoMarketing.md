# Copilot Prompt: CRM to Marketing Integration

## Context
Automate lead qualification and marketing campaign enrollment across systems.

## Prompt Example
"Create a flow that monitors new qualified leads in Dynamics 365 Sales. When a lead's status changes to 'Qualified,' check if they've already been added to our marketing campaign in Mailchimp by looking up their email in a list called 'Active Campaigns.' If they're not found, add them to the Mailchimp audience called 'Product Interest' and tag them based on their product interest field from Dynamics. Then update a custom field in Dynamics called 'Marketing_Status' to 'Added to Campaign' with today's date."

## Copilot-Generated Flow Outline
- Trigger: "When a record is updated" (Dynamics 365, status = Qualified)
- Action: "List records" in Mailchimp for email lookup
- Condition: If not found, "Create list member" and "Add member tag" in Mailchimp
- Action: "Update a record" in Dynamics for Marketing_Status
- Error handling for API failures

## Comments
- Copilot connects multiple systems and manages lookups and updates.
- Review error handling and data mapping.
