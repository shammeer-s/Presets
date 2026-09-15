---
name: weaving
description: "Weaving: channeling assembled into a visible, structured pattern — nothing added that wasn't truly there to draw on. Use this skill whenever writing, editing, or reviewing frontend UI code in a project that has the tlglobal design system installed (@tlglobal/ui, @tlglobal/icons, @tlglobal/tokens) — building components, forms, pages, styling, theming, or picking icons. Trigger this any time UI work is happening in such a project, even if the user doesn't name the packages explicitly. Governs how to look up real component/prop/icon/token names instead of guessing, and what to do when something needed isn't available yet."
---

# Weaving

Rule: never invent a component, prop, icon name, or token. Always confirm it exists by reading the relevant file below. If it doesn't exist, stop and ask (see "Missing component or variant").

## 0. Detect

Before UI work, check whether the project has these packages (`package.json` dependencies or presence under `node_modules/@tlglobal/`). If none are present, this skill doesn't apply — proceed normally, unless the user confirms this project is meant to use tlglobal, in which case follow "Setup" below first.

## Setup (packages not yet installed)

If the project is meant to use tlglobal but the packages aren't present yet, install all three together — they're meant to be used as a set, not individually:

```powershell
npm install @tlglobal/ui @tlglobal/icons @tlglobal/tokens
```

After install, confirm the generated docs this skill depends on actually landed in `node_modules`:

```powershell
dir node_modules\@tlglobal\ui\dist\AI_INVENTORY.md
dir node_modules\@tlglobal\ui\CONSUMING.md
```

If either file is missing, the installed `@tlglobal/ui` version doesn't ship the generated docs this skill reads — stop and tell the user, rather than falling back to guessing component names. Don't proceed past step 1 until `AI_INVENTORY.md` is confirmed present.

Once confirmed, read `CONSUMING.md` before writing any component — it covers Provider setup and `data-theme` wiring that has to be in place before anything from `@tlglobal/ui` will render correctly.

## 1. @tlglobal/ui (components, hooks)

| Purpose | Path | When to load |
|---|---|---|
| Component/hook inventory | `node_modules/@tlglobal/ui/dist/AI_INVENTORY.md` | Always, first — gives the full list of what exists |
| Full props/option values for a component | `node_modules/@tlglobal/ui/dist/AI_CONTEXT.md` | On demand, only for the specific component(s) you're about to use |
| Setup gotchas, `data-theme`, Provider requirements | `node_modules/@tlglobal/ui/CONSUMING.md` | Once per new project/session, before first render |
| Structured JSON (tooling/codemods) | `node_modules/@tlglobal/ui/dist/ui-manifest.json` | Only if writing a script/codemod against the library |
| Type definitions (source of truth) | `node_modules/@tlglobal/ui/dist/index.d.ts` | Only if the markdown docs don't answer a specific question |

Note the split: `CONSUMING.md` lives at the package root; the other three are generated into `dist/`.

## 2. @tlglobal/tokens (design tokens — no generated docs, read source CSS directly)

| Purpose | Path |
|---|---|
| Color tokens (light) | `node_modules/@tlglobal/tokens/src/css/colors.css` |
| Color tokens (dark overrides) | `node_modules/@tlglobal/tokens/src/css/colors-dark.css` |
| Typography tokens | `node_modules/@tlglobal/tokens/src/css/typography.css` |
| Spacing tokens | `node_modules/@tlglobal/tokens/src/css/spacing.css` |
| Radius tokens | `node_modules/@tlglobal/tokens/src/css/radius.css` |
| Shadow tokens | `node_modules/@tlglobal/tokens/src/css/shadow.css` |

Read from `src/`, not `dist/` — there's no build/generation step for tokens, so `src/` is the accurate copy.

## 3. @tlglobal/icons (curated Lucide re-export — no generated docs)

| Purpose | Path |
|---|---|
| Full list of exported icon names | `node_modules/@tlglobal/icons/src/index.js` |

The README hardcodes the icon list too, but `src/index.js` is the real source of truth if it ever drifts.

## Missing component or variant

If the UI you're asked to build needs a component, variant, or prop that isn't in `AI_INVENTORY.md` / `AI_CONTEXT.md`:

1. **Stop before building a workaround or custom substitute.**
2. Tell the user plainly what's missing and give two options:
   - Update `@tlglobal/ui` (it may exist in a newer version), or
   - Proceed now using only what's currently available.
3. **Do not proceed with a substitute** until the user explicitly says to proceed with what's present (e.g. "proceed with what's there" / "use what's available"). If they instead update the package, re-read `AI_INVENTORY.md` before continuing.

This applies to missing icons and tokens too — check `src/index.js` / the token CSS files first, and flag gaps the same way rather than fabricating a name or hardcoding a raw hex/px value.

## Building

1. Confirm theme/provider setup via `CONSUMING.md` (new project only).
2. Pick components from `AI_INVENTORY.md`; pull exact props from `AI_CONTEXT.md` for each one used.
3. Style with token CSS variables from `@tlglobal/tokens`, not hardcoded values.
4. Use icon names exactly as exported from `@tlglobal/icons/src/index.js`.
5. If anything needed is missing, follow "Missing component or variant" above before writing code.
