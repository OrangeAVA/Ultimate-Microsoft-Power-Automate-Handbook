# Chapter 8 — Advanced Expression Techniques (Conceptual)

---

## 1) Composition of Functions — Word Frequency
**Description:** Count frequency of words longer than 3 characters.

```plaintext
@[
 // Pseudocode-style pipeline for clarity (conceptual)
 // 1. Normalize text
 varText: toLower(triggerBody()?['comment']);
 // 2. Split words
 varWords: split(varText, ' ');
 // 3. Filter long words
 varLongWords: filter(varWords, w => greater(length(w), 3));
 // 4. Group and count (conceptual)
]
```

**Comments:** Implement grouping/counting via **Apply to each** and array actions.

---

## 2) Dynamic Arrays — Field Extraction
**Description:** Project specific fields from items.

```plaintext
@[map(triggerBody()?['items'], i => json('{"id": ' + string(i?['id']) + ', "name": "' + string(i?['name']) + '", "email": "' + string(i?['email']) + '" }'))]
```

**Comments:** Use **Select** action to project properties.
