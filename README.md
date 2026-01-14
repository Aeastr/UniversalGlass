<div align="center">
  <img width="270" height="270" src="/Resources/icon.png" alt="UniversalGlass Icon">
  <h1><b>UniversalGlass</b></h1>
  <p>
    Bring SwiftUI's iOS 26 glass APIs to earlier deployments with lightweight shims—keep your UI consistent on iOS 17+, yet automatically defer to the real implementations wherever they exist.
  </p>
</div>

<p align="center">
  <a href="https://swift.org"><img src="https://img.shields.io/badge/Swift-6.0+-F05138?logo=swift&logoColor=white" alt="Swift 6.0+"></a>
  <a href="https://developer.apple.com"><img src="https://img.shields.io/badge/iOS-17+-000000?logo=apple" alt="iOS 17+"></a>
  <a href="https://developer.apple.com"><img src="https://img.shields.io/badge/macOS-13+-000000?logo=apple" alt="macOS 13+"></a>
  <a href="https://developer.apple.com"><img src="https://img.shields.io/badge/tvOS-17+-000000?logo=apple" alt="tvOS 17+"></a>
  <a href="https://developer.apple.com"><img src="https://img.shields.io/badge/watchOS-10+-000000?logo=apple" alt="watchOS 10+"></a>
  <a href="https://developer.apple.com"><img src="https://img.shields.io/badge/visionOS-1+-000000?logo=apple" alt="visionOS 1+"></a>
</p>

<div align="center">
  <img width="600" src="https://github.com/user-attachments/assets/f66f2c6d-7f51-441c-9022-fcd0280abb5a" alt="Preview">
</div>


## Overview

OS 26 introduces new SwiftUI glass APIs, but these only ship on the latest platforms. UniversalGlass offers compatibility layers so your code stays unified on older systems, then quietly defers to Apple's implementation where available.

- **Glass for every surface** – Apply `universalGlassEffect` to any view with tinting and interactivity
- **Native-feeling buttons** – `.universalGlass()` and `.universalGlassProminent()` button styles
- **Containers & morphing** – `UniversalGlassEffectContainer` with union/ID helpers for glass grouping
- **Backports** – Optional `UniversalGlassBackports` target for `.glass` and `.glassEffect` syntax


## Installation

```swift
dependencies: [
    .package(url: "https://github.com/Aeastr/UniversalGlass.git", branch: "main")
]
```

```swift
import UniversalGlass
```

| Target | Description |
|--------|-------------|
| `UniversalGlass` | Main module with glass effects, button styles, and containers |
| `UniversalGlassBackports` | Optional shorthand APIs (`.glass`, `.glassEffect`, etc.) |


## Usage

### Glass Effects

```swift
Text("Hello")
    .universalGlassEffect(.regular.tint(.purple))
```

With a custom shape:

```swift
Circle()
    .frame(width: 120, height: 120)
    .universalGlassEffect(in: Circle())
```

See [Effects](docs/Effects.md) for configurations, fallback customization, and transitions.

### Button Styles

```swift
Button("Join Beta") { }
    .buttonStyle(.universalGlassProminent())
    .tint(.pink)
```

See [Button Styles](docs/ButtonStyles.md) for routing behaviour and material fallback details.

### Effect Containers

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

See [Containers](docs/Containers.md) for grouping behaviour and [Container Internals](docs/ContainerInternals.md) for the fallback pipeline.

### Backports

```swift
import UniversalGlassBackports

Button("RSVP") {}
    .buttonStyle(.glassProminent)
```

See [Backports](docs/Backports.md) for the full API surface.


## Customization

### Tinting and Interactivity

```swift
.universalGlassEffect(.regular.tint(.cyan))
.universalGlassEffect(.regular.interactive())
```

### Fallback Materials

Override what renders on older OS versions:

```swift
.universalGlassEffect(.regular.fallback(material: .thin))
.universalGlassEffect(.thick.fallback(material: .regular, tint: .blue.opacity(0.2)))
```

### Shadows

```swift
.universalGlassEffect(.regular.shadow(UniversalGlassShadow(color: .red, radius: 12)))
.universalGlassEffect(.regular.shadow(.none))
```

### Global Rendering Mode

Force all effects in a hierarchy to use a specific renderer:

```swift
MyApp()
    .universalGlassRenderingMode(.material)  // Force fallback
```

### Chaining

All modifiers chain:

```swift
.universalGlassEffect(
    .ultraThick
        .fallback(material: .thin, tint: .cyan.opacity(0.3))
        .shadow(UniversalGlassShadow(color: .blue, radius: 16))
        .tint(.purple)
        .interactive()
)
```

See [Effects](docs/Effects.md) for the full configuration API.


## How It Works

UniversalGlass uses runtime availability checks to route calls to native SwiftUI APIs on OS 26+ or fall back to material-based approximations on earlier systems. The fallback renderer registers effects as participants via anchor preferences, groups them by union keys, and draws composite overlays.

**Known limitation:** On pre-OS 26 systems, the fallback container ignores the `spacing` parameter.


## Contributing

Contributions welcome. Before submitting a PR:

1. Create an issue outlining the change (optional for small fixes)
2. Follow the existing Swift formatting and file organisation
3. Ensure `swift build` succeeds and add previews/tests where relevant


## License

MIT. See [LICENSE](LICENSE) for details.
