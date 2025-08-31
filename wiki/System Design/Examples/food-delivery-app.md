# Food delivery app — System Design

# Goal: A practical, review-ready design for browsing restaurants, selecting items, building a cart, and placing an order.

## Refer the [system design playbook](../system-design-playbook.md) if you have a question on any of the points. We are going to start from the section **3.1 of the playbook**

---

## 3.1. Module Overview

- **Name:** Food Ordering
- **Purpose:** Let users discover restaurants, browse menus, customize items, add to cart, check out, pay, and track their order.
- **Scope:** Discovery, Menu, Cart, Checkout, Payments (SDK-based), Order Tracking.
- **Out of scope:** Courier app, merchant tooling, payment processing backend.

---

## 3.2. Goals & Non-Goals

**Goals**

- Smooth, reliable ordering under variable networks.
- Accurate pricing (items, modifiers, taxes, fees, tip).
- Exactly-once order submission with clear recovery if something fails.
- Reorder from history; apply promos; schedule orders.

**Non-Goals**

- Native payment handling.
- Inventory management for merchants.
- Complex loyalty engines (basic promo codes only for v1).

---

## 3.3. High-Level Architecture

> **Note:** MVVM is used here **as an example** to demonstrate how **UseCases** fit into an architecture.  
> The term **UseCase** comes from **Domain-Driven Design (DDD)**, where it represents an **encapsulation of business rules and domain logic**.  
> The idea is to **start with one architectural pattern**, evaluate it against your needs, and pivot if it doesn't hold up for your use case.

## **Why Start with a Pattern**

When designing a system, you need a **starting point** to organize responsibilities and boundaries.

- Begin with a pattern like **MVVM** (or TCA, VIPER, Clean Architecture, etc.).
- Build a feature or two, test assumptions, and evaluate:
  - Does it keep the code **maintainable**?
  - Does it make the app **testable**?
  - Does it handle **scaling and complexity** well?
- If not, **adapt or switch** to another approach architecture is a tool, not a rule.

> **Rule** I would strongly not recommend _tanks_ like TCA or VIPER for something that's easily achievable by less work and zero cyclomatic complexity.

## **Why UseCases with MVVM**

| Without UseCases                                         | With UseCases                                                |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| ViewModel becomes a **“God class”** with too much logic. | ViewModel delegates logic → smaller, testable ViewModels.    |
| Business rules scattered across multiple ViewModels.     | **Centralized, reusable domain logic** in one place.         |
| Harder to test because ViewModel depends on UI context.  | **UseCases are independent** and easily testable without UI. |

---

## 3.4. Data Flow & State Management

**Source of truth**

- **CartStore (actor):** The single source of truth for cart line items, modifiers, computed totals, and checkout readiness.
- **OrderStore (actor):** Tracks order placement lifecycle and status updates.
- **Menu cache:** Local database copy (menu, prices, availability timestamps) with TTL (Time to live).

**Concurrency model**

- Swift Concurrency; actor-isolated state for CartStore and OrderStore to prevent race conditions (e.g., double-add on fast taps).
- Long-running calls (pricing, availability) are cancelable; UI keeps optimistic state while server confirms.

**Textual flow: Add Item to Cart**
I wish I could add a diagram here but I have added code which would give an idea of what's happening in these points

1. User taps “Add to cart” with selected modifiers.
2. ViewModel does necessary validation and sends `AddItemIntent(itemId, modifiers)` to **CartUseCase**.
3. **PricingService** computes provisional (subtotals, taxes, discounts) totals locally; **CartStore** updates state.
4. In parallel, **AvailabilityService** confirms current price/stock with server.
5. If server differs, **CartStore** reconciles and ViewModel shows the updated price.

## **CartViewModel**

```swift
// CartViewModel.swift
final class CartViewModel: ObservableObject {
    private let addToCart: AddItemToCartUseCase

    init(addToCart: AddItemToCartUseCase) {
        self.addToCart = addToCart
    }

    func addToCart(itemId: String, modifiers: [String]) {
        Task { @MainActor in
            do {
                // 1. Validate that the item exists in the menu via MenuRepository.
                //    If invalid, throw an error.

                // 2. Validate that selected modifiers are allowed for this item.
                //    If not, throw a ValidationError.

                // Use factory pattern or other creation pattern to create the object of AddItemToCartUseCase
                try await addToCart.execute(itemId: itemId, modifiers: modifiers)
            } catch {
                // TODO: Handle error gracefully (e.g., show a banner or alert)
            }
        }
    }
}
```

