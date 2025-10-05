# Copilot Prompt: Scheduled Data Consolidation

## Context
Automate weekly data consolidation and reporting using Copilot.

## Prompt Example
"Create a flow that runs every Monday at 8 AM, exports data from three different SharePoint lists ('Sales-North,' 'Sales-South,' and 'Sales-West'), combines the data into a single Excel file with separate worksheets for each region, and saves it to a SharePoint document library called 'Weekly Reports.' Then send the Excel file to the sales management team via email."

## Copilot-Generated Flow Outline
- Trigger: "Recurrence" (every Monday at 8 AM)
- Actions: "Get items" for each SharePoint list
- Actions: "Initialize variable" (array) and "Append to array variable" for data
- Actions: "Create Excel file" with dynamic worksheets
- Action: "Create file" in SharePoint
- Action: "Send an email" with Excel attachment

## Comments
- Copilot handles connectors, loops, and file creation logic.
- Review dynamic content and worksheet creation in the generated flow.
