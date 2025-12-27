# macOS Native Design Patterns

Reference patterns for macOS native applications using Swift and AppKit/SwiftUI.

## Window Patterns

### Document-Based Application
- Multiple windows, each representing a document
- Standard File menu (New, Open, Save, Save As)
- Window restoration on relaunch
- Example: TextEdit, Xcode

```swift
// Document-based apps use NSDocument subclass
class MyDocument: NSDocument {
    override func makeWindowControllers() {
        let windowController = NSWindowController(windowNibName: "Document")
        addWindowController(windowController)
    }
}
```

### Single-Window Application
- One main window (may have auxiliary windows)
- Sidebar navigation or tab-based sections
- Example: System Preferences, Music

```swift
// Single window with sidebar
class MainWindowController: NSWindowController {
    @IBOutlet var sidebar: NSOutlineView!
    @IBOutlet var contentView: NSView!
}
```

### Utility Window
- Floating panels for tools/inspectors
- Non-activating (don't steal focus)
- Example: Color picker, Font panel

```swift
let panel = NSPanel(
    contentRect: rect,
    styleMask: [.titled, .closable, .utilityWindow, .nonactivatingPanel],
    backing: .buffered,
    defer: true
)
panel.isFloatingPanel = true
```

## Titlebar Patterns

### Standard Titlebar
- Title, close/minimize/zoom buttons
- Optional toolbar below

### Unified Titlebar + Toolbar
- Toolbar integrated into titlebar space
- More vertical content space
- `window.titlebarAppearsTransparent = true`

### Full Content Under Titlebar
- Content extends behind titlebar
- Good for media apps
- `window.styleMask.insert(.fullSizeContentView)`

### Titlebar Accessories
- Custom views in titlebar (like Safari tabs)
- Use `NSTitlebarAccessoryViewController`

```swift
let accessory = NSTitlebarAccessoryViewController()
accessory.view = customTabBar
accessory.layoutAttribute = .bottom
window.addTitlebarAccessoryViewController(accessory)
```

## Navigation Patterns

### Sidebar Navigation
- NSOutlineView or List (SwiftUI)
- Collapsible sections
- Selection drives main content

### Tab Bar
- NSTabView or custom tab bar
- Document tabs (like browsers)
- Feature tabs (like System Preferences sections)

### Toolbar Navigation
- NSToolbar with selectable items
- Each item shows different content
- Example: Finder view modes

### Source List
- Sidebar with library/sources
- User-created groups
- Smart folders/saved searches

## Split View Patterns

### Two-Pane (Master-Detail)
- Left: list/outline, Right: detail
- Adjustable divider
- Collapsible sidebar

```swift
let splitView = NSSplitViewController()
splitView.splitViewItems = [
    NSSplitViewItem(sidebarWithViewController: sidebarVC),
    NSSplitViewItem(viewController: contentVC)
]
```

### Three-Pane (Navigator-List-Detail)
- Mail-style layout
- Source list → Message list → Message content

### Inspector Panel
- Main content with right-side inspector
- Inspector shows properties of selection
- Toggle visibility

## State Management Patterns

### @Observable (Swift 5.9+)
```swift
@Observable
class AppState {
    var documents: [Document] = []
    var selectedDocumentID: UUID?

    var selectedDocument: Document? {
        documents.first { $0.id == selectedDocumentID }
    }
}
```

### Actor for Thread Safety
```swift
actor DocumentManager {
    private var documents: [UUID: Document] = [:]

    func load(_ url: URL) async throws -> Document {
        // Thread-safe document loading
    }
}
```

### UserDefaults for Preferences
```swift
extension UserDefaults {
    @objc dynamic var fontSize: CGFloat {
        get { double(forKey: "fontSize") as CGFloat }
        set { set(newValue, forKey: "fontSize") }
    }
}
```

## Menu Patterns

### Application Menu
- About, Preferences, Services, Hide, Quit
- Standard for all Mac apps

### Context Menu
- Right-click actions
- Context-sensitive to selection

### Menu Bar Extra
- Status item in menu bar
- Dropdown menu or popover

```swift
let statusItem = NSStatusBar.system.statusItem(withLength: NSStatusItem.squareLength)
statusItem.button?.image = NSImage(named: "MenuIcon")
statusItem.menu = statusMenu
```

## Keyboard Shortcuts

### Standard Shortcuts (Never Override)
- ⌘Q: Quit
- ⌘W: Close Window
- ⌘H: Hide
- ⌘,: Preferences
- ⌘N: New
- ⌘O: Open
- ⌘S: Save

### App-Specific Shortcuts
- Register in menu items
- Show in menu with key equivalents
- Support customization if possible

### Global Hotkeys
- Use MASShortcut or similar library
- Register with system when app launches
- Respect user's existing shortcuts

## File Handling

### Open/Save Panels
```swift
let panel = NSOpenPanel()
panel.allowedContentTypes = [.markdown, .plainText]
panel.allowsMultipleSelection = true
panel.begin { response in
    if response == .OK {
        self.openFiles(panel.urls)
    }
}
```

### Drag and Drop
```swift
view.registerForDraggedTypes([.fileURL])

override func performDragOperation(_ sender: NSDraggingInfo) -> Bool {
    guard let urls = sender.draggingPasteboard.readObjects(
        forClasses: [NSURL.self]
    ) as? [URL] else { return false }
    return handleDroppedFiles(urls)
}
```

### Recent Documents
- Managed by NSDocumentController
- Appears in File menu automatically

## Performance Considerations

### Lazy Loading
- Load content on-demand
- Use NSTableView/NSOutlineView cell reuse
- Prefetch adjacent content

### Background Processing
- Use DispatchQueue or async/await
- Never block main thread
- Show progress for long operations

### Memory Management
- Use weak references for delegates
- Clear caches under memory pressure
- Profile with Instruments
