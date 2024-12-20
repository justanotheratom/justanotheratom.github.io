# Handling Multiple Drag-and-Drop Types in SwiftUI with Transferable

SwiftUI makes it easy to implement drag-and-drop functionality with the `Transferable` protocol. However, handling multiple data types like `String` and `UIImage` in a single drop target requires some additional setup. In this post, we’ll explore how to create a robust solution for handling both text and images in a SwiftUI app.

---

## Problem

By default, SwiftUI's `dropDestination` modifier can handle a single type at a time. While chaining multiple `dropDestination` modifiers might seem like a solution, only the first match is executed. This limitation complicates scenarios where you want to support multiple types in the same drop area.

For example, when attempting to handle both `String` and `UIImage` drops, you might encounter issues where:
- Plain text drops fail to populate the `items` array.
- Chained `dropDestination` modifiers only recognize the first matching type.

---

## Solution: Custom `Transferable` for Multiple Types

To solve this, we define a custom `Transferable` type that supports both `String` and `UIImage`. This allows us to use a single `dropDestination` modifier to handle both types seamlessly.

### Final Implementation

Below is the complete code for our custom `Transferable` type and its integration with `dropDestination`:

```swift
import SwiftUI
import UniformTypeIdentifiers

enum TransferError: Error {
    case importFailed
}

enum MultiTypeTransferable: Transferable {
    case text(String)
    case image(UIImage)

    static var transferRepresentation: some TransferRepresentation {
        // String Representation
        DataRepresentation(contentType: .plainText) { transferable in
            if case let .text(text) = transferable {
                return text.data(using: .utf8) ?? Data()
            }
            return Data()
        } importing: { data in
            guard let text = String(data: data, encoding: .utf8) else {
                throw TransferError.importFailed
            }
            return .text(text)
        }

        // UIImage Representation
        DataRepresentation(contentType: .image) { transferable in
            if case let .image(image) = transferable {
                return image.pngData() ?? Data()
            }
            return Data()
        } importing: { data in
            guard let image = UIImage(data: data) else {
                throw TransferError.importFailed
            }
            return .image(image)
        }
    }
}

struct RootView: View {
    private var vm = CalendarAgentViewModel.shared

    var body: some View {
        ZStack(alignment: .bottom) {
            HomeTab()
            ConversationSheet()
        }
        .dropDestination(for: MultiTypeTransferable.self) { items, _ in
            print(items) // Debug: Check dropped items
            for item in items {
                switch item {
                case .text(let text):
                    vm.handleSharedText(text)
                case .image(let image):
                    vm.processImage(image)
                }
            }
            return true
        }
    }
}
