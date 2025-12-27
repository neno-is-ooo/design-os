# iOS Native Design Patterns (SwiftUI-First)

Reference patterns for iOS native applications using Swift and SwiftUI. These patterns target iOS 17+ with SwiftUI as the primary UI framework.

> **Note**: For UIKit-based apps or iOS 16 and earlier, adapt these patterns accordingly. UIKit equivalents are noted where significantly different.

## Navigation Patterns

### Tab Bar Navigation
- Bottom tab bar (max 5 items)
- Each tab is independent navigation stack
- Badge support for notifications

```swift
// SwiftUI
TabView {
    HomeView()
        .tabItem { Label("Home", systemImage: "house") }
    SearchView()
        .tabItem { Label("Search", systemImage: "magnifyingglass") }
    ProfileView()
        .tabItem { Label("Profile", systemImage: "person") }
}
```

### Navigation Stack
- Hierarchical drill-down
- Back button automatically managed
- Large title → small title on scroll

```swift
// SwiftUI
NavigationStack {
    List(items) { item in
        NavigationLink(item.title) {
            DetailView(item: item)
        }
    }
    .navigationTitle("Items")
}
```

### Modal Presentation
- Sheet (partial cover, swipe to dismiss)
- Full screen cover
- Popover (iPad)

```swift
.sheet(isPresented: $showSettings) {
    SettingsView()
}

.fullScreenCover(isPresented: $showOnboarding) {
    OnboardingView()
}
```

### Sidebar Navigation (iPad)
- Three-column layout on large screens
- Collapses to navigation stack on compact

```swift
NavigationSplitView {
    SidebarView()
} content: {
    ContentListView()
} detail: {
    DetailView()
}
```

## List Patterns

### Standard List
- Full-width rows
- Swipe actions (delete, archive, etc.)
- Pull to refresh

```swift
List {
    ForEach(items) { item in
        ItemRow(item: item)
    }
    .onDelete(perform: deleteItems)
}
.refreshable {
    await loadItems()
}
```

### Grouped List
- Sections with headers/footers
- Good for settings screens
- Inset grouped style for modern look

```swift
List {
    Section("Account") {
        // Account settings
    }
    Section("Preferences") {
        // Preference settings
    }
}
.listStyle(.insetGrouped)
```

### Search
- Search bar in navigation
- Real-time filtering
- Search suggestions

```swift
.searchable(text: $searchText, suggestions: {
    ForEach(suggestions) { suggestion in
        Text(suggestion).searchCompletion(suggestion)
    }
})
```

## Form Patterns

### Settings Form
```swift
Form {
    Section("Appearance") {
        Picker("Theme", selection: $theme) {
            Text("Light").tag(Theme.light)
            Text("Dark").tag(Theme.dark)
            Text("System").tag(Theme.system)
        }
        Toggle("Show Badges", isOn: $showBadges)
    }

    Section("Notifications") {
        Toggle("Push Notifications", isOn: $pushEnabled)
    }
}
```

### Input Form
```swift
Form {
    TextField("Name", text: $name)
    TextField("Email", text: $email)
        .keyboardType(.emailAddress)
        .textContentType(.emailAddress)
    SecureField("Password", text: $password)
        .textContentType(.newPassword)
}
```

## State Management Patterns

### @Observable (iOS 17+)
```swift
@Observable
class AppState {
    var user: User?
    var items: [Item] = []
    var isLoading = false
}

// In view
@State private var appState = AppState()
```

### Environment for Shared State
```swift
@Observable
class NavigationState {
    var selectedTab: Tab = .home
    var path = NavigationPath()
}

// Root view
ContentView()
    .environment(navigationState)

// Child view
@Environment(NavigationState.self) private var navigation
```

### SwiftData for Persistence
```swift
@Model
class Item {
    var title: String
    var createdAt: Date
    var isCompleted: Bool

    init(title: String) {
        self.title = title
        self.createdAt = .now
        self.isCompleted = false
    }
}
```

## Gesture Patterns

### Swipe Actions
```swift
.swipeActions(edge: .trailing) {
    Button(role: .destructive) {
        deleteItem(item)
    } label: {
        Label("Delete", systemImage: "trash")
    }
}
.swipeActions(edge: .leading) {
    Button {
        archiveItem(item)
    } label: {
        Label("Archive", systemImage: "archivebox")
    }
    .tint(.blue)
}
```

### Long Press
```swift
.contextMenu {
    Button("Copy") { copy(item) }
    Button("Share") { share(item) }
    Button("Delete", role: .destructive) { delete(item) }
}
```

### Drag and Drop
```swift
.draggable(item)
.dropDestination(for: Item.self) { items, location in
    handleDrop(items, at: location)
    return true
}
```

## Loading & Empty States

### Loading State
```swift
if isLoading {
    ProgressView()
        .frame(maxWidth: .infinity, maxHeight: .infinity)
} else {
    ContentView()
}
```

### Empty State
```swift
if items.isEmpty {
    ContentUnavailableView(
        "No Items",
        systemImage: "tray",
        description: Text("Add items to get started")
    )
} else {
    List(items) { ... }
}
```

### Error State
```swift
ContentUnavailableView {
    Label("Connection Error", systemImage: "wifi.slash")
} description: {
    Text("Check your internet connection and try again.")
} actions: {
    Button("Retry") {
        Task { await retry() }
    }
}
```

## Haptics & Feedback

```swift
// Impact feedback
let impact = UIImpactFeedbackGenerator(style: .medium)
impact.impactOccurred()

// Success/error feedback
let notification = UINotificationFeedbackGenerator()
notification.notificationOccurred(.success)

// Selection feedback
let selection = UISelectionFeedbackGenerator()
selection.selectionChanged()
```

## Keyboard Handling

### Dismiss Keyboard
```swift
.scrollDismissesKeyboard(.interactively)

// Or programmatically
@FocusState private var isFocused: Bool
// Set isFocused = false to dismiss
```

### Keyboard Toolbar
```swift
TextField("Note", text: $note)
    .toolbar {
        ToolbarItemGroup(placement: .keyboard) {
            Spacer()
            Button("Done") {
                isFocused = false
            }
        }
    }
```

## Accessibility

### VoiceOver Support
```swift
Image(systemName: "heart.fill")
    .accessibilityLabel("Favorite")
    .accessibilityHint("Double tap to remove from favorites")

// For custom controls
.accessibilityElement(children: .combine)
.accessibilityAddTraits(.isButton)
```

### Dynamic Type
- Use system fonts (`.body`, `.headline`, etc.)
- Test with larger text sizes
- Ensure layouts adapt

## Performance Considerations

### Lazy Loading
```swift
LazyVStack {
    ForEach(items) { item in
        ItemRow(item: item)
    }
}
```

### Image Loading
```swift
AsyncImage(url: imageURL) { image in
    image.resizable().aspectRatio(contentMode: .fill)
} placeholder: {
    ProgressView()
}
```

### Background Tasks
```swift
Task {
    await loadData()
}

// For long-running tasks
Task.detached(priority: .background) {
    await processData()
}
```
