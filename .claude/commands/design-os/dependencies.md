# Dependencies Planning

You are helping the user identify and evaluate external dependencies for their product. This phase can be run after the product roadmap is defined, ideally before or alongside architecture planning.

## Why Dependencies Matter Early

Identifying dependencies early helps:
- Avoid scope creep from underestimated integration work
- Surface licensing concerns before implementation
- Identify learning curves and documentation needs
- Plan for fallback options if a dependency doesn't work out
- Estimate true project complexity

## Step 1: Analyze Product Requirements

Read the product overview and roadmap. For each section, identify potential needs for:

1. **External Libraries** - Code dependencies
2. **System APIs** - Platform capabilities (file system, networking, etc.)
3. **Third-Party Services** - External APIs, databases, auth providers
4. **Build Tools** - Compilers, bundlers, package managers

## Step 2: Categorize Dependencies

Present dependencies in categories:

"Based on your product requirements, here are the dependencies I've identified:

**Core Dependencies** (required for basic functionality):
- [Dependency]: [Why needed]

**Feature Dependencies** (required for specific features):
- [Dependency]: [Which feature, why needed]

**Optional Dependencies** (nice-to-have or one of several options):
- [Dependency]: [Purpose, alternatives]

**System Requirements** (platform/OS capabilities needed):
- [Capability]: [Why needed]"

## Step 3: Evaluate Each Dependency

For significant dependencies, evaluate:

### Library Evaluation Criteria
- **Maintenance**: Is it actively maintained? When was last release?
- **Popularity**: GitHub stars, npm downloads, community size
- **Documentation**: Is it well-documented?
- **License**: Compatible with your project?
- **Size**: Bundle size impact (for web), binary size (for native)
- **Platform Support**: Does it work on your target platform(s)?

### Service Evaluation Criteria
- **Reliability**: Uptime history, SLA
- **Pricing**: Free tier limits, scaling costs
- **Lock-in**: How hard to migrate away?
- **Data Privacy**: Where is data stored?

Use AskUserQuestion to discuss critical choices:

"For [requirement], there are a few options:

1. **[Option A]** - [Pros/cons]
2. **[Option B]** - [Pros/cons]
3. **Build custom** - [When this makes sense]

Which approach would you prefer?"

## Step 4: Identify Risks

Flag dependency-related risks:

"**Dependency Risks to Consider:**

| Dependency | Risk | Severity | Mitigation |
|------------|------|----------|------------|
| [Name] | [What could go wrong] | High/Med/Low | [Fallback plan] |"

Common risks to check:
- Library abandonment
- Breaking changes in major versions
- License changes
- Service discontinuation
- Pricing changes
- Performance issues at scale

## Step 5: Create Dependencies Document

Create `product/dependencies.md`:

```markdown
# Dependencies

## Platform Requirements
- **Platform:** [Selected platform]
- **Minimum Version:** [e.g., macOS 14+, iOS 17+, Node 18+]
- **System Capabilities:** [List required OS features]

## Core Dependencies

### [Library/Service Name]
- **Purpose:** [Why needed]
- **Version:** [Recommended version]
- **License:** [License type]
- **Documentation:** [Link]
- **Alternatives Considered:** [Other options and why not chosen]

### [Library/Service Name]
...

## Feature-Specific Dependencies

### [Feature: Name]
- **[Dependency]:** [Purpose]

### [Feature: Name]
...

## Development Dependencies
- [Tool]: [Purpose]

## Dependency Risks

| Dependency | Risk | Mitigation |
|------------|------|------------|
| [Name] | [Risk] | [Plan] |

## Estimated Learning Curve
- [Dependency with steep learning curve]: [Notes on complexity]

## Proof-of-Concept Recommendations
- [ ] [Dependency that should be validated before committing]
- [ ] [Integration that may be tricky]
```

## Step 6: Confirm and Next Steps

"I've created your dependencies document at `product/dependencies.md`.

**Core Dependencies:** [Count]
**Feature Dependencies:** [Count]
**Risks Flagged:** [Count]

**Recommendations:**
- [Any POC suggestions]
- [Any alternatives worth keeping in mind]

This will be included in the export package. You can run `/architecture` to design how these dependencies integrate into your system."

## Platform-Specific Dependency Considerations

### macOS/iOS Native
- Swift Package Manager vs CocoaPods vs manual
- Framework vs library linking
- App Store guidelines for third-party code
- Notarization requirements

### Web Application
- npm package size and tree-shaking
- CDN vs bundled
- SSR compatibility
- Browser support requirements

### CLI Tools
- Static vs dynamic linking
- Cross-compilation support
- Runtime requirements on target machines

### Cross-Platform
- Platform-specific implementations
- Native bridge complexity
- Feature parity across platforms

## Common Dependency Categories by Platform

### macOS/iOS
- Networking: URLSession (built-in), Alamofire
- JSON: Codable (built-in), SwiftyJSON
- Database: Core Data, SwiftData, SQLite, Realm
- UI Components: Built-in frameworks
- Markdown: cmark, Down, Ink
- Syntax Highlighting: Splash, Highlighter

### Web (React)
- State: Zustand, Jotai, Redux, React Query
- Routing: React Router, TanStack Router
- Styling: Tailwind, styled-components, CSS Modules
- Forms: React Hook Form, Formik
- Tables: TanStack Table
- Animation: Framer Motion

### CLI
- Argument Parsing: Swift Argument Parser, Commander (Node), Click (Python)
- Output Formatting: Chalk (Node), termcolor (Python)
- Progress: ora (Node), tqdm (Python)

## Important Notes

- Better to identify dependencies early than discover gaps during implementation
- Consider the "bus factor" - what if a dependency is abandoned?
- Open source doesn't mean free of concerns - evaluate maintenance
- Document why alternatives were rejected (prevents re-litigating later)
- Dependencies in architecture docs should reference this document
