# Developer Quality Guide

A reminder for every dev out there: Clean, tested, and thoughtful code isn’t a luxury it’s the baseline. Follow these practices not just to ship, but to grow.

**Better habits = better dev = better career.**

Let’s level up.

### If you want to download/print the quality guide [click here](https://docs.google.com/document/d/1WF6FF_aZ7nbHeitiHgy2y8PVomyh2KCViY6cudhdFrg)

### 1. Start with a Test Plan

Before touching code, write a point-by-point manual test plan based on the story's acceptance criteria. This becomes your checklist to validate the work.

**Ask Yourself:**

1. What exactly does success look like for this story?

2. Are there edge cases the ACs (acceptance criteria of the user story) don’t cover?

3. How will I verify every functional piece, manually?

---

### 2. Design the Implementation

Sketch out how you’ll build it. Include data models, protocols, data flow, error handling, and any necessary cleanup.

**Ask Yourself:**

1. What's the cleanest way to build this with how the current code base is?

2. What are the "risky" parts that might cause bugs or change often?

3. Do I need to modify any existing code? If yes, could that break something? If yes then how would I account for that?

---

### 3. Model Data

Be explicit about your inputs and outputs. Only return what’s necessary—and nothing more.

**Ask Yourself:**

1. What data does the caller really need?

2. Could this response shape change soon?

3. Am I leaking information unnecessarily?

---

### 4. Code the Story, End-to-End

Write the full feature flow. Don’t stop at “it works”, run through the entire user journey and edge cases using your test plan you wrote in point 1.

**Ask Yourself:**

1. Did I implement the full flow, not just the happy path?

2. Would someone new to this code get what’s going on?

3. Did I handle common failure states or retries?

---

### 5. Write Unit Tests with Purpose

Use your manual test plan and any new edge cases discovered while coding. Don't just test that it works—test that it fails correctly too.

**Ask Yourself:**

1. Am I testing both success and failure paths?

2. Did I mock/stub any third-party calls realistically?

3. Can these tests catch future regressions?

---

### 6. Try to Break It

Challenge your own code like you didn’t write it. Be aggressive and neutral. Think like a tester or even a malicious user.

**Ask Yourself:**

1. What would I do if I wanted to make this fail?

2. What areas around my code could this unintentionally impact?

3. What happens with invalid inputs, bad timing, or flaky network?

---

### 7. Review for Cleanliness and Consistency

Check naming, indentation, abstraction, DRYness, and if it fits your project’s code style and conventions, how SOLID is the code?

**Ask Yourself:**

1. Would I be proud to show this code to a new teammate?

2. Did I leave unnecessary logs, comments, or dead code?

3. Am I using naming that makes the logic self-explanatory?

---

### 8. Retest with a Fresh Mindset

Step back, come back, and test again like you’ve never seen the code. Context-switching for a few minutes helps reset bias.

**Ask Yourself:**

1. Did I miss anything because I assumed it was “obvious”?

2. Am I testing only what I built—or the system around it too?

---

### 9. Audit Dependencies & Integrations

Check external APIs, modules, and shared services. Don’t assume they work or stay the same.

**Ask Yourself:**

1. What does this code rely on externally, and is it stable?

2. What happens if that thing fails, times out?

3. Did I cover this in mocks/tests?

---

### 10. Consider Race Conditions & State

Especially in async code or UI work, ensure consistency and correctness under stress or concurrency.

**Ask Yourself:**

1. What happens if this runs twice—or not at all?

2. Could I be reading or updating stale state?

3. Is any shared state protected or isolated?

---

### 11. Leave a Trail in Your Code

If the code you wrote has tricky business logic, write code comments as to what the logic does or even in general it’s good to write comments for something that you will forget after 3 months when you look at the code

**Ask Yourself:**

1. Is this code readable/understandable to a new developer? If not then add comments

---

### 12. Do a Regression Risk Sweep

Look around the change area and beyond. Anything that could be impacted even subtly should be validated or mentioned.

**Ask Yourself:**

1. What shared components or utilities did I touch?

2. Could I have accidentally broken another flow?

---

### 13. Log Your Learnings

Capture new commands, tricks, patterns, or gotchas you discovered. Revisit often, it’s your personal changelog for growth. I use google doc app for doing this.

**Ask Yourself:**

1. What did I learn that I want to remember?

2. What went wrong and how did I fix it?

3. Could this help Future Me (or teammates) later?

---

### 14. Log Your Mistakes

Every bug you caused, every “aare yaar 🤦‍♂️” moment, every missed edge case write it down. Not to shame yourself, but to build your own pattern recognition and future immunity.

The best devs are repeat offenders until they start logging. Hold yourself accountable when you repeat the same patterns

**Ask Yourself:**

1. What did I miss that caused this bug or oversight?

2. Why didn’t I catch it sooner was it a blind spot or assumption?

3. What guardrail can I build (test, checklist, habit) to avoid it next time?
