# UniversalGlass - Backports

Optional `UniversalGlassBackports` target that forwards SwiftUI's new API names to the universal implementations.

## Quick Start

```swift
import UniversalGlass
import UniversalGlassBackports

Button("RSVP") {}
    .buttonStyle(.glassProminent)

CardView()
    .glassEffect(.regular.tint(.mint))
```

## Adding the Target

```swift
.package(url: "https://github.com/Aeastr/UniversalGlass", from: "1.0.0"),

.target(
    name: "YourApp",
    dependencies: [
        .product(name: "UniversalGlass", package: "UniversalGlass"),
        .product(name: "UniversalGlassBackports", package: "UniversalGlass"),
    ]
)
```

Keep both imports together—`UniversalGlass` provides the rendering logic, while the backport target exposes the Apple-style symbols when SwiftUI does not.

## What Gets Exposed

The target recreates the same entry points Apple ships, guarded with availability so they disappear once the native SDK provides them.

| File | Symbols |
|------|---------|
| `BackportGlassButtonStyles.swift` | `.buttonStyle(.glass)`, `.buttonStyle(.glassProminent)` |
| `BackportGlassEffect.swift` | `View.glassEffect` overloads |
| `BackportGlassContainers.swift` | `GlassEffectContainer`, `View.glassEffectUnion`, `View.glassEffectID`, `View.glassEffectTransition` |

Every symbol is marked `obsoleted: 26.0` across Apple's platforms, so you never collide with the native definitions once you build against OS 26 SDKs.

## How Forwarding Works

The backport layer is intentionally thin:

1. Each extension or global function lives behind availability gates
2. The body simply calls through to the universal API (`universalGlassEffect`, `UniversalGlassEffectContainer`, etc.)
3. Because it never reimplements behaviour, you get the same fallback routing and rendering options described in the other docs

Improvements to the universal implementations automatically flow into the backported names.

## When to Use It

Reach for the backport target when you:

- Maintain codebases that already use Apple's `glass` syntax and want them to compile against pre-OS 26 SDKs
- Build shared components that must compile on Xcode 15+ but still expose the future-facing API surface
- Want to adopt `.glass` now without wrapping every call site in `#if` checks

If you prefer to keep the explicit universal prefixes, omit the backport target—the core module still works on all supported OS versions.
