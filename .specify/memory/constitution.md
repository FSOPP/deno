<!--
  ╔══════════════════════════════════════════════════════════════╗
  ║                   SYNC IMPACT REPORT                        ║
  ╠══════════════════════════════════════════════════════════════╣
  ║ Version change: N/A (initial) → 1.0.0                      ║
  ║                                                             ║
  ║ Added principles:                                           ║
  ║   I.   Static First Delivery                                ║
  ║   II.  Simplicity over Tooling                              ║
  ║   III. Accessibility (a11y) & SEO                           ║
  ║   IV.  Architectural Sovereignty                            ║
  ║                                                             ║
  ║ Added sections:                                             ║
  ║   - Core Principles (4 articles)                            ║
  ║   - Constraints & Boundaries                                ║
  ║   - Development Workflow                                    ║
  ║   - Governance                                              ║
  ║                                                             ║
  ║ Removed sections: None (initial creation)                   ║
  ║                                                             ║
  ║ Template compatibility:                                     ║
  ║   .specify/templates/plan-template.md        ✅ compatible  ║
  ║     — "Constitution Check" section dynamically resolves     ║
  ║       against these 4 principles. No update required.       ║
  ║   .specify/templates/spec-template.md        ✅ compatible  ║
  ║     — Requirements & success criteria sections are generic  ║
  ║       enough to accommodate a11y/SEO gates. No update.      ║
  ║   .specify/templates/tasks-template.md       ✅ compatible  ║
  ║     — Phase structure and parallel markers are principle-    ║
  ║       agnostic. No update required.                         ║
  ║   .specify/templates/agent-file-template.md  ✅ compatible  ║
  ║   .specify/templates/checklist-template.md   ✅ compatible  ║
  ║                                                             ║
  ║ Follow-up TODOs: None                                       ║
  ╚══════════════════════════════════════════════════════════════╝
-->

# Deno Project Constitution

> **This document is the supreme governing reference for all AI-assisted
> development on this project.** Every code generation request, architectural
> proposal, library suggestion, and pull-request review MUST be evaluated
> against the principles codified below. When ambiguity arises, this
> constitution takes precedence over ad-hoc instructions.

---

## Core Principles

### I. Static First Delivery 🌐

All deliverable output MUST consist exclusively of **pure HTML, CSS, and
JavaScript**. No server-side runtime execution is permitted in any
production artifact.

**Non-negotiable rules:**

- The final build artifact MUST be a collection of static files
  deployable to any CDN (e.g., Cloudflare Pages, Netlify, GitHub Pages,
  Vercel static, AWS S3 + CloudFront) without a compute layer.
- Server-side languages (Node.js, Python, Ruby, Go, PHP, etc.) MUST NOT
  appear in production delivery paths. If a build-time tool written in
  such a language is used, it MUST be confined to CI/CD and MUST NOT be
  required at runtime.
- Dynamic behavior MUST be achieved through client-side JavaScript,
  Web APIs, or third-party client-side SDKs only.
- Any data fetching MUST target external APIs or static JSON/data files;
  no custom backend endpoints are permitted unless explicitly authorized
  by a constitutional amendment.

**Rationale:** Static delivery guarantees predictable performance,
eliminates server-side failure modes, simplifies caching, and minimizes
attack surface. CDN-first architecture ensures global low-latency access.

---

### II. Simplicity over Tooling 🛠️

All implementations MUST default to **vanilla HTML, CSS, and JavaScript**.
Heavy frameworks, transpilers, and complex build pipelines are forbidden
unless explicitly authorized through a documented governance exception.

**Non-negotiable rules:**

- Vanilla JS MUST be the default. Introduction of frameworks (React, Vue,
  Angular, Svelte, etc.) requires a formal exception recorded in the
  Governance section of this document.
- CSS MUST be written as plain CSS. Preprocessors (Sass, Less, PostCSS
  with non-standard plugins) MUST NOT be introduced without authorization.
- Build steps MUST be minimal. A project MAY use a static-site generator
  or simple bundler only when the complexity is justified in writing and
  approved. The default expectation is zero build steps—files are authored
  as they are served.
