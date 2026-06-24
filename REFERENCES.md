# References — ddruids/mobbin-skill

Docs copiados do repositório (4 arquivos).

---

## `README.md`

# Mobbin UI Research Skill

Turn [Mobbin](https://mobbin.com)'s 300K+ screen library into structured UI research. Give your agent a research goal — it searches Mobbin for screens and flows, visually analyzes the results, and returns pattern clusters, design system components, or competitive comparisons.

Works with Claude Code, Cursor, Gemini CLI, Lovable, and any MCP-compatible agent.

## What it does

- Translates abstract UX goals into concrete Mobbin search queries
- Searches both **individual screens** and **multi-step user flows** (onboarding, checkout, etc.)
- Visually analyzes returned screenshots (layout, hierarchy, components, color)
- Clusters findings into recurring UI patterns with linked references
- Derives research queries from PRDs and product briefs automatically
- Surfaces atomic design system primitives when asked about components
- Supports three output modes: research summary, competitive comparison, product decision log

## Install

Requires [Mobbin MCP](#setup-mobbin-mcp) to be connected first. Copy and Paste in Terminal to install skill

```bash
npx skills add https://github.com/ddruids/mobbin-skill
```

## Usage

Once installed, the skill activates when you ask about UI research, screen patterns, or design system components:

```
"Research how fintech apps handle onboarding"
"Compare how crypto wallets display transaction confirmation"
"What are common components for a SaaS dashboard design system?"
"How do AI apps handle empty states on iOS?"
"Here's my PRD — research the key screens and flows"
```

The skill automatically:
1. Resolves the target platform (iOS or web)
2. Picks the right tool mix — `search_screens` for specific screens, `search_flows` for multi-step journeys
3. Generates 3-7 concrete search queries (or derives them from your PRD)
4. Analyzes the returned screenshots and flow sequences
5. Synthesizes findings into structured output with Mobbin URLs

## What's inside

| File | Purpose |
|------|---------|
| [`SKILL.md`](SKILL.md) | Core skill — screen/flow tool selection, query construction, PRD-driven research, analysis framework, output formats, design system component mode |
| [`references/query-patterns.md`](references/query-patterns.md) | Query formula, platform-specific guidance, 20+ screen query examples, flow query examples by category |
| [`references/synthesis-framework.md`](references/synthesis-framework.md) | 8 analysis lenses, 3 output templates (research summary, competitive comparison, product decision log) |

---

<details id="setup-mobbin-mcp">
<summary><strong>Setup Mobbin MCP</strong></summary>

1. Sign up at [mobbin.com](https://mobbin.com)
2. Click your profile icon (top right) → **Settings** → **MCP**
3. Select your tool and follow the instructions, or use one of these:

**Claude Code:**
```bash
claude mcp add mobbin \
  --transport http https://api.mobbin.com/mcp
```

**Cursor:**
```json
{
  "mcpServers": {
    "mobbin": {
      "serverUrl": "https://api.mobbin.com/mcp"
    }
  }
}
```

**Codex:**
```bash
codex mcp add mobbin --url https://api.mobbin.com/mcp
```

**Lovable:**
```
https://api.mobbin.com/mcp
```

</details>

## License

MIT

---

## `SKILL.md`

---
name: mobbin-ui-research
description: "Use when the user needs UI/UX research using Mobbin MCP — screen references, flow research, competitive audits, pattern exploration, or PRD-driven research. Triggers on: Mobbin search, mobbin research, screen references, product flow research, pattern analysis, competitive UI audit, UX benchmarking, onboarding flow, checkout flow, empty states, error states, confirmation screens, fintech UX, crypto UX, AI product patterns, SaaS dashboard patterns, ecommerce checkout patterns, PRD research, or turning abstract product questions into concrete screen and flow searches. This skill uses the Mobbin MCP server specifically — not Refero or other design reference tools."
---

# Mobbin UI Research Skill

Help the user research UI patterns using Mobbin MCP. Mobbin exposes two tools:

- `search_screens` — AI-powered search that returns individual app screenshots with metadata. Best for studying specific screen types, components, layouts, and states.
- `search_flows` — AI-powered search that returns multi-step user flows (e.g., onboarding, checkout, settings changes). Best for understanding how apps sequence screens across a journey.

Use `image_format: "webp"` on all calls (smaller than jpg, reduces context pressure).

## Core Rule

Translate abstract product or UX goals into concrete screen descriptions before calling `search_screens`. Describe what is visible on the screen, not abstract UX concepts.

Do not search:
- "trust patterns"
- "good onboarding"
- "transaction confidence"

Search:
- "signup screen with progress indicator, phone number input, security message, and continue button"
- "bank transfer pending screen showing amount, recipient, estimated arrival time, and progress indicator"
- "dashboard empty state with illustration, explanatory text, and primary call to action"

## Platform Resolution

The `platform` parameter is required on every call ("ios" or "web").

1. **Explicit**: User says "mobile app", "iOS", "iPhone" -> `ios`. User says "website", "web app", "SaaS", "desktop" -> `web`.
2. **Inferrable**: If the user names a product that is clearly one platform (e.g., "Stripe dashboard" -> web, "Duolingo onboarding" -> ios), infer it.
3. **Ambiguous**: If unclear, ask: "Should I search iOS apps, web apps, or both?"
4. **Both**: For cross-platform comparison, run separate searches per platform and note which results come from which in the synthesis.

## Screens vs. Flows: When to Use Each

| Goal | Tool | Why |
|------|------|-----|
| Study a specific screen type (empty state, confirmation, dashboard) | `search_screens` | Returns individual screens you can compare side by side |
| Study a multi-step journey (onboarding, checkout, account setup) | `search_flows` | Returns the full sequence of screens users go through |
| Component-level or design system research | `search_screens` | Need individual screens to decompose into primitives |
| Understand how apps transition between steps | `search_flows` | Shows the screen-to-screen progression |
| Broad exploratory research | Both | Start with flows for journey context, then drill into specific screens |

For most PRD-driven research, use **both tools**: flows to understand journey structure, screens to study specific moments in depth.

## Workflow

1. Understand the user's research goal.
2. Resolve the target platform.
3. Decide the tool mix: flows for journeys, screens for specific moments, both for broad research.
4. Break the goal into concrete queries — screen descriptions for `search_screens`, journey descriptions for `search_flows`.
5. Generate 3-7 search queries using the query construction rules below.
6. Run searches in batches of 2-3 and **analyze each batch immediately** — write down app names, URLs, and key observations as text before moving to the next batch. Screenshots are large and will be dropped from context if you accumulate too many before synthesizing.
7. After all batches are analyzed, merge the per-batch notes into pattern clusters.
8. Synthesize what the patterns mean for product design.
9. Return actionable recommendations using the output format below.

### Limits

- `search_screens`: Use `limit: 20-30` for broad research, `limit: 10-15` for focused queries. Pass prior result IDs via `exclude_screen_ids` to get fresh results across batches.
- `search_flows`: Use `limit: 3-5` (flows contain multiple screens each, so fewer results = more content). Use `page` to paginate for more variety.

**Important: analyze incrementally.** Do not accumulate all search results before writing. Each batch of screenshots consumes significant context. Write your visual analysis (app names, URLs, layout observations, component notes) as text immediately after each batch returns. Text persists through context compaction; base64 images do not.

## PRD-Driven Research

When the user provides a PRD, architecture doc, or product brief alongside a research request:

1. **Extract key screens and flows** — Read the document and identify the distinct screens, flows, and interaction moments the product needs.
2. **Prioritize by risk** — Research the screens where design decisions are hardest first: novel interactions, complex information density, trust-critical moments, or areas where the team lacks precedent.
3. **Map PRD sections to queries** — Each major feature or flow in the PRD should produce 1-3 search queries. Use `search_flows` for end-to-end journeys described in the PRD and `search_screens` for specific screen types.
4. **Anchor findings to PRD sections** — In the output, reference which part of the PRD each finding applies to so the research directly informs implementation planning.

## Query Construction

### Screen queries (`search_screens`)

Each query should include as many of these as relevant:

- **Product category**: fintech, crypto, banking, SaaS, AI, ecommerce, health, social
- **Screen type**: onboarding, dashboard, settings, confirmation, modal, empty state, error state
- **Visible components**: input field, progress bar, tabs, card, warning banner, timeline, bottom sheet
- **User action**: sign up, transfer money, connect wallet, confirm withdrawal
- **State**: loading, pending, failed, completed, empty, disabled
- **Trust elements**: security message, fee breakdown, confirmation checklist, support link

### Flow queries (`search_flows`)

Describe the journey, not a single screen:

- **Good**: "user onboarding flow with account creation email verification and profile setup"
- **Good**: "checkout flow from cart through payment to order confirmation"
- **Good**: "crypto wallet setup flow with seed phrase backup and verification"
- **Bad**: "onboarding" (too vague)
- **Bad**: "signup screen with email input" (this is a screen query, not a flow)

All queries must be under 500 characters. Use concrete visual language, not product theory.

See `references/query-patterns.md` for the full formula and example catalog with platform-specific tips.

## Working with Results

### Screen results
`search_screens` returns inline images alongside metadata (app name, Mobbin URL, image URL).

- **Visually analyze each screen**: Examine layout, hierarchy, component placement, color usage, and interaction patterns. This visual analysis is the core value.
- **Attribute findings**: Every screen referenced in the output MUST include the app name and its Mobbin screen URL as a clickable markdown link (e.g., [Wise](https://mobbin.com/screens/...)). Never reference an app or screen without its URL.
- **Compare across results**: Look for recurring patterns, outliers, and platform-specific conventions.
- **Progressive depth**: Start with the strongest 3-5 queries. If results are thin, expand with additional queries or broaden search terms.

### Flow results
`search_flows` returns evenly-spaced preview images from each flow alongside metadata (app name, Mobbin URL, screen count).

- **Analyze the journey structure**: How many steps? What's the entry point? Where are decision points? What's the exit?
- **Note step-to-step transitions**: What information carries forward between screens? Where does the app ask for input vs. show confirmation?
- **Compare flow length**: Shorter flows reduce drop-off but may overload individual screens. Note how different apps balance depth vs. breadth.
- **Attribute findings**: Same rule as screens — every flow referenced MUST include app name and Mobbin URL as a clickable link.

## Analysis Framework

For each useful result, analyze through these lenses (full detail in `references/synthesis-framework.md`):

1. **Screen Moment** — What part of the journey is this?
2. **User Job** — What is the user trying to accomplish?
3. **Information Hierarchy** — What appears first, second, last?
4. **Trust Mechanism** — How does the UI reduce uncertainty?
5. **Action Model** — What is the primary CTA? What is secondary?
6. **Progressive Disclosure** — What is hidden until needed?
7. **Recovery** — What happens when the user is blocked?
8. **Reusable Pattern** — What can be applied to the user's product?

## Output Format

Use this structure by default:

```
# Mobbin Research Summary

## Research Goal
Briefly restate the goal.

## Searches Used
List the concrete Mobbin queries used and platform(s) searched.

## Pattern Clusters
Group findings into recurring UI patterns. Every app reference MUST be a markdown link to its Mobbin screen URL.

## Key Observations
Explain what the screens reveal about product design thinking.

## UX Opportunities
Translate findings into design opportunities for the user's product.

## Recommendations
Give concrete, actionable design recommendations.

## Reusable Patterns
List reusable UI patterns the user can apply. Every app reference MUST link to its Mobbin screen URL.
```

If the user asks for a competitive comparison or product decision log, see the templates in `references/synthesis-framework.md`.

## Design System Component Research

When the user asks about **design system components**, **common components**, **UI kit**, or **component library** for a product category, shift the output from screen-level patterns to **atomic UI primitives**.

### Detection
Trigger this mode when the request includes phrases like: "design system", "component library", "common components", "UI kit", "what components do I need", "atoms", "primitives", "building blocks".

### How to analyze
After running searches, do NOT cluster by screen type (e.g., "Home Screen Pattern", "Swap Screen Pattern"). Instead, decompose each screen into its **individual reusable UI elements** — the atoms and molecules that appear across multiple screens and apps.

For each component, identify:
- **What it is**: A single, named primitive (e.g., "Token Icon", not "Portfolio Dashboard")
- **Variants**: Size, state, or style variations observed across apps
- **Where it appears**: Which screens and apps use it (with Mobbin URLs)
- **What makes it domain-specific**: Why this component wouldn't exist in a generic design system

### Output format for design system research

```
# Design System Components: [Category]

## Primitives

### [Component Name]
[One-line description of what the element is]
- **Variants**: [sizes, states, styles observed]
- **Seen in**: [App](url), [App](url) — [brief note on how each implements it]

### [Component Name]
...

## Domain-Specific Components
List components that are unique to this product category and would not exist in a generic design system.

## Standard Components with Domain Variants
List components that exist in any design system but need product-specific adaptations.
```

### Key distinction
- **Screen pattern** (wrong): "Send Transaction Screen — shows amount, address, fee, and confirm button"
- **Primitives** (right): `Truncated Address`, `Currency Display`, `Fee Breakdown Row`, `Slide-to-Confirm`, `Numeric Keypad` — each as independent, reusable atoms

Always decompose screens into their smallest reusable parts. A screen is a composition of primitives, not a component itself.

## Important Behavior

**Every screen or app mentioned in the output must include a clickable Mobbin URL.** No exceptions. If a screen's URL was not returned by the tool, do not reference that screen.

Do not only describe screens visually. Always synthesize the product thinking behind them.

Avoid: "Here are some nice references."
Prefer: "These examples reduce uncertainty by repeating critical transaction details before and after confirmation."

## Scope

This skill uses `search_screens` and `search_flows` from the Mobbin MCP server. Do not invent tools that don't exist (e.g., `get_screen`, `compare_flows`). If a tool call fails, check the tool name — only these two are available.

---

## `references/query-patterns.md`

# Mobbin Query Patterns

## Query Formula

```
[product category] + [screen type] + [visible UI components] + [user state/action]
```

Each query should describe what a person would see on screen. Maximum 500 characters. Use `mode: "deep"` for nuanced queries, `mode: "fast"` for quick lookups.

## Platform-Specific Guidance

### iOS Queries
iOS apps use mobile-native patterns. Include these when relevant:
- Bottom sheets, action sheets, half-modals
- Tab bar navigation, swipe gestures
- Pull-to-refresh, floating action buttons
- System-style alerts and permission dialogs
- Full-screen takeovers for onboarding

### Web Queries
Web apps use browser-native patterns. Include these when relevant:
- Sidebar navigation, top navigation bar, breadcrumbs
- Modals, drawers, popovers, dropdown menus
- Multi-column layouts, data tables, dashboards
- Toast notifications, inline validation
- Sticky headers, scroll-based interactions

## Example Queries by Goal

### Onboarding
- "fintech signup screen with phone number input progress indicator security message and continue button"
- "AI app onboarding screen asking user goal with selectable cards and continue button"
- "AI chat welcome screen with suggested prompts and empty conversation state"
- "AI assistant setup screen with permissions integrations and progress steps"
- "SaaS onboarding checklist with completed and pending steps progress bar and skip option"

### Empty States
- "SaaS dashboard empty state with illustration explanatory text primary CTA and secondary link"
- "inbox empty state with illustration and suggested actions"
- "search results empty state with alternative suggestions and filters"

### Error States
- "payment failed screen with error message retry button support link and transaction details"
- "network error screen with retry button offline indicator and cached content"
- "form validation error state with inline error messages and highlighted fields"

### Progress and Pending States
- "transfer pending screen with amount recipient estimated arrival time progress tracker and help link"
- "file upload progress screen with progress bar cancel button and file details"
- "order tracking screen with timeline status steps estimated delivery and contact support"

### Confirmation Screens
- "crypto withdrawal confirmation screen with recipient address network fee warning text and confirm button"
- "money transfer confirmation screen with fee breakdown recipient details and confirm button"
- "subscription upgrade confirmation with plan comparison price change and billing date"

### Success States
- "payment success screen with receipt transaction ID and share button"
- "account created success screen with next steps checklist and primary CTA"
- "transfer completed screen with transaction summary download receipt and done button"

### Settings
- "notification settings screen with grouped toggles channels frequency controls and save button"
- "privacy settings screen with data sharing toggles delete account option and explanation text"
- "security settings screen showing recovery phrase backup option and warning banner"

### Wallet and Crypto
- "crypto wallet seed phrase backup screen with warning text and continue button"
- "wallet recovery phrase confirmation screen with numbered words and validation error"
- "crypto portfolio dashboard with token balances price charts and send receive buttons"

### Fintech Transfers
- "bank transfer pending screen showing amount recipient estimated arrival time and progress status"
- "international transfer screen with exchange rate fee breakdown and delivery estimate"
- "recurring payment setup screen with frequency amount date and recipient fields"

## Flow Query Examples (`search_flows`)

Flow queries describe journeys, not individual screens. Focus on what the user goes through from start to finish.

### Onboarding & Signup
- "user signup flow with email verification profile creation and first-use tutorial"
- "AI app onboarding flow with goal selection permissions and first interaction"
- "SaaS trial onboarding flow with workspace setup team invite and feature tour"

### Checkout & Payment
- "ecommerce checkout flow from cart review through shipping payment to order confirmation"
- "subscription upgrade flow with plan comparison billing entry and confirmation"
- "in-app purchase flow with feature preview pricing and payment"

### Account & Settings
- "account setup flow with profile photo username bio and preferences"
- "two-factor authentication setup flow with method selection and verification"
- "notification preferences flow with channel selection frequency and confirmation"

### Crypto & Fintech
- "crypto wallet creation flow with seed phrase generation backup and verification"
- "money transfer flow from recipient selection through amount entry to confirmation"
- "KYC verification flow with document upload selfie and review status"

### Tips
- Keep flow queries broader than screen queries — describe the journey arc, not UI components
- Use `limit: 3-5` since each flow contains multiple screens
- Combine with `search_screens` to drill into specific steps that need deeper study

## Multi-Batch Research

For broad research topics, use `exclude_screen_ids` to get fresh results across multiple calls:

1. Run the first search with `limit: 20`.
2. Collect the screen IDs from the results.
3. Pass them as `exclude_screen_ids` in the next call with a varied query.
4. Repeat to build a diverse set of references.

This is useful when the user wants comprehensive coverage across many apps rather than the top results for a single query.

---

## `references/synthesis-framework.md`

# UX Synthesis Framework

Analyze Mobbin results through these lenses, then structure output using the appropriate template.

---

## Part 1: Analysis Lenses

### 1. Screen Moment
What part of the user journey is this? (First launch, onboarding, core task, settings, error recovery, etc.)

### 2. User Job
What is the user trying to accomplish at this moment? What triggered them to be on this screen?

### 3. Information Hierarchy
What appears first, second, and last? What gets the most visual weight? What is the reading order?

### 4. Trust Mechanism
How does the interface reduce uncertainty or anxiety? Look for: security messages, fee breakdowns, confirmation checklists, familiar brand cues, social proof, progress indicators.

### 5. Action Model
What is the primary CTA? What is secondary? Is there a destructive action, and how is it visually separated? How many actions compete for attention?

### 6. Progressive Disclosure
What is hidden until needed? Look for: expandable sections, "learn more" links, advanced settings behind toggles, tooltips, info icons.

### 7. Recovery
What happens when the user is blocked, makes an error, or wants to go back? Look for: error states, undo options, back buttons, help links, retry mechanisms.

### 8. Reusable Pattern
What specific UI pattern could be extracted and applied to the user's product? Name it concretely (e.g., "inline fee breakdown with expandable details" not "good transparency").

---

## Part 2: Output Templates

Choose the template that best fits the user's research goal.

### Template A: Research Summary (default)

Use when the user wants to understand patterns across a category or flow type.

```
# Mobbin Research Summary

## Research Goal
Briefly restate the goal.

## Searches Used
List the concrete Mobbin queries used and platform(s) searched.

## Pattern Clusters
Group findings into recurring UI patterns. Attribute each to specific apps.

## Key Observations
Explain what the screens reveal about product design thinking.

## UX Opportunities
Translate findings into design opportunities for the user's product.

## Recommendations
Give concrete, actionable design recommendations.

## Reusable Patterns
List reusable UI patterns the user can apply, with app references.
```

### Template B: Competitive Comparison

Use when the user wants to compare how different apps handle the same screen or flow.

```
# Competitive UI Comparison

## Screen / Flow Compared
What specific moment is being compared.

## Comparison Table

| App | Pattern | Strength | Weakness | Reusable Idea |
|-----|---------|----------|----------|---------------|
| ... | ...     | ...      | ...      | ...           |

## Winner Analysis
Which approach works best and why.

## Recommendation
What the user's product should adopt, combine, or avoid.
```

### Template C: Product Decision Log

Use when the user wants to formalize research into a design decision.

```
# Product Decision Log

## Context
What prompted this research. What product area is affected.

## Problem
The specific UX problem being solved.

## Options Considered
List the patterns found, with app references.

## Decision
The recommended approach.

## Rationale
Why this approach, grounded in the Mobbin findings.

## Consequences
What this decision enables and constrains.

## Success Metrics
How to measure whether this decision worked.
```

---

## Choosing a Template

- User says "research X patterns" or "how do apps handle X" -> **Template A**
- User says "compare how apps do X" or "competitive audit" -> **Template B**
- User says "help me decide" or "what should we do for X" -> **Template C**
- If unclear, default to **Template A** and offer to convert to another format.