## AddItemToCartUseCase

```swift
// AddItemToCartUseCase.swift
final class AddItemToCartUseCase {
    func execute(itemId: String, modifiers: [String]) async throws {

        // 1. Update the CartStore optimistically:
        //    - Add item to cart.
        //    - Calculate provisional totals locally via PricingService.

        // 2. In the background, confirm availability and price with AvailabilityService.
        //    If the server price or availability differs:
        //       - Reconcile the cart to match server data.
        //       - Update CartStore totals.
        //       - Persist the updated snapshot.
        //       - Optionally, notify the ViewModel to display a non-blocking banner
        //         (e.g., “Price updated” or “Item unavailable”).
    }
}

```

## With this design you can already tell which layer is handling what and how and if any layer is doing too much or needs refactoring.

**Textual flow: Place Order (happy path)**

1. CheckoutViewModel assembles payload from **CartStore**, address, timeslot, promo, and payment token.
2. Generates **clientOrderId** for idempotency.
3. **OrderUseCase** calls backend `CreateOrder` with idempotency key; receives `orderId`.
4. **OrderStore** persists order locally with `status = pending`.
5. Payment SDK confirms charge; backend flips order to `confirmed`.
6. User navigates to Order Tracking; periodic polling or push updates status (accepted → preparing → out for delivery → delivered).

## **CheckoutViewModel**

```swift
final class CheckoutViewModel: ObservableObject {
    private let placeOrder: PlaceOrderUseCase
    private let cartStore: CartStore
    private let orderStore: OrderStore

    init(placeOrder: PlaceOrderUseCase, cartStore: CartStore, orderStore: OrderStore) {
        self.placeOrder = placeOrder
        self.cartStore = cartStore
        self.orderStore = orderStore
    }

    func placeOrder() {
        Task { @MainActor in
            do {
                // Use factory pattern or other creation pattern to create the object of PlaceOrderUseCase
                try await placeOrder.execute()
                // TODO: Navigate to order tracking screen
            } catch {
                // TODO: Show error banner or alert on failure
            }
        }
    }
}
```

## PlaceOrderUseCase.swift

```swift
final class PlaceOrderUseCase {
    private let cartStore: CartStore
    private let orderStore: OrderStore

    init(cartStore: CartStore, orderStore: OrderStore) {
        self.cartStore = cartStore
        self.orderStore = orderStore
    }

    func execute() async throws {
        // 1. Assemble payload from CartStore and other inputs:
        //    - Items in cart
        //    - Delivery address
        //    - Selected timeslot
        //    - Applied promo code
        //    - Payment token from SDK

        // 2. Generate a unique clientOrderId for idempotency:
        //    - Prevents duplicate orders if request is retried.

        // 3. Call backend API to create the order:
        //    - Pass payload + clientOrderId.
        //    - Receive back an orderId from the server.

        // 4. Persist the order locally via OrderStore:
        //    - Save orderId with status = pending.

        // 5. Confirm payment through the Payment SDK:
        //    - Handle approval, 3-D Secure, or declines.
        //    - Wait for backend to flip order status to confirmed.

        // 6. Transition user to Order Tracking screen:
        //    - Tracking updates come via push notifications or polling.
        //    - Status moves through accepted → preparing → out for delivery → delivered.
    }
}
```

**Technical reason:** Now some of you may think that the **PlaceOrderUseCase** is handling too many responsibilities, which I agree but this is by design because If the VM assembles payloads or generates keys, every screen must copy that logic. The **PlaceOrderUseCase** centralizes it and makes it reusable, what we can do is maybe create helper classes that can do this.

**Error handling (Place Order)**

- If network fails after backend receives the request: retry **lookup by clientOrderId** before resubmitting.
- If payment fails: keep cart; surface retry with a different method; record failed attempt telemetry.
- If inventory/price changed: prompt user to accept updated total or edit cart.
- Log exception using any telemetry client that your company approves of.

**Conflict resolution**

- Server is **pricing authority**. Client shows optimistic totals but reconciles on checkout.
- Cart merges are last-write-wins per line item, serialized through **CartStore**. In short whatever price the server has we need to respect the same via **CartStore**

