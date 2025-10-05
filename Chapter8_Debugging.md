# Chapter 8 — Debugging & Troubleshooting Expressions

---

## 1) Safe Navigation
**Description:** Avoid null reference errors when accessing nested properties.

```plaintext
triggerBody()?['customer']?['name']
```

---

## 2) coalesce()
**Description:** Provide fallback when a value is null/undefined.

```plaintext
@coalesce(triggerBody()?['name'], 'Anonymous')
```

**Comments:** Useful for robust logging and default values.
