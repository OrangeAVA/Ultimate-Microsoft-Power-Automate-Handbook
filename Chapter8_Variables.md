# Chapter 8 — Variables & Expressions

---

## 1) Access a Variable
**Description:** Refer to a variable's current value inside expressions.

```plaintext
@variables('myVar')
```

---

## 2) Complex Object in a Variable (Example)
**Description:** Store a nested object for simpler downstream access.

```plaintext
@[
 json('{
  "name": "Sample Customer",
  "contact": {
    "email": "sample@example.com",
    "phone": "1234567890"
  },
  "preferences": {
    "newsletter": true,
    "notifications": ["email", "sms"]
  }
 }')
]
```

**Comments:** Initialize type **Object**; update via **Set variable**.

---

## 3) Incremental Processing (Conceptual)
**Description:** Pattern to batch results and reset counters.

```plaintext
// Initialize: counter = 0 (Integer), results = [] (Array)
// For each item:
//  - Process item → result
//  - Append to results
//  - Increment counter
//  - If counter == 100 → persist results and reset counter
```

**Comments:** Use **Append to array variable**, **Increment variable**, and conditional checks.
