---
description: Review iOS Swift/SwiftUI code for architecture, concurrency safety, and App Store readiness. Invokes ios-reviewer agent.
---

Review the iOS Swift/SwiftUI code in this project.

Use the `ios-reviewer` agent to:

1. Check MVVM separation (no business logic in View body)
2. Verify Swift Concurrency usage (@MainActor, actors, Sendable)
3. Review @Observable / @StateObject usage (correct pattern for iOS target)
4. Check networking (protocol-backed, async/await, no URLSession in View)
5. Validate error handling (no silently swallowed errors)
6. Identify memory leaks (missing cancellation, retain cycles)
7. Check for force unwraps (`!`) in production code

Report findings as: **Crash Risk**, **Concurrency Bug**, **Architecture**, **Performance**, **Maintainability**.
