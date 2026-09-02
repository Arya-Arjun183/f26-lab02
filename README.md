# Lab 2 Starter: Availability Calculator

A small reservation component. Given a room's bookings and the day's business hours,
`AvailabilityCalculator.freeSlots` computes when the room is free. It is the code you
work in for Lab 2.

It ships with a generated test suite that passes, and a property-based test harness
(jqwik) with one example property. Everything is green. Your job in Lab 2 is to decide
whether green actually means correct.

**Read `ARCHITECTURE.md` before the code.**

## Build and test

```
mvn test
```

`mvn test` runs both files, the ordinary example-based tests (`AvailabilityCalculatorTest`)
and the property-based tests (`AvailabilityProperties`). A code-coverage report is written
to `target/site/jacoco/index.html`.

## Continuous integration

This repository has CI configured in `.github/workflows/ci.yml`. GitHub disables workflows on a
fresh fork, so enable them once on your fork (the handout shows where). After that, every
push runs `mvn test`. You will watch the gate go red when your new property finds the bug, then
green once you fix it.

## Where things are

- Component: `src/main/java/edu/cmu/cs214/availability/`
- Example-based tests: `src/test/java/edu/cmu/cs214/availability/AvailabilityCalculatorTest.java`
- Property-based tests: `src/test/java/edu/cmu/cs214/availability/AvailabilityProperties.java`
- Setup: `SETUP.md`

See the Lab 2 handout on the course page for the three milestones you show a TA.

Used: Antigravity with Gemini 3.1 Pro

## 3 Weaknesses and why high coverage did not save them

1. **Controllability Gap:** All of the tests with strict assertions (tests using assertEquals) always end with a booking that goes exactly to DAY_END (like new TimeInterval(720, DAY_END)). They never drive an input where there is free time at the end of the day, so they never put the calculator in a state where it needs to append a final free slot.
2. **Observability Gap:** The only test that has an input with free time at the end of the day is returnedSlotsNeverOverlapABooking (it passes a single booking from 600 to 660). However, its assertion is too weak because it only checks that the slots returned don't overlap the booking. Because it doesn't verify that all free time is returned, it runs the bug but fails to notice the missing slot at the end of the day.
3. **Controllability Gap:** The suite never tests the simplest edge case: a completely empty day with no bookings (an empty list). This input would have immediately exposed the bug since the method would return an empty list instead of a slot for the whole day, but this input was never driven by the tests.

**Why high coverage didn't save it:**
Code coverage tools like JaCoCo only measure the lines of code that are actually written in the file. Because the bug was an **omission** (entirely missing logic to append the remaining time at the end), there were no unexecuted lines to flag as missed. The test suite successfully executed 100% of the flawed, incomplete logic, giving a false sense of security.
