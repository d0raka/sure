# RTL spike (throwaway, time-boxed)

Status: conversion of the three screens landed as a **draft-only** sketch. Not a shippable RTL implementation. Do not merge.

## Verdict (observed)

**Medium** for the three spike screens under Strategy A (logical Tailwind + `dir=rtl` + locale). **Hard** for full-app RTL.

One-line: chrome and lists flip cheaply; leftover interpolated physical classes, English DS/chart copy, and mixed-script money are the remaining cost.

## Order of work (this pass)

1. Rails + Postgres + demo seed were already up from the boot pass.
2. **English baseline tests** (before convert): `210 runs, 770 assertions, 0 failures, 0 errors` in 14.9s on helpers/controllers for the three screens.
3. **English before screenshots** (fake demo data):
   - `/opt/cursor/artifacts/before-en-home.png` — **misnamed**: this is the English **transactions** list, not the dashboard. Same bytes as `before-en-transactions.png`.
   - `/opt/cursor/artifacts/before-en-transactions.png` — English transactions list.
   - `/opt/cursor/artifacts/before-en-budget.png` — English budget with category bars.
   - No English dashboard screenshot was captured (filename collision).
4. Convert logical classes + `he.yml` + `money_tag` (Tech Lead / Architect / Domain Expert fixes applied).
5. **English tests after convert**: same focused set, `210 runs, 770 assertions, 0 failures, 0 errors` in 12.1s. LTR still passes.
6. **RTL screenshots** (demo user locale `he`, fake data):
   - `/opt/cursor/artifacts/rtl-home.png` — dashboard, `dir=rtl`, Hebrew chrome.
   - `/opt/cursor/artifacts/rtl-transactions.png` — transactions list.
   - `/opt/cursor/artifacts/rtl-budget.png` — budget + category bars + donut.
   - `/opt/cursor/artifacts/rtl-chart.png` — sankey on the dashboard (copy of rtl-home).
   - `/opt/cursor/artifacts/rtl-date-picker.png` — budget month picker overlay.

Stopped after those screenshots, the focused English re-run, and the draft PR update. Full-app suite was **not** re-run after convert (a pre-convert full run finished `10886 runs, 3 failures, 3 errors, 34 skips`; one sampled failure was `Eval::Langfuse`, unrelated). No merge, no deploy, no spend.

## Minutes per screen (observe, not polish)

| Screen | Minutes | Notes |
| --- | --- | --- |
| Dashboard / home | ~8 | Convert + Hebrew strings + sankey/donut observe. Mechanical class swap; chart labels still English. |
| Transactions list | ~5 | Convert + list/header strings. Amounts isolate in `<bdi>`; USD still reads mixed. |
| Budget | ~6 | Convert + category bars + donut + month picker. Category names/status still English. |

Boot (Ruby PATH, Postgres, bundle, `db:prepare`, `demo_data:default`, Puma): ~2–8 minutes earlier in the window. Wrap target 18:47 IDT.

## Tech Lead / Architect / Domain Expert fixes

1. `script/rtl_logical_classes` only rewrites quoted `class` / `class:` / `class_names` strings in **`.html.erb`**. No `.rb` or `.js`. **No `space-x-` → `gap-`** (Tailwind v4 `space-x` already uses logical margins). Skips `--left-` / `--right-` CSS vars and `left-1/2` (etc.) centering.
2. `format_money` is plain `Money#format` in every locale (no LRI/PDI/RLM). Isolation is view-only `money_tag` → `<bdi>…</bdi>`, used on the converted screens. Interpolation/`t(..., amount: format_money(...))` stays plain text.
3. Hebrew money format is `"%n\u00A0%u"` (number, NBSP, symbol). Example: `1,234.50 ₪` for ILS. Demo family is USD, so live UI shows `15,866.14 $` / `9,384.42 $`.

## What flipped

