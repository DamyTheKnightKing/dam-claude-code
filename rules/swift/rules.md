---
description: Swift and SwiftUI coding standards for iOS app development.
---

# Swift Rules

## Always

- Use `@Observable` (iOS 17+) instead of `ObservableObject` + `@Published`
- Mark ViewModels `@MainActor` to ensure UI updates on main thread
- Use `.task {}` modifier for async work in Views (auto-cancels on disappear)
- Use `actor` for shared mutable state accessed from multiple async contexts
- Define networking behind a protocol for testability
- Use `Sendable` on types crossing actor boundaries

## Never

- Business logic in `View.body`
- Force unwraps (`!`) in production code — use `guard let` or `if let`
- `DispatchQueue.main.async` (use `@MainActor` or `.receive(on: .main)`)
- `URLSession` calls directly in Views
- Silent error swallowing (`catch {}` with empty body)
- `@StateObject` for injected dependencies (use `@State` with `@Observable`)

## Architecture

- MVVM: View → ViewModel (@Observable) → Repository (protocol) → Implementation
- Dependency injection via initializer (not singleton access)
- One ViewModel per screen; shared state via `@Environment` or parent ViewModel
- Feature folders: group View + ViewModel + subviews by feature, not by type
