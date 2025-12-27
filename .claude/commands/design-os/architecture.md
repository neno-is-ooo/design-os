# Architecture Planning

You are helping the user define the technical architecture for their product. This phase comes after section specifications are complete and before export.

## Prerequisites

Before running this command, ensure:
- Product overview exists (`product/product-overview.md`)
- Platform is defined
- Data model is complete
- Section specs are finished

## Step 1: Identify Architecture Domains

Based on the platform and product requirements, identify which architecture domains need documentation:

### Web Application
- State Management (Redux, Zustand, Context, etc.)
- API Integration (REST, GraphQL, data fetching)
- Authentication & Authorization
- Routing & Navigation
- Performance (code splitting, caching, SSR/SSG)

### macOS/iOS Native
- State Management (@Observable, Combine, actors)
- Data Persistence (Core Data, SwiftData, UserDefaults)
- C/System Integrations (if any)
- Window/View Architecture
- Performance & Memory Management

### Cross-Platform Desktop (Electron/Tauri)
- IPC Architecture (main/renderer communication)
- State Management
- Native Integrations
- File System Access
- Performance (memory, startup time)

### Mobile Cross-Platform
- State Management
- Navigation Architecture
- Platform-Specific Modules
- Offline Support
- Performance (animations, rendering)

### CLI Tool
- Command Structure & Parsing
- Configuration Management
- Output Formatting
- Error Handling

### API/Backend
- API Design (REST/GraphQL)
- Database Schema
- Authentication
- Caching Strategy
- Error Handling & Logging

## Step 2: Assess Technical Risks

For each section in the product roadmap, identify:

1. **Complexity Rating** (Low/Medium/High)
   - What makes this technically challenging?

2. **Dependencies**
   - External libraries or services required
   - System capabilities needed
   - Third-party integrations

3. **Risk Areas**
   - What could go wrong?
   - What has tight performance requirements?
   - Where are the unknowns?

4. **Mitigation Strategies**
   - How to reduce each risk
   - Fallback approaches
   - Proof-of-concept suggestions

Present this as a table:

```
| Section | Complexity | Key Risks | Mitigation |
|---------|------------|-----------|------------|
| [Name]  | High       | [Risk]    | [Strategy] |
```

## Step 3: Create Architecture Documents

For each relevant domain, create a detailed architecture document. Use the AskUserQuestion tool to determine which domains are most critical.

Ask: "Which areas need the most architectural guidance?"

Options based on platform:
- For web: State Management, API Integration, Performance
- For native: State Management, System Integrations, Window Architecture
- For cross-platform: IPC, Native Bridges, State Management
- For CLI: Command Structure, Configuration
- For API: Database Schema, API Design, Caching

## Step 4: Document Each Architecture Domain

For each selected domain, create a file at `product/architecture/[domain].md` with:

```markdown
# [Domain Name] Architecture

## Overview
[High-level approach in 2-3 sentences]

## Design Decisions

### Decision 1: [What]
**Choice:** [Selected approach]
**Rationale:** [Why this approach]
**Alternatives Considered:** [What else was evaluated]

### Decision 2: [What]
...

## Implementation Patterns

### Pattern 1: [Name]
[Description with code examples]

    // Example code showing the pattern (use 4-space indent for code in templates)
    func example() {
        // ...
    }

### Pattern 2: [Name]
...

## Integration Points
[How this domain connects to other parts of the system]

## Performance Considerations
[Specific optimizations and constraints]

## Testing Strategy
[How to test this architectural layer]
```

## Step 5: Create Architecture Overview

After individual domains are documented, create `product/architecture/overview.md`:

```markdown
# Architecture Overview

## Platform
[Selected Platform]

## High-Level Architecture
[Diagram or description of main components and data flow]

## Architecture Documents
- [Domain 1](./domain-1.md) - [One-line description]
- [Domain 2](./domain-2.md) - [One-line description]

## Technical Risks Summary

| Risk | Severity | Mitigation Status |
|------|----------|-------------------|
| [Risk] | High/Medium/Low | [Documented/Needs POC/Resolved] |

## Dependencies

### External Libraries
| Library | Purpose | Version | License |
|---------|---------|---------|---------|
| [Name] | [Why needed] | [Version] | [License] |

### System Requirements
- [Requirement 1]
- [Requirement 2]

## Implementation Order

Based on dependencies and risk, the recommended build order is:

1. **Foundation** - [Core systems that everything depends on]
2. **[Section]** - [Why this order makes sense]
3. **[Section]** - [Dependencies on previous]
...
```

## Step 6: Confirm Completion

Let the user know:

"I've created your architecture documentation at `product/architecture/`:

- `overview.md` - High-level architecture and risk summary
- `[domain-1].md` - [Domain description]
- `[domain-2].md` - [Domain description]

**Technical Risks Identified:**
- [High risk items with mitigation status]

**Recommended Implementation Order:**
1. [First section/milestone]
2. [Next section]
...

You can run `/export-product` to generate the full handoff package with these architecture docs included."

## Platform-Specific Guidance

### macOS Native Architecture Domains

**State Management Options:**
- `@Observable` (Swift 5.9+) - Modern, simple observation
- Combine framework - Reactive streams
- Actor pattern - Thread-safe shared state

**Window Architecture:**
- Document-based vs single-window
- Window restoration
- Full-screen support
- Titlebar customization (NSTitlebarAccessoryViewController)

**C/System Integrations:**
- Swift C interop patterns
- Bridging headers
- Memory management across boundary
- Error handling from C APIs

### iOS Native Architecture Domains

**Navigation Patterns:**
- NavigationStack (SwiftUI) / UINavigationController
- Tab-based navigation
- Modal presentation
- Deep linking

**Data Persistence:**
- SwiftData vs Core Data
- UserDefaults for settings
- Keychain for secrets
- File coordination for documents

### Web Application Architecture Domains

**State Management:**
- Component state vs global state
- Server state (React Query, SWR)
- URL state for shareable views

**Performance:**
- Code splitting boundaries
- Image optimization
- Caching strategies
- Bundle size management

## Platform Pattern References

For platform-specific patterns, consult the patterns library:

- **macOS Native**: See `.claude/commands/design-os/patterns/macos-patterns.md`
- **iOS Native**: See `.claude/commands/design-os/patterns/ios-patterns.md`
- **CLI Tools**: See `.claude/commands/design-os/patterns/cli-patterns.md`

These provide ready-to-use patterns for window management, navigation, state management, and common UI components.

## Important Notes

- Architecture docs should be practical, not theoretical
- Include code examples for every pattern
- Focus on decisions that are hard to change later
- Document the "why" as much as the "what"
- Identify proof-of-concept needs for high-risk areas
- Keep the implementation agent's perspective in mind - what would they need to know?
- Reference platform patterns library for established patterns
