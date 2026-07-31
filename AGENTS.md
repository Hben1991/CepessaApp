# Cepessa Agent Instructions

## SwiftUI and AppKit Policy

SwiftUI is the default and primary UI framework for Cepessa. Treat AppKit as a targeted escape hatch, not as an alternative application architecture. This policy is an architectural guardrail, not a mandate to introduce AppKit.

Use AppKit only for a small, isolated component when at least one of these conditions is true:

- A reproducible performance problem exists under realistic usage, and profiling or diagnostics indicate that SwiftUI is the relevant bottleneck.
- A required macOS behavior is unavailable, unreliable, or materially inferior in SwiftUI.
- Direct access to a native AppKit API is required.

Do not introduce AppKit because it may be smoother, because of general claims about SwiftUI performance, or because AppKit feels more native. Do not replace working SwiftUI code as a speculative optimization. When evidence is unclear, stay with SwiftUI and measure first.

Before introducing AppKit, record the following in the implementation notes, commit, or pull request:

- The concrete requirement or reproducible problem.
- The evidence or confirmed SwiftUI limitation.
- Why a SwiftUI-only solution is insufficient.
- The smallest AppKit bridge that solves the problem.

Implementation boundaries:

- Prefer a narrow `NSViewRepresentable`, `NSViewControllerRepresentable`, or focused coordinator.
- Keep domain state, business logic, and navigation in the existing application architecture. AppKit may own only transient native control state when necessary.
- Maintain a single source of truth. Do not duplicate application state between SwiftUI and AppKit.
- Do not rewrite complete screens, introduce broad AppKit dependencies, or migrate unrelated working code.
- Preserve accessibility, keyboard navigation, focus behavior, lifecycle handling, cancellation, and MainActor safety.
- When performance is the justification, validate with realistic data and compare before and after behavior.

If the AppKit bridge does not provide a clear material advantage after validation, remove it and keep the SwiftUI implementation.
