# RTL spike (throwaway, time-boxed)

Status: conversion of the three screens landed as a **draft-only** sketch. Not a shippable RTL implementation. Do not merge.

## Verdict (observed)

**Medium** for the three spike screens under Strategy A (logical Tailwind + `dir=rtl` + locale). **Hard** for full-app RTL.

One-line: chrome and lists flip cheaply; leftover interpolated physical classes, English DS/chart copy, and mixed-script money are the remaining cost.

## Order of work (this pass)

1. Rails + Postgres + demo seed were already up from the boot pass.
2. **English baseline tests** (before convert): `210 runs, 770 assertions, 0 failures, 0 errors` in 14.9s on helpers/controllers for the three screens.
3. **English before screenshots** (fake demo data). `before-en-home.png` was recaptured from checkout `b94c64a13` with the demo user locale `en`:
   - `/opt/cursor/artifacts/before-en-home.png` — English LTR **dashboard** (`Welcome back, Jack`). md5 `35867f4244d352ca2d9465e8c0fd578a`. Distinct from transactions (`d9200a640244d991783f6fbaf86d9cdd`).
   - `/opt/cursor/artifacts/before-en-transactions.png` — English transactions list.
   - `/opt/cursor/artifacts/before-en-budget.png` — English budget with category bars.
4. Convert logical classes + `he.yml` + `money_tag` (Tech Lead / Architect / Domain Expert fixes applied). Minus on earmarked cash moved inside `money_tag`; Hebrew negatives prefix LRM inside `<bdi>`.
5. **English tests after convert**: same focused set, `210 runs, 770 assertions, 0 failures, 0 errors` in 12.1s. LTR still passes. Helper tests including the new `money_tag` cases: `53 runs, 159 assertions, 0 failures`.
6. **RTL screenshots** (demo user locale `he`, fake data):
   - `/opt/cursor/artifacts/rtl-home.png` — dashboard, `dir=rtl`, Hebrew chrome. md5 `89db0f0c205fdaefd339aa976904d521`.
   - `/opt/cursor/artifacts/rtl-transactions.png` — transactions list.
   - `/opt/cursor/artifacts/rtl-budget.png` — budget + category bars + donut.
   - `/opt/cursor/artifacts/rtl-chart.png` — Sankey **close-up** (535×449, not a copy of rtl-home). md5 `18fcb3bf051448a70fe4bc3dd232bbce`.
   - `/opt/cursor/artifacts/rtl-date-picker.png` — budget month picker overlay.

Stopped after the full-suite before/after reruns, the `money_tag` minus/LRM fix, recaptured screenshots, and the draft PR update. No merge, no deploy, no spend.

## Minutes per screen (observe, not polish) including known-gap estimates

| Screen | Observed minutes | Known remaining gap (estimate) |
| --- | --- | --- |
| Dashboard / home | ~8 convert + observe | Sankey labels clipped at the chart edges: **2–4 hours** (JS layout / padding, not a library fork). Untranslated English sentences (period on the wrong side in RTL): **2–4 hours** app-wide, not this screen alone. |
| Transactions list | ~5 | Category badges clipped at the start of the word: **30–60 min** (overflow + logical padding). |
| Budget | ~6 including month picker | Month picker names still English `Jan`…`Dec`: **15–30 min** (`I18n.l` / locale data). |
| Shared chrome | included in the three screens | Right sidebar account groups not converted; text runs into amounts: **30–45 min** (sidebar partials + logical classes). |

Boot (Ruby PATH, Postgres, bundle, `db:prepare`, `demo_data:default`, Puma): ~2–8 minutes. This pass wrap 19:05 IDT.

## Tech Lead / Architect / Domain Expert fixes

