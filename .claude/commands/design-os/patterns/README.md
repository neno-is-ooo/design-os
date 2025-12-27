# Platform Patterns Library

Reference patterns for different platforms. These patterns inform architecture decisions and screen designs when working with non-web platforms.

## Available Patterns

- **[macOS Patterns](./macos-patterns.md)** - Swift/AppKit/SwiftUI desktop patterns
- **[iOS Patterns](./ios-patterns.md)** - Swift/UIKit/SwiftUI mobile patterns
- **[CLI Patterns](./cli-patterns.md)** - Command-line interface patterns

## How to Use

When working on a native platform project:

1. **During Architecture Phase** - Reference patterns for state management, navigation, and system integration approaches
2. **During Shell Design** - Reference window/navigation patterns appropriate for the platform
3. **During Section Design** - Reference UI component patterns (lists, forms, etc.)

## Adding New Patterns

To add patterns for a new platform (e.g., Android, Electron, Flutter):

1. Create a new file: `[platform]-patterns.md`
2. Include sections for:
   - Navigation patterns
   - UI component patterns
   - State management patterns
   - Platform-specific considerations
   - Performance considerations
3. Update this README with a link to the new file
