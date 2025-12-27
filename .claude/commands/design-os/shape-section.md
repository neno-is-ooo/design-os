# Shape Section

You are helping the user define the specification for a section of their product. This is a conversational process to establish the scope of functionality, user flows, UI requirements, and technical feasibility.

## Step 1: Check Prerequisites

First, verify that `/product/product-roadmap.md` exists. If it doesn't:

"I don't see a product roadmap defined yet. Please run `/product-roadmap` first to define your product sections, then come back to shape individual sections."

Stop here if the roadmap doesn't exist.

## Step 2: Identify the Target Section

Read `/product/product-roadmap.md` to get the list of available sections.

If there's only one section, auto-select it. If there are multiple sections, use the AskUserQuestion tool to ask which section the user wants to work on:

"Which section would you like to define the specification for?"

Present the available sections as options.

## Step 3: Gather Initial Input

Once the section is identified, invite the user to share any initial thoughts:

"Let's define the scope and requirements for **[Section Title]**.

Do you have any notes or ideas about what this section should include? Share any thoughts about the features, user flows, or UI patterns you're envisioning. If you're not sure yet, we can start with questions."

Wait for their response. The user may provide raw notes or ask to proceed with questions.

## Step 4: Ask Clarifying Questions

Use the AskUserQuestion tool to ask 4-6 targeted questions. Adapt based on the platform:

### For UI Platforms (Web, Desktop, Mobile):
- **Main user actions/tasks** - What can users do in this section?
- **Information to display** - What data and content needs to be shown?
- **Key user flows** - What are the step-by-step interactions?
- **UI patterns** - Any specific interactions, layouts, or components needed?
- **Scope boundaries** - What should be explicitly excluded?

### For CLI Tools:
- **Commands/subcommands** - What commands belong to this section?
- **Flags and arguments** - What options does each command accept?
- **Input/output** - What data comes in and what output is produced?
- **Error cases** - What errors can occur and how are they reported?
- **Scope boundaries** - What's explicitly out of scope?

### For API/Backend:
- **Endpoints** - What routes belong to this section?
- **Request/response** - What data structures are used?
- **Authentication** - What authorization is required?
- **Error cases** - What errors can occur and how are they returned?
- **Scope boundaries** - What's explicitly out of scope?

Ask questions one or two at a time, conversationally.

## Step 5: Technical Feasibility Check

Before proceeding, assess the technical complexity of this section. Consider:

1. **Platform Constraints** - Does this section require capabilities the platform may not support well?
2. **External Dependencies** - Are there third-party libraries, APIs, or services needed?
3. **Performance Concerns** - Are there operations that could be slow or resource-intensive?
4. **Known Hard Problems** - Does this involve technically challenging areas (e.g., real-time sync, offline support, complex animations)?

If any high-risk areas are identified, flag them for the user:

"I want to flag some technical considerations for **[Section Title]**:

**Potential Complexity:**
- [Issue 1] - [Why it's complex]
- [Issue 2] - [Why it's complex]

**Questions to consider:**
- [Question about approach or trade-off]

These don't prevent us from proceeding, but they should be addressed in the architecture phase. Want to adjust the scope, or continue as planned?"

Use AskUserQuestion with options:
- "Continue as planned" - Accept the complexity and document for architecture phase
- "Simplify scope" - Reduce scope to lower complexity
- "Discuss alternatives" - Explore different approaches

## Step 6: Define Test Scenarios

Ask the user about key test scenarios early to clarify expected behavior:

"Let's define how we'll know this section works correctly. What are the key scenarios to test?"

Use AskUserQuestion to gather:
- **Happy path** - Main flow works as expected
- **Empty states** - What shows when there's no data?
- **Error cases** - What happens when things go wrong?
- **Edge cases** - Unusual but valid situations

Example questions:
- "What should happen when there's no [data type] yet?"
- "How should errors be communicated to the user?"
- "Are there any edge cases we should handle specially?"

Document 3-5 core test scenarios that will be included in the spec.

## Step 7: Ask About Shell Configuration (UI Platforms Only)

**Skip this step for CLI and API projects.**

For UI platforms, check if a shell design exists:
- **Web**: Check for `/src/shell/components/AppShell.tsx`
- **macOS**: Check for window controller or main window spec
- **iOS**: Check for tab bar or navigation structure

If a shell exists, ask:

"Should this section be displayed **inside the app shell** (with navigation), or as a **standalone view** (without the shell)?

Most sections use the app shell, but some views like onboarding, public pages, or embedded widgets should be standalone."

Use AskUserQuestion with options:
- "Inside app shell" - The default for most in-app sections
- "Standalone (no shell)" - For public pages, onboarding, or embeds

If no shell design exists yet, skip this question and default to using the shell.

## Step 8: Present Draft and Refine

Once you have enough information, present a draft specification:

"Based on our discussion, here's the specification for **[Section Title]**:

**Overview:**
[2-3 sentence summary of what this section does]

**User Flows:**
- [Flow 1]
- [Flow 2]
- [Flow 3]

**UI Requirements:**
- [Requirement 1]
- [Requirement 2]
- [Requirement 3]

**Test Scenarios:**
- [Scenario 1: Happy path]
- [Scenario 2: Empty state]
- [Scenario 3: Error case]

**Technical Notes:**
- Complexity: [Low/Medium/High]
- [Any flagged technical considerations]

**Display:** [Inside app shell / Standalone]

Does this capture everything? Would you like to adjust anything?"

Iterate until the user is satisfied. Don't add features that weren't discussed. Don't leave out features that were discussed.

## Step 9: Create the Spec File

Once the user approves, create the file at `product/sections/[section-id]/spec.md` with this exact format:

```markdown
# [Section Title] Specification

## Overview
[The finalized 2-3 sentence description]

## User Flows
- [Flow 1]
- [Flow 2]
- [Flow 3]
[Add all flows discussed]

## UI Requirements
- [Requirement 1]
- [Requirement 2]
- [Requirement 3]
[Add all requirements discussed]

## Test Scenarios

### Happy Path
- [Main flow scenario]

### Empty States
- [What to show when no data exists]

### Error Cases
- [How errors are handled and displayed]

### Edge Cases
- [Any special situations identified]

## Technical Notes
- **Complexity:** [Low/Medium/High]
- **Dependencies:** [Any external libraries or services needed]
- **Risk Areas:** [Technical challenges to address in architecture phase]

## Configuration
- shell: [true/false]
```

**Important:**
- Set `shell: true` if the section should display inside the app shell (this is the default)
- Set `shell: false` if the section should display as a standalone page without the shell

The section-id is the slug version of the section title (lowercase, hyphens instead of spaces).

## Step 10: Confirm and Next Steps

Let the user know:

"I've created the specification at `product/sections/[section-id]/spec.md`.

**Complexity:** [Low/Medium/High]
**Technical flags:** [Any issues to address in architecture phase, or 'None']

You can review the spec on the section page. When you're ready, run `/sample-data` to create sample data for this section.

After all sections are specified, run `/architecture` to plan the technical architecture before exporting."

## Important Notes

- Be conversational and helpful, not robotic
- Ask follow-up questions when answers are vague
- Focus on UX and UI - but flag technical complexity when spotted
- Keep the spec concise - only include what was discussed, no bloat
- The format must match exactly for the app to parse it correctly
- Test scenarios help clarify behavior early, preventing rework later
- Technical flags feed into the architecture phase