---

## 3.5. Persistence Plan

- **Local DB (SQLite/GRDB or Core Data):** Restaurants, menus, categories, items, modifier groups, recent orders, and lightweight restaurant “cards.”
- **Keychain:** Tokens/credentials, payment SDK keys.
- **UserDefaults:** Feature flags, lightweight preferences (recent filters), last-used address id.
- **Eviction:** LRU by restaurant access; TTL on menus (e.g., 24h) with ETag; hard cap ~200MB for images + data.
- **Migrations:** Lightweight schema migration with checksum; fail-closed (log, rollback, request fresh data).

---

## 3.6. Security & Privacy

- **Payments:** Use PCI-compliant SDK (e.g., Apple Pay / payment provider). App never handles raw PANs.
- **PII:** Names, addresses, and phone numbers are encrypted at rest (DB file encryption) and never logged.
- **Transport:** TLS everywhere; optional certificate pinning for payment endpoints.
- **Screenshots/obscure:** Hide sensitive views in App Switcher during payment entry.
- **Fraud & abuse:** Server-side rate limits; device fingerprint from SDK; basic jailbreak detection (best-effort).
- **Analytics:** Redact PII; use event ids, not strings with user data.

---

## 3.7. Testing Strategy

**Unit**

- ViewModels: filters, sorting, add/remove item flows, promo application logic.
- UseCases: idempotent order submission, pricing math (items, modifiers, taxes, fees, tips).
- Repositories: caching behavior, ETag/TTL, version fallbacks.

**UI (XCUITest)**

- Happy paths: browse → add → checkout → pay (sandbox) → track.
- Edge: price change on checkout, item out of stock, network flakiness during payment.

**Contract/Integration (with mocks or staging)**

- Pricing round-trip (ensure client and server agree within 1¢).
- Idempotency: duplicate submit must yield same `orderId`.
- Order status stream: polling interval, push notification payload shape (validated without PII).

**Non-functional**

- Load tests on Restaurant list and Menu rendering with large payloads.
- Snapshot tests for cart and checkout UI to catch regressions.

**Test data**

- Seed restaurants (small, medium, huge menus).
- Known promos (percent, fixed, minimum-basket).
- Payment sandbox tokens (approved, declined, 3-D Secure).

---

## 3.8. Performance Guidelines

- **Startup:** p95 cold start ≤ 1.8s; warm ≤ 700ms.
- **Discovery List:** first contentful paint ≤ 800ms with cached cards; lazy image loading; pagination 20–30 items.
- **Menu:** initial render ≤ 1s with cached data; progressive load modifiers on demand.
- **Cart:** recompute totals in < 50ms on device-class baseline.
- **Memory:** p95 ≤ 350MB on iPhone 12; image cache capped; purge on background.
- **Network:** retries with jittered backoff; cache menus aggressively; avoid chatty calls in checkout.

**Profiling checklist**

- Instruments: Time Profiler (menu render), Allocations (cart updates), Network (checkout), Energy (tracking screen).
- Watch for main-thread blocking in image decoding and JSON parsing.

---

## 3.9. Risks & Open Questions

- **Promo/tax complexity:** Jurisdiction rules are tricky; unit tests + server truth mandatory.
- **Availability drift:** Menus change mid-session; reconcile often; UX for “item unavailable.”
- **Payment edge cases:** 3-D Secure flows, partial authorizations, timeouts.
- **Scheduling:** Time slot accuracy vs restaurant prep time; need SLA from backend.
- **Address validation:** International formats, apartments, gate codes; consider SDK for validation.
- **Order tracking:** Push vs polling; partners may have delayed status updates.
- **Accessibility debt:** Complex modifier UIs can be hard for VoiceOver — needs early audits.

---

## Developer Checklist (for this feature)

- [ ] Clear ownership of cart/order state (actors).
- [ ] Idempotent order submission with `clientOrderId`.
- [ ] Pricing reconciliation path implemented and UX-approved.
- [ ] Menu caching (TTL + ETag) with version fallback.
- [ ] Payment sandbox tests pass (approve/decline/3DS).
- [ ] Performance budgets met on target devices.
- [ ] Logging redacts PII; no PAN handling in app.

---
