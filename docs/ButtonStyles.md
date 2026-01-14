# UniversalGlass - Button Styles

`PrimitiveButtonStyle` helpers that mirror SwiftUI's `.glass` APIs while keeping the real styles when they exist.

## Quick Start

```swift
Button("Action") { }
    .buttonStyle(.universalGlassProminent())
    .tint(.pink)
    .controlSize(.large)
```

## API Surface

| Type | Maps to |
|------|---------|
| `UniversalGlassButtonStyle` | `.glass` |
| `UniversalGlassProminentButtonStyle` | `.glassProminent` |

Static helpers:
- `PrimitiveButtonStyle.universalGlass(rendering:)`
- `PrimitiveButtonStyle.universalGlassProminent(rendering:)`

Shorthand modifiers:
- `View.universalGlassButtonStyle(isProminent:rendering:)`
- `View.universalGlassProminentButtonStyle(rendering:)`

Every entry point takes a `UniversalGlassRendering` argument so callers can opt into the native API (`.automatic` / `.glass`) or force the material fallback (`.material`).

## Runtime Routing

`resolveShouldUseGlass(for:)` guards the primitive styles. The helper is marked `@inline(__always)` so the availability-heavy branching is substituted at the call site. The logic:

1. If the platform ships OS 26 APIs and the caller did not force `.material`, call straight into `GlassButtonStyle` / `GlassProminentButtonStyle`
2. Otherwise, render through the legacy material style

Because `_Configuration` values from `PrimitiveButtonStyle` are still passed along, gesture handling and the `trigger` action behave the same in either branch.

## Material Fallback Styling

When the native glass styles are unavailable (or `.material` is forced) we wrap the configuration inside `UniversalGlassLegacyMaterialStyle`. That view:

- Applies control-size aware padding so the capsule footprint matches SwiftUI's default metrics
- Draws an interactive material plate via `View.universalGlassEffect(_:in:rendering:)`—which itself routes to `Material` on pre-26 systems
- Adds a subtle white stroke and drop shadow to mimic the elevated look of SwiftUI's liquid glass
- Drives press feedback with a scale animation (`reduceMotion` disables it), matching the built-in `ButtonStyle` affordance

Prominent buttons reuse the same pipeline with two tweaks: the fallback fills the capsule with `.tint` and switches the text to an off-white foreground to match Apple's design.

## SwiftUI Interop

Because these are real `PrimitiveButtonStyle`s you can:

- Layer additional modifiers (`.controlSize`, `.tint`, `.font`) exactly like the system styles
- Use the static helpers (`.buttonStyle(.universalGlass())`) alongside SwiftUI's `.borderedProminent` etc.
- Opt individual call sites into `.material` to debug or maintain platform parity

The native `.glass` replacement loads as soon as your deployment target reaches OS 26.

## Notes

- The result is a single call that yields authentic liquid glass on new OS releases and a faithful material approximation everywhere else