- Third-party libraries MUST be evaluated against the **weight test**:
  if the functionality can be achieved in ≤50 lines of vanilla code, a
  dependency MUST NOT be added.
- Package managers (npm, yarn, pnpm) MUST NOT be used in production
  builds unless a governance exception exists. CDN-hosted ES modules or
  vendored scripts are the approved distribution method.

**Rationale:** Tooling complexity is the primary source of maintenance
debt in frontend projects. Vanilla implementations are universally
understood, require no version management, and produce the smallest
possible payloads.

---

### III. Accessibility (a11y) & SEO 🔍

Every page and component MUST meet a baseline of accessibility and search
engine discoverability. This is a non-negotiable quality gate—not an
afterthought.

**Non-negotiable rules:**

- **Semantic HTML**: All markup MUST use appropriate semantic elements
  (`<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<footer>`,
  `<aside>`, `<figure>`, `<time>`, etc.) rather than generic `<div>` or
  `<span>` wrappers. Heading hierarchy (`<h1>`–`<h6>`) MUST be logical
  and unbroken.
- **ARIA roles**: Where native semantics are insufficient (e.g., custom
  widgets, dynamic content regions), appropriate `role`, `aria-label`,
  `aria-live`, `aria-expanded`, and related attributes MUST be applied.
- **Keyboard navigation**: All interactive elements MUST be reachable
  and operable via keyboard alone. Focus order MUST follow logical
  document flow. Visible focus indicators MUST NOT be suppressed.
- **Color contrast**: Text MUST meet WCAG 2.1 AA minimum contrast ratios
  (4.5:1 for normal text, 3:1 for large text).
- **Image alt text**: Every `<img>` MUST include a meaningful `alt`
  attribute. Decorative images MUST use `alt=""` with `role="presentation"`.
- **Meta-tag structure**: Every page MUST include at minimum:
  `<title>`, `<meta name="description">`, `<meta name="viewport">`,
  `<link rel="canonical">`, and Open Graph tags (`og:title`,
  `og:description`, `og:image`, `og:url`).
- **Structured data**: Pages SHOULD include JSON-LD structured data
  (`<script type="application/ld+json">`) appropriate to the content type.
- **Language attribute**: The `<html>` element MUST declare `lang`.

**Rationale:** Accessible code is correct code. Semantic markup improves
machine readability, which directly benefits search ranking, screen
readers, and future maintainability. These are not optional polish—they
are structural requirements.

---

### IV. Architectural Sovereignty ⚖️

This constitution is the **authoritative source of truth** for all
architectural decisions. Any AI agent or human contributor MUST consult
this document before proposing structural changes, adding dependencies,
or altering the delivery model.

**Non-negotiable rules:**

- Before proposing any new library, framework, build tool, or structural
  change, the proposer MUST verify compliance with all four principles
  in this constitution and cite which articles are satisfied.
- No architectural change is valid unless it can demonstrate that it does
  not violate any principle herein, or a formal amendment has been ratified
  to permit the exception.
- AI agents MUST include an explicit **Constitution Compliance Statement**
  when suggesting code that introduces external dependencies, new tooling,
  or deviates from vanilla implementations. This statement MUST reference
  the specific articles consulted.
- Proposals that conflict with this constitution MUST be flagged
  immediately with a clear explanation of the conflict, alternative
  approaches that comply, and (if the proposer believes the constitution
  should change) a formal amendment proposal per the Governance section.
- The burden of proof lies with the party proposing deviation—not with
  the constitution.

**Rationale:** Without a single authoritative reference, architectural
decisions fragment across conversations and context windows. This
principle ensures coherent, long-term governance regardless of which
agent or contributor is active.

---

## Constraints & Boundaries

### Technology Stack

| Layer        | Permitted                              | Forbidden                          |
|------------- |----------------------------------------|------------------------------------|
| Markup       | HTML5                                  | JSX, template engines              |
| Styling      | CSS3, CSS Custom Properties            | Sass, Less, CSS-in-JS, Tailwind*   |
| Scripting    | Vanilla JS (ES2020+), ES Modules       | TypeScript*, React, Vue, Angular   |
| Delivery     | Static CDN, GitHub Pages, Netlify      | Node servers, serverless functions  |
| Data         | Static JSON, external APIs (client)    | Custom backends, databases          |
| Assets       | Self-hosted or CDN-hosted              | npm/yarn runtime dependencies       |

