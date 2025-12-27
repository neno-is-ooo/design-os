# Design OS

**Design OS is a methodology, not a starter template.**

Do NOT edit source files directly. Instead, run the planning commands to define your product.

## How to Start

1. Run `/product-vision` — Define what you're building and choose your platform
2. Run `/product-roadmap` — Break the product into sections
3. Run `/data-model` — Define core entities
4. Continue through the planning flow...

The methodology guides you through structured product planning before any code is written.

## Quick Reference

| Command | Purpose |
|---------|---------|
| `/product-vision` | Define product, platform, problems/solutions |
| `/product-roadmap` | Break into sections with complexity assessment |
| `/data-model` | Define entities and relationships |
| `/dependencies` | Evaluate external libraries and services |
| `/design-tokens` | Choose colors and typography (web only) |
| `/design-shell` | Design app navigation/chrome (UI platforms only) |
| `/shape-section` | Define section specs with feasibility checks |
| `/sample-data` | Create sample data and types |
| `/architecture` | Plan technical architecture |
| `/design-screen` | Create screen designs (UI platforms only) |
| `/export-product` | Generate implementation handoff package |

## Platform Support

Design OS supports multiple platforms:
- **Web Application** — React/Tailwind exports
- **macOS/iOS Native** — Swift type definitions and UI specs
- **CLI Tool** — Command structure and output specs
- **API/Backend** — Endpoint and schema specs

The platform choice (set in `/product-vision`) determines which phases apply and what gets exported.

---

For detailed documentation, refer to @agents.md
