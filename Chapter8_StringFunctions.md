# Chapter 8 — String Functions

This file includes common string-manipulation expressions.

---

## 1) concat()
**Description:** Combine multiple strings into one.

```plaintext
@concat('Hello', ' ', 'World')
```

**Comments:** Result → `Hello World`.

---

## 2) substring()
**Description:** Extract portion of a string from a starting index and length.

```plaintext
@substring('Power Automate', 6, 8)
```

**Comments:** Result → `Automate`.

---

## 3) replace()
**Description:** Replace all occurrences of a substring.

```plaintext
@replace('Power Platform', 'Platform', 'Automate')
```

**Comments:** Result → `Power Automate`.

---

## 4) indexOf()
**Description:** Find starting position of a substring.

```plaintext
@indexOf('Power Automate', 'Auto')
```

**Comments:** Result → `6`.

---

## 5) toLower() / toUpper()
**Description:** Convert text case.

```plaintext
@toLower('Power Automate')
@toUpper('Power Automate')
```

**Comments:** Results → `power automate` and `POWER AUTOMATE`.

---

## 6) Practical — Name Formatting
**Description:** Proper-case first & last names from trigger body.

```plaintext
@[concat(
  toUpper(substring(triggerBody()?['firstName'], 0, 1)),
  toLower(substring(triggerBody()?['firstName'], 1)),
  ' ',
  toUpper(substring(triggerBody()?['lastName'], 0, 1)),
  toLower(substring(triggerBody()?['lastName'], 1))
)]
```

**Comments:** Capitalizes initial letters, lowercases remaining characters.

---

## 7) Try It Yourself — Email Username Extractor
**Description:** Extracts the part before `@` from an email input.

```plaintext
@[substring(
  triggerBody()?['text'],
  0,
  indexOf(triggerBody()?['text'], '@')
)]
```

**Comments:** Test with different email addresses; consider error-handling if `@` is missing.
