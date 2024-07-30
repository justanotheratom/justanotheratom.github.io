In this post, I'll walk you through the process of adding custom icons to your SwiftUI iOS app, from obtaining the SVG files to using them efficiently in your code.

## Installing Inkscape and Converting SVGs to PDFs

Many icon libraries provide SVGs, which are great for web development but not ideal for iOS. Here's how I install Inkscape and use it to convert SVGs to PDFs efficiently:

### Installing Inkscape

On macOS, I prefer using Homebrew to install Inkscape:

1. Open Terminal.
2. If you don't have Homebrew installed, install it first:
   ```
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
3. Install Inkscape using Homebrew:
   ```
   brew install inkscape
   ```

### Converting SVGs to PDFs using Inkscape Command Line

Once Inkscape is installed, I use this process to convert SVGs to PDFs:

1. Copy the SVG code from the icon library website.
2. Open a text editor and create a new file. Paste the SVG code and save it with a `.svg` extension.
3. Open Terminal and navigate to the directory containing your SVG file.
4. Run the following command to convert the SVG to PDF:
   ```
   inkscape --export-type=pdf --export-filename=output.pdf input.svg
   ```
   Replace `input.svg` with your SVG filename and `output.pdf` with your desired PDF filename.

This command-line method is great for batch processing multiple SVGs. I can create a simple shell script to convert all SVGs in a directory:

```bash
#!/bin/bash
for file in *.svg
do
    inkscape --export-type=pdf --export-filename="${file%.svg}.pdf" "$file"
done
```

I save this as `convert_svg_to_pdf.sh`, make it executable with `chmod +x convert_svg_to_pdf.sh`, and run it in the directory containing my SVGs.

## Adding PDF Assets to Xcode

Now that I have my PDF icons, it's time to add them to Xcode:

1. Open my Xcode project and find the Assets.xcassets catalog in the project navigator.
2. Right-click on the asset catalog and select "New Image Set".
3. Name the new image set something descriptive, like "icon-home".
4. Drag and drop the PDF file into the Universal slot in the image set.
5. In the Attributes Inspector, set:
   - Scales: Single Scale
   - Resizing: Preserve Vector Data
   - Render As: Template Image (if I want to be able to tint the icon)

## Configuring the Image View

In my SwiftUI code, I can now use the icon like this:

```swift
Image("icon-home")
    .resizable()
    .aspectRatio(contentMode: .fit)
    .frame(width: 24, height: 24)
    .foregroundColor(.blue) // If set as template image
```

## Adding IconName Enum for Type Safety

To make my code more robust and prevent typos, I like to use an enum for icon names:

```swift
enum IconName: String {
    case home = "icon-home"
    case settings = "icon-settings"
    case profile = "icon-profile"
    // Add more cases as needed
}

struct Icon: View {
    let iconName: IconName
    var size: CGFloat = 24
    var color: Color = .primary

    var body: some View {
        Image(iconName.rawValue)
            .resizable()
            .aspectRatio(contentMode: .fit)
            .frame(width: size, height: size)
            .foregroundColor(color)
    }
}
```

Now I can use icons in my views like this:

```swift
Icon(iconName: .home, size: 28, color: .blue)
```

This approach gives me type safety, autocomplete suggestions, and a centralized place to manage my icon names.

By following these steps, I've successfully incorporated third-party icons into my SwiftUI iOS app, enhancing its visual appeal while maintaining code quality and efficiency. Happy coding!