- `<html lang="he" dir="rtl">` via `rtl?`.
- Nav: בית / תנועות / דוחות / תקציב. Dashboard greeting שלום Jack. Budget month אוקטובר 2026. Transaction dates like `5 בנובמבר, 2026`.
- Logical utilities (`ms`/`me`/`ps`/`pe`/`text-start`/`text-end`/`border-s`/`border-e`) on dashboard, transactions, budgets, and application chrome.
- Directional icons get `rtl:-scale-x-100`.
- Amounts in converted views wrap `<bdi>`.

## Visual RTL breakage (observed)

- Sankey node labels stay English; flow is left-to-right-ish inside an RTL page (see `rtl-chart.png` / `rtl-home.png`). **No chart-library fork.**
- Budget donut works in RTL; category **names and "Over Budget"/"On Track"** are still English. Bars themselves LTR-fill from the inline start of the row (visually right in RTL) — acceptable first cut.
- Month picker is an in-app popover (not a JS date-library fork). Trigger is Hebrew (`אוקטובר 2026`); grid labels are English `Jan`…`Dec` and sit RTL (Jan on the right). See `rtl-date-picker.png`.
- Mixed-script money: Hebrew page + USD `$` inside `<bdi>` still looks odd (`$` / minus placement). ILS with `%n NBSP %u` is untested in the live demo because the seeded family is USD.
- Breadcrumb `Home > Transactions` and several DS/search placeholders stay English.
- Sidebar account names stay English (not in spike string set).
- `ml-21 mr-4` on `render "shared/ruler", classes: "..."` was **not** converted — the lever only touches quoted class attributes, not `classes:` kwargs.

## Remaining physical-direction classes

Census after convert (`.html.erb` only, three screens + layouts): **23** leftover matches. Several are `rounded-t-` / `rounded-b-` (not directional). Real leftovers:

- Interpolated `class_names` / `classes:` kwargs (`ml-21 mr-4`, some `pl-`/`pr-`).
- `left-1/2` centering and `--left-sidebar-width` (intentionally skipped).
- Layouts outside the spike (`settings`, `mailer`, `doorkeeper`).

App-wide class volume is still a multi-PR job. The lever is safe to re-run on more `.html.erb` trees.

## Tests

| When | Command | Result |
| --- | --- | --- |
| Baseline (English, pre-convert) | helpers + pages/transactions/budgets controllers | 210 / 770 / 0 / 0 |
| After convert (English locale) | same | 210 / 770 / 0 / 0 |
| Full suite | started pre-convert | 10886 runs, 3 failures, 3 errors, 34 skips (unrelated sample: Langfuse). Not re-run after convert. |

New coverage: `format_money` has no bidi marks in `en` or `he`; `money_tag` wraps `<bdi>`; Hebrew ILS formats as `1,234.50\u00A0₪`.

## ADR 0002 no-go triggers

| Trigger | Observed |
| --- | --- |
| One screen takes more than a day | **No.** Minutes per screen. |
| Chart library needs a fork | **No fork.** Sankey/donut/time-series render; x-axis / node labels stay LTR/English. |
| Date picker needs a fork | **No fork.** Month-grid popover; English abbreviations, RTL layout. |
| LTR tests break | **No** on the focused English suite (210/210 before and after). |

Go on Strategy A for these three screens. Full-app RTL stays hard (volume, charts copy, interpolated classes, DS primitives).

## Files

| Area | Change |
| --- | --- |
| Dashboard / home | logical classes + `money_tag` + `config/locales/views/pages/he.yml` |
| Transactions list | logical classes + `money_tag` + `config/locales/views/transactions/he.yml` |
| Budget | logical classes + `money_tag` + `config/locales/views/budgets/he.yml` |
| Chrome | application layout, `_htmldoc`, nav, insights badge; `config/locales/views/layout/he.yml` |
| Helpers | `money_tag`; `format_money` plain; `he` format `%n\u00A0%u` |
| Lever | `script/rtl_logical_classes` (erb class attrs only, no space-x swap) |