1. `script/rtl_logical_classes` only rewrites quoted `class` / `class:` / `class_names` strings in **`.html.erb`**. No `.rb` or `.js`. **No `space-x-` → `gap-`** (Tailwind v4 `space-x` already uses logical margins). Skips `--left-` / `--right-` CSS vars and `left-1/2` (etc.) centering.
2. `format_money` is plain `Money#format` in every locale (no LRI/PDI/RLM/LRM). Isolation is view-only `money_tag` → `<bdi>…</bdi>`, used on the converted screens. Interpolation/`t(..., amount: format_money(...))` stays plain text.
3. Hebrew money format is `"%n\u00A0%u"` (number, NBSP, symbol). Example: `1,234.50 ₪` for ILS. Demo family is USD, so live UI shows `15,866.14 $` / `9,384.42 $`.
4. Negative amounts: the literal `−` outside `money_tag` in `budgets/_available_cash` is gone. The helper receives `-budget.earmarked_for_goals_money`. For `he`, `money_tag` prefixes U+200E (LRM) inside `<bdi>` so a negative ILS amount is LRM + `-1,234.50` + NBSP + `₪` on screen (CLDR 46 / `Intl.NumberFormat('he-IL')`). English negatives stay `-$1,234.50` with no bidi mark.

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
| Full suite **before** convert | `bin/rails test` on `b94c64a13` | 10883 runs, 46398 assertions, **3 failures, 3 errors**, 34 skips |
| Full suite **after** convert | `bin/rails test` on `cba9d9a49` | 10888 runs, 46423 assertions, **3 failures, 3 errors**, 34 skips |

After has 5 extra runs (new `money_tag` / `format_money` assertions). The failing/erroring **names match**.

### Failing / erroring names (before = after)

1. `Eval::Langfuse::ExperimentRunnerTest#test_still_reads_provider_config_written_with_symbol_keys` (`test/models/eval/langfuse/experiment_runner_test.rb:40`)
2. `Settings::HostingsControllerTest#test_can_clear_openai_access_token_by_submitting_a_blank_value` (`test/controllers/settings/hostings_controller_test.rb:307`)
3. `Settings::HostingsControllerTest#test_can_clear_an_encrypted_api_key_by_submitting_a_blank_value` (`test/controllers/settings/hostings_controller_test.rb:261`)
4. `SimplefinEntry::ProcessorTest#test_flags_pending_when_provider_pending_flag_is_true_(even_if_posted_provided)` (`test/models/simplefin_entry/processor_test.rb:101`)
5. `SimplefinEntry::ProcessorTest#test_posted==0_treated_as_missing,_entry_uses_transacted_at_date_and_flags_pending` (`test/models/simplefin_entry/processor_test.rb:121`)
6. `SimplefinEntry::ProcessorTest#test_infers_pending_when_posted_is_explicitly_0_and_transacted_at_present_(no_explicit_pending_flag)` (`test/models/simplefin_entry/processor_test.rb:238`)

### Evidence (same message before and after)

Langfuse (this **is** the Langfuse experiment runner). Mocha expected `uri_base: nil`; `Eval::ProviderFactory` passed `uri_base: ""` because `ENV["OPENAI_URI_BASE"].presence` is the empty string in this environment:

```
Failure:
Eval::Langfuse::ExperimentRunnerTest#test_still_reads_provider_config_written_with_symbol_keys [app/models/eval/provider_factory.rb:49]:
unexpected invocation: Provider::Openai.new("symbol-key-token", uri_base: "", model: "gpt-4.1")
unsatisfied expectations:
- expected exactly once, invoked never: Provider::Openai.new("symbol-key-token", uri_base: nil, model: "gpt-4.1")
```

The two Hostings failures are the same empty-string env: `Expected "" to be nil` when clearing `openai_access_token` / an encrypted API key. They are LLM-hosting settings tests, not RTL.

The three Simplefin errors are `ActiveRecord::RecordNotFound` looking up `entries` by `account_id` / `external_id` / `source`. They are **not** Langfuse. They appear on both `b94c64a13` and `cba9d9a49`, so they are not a conversion regression.

New helper coverage: `format_money` has no bidi marks (including LRM) in `en` or `he`, positives and negatives; `#money_tag wraps a Hebrew amount in bdi without bidi marks on positives`; `#money_tag keeps the minus inside the isolate in English and Hebrew` (`<bdi>\u200E-1,234.50\u00A0₪</bdi>`).

## Known gaps not fixed this pass

| Gap | Estimate |
| --- | --- |
| (1) Right sidebar account groups aren't converted; text runs into amounts | 30–45 min |
| (2) Untranslated English sentences show the period on the wrong side in RTL | 2–4 hours (remaining `en.yml`) |
| (3) Sankey labels are cut off at the chart edges (visible in `rtl-chart.png`) | 2–4 hours (chart JS padding) |
| (4) Category badges are cut off at the start of the word | 30–60 min |
| (5) Month picker names are in English (`Jan`…`Dec`) | 15–30 min |

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
| Helpers | `money_tag` (`<bdi>` + LRM on `he` negatives); `format_money` plain; `he` format `%n\u00A0%u` |
| Lever | `script/rtl_logical_classes` (erb class attrs only, no space-x swap) |
