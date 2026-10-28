---
name: jira-report-page
description: Use when building an internal report page that presents Jira ticket completeness/compliance review results — covers the stack, Tailwind v4 + shadcn tokens, the "Audit Dossier" visual direction, and a ready-to-paste design brief for a coding agent.
---

# Jira Ticket Evaluation Report: Agent Toolkit & Design Direction

> Context: internal deployment. The page presents completeness and compliance review results for Jira tickets.
> Goal: get a coding agent to produce a **credible, restrained, readable** business report — not another SaaS template.

## When to use

- You need an internal report page showing whether Jira tickets are complete and follow the writing standard
- You need to decide the stack, which skills to install, and how to describe the visual direction to the agent
- A first version already exists but reads like an admin template or a SaaS landing page, and needs to be pulled back toward an "audit report"

## How to use

1. Treat the table in §1 as the baseline stack; swap the component layer only if the team already standardizes on antd/Mantine
2. Install the five core skills in §3. Add marketing-oriented ones (`frontend-design`) only when explicitly needed
3. Paste the markdown block in §4 into `docs/design-brief.md`, or inline into the prompt
4. Drop the §5 snippet into the repo root as `AGENTS.md` so the constraints persist across sessions
5. Before shipping, run the §6 checklist and have the agent verify with Playwright screenshots

---

## 1. One stack is enough

| Layer | Choice |
|---|---|
| Build | Vite 8 + React 19 + TypeScript |
| Styling | Tailwind CSS 4.3 (`@tailwindcss/vite`, CSS-first `@theme`) |
| Components | shadcn/ui (`-b base`, components vendored into the repo) + Base UI |
| Charts | CSS bars for scores; Recharts 3 for trends/composition; ECharts 6 for complex charts |
| Icons | lucide-react (imported per-icon, bundled) |
| Type | Source Han Sans VF body + Inter numerals + Source Han Serif (display only) + Sarasa Mono SC (evidence) |
| Tables | Plain `<table>`; TanStack Table 9 only for sorting, multi-column filtering, or virtualization |
| Testing | Playwright (agent self-verification + screenshot baselines) + Vitest (pure functions) |

**Why not Mantine / antd**: they get you there faster, but their default look *is* the admin template. This page's job is to make the verdict believable — it needs document feel and evidence feel, not stacked components. When resources are not a constraint, there is no reason to trade that away.

---

## 2. Resources are not the bottleneck — spend on these three

With an internal npm mirror and a full font library, the usual offline compromises are off the table. What actually decides quality is the following.

### 2.1 Typography

| Purpose | Face | Notes |
|---|---|---|
| Body | Source Han Sans VF (Noto Sans SC VF) / HarmonyOS Sans | 14px, line-height 1.65; never use weight 300 for CJK |
| Numerals, Latin, units | Inter VF | Global `font-variant-numeric: tabular-nums`; scores and percentages must align |
| Display (report title, total score) | Source Han Serif | **Only at ≥24px**; serif at 13–14px renders muddy on Windows |
| Evidence quotations | Sarasa Mono SC | Best mono for mixed CJK/Latin; one more CJK set than JetBrains Mono |

A full CJK font is 5–10MB. Slice it with `cn-font-split` or `fontmin` by `unicode-range`, serve `woff2`, and `preload` the numeral face separately. Internal bandwidth is not the issue — first-paint font swapping is what destroys the formality of a report.

### 2.2 Component foundation

An npm mirror solves **packages**; it does not solve the **shadcn registry** (the CLI fetches JSON from ui.shadcn.com). Pick one path and set it up once:

- Host the registry statically on the intranet and point `registries` in `components.json` at it, so the agent can keep using `shadcn add`;
- Or run `init` + `add` on a networked machine, commit `components/ui/*` into the repo, and let the agent edit local files only.

Also vendor the docs the agent will need: `ui.shadcn.com/llms.txt`, `base-ui.com/llms.txt`, and the Tailwind theme/utility pages into `docs/vendor/`. Write "read the vendored docs before writing component code" into AGENTS.md.

### 2.3 Verification loop

