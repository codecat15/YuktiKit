# 📱 iOS Interview Preparation Roadmap for Developers

---

## 👋 Note to Developers

This document serves as a comprehensive roadmap and checklist for iOS developers preparing for interviews, especially in mid to large companies.  
It will be updated every 3–4 months to ensure all questions and topics remain current and relevant.

> **Note:** These are _questions only_ and do **not** include answers. **The goal is not to memorize answers like a parrot**, but to encourage genuine understanding. This approach helps you grow as a thoughtful and capable developer, not just someone who regurgitates material.

## If you would like to print/download this roadmap there [please use the google drive link.](https://docs.google.com/document/d/1ZnELwqC86jUSm6FboCDJ8TNTylAJIKFmAdeJ8Igj2RQ) Feel free to share this with whomever you feel might benefit from this help a fellow iOS dev.

## ✏️ From a Fellow Developer to Other

Hey there, this is **Ravi**.

iOS job market kaafi challenging hai, aur vo din door nahi, jab aapko multiple tech stack seekhne k liye company pressure banayegi.
Maine is baare mai video bhi banaya tha ki aisa kyu hai, to agar aapko job market ki mentality jaani hai to is video ko zaroor check kare

🎥 **[iOS Developer Job Market 2025](https://youtu.be/NWl-A48wYGg)**

To agar job market mai aacha perform karna hai to khud se aapki pasand ki koi bhi technology choose karo aur uske basics seekho.
Developer to aap ho hi, concepts sab same hai bus aap basic seekh lo.

> Keep learning, stay humble, and grow every day.

Always feel free to ask questions.

**Happy iCoding 🙂**

---

## Master the iOS Interview Process

Before diving into the questions below, check out this blog post that outlines the **interview process basics**:  
[Mastering iOS App Development Interviews](https://codecat15.medium.com/mastering-ios-app-development-interviews-tips-for-success-96702d70d8fd)

---

## 🛠️ 1. Language Fundamentals: Swift, OOP & POP

### Core Topics

- Value types vs Reference types

- [ARC](https://youtu.be/XAlXnApplk0)

- [Weak vs Unowned](https://youtu.be/HdhJjc7Y6iw)

- [Closures: capturing values, escaping vs non-escaping](https://youtu.be/ZP4pEHtndFE)

- Higher-order functions: [`map`](https://youtu.be/96rS9jk4YLg), `filter`, `reduce`
- `defer`

- `guard` vs `if`

- `public` vs `open`

- `map` vs `flatMap`

- Enums with associated values

- Tuples and pattern matching

- Delegate vs Closure: When will you use a delegate over a closure in your code? Explain with an example.
  > **Dev Note:** This is a trick question begin with basics but lean into Apple’s best practices.
- Why use a higher order function over for or foreach loop?

### OOP

- Classes, Inheritance

- Polymorphism, Encapsulation

- Difference between polymorphism and abstraction

  📖 [OOP Principles Simplified](https://codecat15.medium.com/object-oriented-programming-principles-simplified-22484edf42d7)

### POP

- Protocols, Associated Types, Protocol Composition

- Extensions, Default Implementations

- Protocols vs Inheritance

- [Dependency Injection vs Inversion](https://youtu.be/Byf-q5WRv7Q)

---

## 🔄 2. iOS Application Lifecycle

- App lifecycle states: Not Running, Inactive, Active, Background, Suspended

- SceneDelegate vs AppDelegate: What’s the difference?

- Lifecycle methods for push notifications and background fetch.

- Storyboards vs XIBs vs Programmatic UI

- ViewController life cycle events

- Migrating from AppDelegate to SceneDelegate

### Questions

- What happens when you tap on a notification and the app is in the background?

- How would you perform a task when the app is killed and reopened?

- How do you optimize cold start vs warm start?

- In which lifecycle method would you make your API calls and why?

- How do you handle deep links when your app supports multiple scenes?

---

## 🧩 3. UIKit Deep Dive

### UIKit Controls

- When to use `UITableView` vs `UICollectionView`

- `UITableView` performance optimization (cell reuse, prefetching)

- `UICollectionView` custom layout: compositional layout, flow layout

- Size classes, intrinsic content size, hugging & compression resistance.

- `DiffableDataSource`, prefetching, cell reuse

- Frames vs Bounds

- Lazy loading

### Navigation

- `UINavigationController`, `UITabBarController`, modals

- Deep linking & custom transitions

- Push notification and navigation: How to go to a particular screen on the tap of a push notification.

- How does [Coordinator pattern](https://youtu.be/Ji0bqURWv68) help with Navigation in iOS app

---

## 📐 4. AutoLayout, Constraints & UI Challenges

### Core Concepts

- Intrinsic Content Size, Hugging/Compression Resistance

- `NSLayoutConstraint`, `NSLayoutAnchor`

- StackViews & dynamic layouts

### Practical UI Layout Challenges

_Think of your strategy for the following think of edge cases, code management, testability_

- **Instagram Feed:** Infinite scroll, image grid, profile header

- **WhatsApp Settings Screen:** Static grouped table, profile header, actions

- **App Store Home:** CollectionView with nested horizontal scrolls

- **Zomato Home:** Search bar, location selector, restaurant grid

- **Google Maps 'From' and 'To':** Overlay views, autocomplete input, dynamic resizing

---

## 🌐 5. Network Layer Design

### Core Networking

- `URLSession`, `URLRequest`, `URLResponse`

- `Codable`, JSON decoding, error handling

- Retry logic, caching, exponential backoff

- [Async/Await](https://youtu.be/8iBK0QxyjgU)

  🎥 [Networking Series](https://youtube.com/playlist?list=PLb5R4QC2DtFuXr4177KQ2lIXOkqwq97a4&feature=shared)

### Error Handling

- Reachability

- HTTP status codes handling

- Timeouts, cancellations

### Libraries

- Alamofire vs Moya

- Combine / async-await

- Detecting internet connectivity

### Questions

- How/Where do you store JWT tokens in your application?

- If the Auth token expires and the api returns 401 status code what would be your strategy?

---

## 🏗️ 6. App Architecture

### Patterns

- **MVC:** Simplicity vs code tangling with drawbacks

- **MVVM:** ViewModel responsibilities

- **VIPER:** When to use in large-scale enterprise apps

- **Clean Architecture:** Separation of concerns with MVVM

- **RxSwift:** Powers complex async flows with reactive streams but comes with a learning curve.

- **Rib:** Uber’s plugin-friendly architecture focused on scalability, testability, and feature isolation

🎥 Guides:

- [MVVM Series](https://youtube.com/playlist?list=PLb5R4QC2DtFs8CsVZztGSJ0TM8J2URfg7&feature=shared)
- [MVC Series](https://youtube.com/playlist?list=PLb5R4QC2DtFsGyVTm1DB4WpgIJQn_g2cb&feature=shared)

### Reactive Patterns

- RxSwift vs Combine
- TCA: Modularity vs boilerplate

### Questions

- Why choose MVVM over MVC?

- When is VIPER justified?

- Compare Clean Architecture with MVVM

- Why is TCA overengineered?

- When would you choose one architecture over the other?

- Is every architecture scalable?
  Justify your answer with an example if yes why if no then what are the roadblocks that makes an architecture non-scalable.

---

## 🧱 7. Design Patterns in iOS

- [Factory](https://youtu.be/faeoSJhN-Ko): Object creation

- Singleton Pattern: Shared managers (e.g., LocationManager)

- Repository: Decoupling data source

- Feature Flags: Toggle logic via remote config

- Decorator, Adapter, Strategy

---

## 🧭 8. Code Principles

- SOLID Principles (Single Responsibility, Open/Closed, etc.)

- DRY (Don't Repeat Yourself)

- KISS (Keep It Simple, Silly)

- YAGNI (You Aren't Gonna Need It)

  🎥 [Code Principles Guide](https://youtube.com/playlist?list=PLb5R4QC2DtFuC7WzUd5bJP3tdVsUcI8E8&feature=shared)

---

## 🧪 9. Testing in iOS

### Types

- Unit tests: `XCTest`, ViewModel mocking

- Integration tests

- UI testing: `XCTest UI`, snapshot tests

### Tools

- XCTest, Quick/Nimble

- Cuckoo, Mockingbird

## Questions

- [Why do we need test cases in project?](https://youtu.be/YA-Ur61XzRI)

---

## 🧍‍♂️ 10. Behavioral Interview Prep

### STAR Format

**S**ituation  
**T**ask  
**A**ction  
**R**esult

Read about the [star format here](https://codecat15.medium.com/mastering-ios-app-development-interviews-tips-for-success-96702d70d8fd)

### Scenarios

- Leading a deadline-bound feature

- Resolving production performance issues

- Refactoring legacy codebase to adopt an architecture

---

## 🧠 11. Mobile System Design

### Sample Questions

- **Offline Notes App:** CoreData/SQLite, sync strategy, background tasks

- **Real-Time Chat App:** Websockets, state management, delivery guarantees

- **Push Notification Architecture:** FCM/APNs, custom payload handling, deep linking

- **Pagination & Lazy Loading:** TableView/CollectionView optimization

- **Modular App Structure:** Feature modules, dynamic framework loading

---

## 🔐 12. iOS Security

- **HTTP vs HTTPS:** TLS, MITM attacks

- **Keychain Storage:** Tokens, sensitive info

- **Biometric Auth:** FaceID/TouchID usage

- Secure Enclave

- SSL Pinning

- Jailbreak detection via code

- Static/dynamic analysis tools for penetration testing

🎥 [Security Guidelines in iOS (Hindi)](https://youtu.be/RkYoNBqWBkk)

---

## 🔁 13. CI/CD Pipeline

### Build Automation

- Xcode Cloud, GitHub Actions, Bitrise, AzureDevOps

- Fastlane for signing/testing/builds

### Quality Gates

- Linting, test coverage

- Crashlytics, Sentry integration

---

## 💾 14. Local Persistence & Databases

### Database

> Note: Learn how to setup each database as well

- **CoreData:** For structured and ORM (Object relation)

- **SQLite:** Manual schema control (nobody uses this much as it's too much work but it's good to know why nobody uses this)

- **Firebase:** Realtime sync, offline support (this is paid)

- **SwiftData:** Modern, built on CoreData with declarative syntax

- **UserDefaults:** For settings and lightweight flags

  🎥 [Core Data Playlist](https://youtube.com/playlist?list=PLb5R4QC2DtFsJwFLf9119aefRnEMDp4aB&feature=shared)

### Questions

- CoreData vs Firebase
- How do you handle data migration?
- How to manage large data sets effectively.
- Why is CoreData an object graph and not a ORM (Object relation mapping)
- Core data [heavyweight](https://youtu.be/GgC4UfyI64M) vs [lightweight migration](https://youtu.be/YPEX9twI-zU)

---

## 🧵 15. Threading

### Concepts

- [Threading](https://youtu.be/OKfv_0FvEf0)

- Main vs Background thread

- Queues, Serial vs Concurrent queues

- `sync` vs `async`

- Race conditions and deadlocks

- Global Queue vs QoS

- DispatchGroup and TaskGroup

- Thread sanitizer

🎥 [Threading Guide](https://youtube.com/playlist?list=PLb5R4QC2DtFuSE3wpJAqvFRbal5-7sAwr&feature=shared)

### Questions

- What’s the difference between `@MainActor` and `DispatchQueue.main.async`?

- What is structured concurrency and how is it different from GCD?

- [How does Actor prevent race conditions?](https://youtu.be/USz17wtIKG8)

- Explain thread safety while updating a shared array in GCD?

- How do you ensure UI updates are thread-safe?

- [What is a deadlock explain with an example?](https://youtu.be/mToZ03k7oEI)

- What’s the difference between a thread and a queue?

- When would you prefer OperationQueue over GCD?

---

## 🧰 16. SwiftUI

### Navigation, Forms, Layouts

- NavigationStack, NavigationLink, Sheet

- [List](https://youtu.be/OLEijRSANpw), LazyVStack

- Form, Picker, DatePicker

- ViewBuilder, GeometryReader, PreferenceKey

🎥 [SwiftUI Basics](https://youtube.com/playlist?list=PLb5R4QC2DtFuRFJ35uPMhpY90s0VMNR-i&feature=shared)

---

## 🔄 17. Data Flow

### Combine + Swift Concurrency

- State, [@Binding](https://youtu.be/g3VOWHYTHzU), [@ObservedObject](https://youtu.be/GG0z7LpNyq4), [@EnvironmentObject](https://youtu.be/oJg5CJSXs6w)

- async/await with SwiftUI and Task

- @Sendable, @MainActor, TaskGroup

- Isolated state, thread-safe classes

- Background processing using .subscribe(on:) and .receive(on:)

🎥 [CoreData ToDo App](https://youtube.com/playlist?list=PLb5R4QC2DtFs8alYSS_cqNjLTDHucbuFl&feature=shared)

---
