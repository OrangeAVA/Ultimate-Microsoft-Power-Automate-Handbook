# Chapter 8 Code Artifacts — README


This folder contains markdown and JSON artifacts with extracted Power Automate expressions from Chapter 8. Each file groups related snippets, with descriptions, complete code, and comments to help you paste them into your flows or document them in your GitHub repository.

## Files
- Chapter8_Expressions.md — Basic expressions
- Chapter8_StringFunctions.md — String functions & examples
- Chapter8_DateTime.md — Date/time handling
- Chapter8_CollectionsArrays.md — Arrays & collection utilities
- Chapter8_MathLogic.md — Math & logical functions
- Chapter8_Conversion.md — Type & format conversions
- Chapter8_Variables.md — Variables with expressions
- Chapter8_ConditionalLogic.md — Conditional patterns
- Chapter8_AdvancedTechniques.md — Advanced/conceptual patterns
- Chapter8_Debugging.md — Safe navigation & coalesce
- Chapter8_CaseStudies_InvoiceProcessing.md — Invoice processing case
- Chapter8_CaseStudies_CustomerJourney.md — Customer journey case
- Chapter8_ExpressionLibrary.json — Reusable expression library

## Usage
1. Copy any snippet into the **Expression** tab of an action (e.g., **Compose**, **Condition**, etc.).
2. Replace placeholders (e.g., `variables('amount')`, `triggerBody()?['field']`) with your actual flow context.
3. For conceptual snippets using `filter`/`map`, implement with **Filter array** / **Select** actions.

## Notes
- Some multi-line blocks (`@[ ... ]`) are shown in conceptual, readable form. In Power Automate, build complex logic across **Compose** actions and variables.
- Validate expressions in the designer; adjust for your environment, locale, and time zone.
