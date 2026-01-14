# UniversalGlass - Effects

Glass effect modifiers that back-deploy SwiftUI's `glassEffect` API while preserving native behaviour on OS 26+.

## Quick Start

```swift
import UniversalGlass

Text("Hello")
    .universalGlassEffect(.regular.tint(.purple))
```

## API Surface

All overloads live in `UniversalGlassEffect+View.swift`:

```swift
View.universalGlassEffect(rendering:)
View.universalGlassEffect(_:rendering:)
View.universalGlassEffect(_:in:rendering:)
View.universalGlassEffect(in:rendering:)
```

The overloads accept an optional `UniversalGlass` configuration and a `UniversalGlassRendering` toggle. `UniversalGlass` bundles the fallback `Material` with an optional `Glass` value for platforms that support it.

## Runtime Routing

Each modifier performs the same availability dance:

1. When the OS ships the native API and the caller picked `.automatic` or `.glass`, SwiftUI's real `glassEffect` is invoked
2. When the caller forces `.material`—or the OS lacks `glassEffect`—the view flows through `UniversalGlassEffectModifier` instead

## Fallback Rendering

`UniversalGlassEffectModifier` handles the back-port:

- Detects whether the view lives inside `UniversalGlassEffectContainer`; if not, it renders its own background plate via `UniversalGlassFallbackBackground`
- Registers itself as a `GlassEffectParticipant` using an anchor preference, capturing geometry, the requested shape, and any union/ID metadata
- Applies a `.transition(.blur)` so insert/remove animations stay soft when native transitions are unavailable

`UniversalGlassFallbackBackground` paints the chosen `Material` and a `shadow(.drop)` to add depth.

## Built-in Configurations

`UniversalGlassConfiguration` exposes convenience cases that mirror Apple's material tiers:

| Configuration | Description |
|---------------|-------------|
| `.identity` | No effect |
| `.ultraThin` | Clear liquid glass over `.ultraThinMaterial` |
| `.thin` | Clear glass tinted slightly toward the platform background |
| `.regular` | Maps to `Glass.regular` |
| `.thick` | Deeper background tint alongside `.thickMaterial` |
| `.ultraThick` | Heavier tint to emulate Apple's deeper plates |
| `.clear` | Clear glass with ultra thin material fallback |

## Configuration Modifiers

### Tinting

```swift
.universalGlassEffect(.regular.tint(.cyan))
```

### Interactivity

```swift
.universalGlassEffect(.regular.interactive())
```

### Custom Fallback

Override the material and tint used on older OS versions:

```swift
// Custom fallback material
.universalGlassEffect(.regular.fallback(material: .thin))

// Custom fallback tint
.universalGlassEffect(.thick.fallback(material: .regular, tint: .blue.opacity(0.2)))

// Remove fallback tint
.universalGlassEffect(.thick.fallback(tint: nil))
```

### Custom Shadows

```swift
// Custom shadow
.universalGlassEffect(
    .regular.shadow(UniversalGlassShadow(color: .red, radius: 12))
)

// No shadow
.universalGlassEffect(.regular.shadow(.none))

// Default shadow (automatically applied)
.universalGlassEffect(.regular.shadow(.default))
```

Shadow presets:
- `.default` – Subtle black blur (8pt radius, 0.04 opacity)
- `.none` – No shadow

## Grouping and Unions

Two environment-driven helpers coordinate multiple views:

```swift
View.universalGlassEffectUnion(id:namespace:rendering:)
View.universalGlassEffectID(_:in:rendering:)
```

On OS 26+ these forward to SwiftUI. On earlier systems they stash metadata in `glassEffectParticipantContext`. The fallback container consumes this context to merge participants, creating a single slab of glass that "melts" neighbouring elements together.

## Transitions

`View.universalGlassEffectTransition(_:)` mirrors SwiftUI's transitions:

- `.materialize` and `.matchedGeometry` map to `.transition(.fallbackBlur)` on older systems
- `.identity` is a no-op
- Once the deployment target hits OS 26, the helper forwards directly to the platform transition APIs

Custom transition per effect:

```swift
VStack {
    Text("Slide Transition")
        .universalGlassEffect()
        .universalGlassTransition(.slide)

    Text("Scale Transition")
        .universalGlassEffect()
        .universalGlassTransition(.scale)

    Text("Blur Without Scale")
        .universalGlassEffect()
        .universalGlassTransition(.universalGlassMaterialBlurWithoutScale)
}
```

## Environment Modifiers

Force all glass effects in a view hierarchy to use a specific rendering mode:

```swift
// Force material fallback (useful for testing)
MyApp()
    .universalGlassRenderingMode(.material)

// Force glass when available
MyApp()
    .universalGlassRenderingMode(.glass)
```

## Chaining

All configuration methods are fully chainable:

```swift
Text("Fully Customized")
    .universalGlassEffect(
        .ultraThick
            .fallback(material: .thin, tint: .cyan.opacity(0.3))
            .shadow(UniversalGlassShadow(color: .blue, radius: 16))
            .tint(.purple)
            .interactive()
    )
    .universalGlassTransition(.opacity)
```

## Notes

- When using custom timing, reach for `AnyTransition.universalGlassMaterialBlur(intensity:scale:)` and apply your animation on the parent view
- `AnyTransition/universalGlassMaterialFallbackBlur` exposes the same asymmetric animation UniversalGlass uses internally
