# Chapter 8 — Case Study: Customer Journey Orchestration

---

## 1) Personalized Greeting by Time of Day
**Description:** Choose greeting based on current hour.

```plaintext
@[
 varHour: int(formatDateTime(utcNow(), 'HH'));
 if(
   less(varHour, 12), 'Good morning',
   if(less(varHour, 18), 'Good afternoon', 'Good evening')
 )
]
```

---

## 2) Days Since Signup
**Description:** Compute day difference using ticks.

```plaintext
@[
 varSignupDate: triggerBody()?['signupDate'];
 int(
   div(
     sub(
       ticks(utcNow()),
       ticks(varSignupDate)
     ),
     864000000000
   )
 )
]
```

---

## 3) Next Best Action Based on Completion Rate
**Description:** Decide messaging based on progress tiers.

```plaintext
@[
 varCompletedSteps: length(
   filter(
     triggerBody()?['onboardingSteps'],
     step => equals(step?['status'], 'completed')
   )
 );
 varTotalSteps: length(triggerBody()?['onboardingSteps']);
 varCompletionRate: div(varCompletedSteps, varTotalSteps);
 if(
   less(varCompletionRate, 0.3), 'Send Helper Email',
   if(
     less(varCompletionRate, 0.7), 'Send Progress Encouragement',
     if(
       less(varCompletionRate, 1), 'Send Almost There Reminder',
       'Send Completion Congratulations'
     )
   )
 )
]
```

**Comments:** Replace conceptual `filter` with **Filter array** if implementing directly.
