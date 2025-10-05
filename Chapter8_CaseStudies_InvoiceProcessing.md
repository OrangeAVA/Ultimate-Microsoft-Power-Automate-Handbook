# Chapter 8 — Case Study: Invoice Processing System

---

## 1) Calculate Total with Tax
**Description:** Derive tax and final total from parsed invoice data.

```plaintext
@[
 varSubtotal: float(outputs('Parse_Invoice')?['subtotal']);
 varTaxRate: 0.07;
 varTax: mul(varSubtotal, varTaxRate);
 add(varSubtotal, varTax)
]
```

---

## 2) Determine Approval Route
**Description:** Route based on total amount thresholds.

```plaintext
@[
 if(
   greater(variables('invoiceTotal'), 10000), 'Director',
   if(greater(variables('invoiceTotal'), 1000), 'Manager', 'Automatic')
 )
]
```

**Comments:** Adjust thresholds/roles per business rules.
