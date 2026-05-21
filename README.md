# @datagym/design

Single source of truth for **colours, fonts, and the per-lab accent contract**
across the DataGym.io ecosystem - the `datagym` content hub and the separate
quest repos (`dbt-quest`, and future `sql-quest` / `data-modeling-quest`).

It does **not** own numeric scales (spacing, radii, shadows, layout, motion) -
those are layout-coupled and stay consumer-local.

## Why this exists

`datagym` consumes tokens as plain `:root` CSS custom properties. The quests use
Tailwind v4 `@theme {}`. Same values, two dialects. This package authors the
values **once** and emits both.

## What it emits (`dist/`)

| File | Form | Consumed by |
|------|------|-------------|
| `tokens.css` | plain `:root` + `[data-theme='dark']` | datagym (Astro, plain CSS) |
| `theme.css` | Tailwind v4 `@theme {}` + `[data-theme='light']` | the quests (Vite + Tailwind) |
| `primitives.css` | raw brand scales as `:root` vars | imported by `tokens.css` |
| `fonts.css` | self-hosted `@font-face` (woff2 in `dist/fonts/`) | both |
| `tokens.js` / `.d.ts` | resolved hex literals (`dag`) | ReactFlow/DagPanel - CSS vars can't reach its style props |

## Source of truth (`tokens/`)

- `primitives.json` - raw, theme-independent values (brand scales + the navy
  surface/ink palette shared by datagym's dark theme and the quests).
- `semantic.datagym.json` - datagym's named tokens, `light` + `dark`.
- `semantic.quest.json` - the quests' named tokens, `light` + `dark`.
- `dag.json` - quest DAG layer/canvas colours.

A value of the form `$group.key` references a primitive. Edit a primitive once
(e.g. the navy) and it propagates to every consumer and both dialects.

Run `npm run build` after any change to `tokens/`. `dist/` is committed.

## Consuming it

**datagym** (`src/styles/tokens.css`):

```css
@import '@datagym/design/tokens.css';
/* local non-colour scales (spacing, radii, ...) stay below */
```

**A quest** (`src/index.css`):

```css
@import "tailwindcss";
@import "@datagym/design/theme.css";
/* app utilities, keyframes, etc. stay below */
```

**DagPanel / JS that needs literal hex:**

```ts
import { dag } from '@datagym/design/tokens';
```

Install (local dev): `npm i file:../datagym-design`.
Install (pinned): `npm i github:bruno-szdl/datagym-design#v1.0.0` once pushed.

## Per-lab accent contract

The quest theme routes its accent through two custom properties:

- `--lab-accent` - the bright accent (dark mode + every fill/edge/focus ring).
- `--lab-accent-strong` - a darker accent for accent-coloured **text on light
  surfaces** (the bright accent fails WCAG AA there).

Both default to dbt's coral (`#ff694a` / `#e05a3a`), so `dbt-quest` needs no
override. A new quest re-flavours every accent surface by setting these once:

```css
:root { --lab-accent: #5b8def; --lab-accent-strong: #2f5fc0; }
```

datagym sets `--lab-accent` scoped to a lab's card/page (value from the lab's
`accent` field) so a lab's signature colour tints its own surfaces without
touching datagym's global brand accent.
