# Agent Directives for Design OS

Design OS is a **product planning and design tool** that helps users define their product vision, structure their data model, design their UI, and prepare export packages for implementation in a separate codebase.

> **Important**: Design OS is a planning tool, not the end product codebase. The screen designs and components generated here are meant to be exported and integrated into your actual product's codebase.

---

## Understanding Design OS Context

When working in Design OS, be aware of two distinct contexts:

### 1. Design OS Application
The React application that displays and manages planning files. When modifying the Design OS UI itself:
- Files live in `src/` (components, pages, utilities)
- Uses the Design OS design system (stone palette, DM Sans, etc.)
- Provides the interface for viewing specs, screen designs, exports, etc.

### 2. Product Design (Screen Designs & Exports)
The product you're planning and designing. When creating screen designs and exports:
- Screen design components live in `src/sections/[section-name]/` and `src/shell/`
- Product definition files live in `product/`
- Exports are packaged to `product-plan/` for integration into a separate codebase
- Follow the design requirements specified in each section's spec

---

## Getting Started — The Planning Flow

Design OS follows a structured planning sequence:

### 1. Product Overview (`/product-vision`)
Define your product's core description, the problems it solves, key features, and **target platform**.
**Output:** `product/product-overview.md`

**Platform options:**
- Web Application (React, Vue, Svelte)
- macOS Native (Swift/AppKit/SwiftUI)
- iOS Native (Swift/UIKit/SwiftUI)
- Cross-Platform Desktop (Electron, Tauri)
- Mobile Cross-Platform (React Native, Flutter)
- CLI Tool
- API/Backend

The platform choice affects which phases apply and what export format is used.

### 2. Product Roadmap (`/product-roadmap`)
Break your product into 3-5 development sections with **initial complexity assessments** and dependencies.
**Output:** `product/product-roadmap.md`

Includes:
- Section descriptions
- Complexity ratings (Low/Medium/High)
- Inter-section dependencies
- Risk areas flagged early

### 3. Data Model (`/data-model`)
Define the core entities and relationships in your product. This establishes the "nouns" of your system and ensures consistency across sections.
**Output:** `product/data-model/data-model.md`

### 4. Dependencies (`/dependencies`) — *Optional but recommended*
Identify and evaluate external libraries, services, and system requirements before deep design work.
**Output:** `product/dependencies.md`

### 5. Design System (`/design-tokens`)
Choose your color palette (from Tailwind) and typography (from Google Fonts). These tokens are applied to all screen designs.
**Output:** `product/design-system/colors.json`, `product/design-system/typography.json`

*Note: For CLI or API-only products, this phase may be skipped.*

### 6. Application Shell (`/design-shell`)
Design the persistent navigation and layout that wraps all sections.
**Output:** `product/shell/spec.md`, `src/shell/components/`

*Note: For CLI or API-only products, this phase may be skipped.*

### 7. For Each Section:
- `/shape-section` — Define the specification (includes **technical feasibility checks** and **test scenarios**)
- `/sample-data` — Create sample data and types
- `/design-screen` — Create screen designs
- `/screenshot-design` — Capture screenshots

### 8. Architecture (`/architecture`)
Plan the technical architecture based on platform, dependencies, and section requirements.
**Output:** `product/architecture/`

Includes:
- State management patterns
- Platform-specific integration approaches
- Risk mitigation strategies
- Implementation order recommendations

**Platform pattern references:**
- macOS: `.claude/commands/design-os/patterns/macos-patterns.md`
- iOS: `.claude/commands/design-os/patterns/ios-patterns.md`
- CLI: `.claude/commands/design-os/patterns/cli-patterns.md`

### 9. Export (`/export-product`)
Generate the complete export package with all components, types, architecture docs, and handoff documentation.
**Output:** `product-plan/`

---

## File Structure

```
product/                           # Product definition (portable)
├── product-overview.md            # Product description, platform, problems/solutions, features
├── product-roadmap.md             # Sections with complexity and dependencies
├── dependencies.md                # External libraries and services
│
├── data-model/                    # Global data model
│   └── data-model.md              # Entity descriptions and relationships
│
├── design-system/                 # Design tokens
│   ├── colors.json                # { primary, secondary, neutral }
│   └── typography.json            # { heading, body, mono }
│
├── shell/                         # Application shell
│   └── spec.md                    # Shell specification
│
├── architecture/                  # Technical architecture
│   ├── overview.md                # High-level architecture and risks
│   └── [domain].md                # Domain-specific architecture docs
│
└── sections/
    └── [section-name]/
        ├── spec.md                # Section spec (with test scenarios, complexity)
        ├── data.json              # Sample data for screen designs
        ├── types.ts               # TypeScript interfaces (or .swift for native)
        └── *.png                  # Screenshots

src/
├── shell/                         # Shell design components
│   ├── components/
│   │   ├── AppShell.tsx
│   │   ├── MainNav.tsx
│   │   ├── UserMenu.tsx
│   │   └── index.ts
│   └── ShellPreview.tsx
│
└── sections/
    └── [section-name]/
        ├── components/            # Exportable components
        │   ├── [Component].tsx
        │   └── index.ts
        └── [ViewName].tsx         # Preview wrapper

product-plan/                      # Export package (generated)
├── README.md                      # Quick start guide
├── product-overview.md            # Product summary with platform
├── prompts/                       # Ready-to-use prompts for coding agents
│   ├── one-shot-prompt.md         # Prompt for full implementation
│   └── section-prompt.md          # Prompt template for incremental
├── instructions/                  # Implementation instructions
│   ├── one-shot-instructions.md   # All milestones combined
│   └── incremental/               # Milestone-by-milestone instructions
│       ├── 01-foundation.md
│       ├── 02-shell.md
│       └── [NN]-[section-id].md   # Section-specific instructions
├── architecture/                  # Technical architecture docs
│   ├── overview.md                # Architecture summary and risks
│   └── [domain].md                # Domain-specific patterns
├── dependencies.md                # External dependencies list
├── design-system/                 # Tokens, colors, fonts
├── data-model/                    # Types and sample data
├── shell/                         # Shell components (or specs for native)
└── sections/                      # Section components (with tests.md each)
```

