# Chapter 8 — Date & Time Functions

---

## 1) Current UTC
**Description:** Get current UTC timestamp.

```plaintext
@utcNow()
```

---

## 2) Format Date
**Description:** Format a timestamp as `yyyy-MM-dd`.

```plaintext
@formatDateTime(utcNow(), 'yyyy-MM-dd')
```

---

## 3) Date Calculations
**Description:** Add seven days to the current timestamp.

```plaintext
@addDays(utcNow(), 7)
```

**Comments:** Similar functions → `addHours()`, `addMinutes()`, `addSeconds()`, `addMonths()`, `addYears()`.

---

## 4) Date Comparison
**Description:** Check if now is after `2023-01-01T00:00:00Z`.

```plaintext
@greater(utcNow(), '2023-01-01T00:00:00Z')
```

---

## 5) Extract Components
**Description:** Get current month number.

```plaintext
@month(utcNow())
```

---

## 6) Practical — Business Day Calculation
**Description:** Calculate a due date 5 business days ahead (skip weekends).

```plaintext
@[
 varDay: dayOfWeek(utcNow());
 varAddDays: if(
   greater(varDay, 5),
   7 - varDay + 5,
   if(
     equals(varDay, 5),
     3,
     5
   )
 );
 addDays(utcNow(), varAddDays)
]
```

**Comments:** Adjusts offset based on current weekday to land on a business day.

---

## 7) Time Zone Conversion
**Description:** Convert UTC to Eastern Standard Time.

```plaintext
@convertTimeZone(utcNow(), 'UTC', 'Eastern Standard Time')
```

**Comments:** Use appropriate IANA/Windows time zone labels.
