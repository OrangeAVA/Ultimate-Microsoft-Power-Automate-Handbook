# Chapter 8 — Expressions (Basics)

This file collects foundational Power Automate expression examples from Chapter 8: **Expressions and Functions in Power Automate**.

---

## 1) Basic Expression Examples
**Description:** Quick examples to evaluate current time, combine strings, get length, and perform addition.

```plaintext
@utcNow()
@concat('Hello', ' ', 'World')
@length('Power Automate')
@add(5, 3)
```

**Comments:** These showcase the `@` expression prefix and simple function-based structure.

---

## 2) Combine Expressions with Static Text
**Description:** Embed expressions inside static messages for dynamic output.

```plaintext
The current time is: @utcNow()
Order total: @formatNumber(variables('orderAmount'), 2)
Customer @concat(variables('firstName'), ' ', variables('lastName')) has placed an order
```

**Comments:** Power Automate auto-converts expression results to strings when mixed with text.

---

## 3) Try It Yourself — Basic Expression
**Description:** Compose action input that says hello.

```plaintext
@[concat('Hello, ', 'Power Automate!')]
```

**Comments:** Use a manual trigger → **Compose** → paste the expression above; output should be `Hello, Power Automate!`.
