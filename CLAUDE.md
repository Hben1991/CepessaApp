# Cepessa

A native macOS application built with SwiftUI.

## Project Overview

- **Language**: Swift 5.0
- **Framework**: SwiftUI
- **Platform**: macOS 26.2+
- **IDE**: Xcode 16.3+

## Project Structure

```
CepessaApp/
├── Cepessa/                    # Main app source
│   ├── CepessaApp.swift        # App entry point (@main)
│   ├── ContentView.swift       # Main UI view
│   └── Assets.xcassets/        # Asset catalogs (icons, colors)
└── Cepessa.xcodeproj/          # Xcode project configuration
```

## Build Commands

```bash
# Build for Debug
xcodebuild -scheme Cepessa -configuration Debug

# Build for Release
xcodebuild -scheme Cepessa -configuration Release

# Open in Xcode
open Cepessa.xcodeproj
```

## Code Conventions

- Use SwiftUI declarative patterns
- Follow Apple's Swift API Design Guidelines
- Use `@main` for app entry point
- Prefer `VStack`, `HStack`, `ZStack` for layouts
- Use SF Symbols for icons (e.g., `Image(systemName: "globe")`)

## Architecture Notes

- **App Sandbox**: Enabled for security
- **ARC**: Automatic Reference Counting enabled
- **Concurrency**: MainActor isolation by default
- **Previews**: SwiftUI previews enabled for development

## Testing

Currently no test targets configured. When adding tests:
- Use XCTest framework
- Add test target via Xcode
- `ENABLE_TESTABILITY` is set for Debug builds