*\* Items marked with \* may be authorized via governance amendment only.*

### Performance Budgets

- **First Contentful Paint**: MUST be under 1.5 seconds on a 3G
  connection (Lighthouse simulated throttling).
- **Total page weight**: SHOULD remain under 500 KB uncompressed for
  initial load (excluding lazy-loaded media).
- **JavaScript payload**: MUST NOT exceed 100 KB uncompressed for any
  single page's critical path.

### Security Baseline

- No inline `<script>` blocks in production HTML; all JS MUST be in
  external files to support Content Security Policy headers.
- Third-party scripts MUST be loaded with `integrity` (SRI) attributes
  when served from external CDNs.
- All external resource URLs MUST use HTTPS.

---

## Development Workflow

### Pre-Implementation Checklist

Before writing any code, the following gates MUST be satisfied:

1. **Principle I check** — Will the output be purely static? If not, STOP.
2. **Principle II check** — Is the approach vanilla? If a dependency is
   proposed, has it passed the weight test (≤50 lines)?
3. **Principle III check** — Does the markup plan include semantic
   elements, ARIA strategy, and meta-tag structure?
4. **Principle IV check** — Has this constitution been consulted? Is a
   compliance statement prepared?

### Code Review Gates

Every pull request MUST be evaluated against:

- [ ] No server-side runtime code in delivery artifacts
- [ ] No unauthorized frameworks or build tools
- [ ] Semantic HTML with correct heading hierarchy
- [ ] ARIA attributes on custom interactive elements
- [ ] Required meta tags present on all pages
- [ ] Keyboard navigability verified
- [ ] Contrast ratios meet WCAG 2.1 AA
- [ ] JavaScript payload within budget
- [ ] External scripts include SRI hashes
- [ ] Constitution Compliance Statement included (for structural changes)

### File Organization

```
project-root/
├── index.html              # Entry point
├── pages/                  # Additional HTML pages
├── css/
│   ├── reset.css           # Minimal CSS reset
│   ├── variables.css       # CSS Custom Properties
│   └── main.css            # Primary styles
├── js/
│   ├── main.js             # Entry module
│   └── modules/            # ES Module components
├── assets/
│   ├── images/
│   ├── fonts/
│   └── data/               # Static JSON data files
├── .specify/               # Project governance & specs
└── docs/                   # Documentation
```

---

## Governance

### Authority

This constitution is the highest-precedence document in the project. It
supersedes all other practices, conventions, and ad-hoc instructions. In
any conflict between this document and other guidance, this document wins.

### Amendment Procedure

1. **Proposal**: Submit a written amendment proposal that identifies the
   article(s) to modify, the proposed change, and a justification.
2. **Impact assessment**: Evaluate how the change affects existing code,
   delivery pipelines, and all four core principles.
3. **Approval**: The project maintainer(s) MUST explicitly approve the
   amendment. AI agents MUST NOT self-approve amendments.
4. **Documentation**: The approved amendment MUST be recorded in this
   file with an updated version number and `LAST_AMENDED_DATE`.
5. **Migration plan**: If the amendment invalidates existing code, a
   migration plan MUST be included before the amendment takes effect.

### Versioning Policy

This constitution follows semantic versioning:

- **MAJOR** (X.0.0): Removal or fundamental redefinition of a core
  principle; backward-incompatible governance changes.
- **MINOR** (0.X.0): Addition of a new principle, section, or materially
  expanded guidance.
- **PATCH** (0.0.X): Clarifications, wording improvements, typo fixes,
  non-semantic refinements.

### Compliance Review

- All pull requests MUST include evidence of constitution compliance.
- Periodic audits (at least once per quarter or per major feature) SHOULD
  verify that accumulated changes have not drifted from these principles.
- AI agents MUST re-read this document at the start of every new
  conversation or context window to prevent drift.

**Version**: 1.0.0 | **Ratified**: 2026-02-09 | **Last Amended**: 2026-02-09
