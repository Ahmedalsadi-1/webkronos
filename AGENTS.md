# AGENTS.md

Development guide for agentic coding in Web - a native macOS AI browser.

## Build & Test Commands

### Build & Run
```bash
# Open project in Xcode
open Web.xcodeproj

# Or build via command line
xcodebuild -project Web.xcodeproj -scheme Web build

# Run from Xcode: ⌘R
```

### Testing
```bash
# Run all tests via Xcode: Product > Test
# Or command line:
xcodebuild test -project Web.xcodeproj -scheme Web

# Run single test:
xcodebuild test -project Web.xcodeproj -scheme Web \
  -only-testing:WebTests/WebTests/example

# Or select specific test in Xcode Test Navigator and run with ⌘U
```

## Architecture & Patterns

### MVVM with SwiftUI + Combine
- **Models/**: Data models (Tab, Bookmark) - use `struct` for value types
- **Views/**: SwiftUI views - use `struct` conforming to `View`
- **ViewModels/**: Business logic - use `class` with `@ObservableObject` and `@Published` properties
- **Services/**: Core services - use `class` with `static let shared` singleton pattern
- **AI/**: AI integration - protocol-based provider system

### Code Style Guidelines

#### Imports
```swift
import SwiftUI
import Combine
import Foundation
import WebKit
import os.log  // For structured logging
```

#### Type Definitions
- **Views**: `struct` conforming to `View`
- **ViewModels**: `class` with `@ObservableObject`
- **Models**: `struct` with `Identifiable`, `Codable`, `Hashable` as needed
- **Services**: `class` with `static let shared = Self()`
- **Protocols**: Use for abstraction (e.g., `AIProvider`, `ExternalAPIProvider`)

#### Naming Conventions
- **Types**: PascalCase (`ContentView`, `TabManager`)
- **Variables/Functions**: camelCase (`isExpanded`, `toggleExpanded()`)
- **Constants**: camelCase or uppercase (`maxTabs`, `MAX_RETRIES`)
- **Private properties**: Prefix with underscore only when needed to distinguish from public

#### State Management
```swift
// Published properties trigger UI updates
@Published var tabs: [Tab] = []
@Published var activeTab: Tab?

// Local view state
@State private var isExpanded: Bool = false
@StateObject private var bookmarkService = BookmarkService.shared
@ObservedObject var tabManager: TabManager
```

#### Error Handling
```swift
// Custom error enums with LocalizedError
enum ServiceError: LocalizedError {
    case invalidURL(String)
    case networkError(Error)

    var errorDescription: String? {
        switch self {
        case .invalidURL(let url): return "Invalid URL: \(url)"
        case .networkError(let err): return err.localizedDescription
        }
    }
}

// Throwing functions
func fetchContent() async throws -> Data

// Result types for optional failure
func processData() -> Result<String, Error>
```

#### Notification Pattern
```swift
// Post notifications
NotificationCenter.default.post(name: .customNotification, object: value)

// Observe in views
.onReceive(NotificationCenter.default.publisher(for: .customNotification)) { notification in
    // Handle notification
}

// Define in Notification.Name extension
extension Notification.Name {
    static let customNotification = Notification.Name("customNotification")
}
```

## Swift 6 Requirements

### Strict Concurrency
- Use `@MainActor` for UI operations and ViewModels
- Mark concurrent code with `async/await`
- Use `Task { }` for background work from main thread
- No data races - protect shared state with actors or `@MainActor`

### Zero Warnings/Errors Policy
- All compiler warnings must be resolved before commit
- No `!` force-unwraps without validation
- No `try!` without guaranteed success
- Use `guard let` for optional unwrapping with early returns

## Testing Patterns

```swift
import Testing
@testable import Web

struct FeatureTests {
    @Test func exampleTest() async throws {
        // Arrange
        let value = 42

        // Act
        let result = value * 2

        // Assert
        #expect(result == 84)
    }
}
```

## AI Provider System

### Protocol-Based Architecture
```swift
@MainActor
protocol AIProvider {
    var providerId: String { get }
    var displayName: String { get }
    func initialize() async throws
    func generateResponse(query: String, ...) async throws -> AIResponse
}
```

### Provider Types
- **Local**: `LocalMLXProvider` - uses Apple MLX for on-device inference
- **External**: `OpenAIProvider`, `AnthropicProvider`, `GeminiProvider` - cloud APIs

### Security
- API keys stored in `SecureKeyStorage` (Keychain)
- Privacy-first: local processing by default
- Context sharing optional for external providers

## Keyboard Shortcuts

Add new shortcuts in `WebApp.swift` → `BrowserCommands`:

```swift
Button("My Action") {
    NotificationCenter.default.post(name: .myActionRequested, object: nil)
}
.keyboardShortcut("k", modifiers: [.command, .option])
```

## Memory Management

### Tab Hibernation
- `TabHibernationManager` manages memory automatically
- Wake up hibernated tabs: `tab.wakeUp()`
- `BackgroundResourceManager` monitors memory pressure

### Memory Best Practices
- Use `[weak self]` in closure subscriptions
- Cancel Combine subscriptions in `deinit`
- Avoid retain cycles with `unowned` or `weak` references

## Project Configuration

### Xcode Settings
- Swift 6 with strict concurrency
- macOS 14.0+ deployment target
- Apple Silicon required for AI features
- No external linting tools configured (Xcode built-in warnings only)

### Directory Structure
```
Web/
├── Models/           # Data models
├── Views/           # SwiftUI views
├── ViewModels/      # ObservableObject view models
├── Services/        # Singleton services
├── AI/             # AI integration
└── Utils/          # Extensions, utilities
```
