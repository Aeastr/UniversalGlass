# UniversalGlass - Container Internals

Deep dive into `UniversalGlassEffectContainer`'s fallback pipeline for debugging and extension.

## When the Fallback Activates

The public entry point lives in `Sources/UniversalGlass/UniversalGlassContainers.swift`. At runtime it checks availability:

- OS 26 directly uses SwiftUI's `GlassEffectContainer` unless you override the rendering mode with `.material`
- Older systems, or calls that force the material path, render through `FallbackGlassEffectContainerRenderer`

## Capturing Child Views

Each child view that calls `universalGlassEffect` registers itself as a `GlassEffectParticipant` via an anchor preference. The participant stores:

- **Geometry** – Through the anchor resolved later in a shared `GeometryReader`
- **Merge hints** – `union` (namespace + id) and optional `effectID`
- **Styling data** – Requested shape, fallback material, universal glass configuration, rendering mode, and whether the child drew its own background
- **Transition metadata** – So appearance/disappearance can mimic `.materialize` or `.matchedGeometry`

Inside the container we flip the `isInFallbackGlassContainer` environment flag to `true`, signalling child modifiers to skip drawing their own glass plate. That prevents double layering—the composite overlay handles the background.

## Grouping Logic

Once SwiftUI resolves the preference, the container gathers every participant and converts the anchors into concrete frames. Participants are then grouped by priority:

1. A `GlassEffectUnion` (id + namespace) if present
2. Otherwise an explicit `glassEffectID`
3. Otherwise the participant's UUID (no grouping)

This mirrors native behaviour where sharing a union key melts multiple views into one continuous sheet.

## Rendering the Material

`GlassEffectFallbackOverlay` is drawn twice:

- **As a background** – Paints the merged glass plate
- **As a mask** – Clips the child content to the same geometry so content does not escape the growing container

For each group the overlay:

- Chooses an "anchor" member (first in declaration order) to supply the primary shape and material hints
- Computes the union of all member frames to determine the composite rect
- Renders a single `GlassEffectUnionOverlay`, which applies the material (`Material` or `Glass` fallback) inside the chosen shape

Where the OS supports `shadow(.drop:)`, the overlay adds the soft highlight to mimic native glass depth.

Because child views skip their own background drawing, the overlay becomes the only composited layer, matching the way the real container treats unions.

## Transition Behaviour

`UniversalGlassEffectTransition` rewrites transition requests:

- `.materialize` and `.matchedGeometry` fall back to `.transition(.fallbackBlur)` to keep insert/remove animations soft, unless the OS ships the native API
- `.identity` becomes a no-op

Grouping ensures that matched geometry transitions operate on the combined glass slab rather than individual participants, so resizing or reordering keeps the illusion of one piece of glass.

## Rendering Modes

The `rendering` parameter travels with each participant. Inside the container you can:

- Leave it `.automatic` (default) so OS 26+ uses native glass and earlier versions use the fallback
- Force `.material` when debugging or when you deliberately want the material aesthetic everywhere
- Use `.glass` on new OS builds to guarantee the real API, bypassing the fallback entirely

## Extending the Container

- Additional metadata can ride along in `glassEffectParticipantContext`; modifiers only need to write to the environment before registering their participant
- If you introduce new grouping semantics, extend `GlassEffectGroupingKey` or swap in a custom overlay that consumes the same participant list
- Heavy work stays out of the body by resolving geometry in one place and reusing it for both mask and background passes—keep extensions mindful of that single-pass approach
