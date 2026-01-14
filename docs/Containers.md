# UniversalGlass - Containers

`UniversalGlassEffectContainer` orchestrates multiple glass effects so they render as one fused surface on earlier OS releases while deferring to SwiftUI's native `GlassEffectContainer` on OS 26.

## Quick Start

```swift
@Namespace private var ns

UniversalGlassEffectContainer {
    HStack {
        AvatarView()
            .universalGlassEffect()
            .universalGlassEffectUnion(id: "profile", namespace: ns)

        DetailsView()
            .universalGlassEffect()
            .universalGlassEffectUnion(id: "profile", namespace: ns)
    }
}
```

## Entry Point

```swift
UniversalGlassEffectContainer(spacing:rendering:content:)
```

- `.automatic` or `.glass` renders the native container whenever it exists
- `.material` forces the legacy renderer even on OS 26+, useful for testing or when you prefer the material aesthetic everywhere

## Fallback Renderer

When the native container is unavailable, the helper swaps in `FallbackGlassEffectContainerRenderer`. The renderer:

1. Projects every child's anchor preference (registered by `universalGlassEffect`) into concrete frames
2. Groups participants by shared union keys or effect IDs so neighbouring elements "melt" into a single slab
3. Draws a `GlassEffectFallbackOverlay` twice—once as a background and once as a mask—so the merged plate perfectly clips the child content

This setup guarantees that the fallback honours matched-geometry unions, respects per-view shapes, and keeps hit-testing consistent.

## Supporting APIs

The container works hand-in-hand with:

```swift
View.universalGlassEffectUnion(id:namespace:rendering:)
View.universalGlassEffectID(_:in:rendering:)
View.universalGlassEffectTransition(_:)
```

Each helper goes straight to SwiftUI's implementation on new OS builds and otherwise stores metadata in the environment for the fallback renderer to consume.

## Notes

- For the full breakdown of the participant pipeline, grouping keys, and overlay drawing order, see [Container Internals](ContainerInternals.md)
- On pre-OS 26 systems, the fallback `UniversalGlassEffectContainer` ignores the `spacing` parameter
