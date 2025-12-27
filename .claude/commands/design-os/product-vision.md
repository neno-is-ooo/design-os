# Product Vision

You are helping the user define their product vision for Design OS. This is a conversational, multi-step process.

## Step 1: Gather Initial Input

First, ask the user to share their raw notes, ideas, or thoughts about the product they want to build. Be warm and open-ended:

"I'd love to help you define your product vision. Tell me about the product you're building - share any notes, ideas, or rough thoughts you have. What problem are you trying to solve? Who is it for? Don't worry about structure yet, just share what's on your mind."

Wait for their response before proceeding.

## Step 2: Determine Platform

After receiving their initial input, use the AskUserQuestion tool to determine the target platform:

"What platform are you building for?"

Options:
- **Web Application** — React with Tailwind CSS (exports are React/Tailwind-specific)
- **macOS Native** — Swift with AppKit or SwiftUI
- **iOS Native** — Swift with UIKit or SwiftUI
- **Cross-Platform Desktop** — Electron or Tauri (exports use React/Tailwind)
- **Mobile Cross-Platform** — React Native or Flutter
- **TUI (Terminal UI)** — Interactive terminal app (Charm/Bubble Tea, Textual, Ratatui, Ink)
- **CLI Tool** — Non-interactive command-line application
- **API/Backend** — Server-side service

**Important:** The platform choice affects:
- Which design phases apply (e.g., no shell design for CLI tools)
- Terminology used (views vs screens, windows vs pages)
- Export format (React/Tailwind for web, Swift types for native, specs for CLI/API)
- Available UI patterns and components

Store the platform choice for use throughout the design process.

## Step 3: Ask Clarifying Questions

After receiving their platform choice, use the AskUserQuestion tool to ask 3-5 targeted questions to help shape:

- **The product name** - A clear, concise name for the product
- **The core product description** (1-3 sentences that capture the essence)
- **The key problems** the product solves (1-5 specific pain points)
- **How the product solves each problem** (concrete solutions)
- **The main features** that make this possible

**Important:** If the user hasn't already provided a product name, ask them:
- "What would you like to call this product? (A short, memorable name)"

Other example clarifying questions (adapt based on their input):
- "Who is the primary user of this product? Can you describe them?"
- "What's the single biggest pain point you're addressing?"
- "How do people currently solve this problem without your product?"
- "What makes your approach different or better?"
- "What are the 3-5 most essential features?"

Ask questions one or two at a time, and engage conversationally.

## Step 4: Present Draft and Refine

Once you have enough information, present a draft summary:

"Based on our discussion, here's what I'm capturing for **[Product Name]**:

**Platform:** [Selected Platform]

**Description:**
[Draft 1-3 sentence description]

**Problems & Solutions:**
1. [Problem] → [Solution]
2. [Problem] → [Solution]

**Key Features:**
- Feature 1
- Feature 2
- Feature 3

Does this capture your vision? Would you like to adjust anything?"

Iterate until the user is satisfied.

## Step 5: Create the File

Once the user approves, create the file at `/product/product-overview.md` with this exact format:

```markdown
# [Product Name]

## Platform
[Selected Platform]

## Description
[The finalized 1-3 sentence description]

## Problems & Solutions

### Problem 1: [Problem Title]
[How the product solves it in 1-2 sentences]

### Problem 2: [Problem Title]
[How the product solves it in 1-2 sentences]

[Add more as needed, up to 5]

## Key Features
- [Feature 1]
- [Feature 2]
- [Feature 3]
[Add more as needed]
```

**Important:** The `# [Product Name]` heading at the top is required - this is what displays as the card title in the app.

## Step 6: Confirm Completion

Let the user know:

"I've created your product overview at `/product/product-overview.md`. The homepage will now display **[Product Name]** with your product vision.

**Platform:** [Selected Platform]

Based on your platform choice, the design process will be tailored accordingly:
- [Platform-specific note about which phases apply]

You can run `/product-roadmap` next to break this down into development sections."

### Platform-Specific Notes

**Web Application:**
- Full design process applies (shell, screen designs, React components)
- Export includes React/Tailwind components

**macOS Native / iOS Native:**
- Shell design focuses on window/navigation patterns
- No React component export — provides Swift type definitions and UI specifications
- Architecture phase is especially important for native platform patterns

**Cross-Platform Desktop (Electron/Tauri):**
- Similar to web, but with desktop window considerations
- Export includes web components plus desktop integration notes

**Mobile Cross-Platform:**
- Shell design focuses on tab bars, navigation stacks
- Export includes component specifications and navigation patterns

**TUI (Terminal UI):**
- Full design process applies (shell becomes terminal layout, screens become views)
- Design tokens focus on ANSI colors and terminal-safe styling
- Shell design defines the main terminal layout (panels, status bar, etc.)
- Export includes component specifications and state patterns
- Architecture phase important for state management (Elm-like for Bubble Tea)

**CLI Tool:**
- Skip shell design and screen design phases
- Focus on command structure, flags, output formatting
- Export includes command specifications and help text

**API/Backend:**
- Skip visual design phases entirely
- Focus on data model, endpoints, authentication
- Export includes API specifications and data schemas

## Important Notes

- Be conversational and helpful, not robotic
- Ask follow-up questions when answers are vague
- Help the user think through their product, don't just transcribe
- Keep the final output concise and clear
- The format must match exactly for the app to parse it correctly
- **Always ensure the product has a name** - if user didn't provide one, ask for it
- **Always determine the platform** - this affects the entire design process
