# Copilot Prompt: Multi-Level Approval Workflow

## Context
Automate expense approvals with conditional logic for different approval levels.

## Prompt Example
"Create an expense approval flow that triggers when a new item is added to an 'Expense Claims' SharePoint list. If the amount is less than $100, automatically approve it. If it's between $100 and $1,000, send an approval request to the employee's direct manager (stored in the 'Manager' field). If it's over $1,000, first send an approval to the direct manager, and if approved, send a second approval to the finance director. Update the SharePoint item status after each approval or rejection and notify the employee of the final decision."

## Copilot-Generated Flow Outline
- Trigger: "When an item is created" (SharePoint)
- Condition: Amount < $100 → "Update item" to Approved
- Condition: $100–$1,000 → "Start and wait for an approval" (Manager)
- Condition: >$1,000 → Two-stage approval (Manager, then Finance Director)
- Actions: "Update item" for status, "Send an email" to employee

## Comments
- Copilot builds nested conditions and multi-stage approvals.
- Review approval assignments and status updates.
