Note: This is the result of a conversation with ChatGPT o1.

# Building an Auto-Expanding, Scrollable Transcript View in SwiftUI

## 1. Product Requirements (PRD)

**High-Level Goal**  
We want to display a **live transcript** (e.g., from a live audio recording) in a SwiftUI app. The transcript is read-only text that grows over time as new lines are appended.

**Key Requirements**  
1. **Expandable Height**  
   - Start at 1 line in height.  
   - Expand up to a maximum of 4 lines as the transcript grows.  
   - Beyond 4 lines, the transcript should become scrollable.  

2. **Scrolling Behavior**  
   - By default, keep the last 4 lines visible.  
   - User can manually scroll up to view older lines if they want.  

3. **Auto-Scroll**  
   - Whenever new text arrives, scroll to the bottom automatically (simple approach).  
   - Potentially add “sticky scroll” logic so it only auto-scrolls if the user is already at the bottom.  

4. **Non-Editable**  
   - The transcript text cannot be edited by the user.

## 2. Limitations of Off-the-Shelf SwiftUI Components

- **`Text`**  
  SwiftUI’s `Text` doesn’t provide a built-in mechanism to auto-expand or clamp its height to a certain number of lines. By default, `Text` can truncate or wrap indefinitely, but not smoothly expand from 1 to 4 lines, then start scrolling.

- **`TextField` / `TextEditor`**  
  SwiftUI’s `TextField` is primarily for single‐line user input. A `TextEditor` can be multi-line and scrollable, but it doesn’t easily cap its height to a certain number of lines (it can grow indefinitely or be fixed in height). It’s also inherently editable; making it truly read-only can be finicky.

- **Scrolling Behavior**  
  A standard `ScrollView` can provide scrolling, but you still need logic to measure the text height to decide whether to expand or scroll. Off-the-shelf components don’t handle the “expand until 4 lines, then scroll” pattern automatically.

Hence, building a custom solution is the best approach to fulfill all requirements.

## 3. Proposed Solution

### Overview

We’ll create a custom SwiftUI view that:

1. Dynamically measures the **height** of the full transcript text “off-screen.”
2. Measures the **height of a single line** using the same font, also “off-screen.”
3. Computes how many lines the transcript would occupy by dividing the full height by the single-line height.
4. Displays only 1 line, 2 lines, 3 lines, or 4 lines as needed. Beyond 4 lines, it scrolls within a `ScrollView`.
5. Automatically scrolls to the bottom when new text arrives (unless we add more advanced “sticky bottom” logic).

### Key Techniques

1. **Geometry Measurement** with a hidden “MeasuredText” view using `GeometryReader` and a `PreferenceKey`.
2. **Overlay** the measuring views so they don’t consume actual layout space in the main UI.
3. **ScrollViewReader** to programmatically scroll to the bottom.

## 4. Step-by-Step Implementation

Below is **all-in-one code** in a single Swift file. It includes:

1. `ViewSizeKey`: A preference key to store a measured `CGSize`.  
2. `MeasuredText`: A small view that renders text invisibly and reports its size via a geometry measurement.  
3. `AutoSizingTranscriptView`: The main custom view that measures a single line, measures the full text, expands up to 4 lines, then scrolls if needed.  
4. `TranscriptDemoView`: A sample container view showing how to append new lines to the transcript and observe the behavior.

> **Fact Check**:  
> - SwiftUI `Text` does not auto-expand or clamp by line count out of the box.  
> - Measuring text size in SwiftUI typically involves using `GeometryReader` with a `PreferenceKey`.  
> - Overlays (with `.hidden()`) keep the measured text from consuming extra layout space.  
> - `ScrollViewReader` and `.scrollTo` let us auto-scroll to the bottom.  

All these points are accurate according to Apple’s SwiftUI documentation and common SwiftUI patterns.

---

