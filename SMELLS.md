# reservation-service: Smells and One Fix

Fill in each section. One section per milestone. Keep it short and specific. Point at files
and methods, not adjectives.

---

## Milestone 1: Three smells

Three smells, each in a different part of the module. For each one, fill in all five parts.

### Smell 1

**The smell.** God Class

**Classic or agent-specific.** Classic. ReservationManager owns room registration, booking creation/cancellation, conflict detection, pricing, availability queries, caching, notifications, and presentation formatting.

**Where in the code.** src/reservationManager.ts:30, especially createBooking() at line 57. That method validates, detects conflicts, calculates prices, generates IDs, persists data, and sends notifications. formatReceipt(), formatDailySummary(), and dispatchNotification() demonstrate additional unrelated responsibilities.

**The principle it violates.** Cohesion. The class has many independent reasons to change: booking policy, pricing rules, persistence, notification behavior, caching, and output formatting.

**What it makes expensive.** Changing one concern requires modifying and retesting the central class. For example, changing the confirmation-message format risks also changing the public receipt because dispatchNotification() uses formatReceipt() as the email body. Adding another booking workflow would also require editing createBooking(), risking unrelated pricing, storage, and notification behavior.

### Smell 2

**The smell.** Excessive Coupling / Shotgun Surgery.

**Classic or agent-specific.** Classic smell. Reporting is coupled to the booking service’s internal pricing rules because it independently recalculates booking prices.

**Where in the code.** ReservationManager.calculatePrice() and applyDiscounts() in src/reservationManager.ts:140, duplicated by ReportGenerator.priceOf() in src/reportGenerator.ts:103. Both files separately define the premium surcharge, long-booking discount, and evening discount.

**The principle it violates.** Low coupling. Pricing policy should live in one place.

**What it makes expensive.** Any pricing change requires coordinated edits in both files. For example, changing the evening discount from 5% to 10% requires updating both sets of constants and calculations. If only ReservationManager changes, new bookings receive the new price while ReportGenerator.revenue() silently recomputes the old price, producing revenue reports that disagree with Booking.priceCents.

### Smell 3

**The smell.** Hidden dependency

**Classic or agent-specific.** Classic smell. The dependency was concealed through a direct call to global state rather than being introduced by agent-generated duplication or scaffolding.

**Where in the code.** QueryCache.get() and QueryCache.set() in src/cache/queryCache.ts:18. Both call Date.now() directly, although the clock is not represented in the constructor or method parameters.

**The principle it violates.** Controllability. Time should be supplied through an abstraction or injectable clock when it controls behavior.

**What it makes expensive.** Testing expiration behavior requires manipulating global fake timers or actually waiting. Tests near an expiration boundary can become nondeterministic. Supporting a different time source would require editing QueryCache instead of supplying another clock implementation.

---

## Milestone 2: One small fix

One fix, behavior preserved, suite green, zero test edits.

**Which smell you attacked.** And why that one.

**What changed.** Files and methods you touched, and what the code does differently now.

**What you deliberately did not touch.** Name the scope line you drew and why you drew it
there. "I ran out of time" is not a scope line.

**How you know behavior is preserved.** Point at the suite, say what it actually covers, and
say what it would not catch.

---

## Milestone 3: Two proposals and one false positive

One proposal for each milestone 1 smell you did not fix.

### Proposal A (not coded)

**The problem.** Name it.

**The decomposition.** What are the pieces, what does each own, and where do the rules live?

**One cost.** Something this actually costs. "No real downside" is not a cost.

### Proposal B (not coded)

**The problem.**

**The decomposition.**

**One cost.**

### The thing that looks smelly but is fine

**What it is.** File and method.

**Why it is fine.** Defend it with properties of the code, not with its line count.

**What would flip your verdict.** Name the change that would turn this into a real problem.
