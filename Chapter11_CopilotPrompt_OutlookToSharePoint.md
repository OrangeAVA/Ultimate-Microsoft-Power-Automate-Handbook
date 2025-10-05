# Copilot Prompt: Outlook Attachments to SharePoint

## Context
Describe your business process to Copilot to generate a complete Power Automate flow.

## Prompt Example
"I need a flow that monitors my Outlook inbox for emails with attachments from my manager. When such an email arrives, save the attachments to a specific SharePoint document library and send me a mobile notification with the email subject and attachment names."

## Copilot-Generated Flow Outline
- Trigger: "When a new email arrives (V3)" (Outlook), filtered for attachments and sender.
- Action: "Get attachments" to retrieve files.
- Action: "Create file" for each attachment in SharePoint.
- Action: "Send a push notification" with email subject and attachment names.

## Comments
- Copilot interprets the prompt and generates the flow draft automatically.
- Review and refine the generated flow as needed.