---

## Design Requirements

When creating screen designs, follow these guidelines:

- **Mobile Responsive**: Use Tailwind's responsive prefixes (`sm:`, `md:`, `lg:`, `xl:`) to ensure layouts adapt properly across screen sizes.

- **Light & Dark Mode**: Use `dark:` variants for all colors. Test that all UI elements are visible and readable in both modes.

- **Use Design Tokens**: When design tokens are defined, apply the product's color palette and typography. Otherwise, fall back to `stone` for neutrals and `lime` for accents.

- **Props-Based Components**: All screen design components must accept data and callbacks via props. Never import data directly in exportable components.

- **No Navigation in Section Screen Designs**: Section screen designs should not include navigation chrome. The shell handles all navigation.

---

## Tailwind CSS Directives

These rules apply to both the Design OS application and all screen designs/components it generates:

- **Tailwind CSS v4**: We always use Tailwind CSS v4 (not v3). Do not reference or create v3 patterns.

- **No tailwind.config.js**: Tailwind CSS v4 does not use a `tailwind.config.js` file. Never reference, create, or modify one.

- **Use Built-in Utility Classes**: Avoid writing custom CSS. Stick to using Tailwind's built-in utility classes for all styling.

- **Use Built-in Colors**: Avoid defining custom colors. Use Tailwind's built-in color utility classes (e.g., `stone-500`, `lime-400`, `red-600`).

---

## The Four Pillars

Design OS is organized around four main areas:

1. **Product Overview** — The "what" and "why"
   - Product name and description
   - Problems and solutions
   - Key features
   - Sections/roadmap

2. **Data Model** — The "nouns" of the system
   - Core entity names and descriptions
   - Relationships between entities
   - Minimal — leaves room for implementation

3. **Design System** — The "look and feel"
   - Color palette (Tailwind colors)
   - Typography (Google Fonts)

4. **Application Shell** — The persistent chrome
   - Global navigation structure
   - User menu
   - Layout pattern

Plus **Sections** — The individual features, each with spec, data, screen designs.

---

## Design System Scope

Design OS separates concerns between its own UI and the product being designed:

- **Design OS UI**: Always uses the stone/lime palette and DM Sans typography
- **Product Screen Designs**: Use the design tokens defined for the product (when available)
- **Shell**: Uses product design tokens to preview the full app experience

---

## Export & Handoff

The `/export-product` command generates a complete handoff package:

- **Ready-to-use prompts**: Pre-written prompts to copy/paste into coding agents
  - `one-shot-prompt.md`: For full implementation in one session
  - `section-prompt.md`: Template for section-by-section implementation
- **Architecture documentation**: Technical decisions and patterns
  - `architecture/overview.md`: High-level architecture and risk summary
  - Domain-specific docs (state management, integrations, etc.)
- **Dependencies**: External libraries, services, and system requirements
- **Implementation instructions**: Detailed guides for each milestone
  - `product-overview.md`: Always provide for context
  - `one-shot-instructions.md`: All milestones combined
  - Incremental instructions in `instructions/incremental/`
- **Test instructions**: Each section includes `tests.md` with TDD specs
- **Portable components**: Props-based components (React for web, Swift types for native)

The prompts guide the implementation agent to ask clarifying questions about authentication, user modeling, and tech stack before building. Test instructions are framework-agnostic and include user flows, empty states, and edge cases.

**Platform-specific exports:**
- **Web**: React/Tailwind components
- **Native (macOS/iOS)**: Swift type definitions and UI specifications
- **CLI**: Command structure specs and output formatting
- **API**: Endpoint specifications and data schemas

---

## Design System (Design OS Application)

The Design OS application itself uses a "Refined Utility" aesthetic:

- **Typography**: DM Sans for headings and body, IBM Plex Mono for code
- **Colors**: Stone palette for neutrals (warm grays), lime for accents
- **Layout**: Maximum 800px content width, generous whitespace
- **Cards**: Minimal borders (1px), subtle shadows, generous padding
- **Motion**: Subtle fade-ins (200ms), no bouncy animations
