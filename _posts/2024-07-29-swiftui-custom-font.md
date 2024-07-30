# Adding Custom Font in SwiftUI: A Step-by-Step Guide

Disclaimer: This blog post was the artifact of a conversation with claude.ai.

Custom fonts can significantly enhance the visual appeal and branding of your iOS app. In this post, we'll walk through the process of implementing the [Figtree font](https://fonts.google.com/specimen/Figtree) in a SwiftUI app, including how to set it as the default font globally. We'll also discuss some challenges we encountered and how to overcome them.

## The Variable Font Challenge

Initially, we attempted to use the variable font file `Figtree-VariableFont_wght.ttf`. However, we ran into issues with font weight variations not rendering correctly. After troubleshooting, we decided to use individual static font files for each weight, which proved to be a more reliable solution.

## Step-by-Step Implementation

### 1. Download and Add Font Files

Download the Figtree font family from Google Fonts. Add the following static font files to your Xcode project:

- Figtree-Light.ttf
- Figtree-Regular.ttf
- Figtree-Medium.ttf
- Figtree-SemiBold.ttf
- Figtree-Bold.ttf
- Figtree-ExtraBold.ttf
- Figtree-Black.ttf

Ensure "Copy items if needed" is checked and add them to your main target.

### 2. Update Info.plist

Add the following to your Info.plist file:

```xml
<key>UIAppFonts</key>
<array>
    <string>Figtree-Light.ttf</string>
    <string>Figtree-Regular.ttf</string>
    <string>Figtree-Medium.ttf</string>
    <string>Figtree-SemiBold.ttf</string>
    <string>Figtree-Bold.ttf</string>
    <string>Figtree-ExtraBold.ttf</string>
    <string>Figtree-Black.ttf</string>
</array>
```

### 3. Create Font Extension

Create a new Swift file named `Font+Figtree.swift` and add the following code:

```swift
import SwiftUI

extension Font {
    static func figtree(_ style: TextStyle, weight: Weight = .regular) -> Font {
        let fontName = figtreeFontName(for: weight)
        let baseSize = UIFont.preferredFont(forTextStyle: style.uiFontTextStyle).pointSize
        let font = UIFont(name: fontName, size: baseSize) ?? .systemFont(ofSize: baseSize, weight: weight.uiFontWeight)
        let scaledFont = UIFontMetrics(forTextStyle: style.uiFontTextStyle).scaledFont(for: font)
        return Font(scaledFont)
    }
    
    private static func figtreeFontName(for weight: Weight) -> String {
        switch weight {
        case .black:
            return "Figtree-Black"
        case .heavy:
            return "Figtree-ExtraBold"
        case .bold:
            return "Figtree-Bold"
        case .semibold:
            return "Figtree-SemiBold"
        case .medium:
            return "Figtree-Medium"
        case .regular:
            return "Figtree-Regular"
        case .light, .thin, .ultraLight:
            return "Figtree-Light"
        @unknown default:
            return "Figtree-Regular"
        }
    }
}

extension Font.TextStyle {
    var uiFontTextStyle: UIFont.TextStyle {
        switch self {
        case .largeTitle: return .largeTitle
        case .title: return .title1
        case .title2: return .title2
        case .title3: return .title3
        case .headline: return .headline
        case .body: return .body
        case .callout: return .callout
        case .subheadline: return .subheadline
        case .footnote: return .footnote
        case .caption: return .caption1
        case .caption2: return .caption2
        @unknown default: return .body
        }
    }
}

extension Font.Weight {
    var uiFontWeight: UIFont.Weight {
        switch self {
        case .black: return .black
        case .heavy: return .heavy
        case .bold: return .bold
        case .semibold: return .semibold
        case .medium: return .medium
        case .regular: return .regular
        case .light: return .light
        case .thin: return .thin
        case .ultraLight: return .ultraLight
        @unknown default: return .regular
        }
    }
}
```

### 4. Create Custom Environment Key

Create a new Swift file named `FigtreeFontKey.swift` and add the following code:

```swift
import SwiftUI

struct FigtreeFontKey: EnvironmentKey {
    static let defaultValue: ((Font.TextStyle) -> Font) = { style in
        .figtree(style, weight: .regular)
    }
}

extension EnvironmentValues {
    var figtreeFont: (Font.TextStyle) -> Font {
        get { self[FigtreeFontKey.self] }
        set { self[FigtreeFontKey.self] = newValue }
    }
}
```

### 5. Create Global Font Modifier

Create a new Swift file named `FigtreeFontModifier.swift` and add the following code:

```swift
import SwiftUI

struct FigtreeFontModifier: ViewModifier {
    @Environment(\.figtreeFont) var figtreeFont
    
    func body(content: Content) -> some View {
        content
            .font(figtreeFont(.body))
            .environment(\.font, .figtree(.body))
    }
}

extension View {
    func withFigtreeFont() -> some View {
        self.modifier(FigtreeFontModifier())
    }
}

struct FigtreeFont: ViewModifier {
    let style: Font.TextStyle
    let weight: Font.Weight
    
    func body(content: Content) -> some View {
        content
            .font(.figtree(style, weight: weight))
    }
}

extension View {
    func figtreeFont(_ style: Font.TextStyle, weight: Font.Weight = .regular) -> some View {
        self.modifier(FigtreeFont(style: style, weight: weight))
    }
}
```

### 6. Apply Global Font in App Struct

In your main app struct, apply the global font:

```swift
@main
struct YourApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .withFigtreeFont()
                .environment(\.figtreeFont) { style in
                    .figtree(style, weight: .regular)
                }
        }
    }
}
```

## Usage

Now, Figtree will be the default font throughout your app. You don't need to specify it for every Text view:

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Text("This will use Figtree font by default")
            Text("This too!")
            
            // If you need a different style or weight:
            Text("This is a title")
                .font(.figtree(.title, weight: .bold))
        }
    }
}
```

If you need to use a different font for a specific part of your app, you can override it:

```swift
Text("This will use the system font")
    .font(.system(.body))
```

## Conclusion

By following these steps, you can successfully implement the Figtree font in your SwiftUI app and set it as the default font globally. This approach provides a good balance between convenience and flexibility, allowing you to maintain a consistent look throughout your app while still being able to use different styles or fonts when needed.

Remember, if you encounter any issues with font rendering or weights, double-check that all font files are correctly added to your project and listed in the Info.plist file. Happy coding!