```swift
import SwiftUI

// MARK: - Preference Key for Measured Size
struct ViewSizeKey: PreferenceKey {
    static var defaultValue: CGSize = .zero
    
    static func reduce(value: inout CGSize, nextValue: () -> CGSize) {
        // In this scenario, just take the latest value
        value = nextValue()
    }
}

// MARK: - MeasuredText
/// A view that measures its rendered size via a GeometryReader.
/// We use this to measure single-line height ("X") and full text height.
struct MeasuredText: View {
    let text: String
    let font: Font
    
    @Binding var measuredSize: CGSize
    
    var body: some View {
        Text(text)
            .font(font)
            .fixedSize(horizontal: false, vertical: true)
            .background(
                GeometryReader { geo in
                    Color.clear
                        .preference(key: ViewSizeKey.self, value: geo.size)
                }
            )
            .hidden() // Keep it invisible
            .onPreferenceChange(ViewSizeKey.self) { size in
                measuredSize = size
            }
    }
}

// MARK: - AutoSizingTranscriptView
/// A read-only transcript view that:
///  • Starts at 1 line in height.
///  • Expands up to 4 lines.
///  • Beyond 4 lines, scrolls.
///  • Automatically scrolls to bottom on new text (simple version).
struct AutoSizingTranscriptView: View {
    
    // The transcript text that grows over time (e.g., from speech recognition).
    let text: String
    
    // Whichever SwiftUI Font you prefer (e.g. .body, .caption, etc.)
    let font: Font
    
    // We'll measure these two sizes "off-screen":
    @State private var singleLineSize: CGSize = .zero   // The height of one line
    @State private var fullTextSize: CGSize = .zero     // The total height of the entire text
    
    var body: some View {
        VStack(spacing: 0) {
            
            // 1) Main scrollable content
            ScrollViewReader { scrollProxy in
                ScrollView(.vertical, showsIndicators: true) {
                    Text(text)
                        .font(font)
                        .fixedSize(horizontal: false, vertical: true)
                        // This ID helps us scroll to the bottom
                        .id("TRANSCRIPT_BOTTOM")
                        .frame(maxWidth: .infinity, alignment: .leading)
                }
                .frame(height: displayHeight)
                .clipped()
                .onChange(of: text) { _ in
                    // Scroll to bottom whenever new text arrives
                    withAnimation {
                        scrollProxy.scrollTo("TRANSCRIPT_BOTTOM", anchor: .bottom)
                    }
                }
            }
            .padding()
        }
        // 2) Place the measuring views in overlays so they don't affect layout height
        .overlay(
            MeasuredText(text: "X", font: font, measuredSize: $singleLineSize)
                .frame(maxWidth: .infinity, alignment: .leading)
                .allowsHitTesting(false)
                .accessibilityHidden(true),
            alignment: .topLeading
        )
        .overlay(
            MeasuredText(text: text, font: font, measuredSize: $fullTextSize)
                .frame(maxWidth: .infinity, alignment: .leading)
                .allowsHitTesting(false)
                .accessibilityHidden(true),
            alignment: .topLeading
        )
    }
    
    // Computed height: 1 line minimum, 4 lines max
    private var displayHeight: CGFloat {
        let oneLineHeight = singleLineSize.height
        let totalHeight   = fullTextSize.height
        
        // If not measured yet, return a small fallback
        guard oneLineHeight > 0 else {
            return 20
        }
        
        let lineCount = totalHeight / oneLineHeight
        
        if lineCount <= 1 {
            return oneLineHeight
        } else if lineCount >= 4 {
            return oneLineHeight * 4
        } else {
            return totalHeight
        }
    }
}

// MARK: - TranscriptDemoView
/// A demo view to showcase how AutoSizingTranscriptView behaves.
/// You can present this from your main app or show it in previews.
struct TranscriptDemoView: View {
    @State private var transcriptText: String = ""
    @State private var counter: Int = 0
    
    var body: some View {
        VStack {
            // Demo: using .body font
            AutoSizingTranscriptView(
                text: transcriptText,
                font: .body
            )
            
            Button("Add Line") {
                counter += 1
                let newLine = "Line \(counter) with more text..."
                
                if transcriptText.isEmpty {
                    transcriptText = newLine
                } else {
                    transcriptText.append("\n" + newLine)
                }
            }
            .padding()
        }
        .navigationTitle("Transcript Demo")
    }
}

// MARK: - Preview
struct TranscriptDemoView_Previews: PreviewProvider {
    static var previews: some View {
        NavigationView {
            TranscriptDemoView()
        }
    }
}
