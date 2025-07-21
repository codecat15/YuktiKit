# 📁 iOS Architecture Boundaries & Folder Structure

This documentation contains architecture boundaries and sample folder structure. The architecture boundaries can be used with any architecture whatsoever the core principals are always going to remain the same.

The folder structure however is pure subjective to your situation and application so take your best pragmatic guess on what should go where.

### Ask yourself

- If this is the right place for a `DomainContract` or a `FieldValidation`?
- Does it make maintainence easy?

---

## 🚨 Code Smell Alert

If your project has [SwiftLint](https://github.com/realm/SwiftLint) and you've started disabling rules left and right just to make the linter shut up **congratulations, you're not enforcing code quality, you're actively bypassing it**.

> You're not "cleaning up warnings" you're giving dirty code a VIP pass into production.

You're not just a developer anymore. You're now:

- The guardian of legacy bugs
- The enabler of inconsistency
- The accidental villain in your own app's architecture story

**SwiftLint exists to enforce discipline, not to be silenced when it's inconvenient.** If a rule needs disabling, then perhaps you need to take a pause and think of why this warning comes and how you can make it right.

---

## 📁 Sample Folder structure

```
App/
├── Core/
│   ├── Constants/
│   ├── DeviceServices/
│   ├── Extensions/
│   ├── Models/
│   │   ├── Domain/
│   │   └── DTO/
│   │   └── Entity/
│   │   └── Client/
│   ├── Validation/
│   └── BusinessLogic/
│
├── Data/
│   ├── API/
│   ├── Persistence/
│   ├── Repository/
│   └── Keychain/
│
├── ThirdParty/
│   └── PackageAdapters/
│
├── Presentation/
│   ├── Features/
│   │   ├── FeatureA/
│   │   ├── FeatureB/
│   │   └── CommonUI/
│   └── Navigation/
│
└── Tests/
    ├── Unit/
    └── Integration/
```

---

### 📁 Folder Purpose Overview

| **Folder**                        | **Purpose**                                                                 |
| --------------------------------- | --------------------------------------------------------------------------- |
| `Core/Constants/`                 | App-level constants (e.g., API URLs, keys, hardcoded values, flags)         |
| `Core/DeviceServices/`            | Isolated access to device capabilities like accelerometer, wallet, etc.     |
| `Core/Extensions/`                | Common extensions on native types like `String`, `Date`, `UIView`, etc.     |
| `Core/Models/Domain/`             | Repository app models clean representations, not tied to API/UI             |
| `Core/Models/DTO/`                | Codable structs that represent external API contracts (data transfer)       |
| `Core/Models/Entity/`             | Repository database tables                                                  |
| `Core/Models/Client/`             | Represents models that are used in the UI                                   |
| `Core/Validation/`                | Centralized validation logic, rules, and reusable error types               |
| `Core/BusinessLogic/`             | Business rules (e.g., discount engine, calculations), reusable and testable |
| `Data/API/`                       | Network layer (clients, endpoints, interceptors)                            |
| `Data/Persistence/`               | Local storage: CoreData, SQLite, SwiftData                                  |
| `Data/Repository/`                | Abstract data sources (each vertical owns its own repo)                     |
| `Data/Keychain/`                  | Secure storage utilities for tokens, passwords, etc.                        |
| `ThirdParty/PackageAdapters/`     | Wrappers over external SPM libraries to isolate dependencies                |
| `Presentation/Features/`          | Modular UI per feature, broken down by screen or module                     |
| `Presentation/Features/CommonUI/` | Reusable UI components like buttons, cells, loading indicators              |
| `Presentation/Navigation/`        | Navigation logic, coordinators, routers etc                                 |
| `Tests/Unit/`                     | Unit tests for small, deterministic components                              |
| `Tests/Integration/`              | End-to-end and multi-layer tests                                            |

---

## 1. Data Models Separating DTO, Domain, Entity, and UI Contracts

### Problem

**Many developers make these mistake:**

- Take the JSON from the API → dump it directly into UI or persistence

- Have no separation between what's stored, what's shown, and what's transferred

- This **breaks abstraction**, causes **tight coupling**, and makes refactoring painful when the backend or UI changes.

- We are breking down the model into categories so that we know which model belongs where.

- Data models should be `structs` unless there's a need to use class `then` use class.

- If you are working with a complex data model then try breaking it down into multiple structs for cohesion.

---

## Data Layering

Let’s define the layers and what goes where:

| Layer                          | Purpose                                    | Usage                                             |
| ------------------------------ | ------------------------------------------ | ------------------------------------------------- |
| **DTO (Data Transfer Object)** | Matches API JSON 1:1 (`Codable`)           | `API Layer` (decoding/encoding network responses) |
| **Entity Model**               | Represents how data is stored in database. | `Persistence Layer` (CoreData, Realm, SQLite)     |
| **Domain Model**               | Represents `Repository` data model         | `Repository`, `UseCase`, `BusinessLogic`          |
| **UI/View Model**              | Represents data for UI consumption         | `Presentation Layer`                              |

---

### 📐 Data Flow Example

#### Imagine this API Response:

```json
{
  "id": "123",
  "user_name": "batman",
  "created_at": "2024-01-01T10:00:00Z"
}
```

---

### 1️⃣ DTO API Representation

```swift
struct UserDTO: Codable {
    let id: String
    let user_name: String
    let created_at: String
}
```

- Used only inside the API layer
- No business logic
- Field names match JSON

---

### 2️⃣ Entity Persistence Model

If you're using CoreData, Firebase etc

```swift
final class UserEntity {
    let id: String
    let username: String
    let createdAt: Date
}
```

> Used in DB-only

---

### 3️⃣ Domain Model Pure Business Model

```swift
struct User {
    let id: String
    let username: String
    let createdDate: Date
}
```

> No DB, no JSON. Just the shape of what your repository actually works with.

If the Database returns the `UserEntity` then we should convert the `UserEntity` into `User` type and return it from the repository.
The repository should never return a database entity to the UI layer, this breaks abstraction.

## Sometimes the repository contract i.e. `User` could return extra information like `createdDate` which is not used by the UI and hence this extra information should not be propagated from the `Repository` layer to the `ViewModel` or `UseCase` class.

---

### 4️⃣ UI Model Tailored for Screens

```swift
struct UserCardModel {
    let displayName: String
    let joinedDateText: String
}
```

> This model is often derived in ViewModel or mapper logic.
> It exists to **shape domain data** into something directly usable in SwiftUI/Views.

---

## 📁 Folder Structure

```
Core/
└── Models/
    ├── Domain/
    │   └── User.swift
    ├── DTO/
    │   └── UserDTO.swift
    ├── Entity/
    │   └── UserEntity.swift
    └── Client/
        └── UserCardModel.swift
```

---

## 🔁 Conversion/Mappers

Each layer has its own translation logic, typically via extensions or mappers.

### DTO → Domain

```swift
extension UserDTO {
    func toDomain() -> User {
        return User(
            id: id,
            username: user_name,
            createdDate: ISO8601DateFormatter().date(from: created_at) ?? Date()
        )
    }
}
```

---

### Domain → UI

```swift
extension User {
    func toCardModel() -> UserCardModel {
        return UserCardModel(
            displayName: "@\(username)",
            joinedDateText: createdDate.formatted(date: .abbreviated, time: .omitted)
        )
    }
}
```

---

## 🧠 Why All This?

| ❌ Anti-Pattern                     | ✅ Better Way                              |
| ----------------------------------- | ------------------------------------------ |
| Reusing `DTO` everywhere            | Only in API layer                          |
| Using CoreData model directly in UI | Abstract via Domain → UI conversion        |
| Having business logic in `UserDTO`  | Keep domain logic in `User` only           |
| Binding UI directly to API response | Use `UserCardModel` for presentation logic |

---

## 🧩 Rule of Thumb

| Concern        | What to Use |
| -------------- | ----------- |
| Network        | `DTO`       |
| Storage        | `Entity`    |
| Business Logic | `Domain`    |
| UI Binding     | `UI Model`  |

---

## 🧠 2. Orchestration Layer – ViewModel / Interactor

### 🎯 Purpose

This layer’s job is to **coordinate data flow between app boundaries**:

- Validation
- Repository (data access)
- Transformation for UI
- State preparation for views
- Trigger navigation (via abstraction, not UI references)

It should be **framework-agnostic**, **UI-independent**, and **testable in isolation**.

## ⚒ Where Does This Layer Live?

| Architecture | Orchestration Component | Notes                                                 |
| ------------ | ----------------------- | ----------------------------------------------------- |
| **MVVM**     | `ViewModel`             | Should be testable, UI-agnostic                       |
| **MVP**      | `Presenter`             | Sometimes pushes to View via protocol                 |
| **VIPER**    | `Interactor`            | Pure business logic; communicates via output protocol |
| **TCA**      | `Reducer` (`Feature`)   | State + actions orchestrated via switch-case          |

## ✅ What the Orchestration Layer Should Do

The following is an example using `MVVM` as it's simple and easy to grasp.

```swift
final class TransferMoneyViewModel {
    @Published var errorMessage: String?
    @Published var transferSuccess: Bool = false

    func transfer(to email: String, amount: Double) {
        if let error = EmailValidator.validate(email) {
            errorMessage = error.localizedDescription
            return
        }

        if let error = AmountValidator.validate(amount) {
            errorMessage = error.localizedDescription
            return
        }

        Task {
            do {
                let domainModel = TransactionRequest(email: email, amount: amount)
                let result = try await transactionRepo.transfer(domainModel)
                transferSuccess = result.success
            } catch {
                errorMessage = error.localizedDescription
            }
        }
    }
}
```

### ✅ What this is doing:

- Validating inputs
- Constructing domain request model
- Talking to repository
- Zero knowledge of `UIView`, `SwiftUI`, `UITableView` etc.

---

## ❌ Anti-pattern: ViewModel Touching UIKit

```swift
final class TransferViewModel {
    func transfer(...) {
        // No.
        let alert = UIAlertController(...)
        UIApplication.shared.windows.first?.rootViewController?.present(alert, animated: true)
    }
}
```

> This is toxic coupling. Your tests can’t mock this. Your logic can’t run outside UIKit.
> **You’ve now created a ViewController that lives inside your ViewModel.**

---

## ❌ Anti-pattern: ViewModel Doing Transformation Logic Inline

```swift
func toDisplayModel(user: User) -> UserCardModel {
    return UserCardModel(
        displayName: "@\(user.username)",
        joinedDateText: DateFormatter.localizedString(...)
    )
}
```

- Move this to a **mapper**, or **extension**

## Opinion

Use an extension if it's simple OR create a mapper class because the same model in some case can have polymorphic behavior

---

## ❌ Anti-pattern: ViewModel Talking to Another ViewModel

```swift
// 💀 Just stop
final class PaymentViewModel {
    let accountVM = AccountViewModel()

    func checkBalance() {
        let balance = accountVM.balance
    }
}
```

> ViewModels should not depend on each other. If you need shared state extract it into a shared service, state store, or use composition at a higher level.

---

## ✅ Clean Code Guidelines

| ✅ Allowed                         | ❌ Forbidden                                                        |
| ---------------------------------- | ------------------------------------------------------------------- |
| Calling validation                 | Accessing UIKit elements directly                                   |
| Invoking repository methods        | Referring to navigation controller                                  |
| Emitting `@Published` or callbacks | Showing alerts or modals directly                                   |
| Mapping Domain → UI via mappers    | **Holding references to another ViewModel**                         |
| Structuring actions/events         | Importing `UIKit`, `SwiftUI`, or `Combine` directly in domain logic |

---

## 📦 Folder Structure (Where this lives)

```
Presentation/
└── Features/
    └── Transfer/
        ├── TransferView.swift        // UI
        ├── TransferViewModel.swift   // Orchestration
        └── TransferViewMapper.swift  // Domain → UI transformation
```

---

## 🧪 Testing the Orchestration Layer

Because it’s framework-free, testing it is straightforward:

```swift
func test_transferWithInvalidEmail_shouldFail() {
    let vm = TransferMoneyViewModel()
    vm.transfer(to: "", amount: 100)

    XCTAssertEqual(vm.errorMessage, "Email is required")
}
```

---

## 🚨 TL;DR This Layer **Can** and **Cannot**

| This Layer **Can**                             | This Layer **Cannot**                            |
| ---------------------------------------------- | ------------------------------------------------ |
| Orchestrate validation, data, state updates    | Touch UIKit, SwiftUI, or navigation stack        |
| Emit UI-bound data via `@Published` or closure | Trigger alerts, modals, or view changes directly |
| Transform domain models to UI models           | Contain formatting logic tied to views           |
| Call business logic or use cases               | Know where in the app the user is                |

---

## 📦 3. Repository Layer Data Access Boundaries

### ❓ What is a Repository?

A Repository in iOS architecture acts as an **abstraction over data sources** like APIs, local databases (Core Data, Firebase), or Keychain. It exposes a clean, unified interface to the rest of the app (typically the ViewModel or Use Case or any orchestrator layer), **hiding implementation details and source juggling**.

### ✅ Responsibilities

- Fetch, persist, and transform data (from network, disk, or secure storage).
- Expose reactive or async methods (`async/await`, Combine publishers, etc.).
- Map Data Transfer Objects (DTOs) to Domain Models.

### ❌ Anti-Pattern: Repositories talking to each other

**Just. Don't. Do. It.**
Repositories are boundary layers. If one repository reaches into another, it breaks modularity, increases coupling, and creates a maintenance nightmare.

Think of this like **microservices reaching into each other’s database** that's an invitation to chaos.

Do not call Repository directly from the view controller or any layer that deals with views as it exposes implementation details

#### ❌ Bad Example

```swift
final class OrderRepository {
    private let userRepo = UserRepository()

    func createOrder(for userId: String) {
        // This is bad: OrderRepository is now tightly coupled to UserRepository
        let userDetails = userRepo.getUserDetails(id: userId)
    }
}
```

```swift
final class ProductViewController {
    private let productRepo = ProductRepository()

    func getProduct(for productId: int) {
        // This is bad: ViewController knows about the implementation details as to what is needed to get the product
        let userDetails = productRepo.getProduct(id: productId)
    }
}
```

#### ✅ Good Example

```swift
final class OrderViewModel {
    let orderRepo: OrderRepository
    let userRepo: UserRepository

    init(orderRepo: OrderRepository, userRepo: UserRepository) {
        self.orderRepo = orderRepo
        self.userRepo = userRepo
    }

    func placeOrder(userId: String) async throws {
        let user = try await userRepo.getUserDetails(id: userId)
        try await orderRepo.createOrder(for: user)
    }
}
```

### Note: `OrderRepository` and `UserRepository` should be protocols to adhere to Dependency inversion principal of SOLID.

### 💡 Why Isolation Matters

- **Unit testable**: You can mock each repo independently.
- **Refactor-safe**: Changing data structure in one module won’t break others.
- **Domain-focused**: Each repo serves a specific feature (auth, products, orders).

### 🧱 Folder Structure

```
Data/
└── Repository/
    ├── AuthRepository.swift
    ├── ProductRepository.swift
    └── UserRepository.swift
```

Each repository should:

- Have a clearly defined protocol.
- Be injected via constructor (dependency injection).
- Communicate only with lower layers (API, DB, etc).

---

### ✅ Sample: Product Repository

```swift
// MARK: - Contract
protocol ProductRepository {
    func getProducts() async throws -> [Product]
}

// MARK: - Implementation
final class ProductRepositoryImpl: ProductRepository {
    private let apiClient: ProductAPIClient

    init(apiClient: ProductAPIClient) {
        self.apiClient = apiClient
    }

    func getProducts() async throws -> [Product] {
        let dto = try await apiClient.fetchProducts()
        let products = dto.map { $0.toDomain() }
        // save the data in the database here if needed else return
        return products
    }
}
```

### 📌 Rules of Thumb

| ❌ Don't                | ✅ Do                              |
| ----------------------- | ---------------------------------- |
| Call another repository | Use composition outside repo       |
| Return DTOs             | Map to domain models               |
| Hard-code dependencies  | Use DI (constructor / environment) |

---

## ✅ 4. Validation Layer – Centralized, Reusable, and Isolated

**📌 Purpose**

The validation layer is responsible for:

- Ensuring inputs meet business rules and formatting requirements.

- Providing a consistent, reusable mechanism for error reporting.

- Keeping validation logic out of ViewModels, UseCases, and UI layers.

> 💡 Principle: This layer should do only validation no API calls, no DB queries, no side effects. Just pure logic.

### 🧱 Folder Structure

```
Core/
└── Validation/
    ├── FieldValidators/
    │   ├── EmailValidator.swift
    │   ├── AmountValidator.swift
    │   └── PromoCodeValidator.swift
    ├── ObjectValidators/
    │   ├── TransactionRequestValidator.swift
    │   └── InviteUserRequestValidator.swift
    ├── ValidationError.swift
    └── Validatable.swift
```

## 🧪 Case 1: ViewModel Function With One Parameter

### 🎯 Example: Validate a promo code

```swift
func applyPromo(code: String)
```

### ✅ Validation Code

## 📦 ValidationError.swift

```swift
enum ValidationError: Error {
    case field(String, reason: String)
}
```

---

## 🔑 Validatable.swift

```swift
protocol Validatable {
    func validate() -> [ValidationError]
}
```

```swift
func applyPromo(code: String) {
    if let error = PromoCodeValidator.validate(code) {
        // Show error on UI
        return
    }
    // Proceed to apply promo
}
```

### PromoCodeValidator

```swift
enum PromoCodeValidator {
    static func validate(_ code: String) -> ValidationError? {
        guard !code.isEmpty else {
            return .field("promoCode", reason: "Promo code cannot be empty")
        }

        guard code.count >= 4 else {
            return .field("promoCode", reason: "Promo code must be at least 4 characters")
        }
        return nil
    }
}
```

> ✅ Lightweight, no need to create a `PromoCodeRequest`, just validate and move on.

---

## 🧪 Case 2 ViewModel Function With a Complex Request Object

### 🎯 Example: Creating a money transfer

```swift
func transferMoney(request: TransactionRequest)
```

### TransactionRequest

```swift
struct TransactionRequest {
    let recipientEmail: String
    let amount: Double
    let note: String?
}
```

### ✅ Validation

> Code for `EmailValidator` and `AmountValidator` is given below

```swift
extension TransactionRequest: Validatable {
    func validate() -> [ValidationError] {
        var errors: [ValidationError] = []

        if let error = EmailValidator.validate(recipientEmail) {
            errors.append(error)
        }

        if let error = AmountValidator.validate(amount) {
            errors.append(error)
        }

        if let note = note, note.count > 140 {
            errors.append(.field("note", reason: "Note cannot exceed 140 characters"))
        }

        return errors
    }
}
```

### 🧠 ViewModel Usage

```swift
func transferMoney(request: TransactionRequest) {
    let errors = request.validate()
    guard errors.isEmpty else {
        // Present errors to user
        return
    }

    // Proceed with transaction
}
```

---

## 🧪 Case 3 – Shared Field Validation Across Different Requests

### 🎯 Example: Invite a user (email required again)

```swift
struct InviteUserRequest {
    let email: String
    let role: String
}
```

Instead of rewriting email validation:

```swift
extension InviteUserRequest: Validatable {
    func validate() -> [ValidationError] {
        var errors: [ValidationError] = []

        if let error = EmailValidator.validate(email) {
            errors.append(error)
        }

        if role.isEmpty {
            errors.append(.field("role", reason: "Role must be selected"))
        }

        return errors
    }
}
```

✅ We reuse `EmailValidator.validate(_:)` here just like in `TransactionRequest`.

---

## Field Validators (Reusable Across the App)

### EmailValidator.swift

```swift
enum EmailValidator {
    static func validate(_ input: String) -> ValidationError? {
        guard !input.isEmpty else {
            return .field("email", reason: "Email is required")
        }

        let regex = #"^\S+@\S+\.\S+$"#
        if !NSPredicate(format: "SELF MATCHES %@", regex).evaluate(with: input) {
            return .field("email", reason: "Invalid email format")
        }

        return nil
    }
}
```

### AmountValidator.swift

```swift
enum AmountValidator {
    static func validate(_ input: Double) -> ValidationError? {
        guard input > 0 else {
            return .field("amount", reason: "Amount must be greater than 0")
        }

        if input > 10_000 {
            return .field("amount", reason: "Amount exceeds transfer limit")
        }

        return nil
    }
}
```

---

## Recap – Validation Strategy

| Situation                          | Strategy                                                |
| ---------------------------------- | ------------------------------------------------------- |
| One simple input (`String`, `Int`) | Use standalone field validators                         |
| Multiple fields (object)           | Let object conform to `Validatable` and validate itself |
| Common fields (email, amount)      | Create reusable stateless validators                    |
| Optional fields                    | Guard and validate only when present                    |

---

Alright, let’s dive into **Section 4: SPM Best Practices** — and we’re going to keep this **realistic**, **pragmatic**, and **opinionated**, just the way you like it.

---

## 📦 5. Swift Package Manager (SPM) – Usage Guidelines & Best Practices

### 🎯 Why This Matters

SPM is a powerful tool, but unmanaged SPM dependencies quickly turn your app into:

- A fragile spaghetti monster of transitive dependencies
- A "version hell" scenario during Xcode upgrades
- A security or bloat risk (unused or shady packages)

## ✅ Golden Rules of SPM Usage

| ✅ Do                                                     | ❌ Don’t                                              |
| --------------------------------------------------------- | ----------------------------------------------------- |
| Only use a package if it solves a **non-trivial** problem | Add a package for something you can write in 10 lines |
| Document why the package is used                          | Leave the team guessing what problem it solves        |
| Periodically audit dependencies (license, size, activity) | Treat SPM dependencies as permanent                   |

---

# Opinion

To keep things simple I prefer directly using the third party classes but over the period of time I learned that if the class name changes I have to modify 10 swift classes which is a royal pain in the neck and hence I tend to create `adapters` which are nothing but wrappers around third party and this way I am just updating one class than 10. This keeps third-party code from bleeding into your repository or orchestration layer,.

## 🧱 Folder Structure for Isolation

```
ThirdParty/
└── PackageAdapters/
    ├── Analytics/
    │   ├── AnalyticsProvider.swift        // Internal protocol
    │   └── FirebaseAnalyticsAdapter.swift // Implementation
    └── Networking/
        └── AlamofireAdapter.swift
```

## 🔄 Example: Wrapping a Third-Party SDK

### 🎯 You’re using Firebase Analytics — but you don’t want it everywhere

#### Step 1: Internal Protocol

```swift
protocol AnalyticsProvider {
    func track(event: String, params: [String: Any]?)
}
```

#### Step 2: Adapter

```swift
import FirebaseAnalytics

final class FirebaseAnalyticsAdapter: AnalyticsProvider {
    func track(event: String, params: [String: Any]?) {
        Analytics.logEvent(event, parameters: params)
    }
}
```

#### Step 3: Use in App Code

```swift
final class CheckoutViewModel {
    let analytics: AnalyticsProvider

    init(analytics: AnalyticsProvider) {
        self.analytics = analytics
    }

    func placeOrder() {
        // ...
        analytics.track(event: "checkout_started", params: nil)
    }
}
```

> 🧪 Bonus: This makes the analytics layer mockable for testing, and swappable if you want to switch providers later.

---

## 🧰 When Not to Use a Package

| Problem                  | Write It Yourself Instead                   |
| ------------------------ | ------------------------------------------- |
| Toasts/snackbars         | Simple SwiftUI overlay or `UIView.animate`  |
| String trimming/slugging | One-liner extensions                        |
| Date formatting          | Use `DateFormatter`                         |
| Simple loaders/spinners  | Use `ActivityIndicator` in UIKit or SwiftUI |
| Wrapping HTTP calls      | Use `URLSession` + async/await + decodable  |

---

## 🛠 Versioning Best Practices

```swift
.package(
    url: "https://github.com/Moya/Moya.git",
    .upToNextMajor(from: "15.0.0")
)
```

✅ **Explicit versioning** ensures:

- You don’t get breaking changes on a `git pull`
- CI behaves consistently
- Xcode upgrades don’t nuke your project

---

## 🔍 Periodic Maintenance

- **Licenses** (especially GPL or closed-source)
- **Last commit** (Is it abandoned?)
- **Binary size impact**
- **Compile time impact**

---

## 🧠 Rule of Thumb – When to Use SPM

| Scenario                                   | Should You Use SPM?    |
| ------------------------------------------ | ---------------------- |
| Adds real value (e.g., CombineExt, Lottie) | ✅ Yes                 |
| Used across multiple features              | ✅ Yes                 |
| You can build it in < 1 hour               | ❌ No                  |
| Has heavy dependencies you don’t control   | ❌ Avoid if possible   |
| Used in 1 place only                       | 🟡 Wrap it and isolate |

---

## 📜 TL;DR

> **Treat third-party code like radiation.**
> It’s powerful, but you keep it in lead casing and use it only when necessary.

---

## 🧰 6. Utilities – The Good, the Bad, and the Maintainable

### 🎯 What Are Utilities?

Utilities are reusable, pure functions, non-domain-specific logic or helpers that make your codebase DRY and cleaner. These can include:

- **Formatting helpers** (dates, currency, strings, averages)
- **Network utilities**
- **File managers**
- **App-wide operations** (e.g., retry logic, debounce, throttling)

> Think of utilities as **stateless functions** or **helpers** that serve _multiple features_ without knowing about _any_.

---

## 🧱 Suggested Folder Structure

```
Core/
└── Utilities/
    ├── Network/
    │   └── HTTPHelper.swift
    ├── Formatters/
    │   ├── DateFormatterFactory.swift
    │   └── CurrencyFormatter.swift
    ├── Files/
    │   └── FileCleanupUtility.swift
    └── Math/
        └── RoundingUtility.swift
```

---

## ✅ Good Practices

### 🔁 1. Use When Reused

**Rule of thumb:** If you're copy-pasting something **more than twice**, turn it into a utility.

```swift
struct RoundingUtility {
    static func roundToTwoDecimals(_ value: Double) -> Double {
        return Double(round(100 * value) / 100)
    }
}
```

### 🧼 2. Keep It Pure

Utilities should be:

- Stateless
- Side-effect free
- Easily testable

Avoid injecting services or using app state inside utility functions.

---

### ✅ Good Example: DateFormatter Factory

```swift
struct DateFormatterFactory {
    static let iso8601: DateFormatter = {
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ssZ"
        return formatter
    }()
}
```

Use it across the app via:

```swift
let date = DateFormatterFactory.iso8601.date(from: dateString)
```

✅ Reusable

✅ Centralized

✅ No coupling to UI, domain, or data layers

---

## ❌ Bad Practices

### 1. Utility Extensions with No Boundaries

```swift
extension String {
    var isOnlyLetters: Bool {
        return self.range(of: "^[A-Za-z]+$", options: .regularExpression) != nil
    }
}
```

> Looks cute. **Until** someone uses it to validate usernames, product codes, or passwords and now you're in maintenance hell with **one regex to rule them all**.

### 🚫 Why This Is Bad

- No context of where it should be used
- Easy to misuse or overreach
- **Global scope leaks everywhere**
- Not testable in isolation
- You can’t tell where it was intended to live or why

#### 🧠 Better Alternative:

```swift
struct ValidationUtility {
    static func containsOnlyLetters(_ input: String) -> Bool {
        return input.range(of: "^[A-Za-z]+$", options: .regularExpression) != nil
    }
}
```

Now you can **mock, test, isolate**, and even evolve that function without killing the whole app.

---

### 2. Massive `Utils.swift` Files

```swift
// Utils.swift
func showToast(_ message: String) { ... }
func convertToJSON(_ obj: Any) -> String? { ... }
func roundUp(_ value: Double) -> Int { ... }
```

> Congratulations, you’ve just created a **dumping ground** OR **GOD Utilites.swift**.

- Impossible to test
- No discoverability
- Can't be auto-injected or modularized

---

## 🧠 Key Principles

| ✅ Do                                | ❌ Don’t                                    |
| ------------------------------------ | ------------------------------------------- |
| Group utilities by category          | Dump everything into one mega-file          |
| Keep them pure and side-effect free  | Read app state or UI from inside a utility  |
| Use structs or static helpers        | Abuse global extensions on `String`, `Date` |
| Test utilities in isolation          | Hide logic in hard-to-reach places          |
| Avoid references to UIKit or SwiftUI | Never put toasts, alerts, or views in utils |

---
