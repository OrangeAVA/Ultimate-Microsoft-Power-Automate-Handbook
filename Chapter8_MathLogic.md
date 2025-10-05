# Chapter 8 — Mathematical & Logical Functions

---

## 1) Arithmetic
**Description:** Basic operations.

```plaintext
@add(5, 6)
@sub(10, 4)
@mul(7, 3)
@div(20, 5)
@mod(17, 5)
```

---

## 2) Statistical
**Description:** Aggregate calculations.

```plaintext
@min(3, 7, 10)
@max(3, 7, 10)
@average(3, 7, 10)
@sum(createArray(3, 7, 10))
```

---

## 3) Logical & Conditional
**Description:** Evaluate comfort based on temperature & humidity.

```plaintext
@[if(
  and(greater(variables('temperature'), 18), less(variables('humidity'), 70)),
  'Comfortable',
  'Uncomfortable'
)]
```

---

## 4) Comparisons
**Description:** Equality and inequality examples.

```plaintext
@equals(5, 5)
@greater(10, 7)
@greaterOrEquals(10, 10)
@less(3, 9)
@lessOrEquals(3, 3)
```

---

## 5) Random Number
**Description:** Random number between 1 and 100.

```plaintext
@rand(1, 100)
```

---

## 6) Practical — Price Calculator (Tiered Discount + Tax)
**Description:** Compute final price after discount and tax.

```plaintext
@[
 varBase: float(triggerBody()?['basePrice']);
 varDiscount: if(
   greater(varBase, 1000), 0.20,
   if(greater(varBase, 500), 0.10, 0.05)
 );
 varAfterDiscount: sub(varBase, mul(varBase, varDiscount));
 varTaxRate: 0.08;
 add(varAfterDiscount, mul(varAfterDiscount, varTaxRate))
]
```

**Comments:** Adjust discount tiers by base price; then apply tax.

---

## 7) Pro Tip — Division by Zero
**Description:** Guard against zero denominator.

```plaintext
@[if(equals(variables('denominator'), 0), 'N/A', div(variables('numerator'), variables('denominator')))]
```
