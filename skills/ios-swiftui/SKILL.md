---
name: ios-swiftui
description: SwiftUI and Swift 6 patterns for iOS app development including MVVM, concurrency, SwiftData, networking, and App Store submission.
origin: dham-claude-code
---

# iOS SwiftUI Patterns

Use this skill for building iOS apps with SwiftUI, Swift 6 concurrency, and modern Apple frameworks.

## Project Structure

```
MyApp/
├── App/
│   └── MyApp.swift              # @main entry point
├── Features/
│   ├── Orders/
│   │   ├── OrdersView.swift
│   │   ├── OrdersViewModel.swift
│   │   └── OrderDetailView.swift
│   └── Settings/
├── Domain/
│   ├── Models/                  # Pure Swift structs/enums
│   └── Repositories/            # Protocols + implementations
├── Infrastructure/
│   ├── Network/                 # URLSession, endpoints
│   └── Persistence/             # SwiftData / CoreData
└── Shared/
    ├── Extensions/
    └── Components/              # Reusable SwiftUI views
```

## MVVM with @Observable

```swift
// Domain model
struct Order: Identifiable, Codable {
    let id: UUID
    var status: OrderStatus
    let customerId: String
    let amount: Decimal
    let createdAt: Date
}

enum OrderStatus: String, Codable, CaseIterable {
    case pending, shipped, delivered, cancelled
}

// Repository protocol for testability
protocol OrderRepository {
    func fetchOrders() async throws -> [Order]
    func updateStatus(_ orderId: UUID, status: OrderStatus) async throws
}

// ViewModel
@Observable
@MainActor
final class OrdersViewModel {
    var orders: [Order] = []
    var isLoading = false
    var error: Error?

    private let repository: any OrderRepository

    init(repository: any OrderRepository = LiveOrderRepository()) {
        self.repository = repository
    }

    func loadOrders() async {
        isLoading = true
        defer { isLoading = false }
        do {
            orders = try await repository.fetchOrders()
        } catch {
            self.error = error
        }
    }
}
```

## SwiftData Patterns

```swift
import SwiftData

@Model
final class CachedOrder {
    @Attribute(.unique) var id: UUID
    var status: String
    var amount: Double
    var createdAt: Date

    init(id: UUID, status: String, amount: Double, createdAt: Date) {
        self.id = id
        self.status = status
        self.amount = amount
        self.createdAt = createdAt
    }
}

// In App entry point
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: [CachedOrder.self])
    }
}

// In View
struct OrdersView: View {
    @Query(sort: \CachedOrder.createdAt, order: .reverse) var orders: [CachedOrder]
    @Environment(\.modelContext) private var context
}
```

## Networking

```swift
enum Endpoint {
    case orders
    case order(UUID)
    case updateOrder(UUID, OrderStatus)

    var urlRequest: URLRequest {
        var request = URLRequest(url: baseURL.appending(path: path))
        request.httpMethod = method
        if let body { request.httpBody = try? JSONEncoder.iso8601.encode(body) }
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        return request
    }
}

final class APIClient {
    func fetch<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        let (data, response) = try await URLSession.shared.data(for: endpoint.urlRequest)
        guard let http = response as? HTTPURLResponse, (200..<300).contains(http.statusCode) else {
            throw APIError.httpError((response as? HTTPURLResponse)?.statusCode ?? 0)
        }
        return try JSONDecoder.iso8601.decode(T.self, from: data)
    }
}
```

## Key Rules

- Use `.task {}` for async work (auto-cancelled on view disappear)
- Use `@Observable` instead of `ObservableObject` + `@Published` (iOS 17+)
- Never put business logic in `View.body`
- Use `actor` for shared mutable state across async contexts
- Use `#Preview` macro for SwiftUI previews
- Target iOS 17+ for full `@Observable` + SwiftData support