Most bugs in a report page appear *after* something is expanded. Give Playwright screenshot baselines (`toHaveScreenshot`) across three viewports and two states (default / expanded). The agent diffs its own changes, which catches layout regressions better than any code review. There is no reason to skip this step.

---

## 3. Skills: install five, three optional

### Core

| Skill | Why this one |
|---|---|
| [`interface-design`](https://skills.sh/dammyjay93/interface-design/interface-design) | Craft guidance aimed squarely at **dashboards, admin panels, and data interfaces**. A better fit than `frontend-design`, which is marketing-page oriented (hero sections, typographic personality) and pulls this page off course |
| [`shadcn`](https://skills.sh/shadcn-ui/ui/shadcn) | Reads `components.json` for project context (base library, icon set, installed components) and generates the correct API, instead of hand-rolling Dialog/Popover focus traps and scroll locks |
| [`webapp-testing`](https://skills.sh/anthropics/skills/webapp-testing) | Playwright-driven: the agent opens the page, screenshots it, expands a row, and checks the result. The classic failure here ("layout collapses after expand") only surfaces when you actually run it |
| [`web-design-guidelines`](https://skills.sh/vercel-labs/agent-skills/web-design-guidelines) | For the review pass: accessibility, semantics, and form details against the Web Interface Guidelines |
| [`accessibility`](https://skills.sh/addyosmani/web-quality-skills/accessibility) | Reviewers live on the keyboard; focus order, contrast, and ARIA need a dedicated pass |

### Optional

| Skill | When to add it |
|---|---|
| [`impeccable`](https://skills.sh/pbakaus/impeccable/impeccable) | The page runs; you want one critique/polish round |
| [`tanstack-table`](https://skills.sh/tanstack-skills/tanstack-skills/tanstack-table) | The ticket list needs sorting, multi-column filters, or virtualization |
| [`frontend-design`](https://skills.sh/anthropics/skills/frontend-design) | Only if you want stronger typographic opinion — and you must tell it explicitly: **this is a dense internal report, not a landing page; ignore the hero and entrance-animation guidance** |

Install (run on a networked machine, then commit `.agents/skills/`):

```bash
npx skills add dammyjay93/interface-design --skill interface-design -a universal
npx skills add shadcn-ui/ui -a universal
npx skills add anthropics/skills --skill webapp-testing -a universal
npx skills add vercel-labs/agent-skills --skill web-design-guidelines -a universal
npx skills add addyosmani/web-quality-skills --skill accessibility -a universal
```

> Do not stack `frontend-design` + `impeccable` + `ui-ux-pro-max` + taste variants. Three or more "taste" skills fight each other and the output gets *more* generic, not less.

---

## 4. Design direction: the brief to hand the agent

Paste the block below verbatim into the prompt or `docs/design-brief.md`. It defines **style constraints**, not implementation. Implementation is the agent's job; the standard of judgment lives here.

```markdown
# Design direction: Audit Dossier

This is an internal report page for Jira ticket evaluation, in English, used by reviewers and team leads.
The register should read like an **audit workpaper / QA report**: precise, restrained, evidence-first, traceable.
It is not a SaaS marketing site, not a data-wall dashboard, not an AI product landing page.

## In one line
Build credibility with typography, not with decoration.

## Page structure (top to bottom)
1. Toolbar, 56px: breadcrumb, ticket key, filters, export, print
2. Report header: ticket title, verdict badge (Pass / Pass with conditions / Fail), total score (large, tabular-nums), review timestamp
3. Score overview: one horizontal bar per dimension with weight and score, so the weak spot is visible at a glance
4. Key findings: grouped as Blocking / Major / Minor; every item must carry quoted evidence
5. Dimension detail: a table, expandable to show the rule hit and the reasoning
6. Raw ticket fields: collapsed by default (title, description, acceptance criteria, links)
7. Footer: rubric version, model/rule source, generated-at, disclaimer

## Layout
- Three columns: 240px ticket rail (collapsible) | body capped at 880px measure | 260px sticky metadata panel
- Left-aligned text, never centered; keep CJK paragraphs at 36–45 characters per line, English at 60–75
- 56px between sections, 32px between sub-blocks; do not wrap every block in a card

## Color (OKLCH, semantic only)
- Canvas oklch(0.98 0.004 95); content surface pure white; divider oklch(0.91 0.006 95)
- Primary text oklch(0.24 0.01 260); secondary oklch(0.52 0.01 260)
- Pass oklch(0.52 0.11 155) | Warning oklch(0.62 0.13 70) | Fail oklch(0.52 0.16 25)
- Single accent for links/selection: ink blue oklch(0.45 0.09 255)
- **Color encodes verdict only, never decoration.** Score magnitude comes from bar length plus the number, not from a traffic light
- Banned: purple/indigo primary, gradients, tinted icon circles, multi-color charts

## Typography
- Body: Source Han Sans VF (Noto Sans SC VF), 14px / 1.65, weight 400
- Numerals and Latin: Inter VF; tabular-nums on every score, percentage, and date
- Display and total score: Source Han Serif, only at ≥24px; never mix serif into body copy
- Evidence quotations: Sarasa Mono SC, 13px

## Shape
- Radius 4px (6px on buttons and inputs); separate with 1px borders; shadows only on overlays
- Table rows 36–40px, cell padding 8/12; numbers right-aligned, labels left-aligned
- Icons from lucide at 16/18px, line style, inheriting text color; no emoji

## Motion
- State feedback only: expand/collapse 140ms ease-out, filtered results 120ms fade
- No page-load choreography, no card hover displacement, no counting-up numbers
- Respect prefers-reduced-motion

## Evidence and copy
- Every deduction quotes the ticket verbatim: 2px left rule, mono 13px, field-name label
- Missing fields read "Not provided" — never invent content, never soften it to "N/A"
- Verdicts are actionable: "Add acceptance criteria and re-review"
- Banned vocabulary: AI-powered, insights, one-click, leverage, ✨ or any decorative glyph
- If the product ships in another language, keep these tone rules and translate labels only

## Print
- @media print: hide the rail and toolbar, single column, A4 pagination, avoid breaking blocks across pages, keep the rubric footer

## Hard bans
1. No hero section, no tagline, no gradient background
2. No grid of "tinted icon + big number + small label" KPI cards
3. No rounded-2xl, no shadow-xl, no glassmorphism
4. No more than two chart colors
5. No placeholder or lorem-ipsum content
6. No fade-in-up on every section
```

---

## 5. AGENTS.md snippet

```markdown
## Project
Internal Jira ticket evaluation report. Vite 8 + React 19 + TS + Tailwind 4.3 + shadcn(base).

## Commands
- pnpm dev / build / lint / test
- Before commit: pnpm build && pnpm lint && pnpm test

## Constraints
- Dependencies come from the internal npm mirror; no CDN, no remote fonts, no runtime calls to external APIs
- Fonts are self-hosted in public/fonts (woff2, sliced); preload the numeral face
- Styling uses Tailwind classes + `@theme` tokens only; no new CSS-in-JS dependency
- Read docs/vendor/ (shadcn, base-ui, Tailwind) before writing component code; do not rely on memory
- Visual direction lives in docs/design-brief.md; it wins any conflict
- Allowed libraries: react, tailwind, shadcn(base), lucide-react, recharts (as needed), date-fns
- Justify any new dependency before adding it
```

---

## 6. Acceptance checklist

- [ ] No horizontal scroll at 1280 / 1440 / 1920; the metadata panel never covers the body
- [ ] Keyboard Tab reaches the ticket rail, the dimension table, and an expanded row; focus is visible
- [ ] Contrast: body and secondary text ≥ 4.5:1; verdict is never conveyed by color alone
- [ ] A 60+ character ticket title truncates at two lines without breaking the layout
- [ ] Layout survives both extremes: all dimensions at 0 and all at full score
- [ ] Print preview: single column, no rail, rubric footer present
- [ ] Empty states: no tickets, no findings, rubric not matched — each has explicit copy
- [ ] No layout shift on first paint (font-display: swap + size fallback)
