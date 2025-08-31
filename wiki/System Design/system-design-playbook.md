# iOS App System Design Playbook

This document acts as a **reference guide** and **template** for designing and documenting iOS app modules and features.  
It ensures consistency, maintainability, and scalability across the codebase while keeping the designs approachable for all developers.

## Checkout some examples using the following playbook

1. [Food Delivery App System design](./Examples/food-delivery-app.md)

---

## 1. Purpose of this playbook

- **What this is:** A practical guideline for designing and documenting iOS modules/features.
- **Who should read it:** iOS developers, architects, QA, and reviewers.
- **Why it exists:**
  - Promote **consistent designs** across modules.
  - Improve **onboarding speed** for new developers.
  - Ensure the app remains **scalable, testable, and maintainable**.
- **When to use it:**  
  Use this document whenever:
  - You design a **new module or feature**.
  - You refactor **existing code**.
  - You integrate **external SDKs or libraries**.

---

## 2. How to Use this playbook

- Start here when you receive a **new feature request**.
- Follow the sections step by step to structure your module design.
- Use diagrams and tables wherever needed for clarity.
- By the end, your design should clearly explain:
  - **What the module does**.
  - **How it fits in the overall app architecture**.
  - **How it meets functional and non-functional requirements**.

---

## 3. Module Design Template

This template defines what to include for every feature or module.

### 3.1. Module Overview

**What to include:**

- Module name and purpose.
- Short summary of what the feature does.

**Why it matters:** Sets the context before diving into implementation details.

### 3.2. Goals & Non-Goals

**What to include:**

- Business and technical objectives.
- Explicit non-goals to prevent scope creep.

**Why it matters:** Helps everyone understand boundaries and avoids unnecessary work.

### 3.3. High-Level Architecture

**What to include:**

- Chosen architectural pattern (MVVM, TCA, VIPER, etc.) and **the reasoning behind it**.
- Module dependencies and boundaries.
- Diagram showing component interactions.

> > ⚠️ CAUTION: Developers must tend to avoid doing trend driven architecture selection and choose the right tool for the job after doing a detailed analysis of the requirement. Failing to do so can cost the project in the future. Do not make the project a playground for something you want to learn

**Why it matters:** Visual representation reduces ambiguity and eases onboarding.

### 3.4. Data Flow & State Management

**What to include:**

- Where the source of truth lives.
- How data moves between layers.
- Caching, synchronization, and concurrency strategy.

**Why it matters:**
Avoids data duplication, race conditions, and inconsistencies.

### 3.5. Persistence Plan

**What to include:**

- What data is stored locally and where (Core Data, SQLite, UserDefaults, Keychain).
- Migration and eviction policies.

**Why it matters:**
Keeps storage predictable and avoids unexpected data loss.

### 3.6. Security & Privacy

**What to include:**

- Sensitive data handling.
- Secure storage strategy.
- Logging policies to avoid PII leaks.

**Why it matters:**
Prevents vulnerabilities and ensures compliance with privacy rules.

### 3.7. Testing Strategy

**What to include:**

- Unit tests, UI tests, and snapshot tests.
- Mocking approach and test data setup.
- Coverage targets.

**Why it matters:**
Ensures confidence in production releases.

### 3.8. Performance Guidelines

**What to include:**

- Expected load times, scrolling FPS, memory ceilings.
- Known heavy operations and mitigation strategies.
- Profiling checklist.

**Why it matters:**
Keeps the app smooth and resource-efficient.

### 3.9. Risks & Open Questions

**What to include:**

- Known technical or architectural risks.
- Unresolved dependencies or external blockers.

**Why it matters:**
Surfaces potential issues early.

### 3.10. Decision Log (Optional)

**What to include:**

- Key technical decisions made during design.
- Links to ADRs (Architectural Decision Records).

**Why it matters:**
Provides a historical record for future reference.

---

# 4 Data flow diagrams

## Why diagrams help

- **Shared mental model:** A diagram lets new teammates see the whole picture in seconds.
- **Faster reviews:** Reviewers can spot missing pieces and risky couplings without reading every paragraph.
- **Boundary clarity:** Cleanly shows what talks to what (and what _shouldn’t_).
- **Future-proofing:** When code evolves, the diagram highlights where the design is drifting.

## What “good” diagram looks like

- **Few boxes, strong labels:** Features/domains as boxes, responsibilities as short nouns, arrows with verbs (“loads,” “updates,” “queues”).
- **Scope-appropriate:** High-level for module design, deeper sequence for critical flows (checkout, payments).
- **Truth over beauty:** Accurate beats pretty. This is a tool, not wall art.

## When to include which diagram

- **High-level component map:** Use when introducing a module; shows features, shared services, and dependencies.
- **Sequence flow:** Use for critical paths (e.g., “Place an Order”). Enumerate steps 1→N.
- **State lifecycle:** Use when entities change over time (Cart, Order). Show states and transitions.
- **Navigation map:** Use when user flows is complex (Discovery → Menu → Cart → Checkout).

## Minimum viable data flow (no drawing required)

- **Write it as steps:**
  1. User action → 2) ViewModel intent → 3) Service/UseCase → 4) Repo/Store → 5) Network/DB → 6) State update → 7) UI renders
- **Call out ownership:** Who is the source of truth? Who reconciles conflicts?
- **Note failure handling:** Timeouts, retries, recovery UI.

## Anti-patterns

- **Kitchen-sink diagrams:** Dumping all 10-20 classes in one single diagram, nobody reads that
- **Unlabeled arrows:** If an arrow doesn’t say _what_ or _why_, it’s noise add a text box that says what it does.
- **Stale artifacts:** A wrong diagram is worse than none. Update or delete.

---

## 5. Developer Checklist

Before submitting your design for review, confirm:

- [ ] Module has a clear **purpose and scope**.
- [ ] Architecture diagram is included.
- [ ] Data flow diagrams and persistence strategy documented.
- [ ] Security and privacy risks addressed.
- [ ] Testing strategy defined.
- [ ] Performance guidelines considered.

---
