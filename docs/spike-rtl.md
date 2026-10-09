# RTL spike (throwaway, time-boxed)

Status: partial. Stopped when the 15-minute cap hit. This is a measurement, not a shippable RTL implementation.

## Verdict

**Hard** for full-app RTL.

## What landed

- `he` is in `LanguagesHelper::SUPPORTED_LOCALES`.
- `ApplicationHelper#rtl?` is true for `he`, `ar`, and `fa`.
- `<html lang dir>` in `app/views/layouts/shared/_htmldoc.html.erb` follows that helper.
- Directional icons get `rtl:-scale-x-100`.
- `format_money` wraps amounts in Unicode LRI/PDI isolates when `rtl?` is true.
- `Money::Formatting` has a first-pass `he` number format (`%u%n`, comma thousands). Stopped there. ILS already has `₪` in `config/currencies.yml`.
- `script/rtl_logical_classes` converts physical Tailwind classes and can census leftovers. It was not run against the three screens.

## Files touched

| Area | Files | Class conversions |
| --- | --- | --- |
| Dashboard / home | 0 views | 0 of 29 physical matches |
| Transactions list | 0 views | 0 of 27 physical matches |
| Budget | 0 views | 0 of 6 physical matches |
| Shared chrome (application layout + shared partials) | `_htmldoc.html.erb` only | 0 of 28 physical matches |
| Helpers / money / locale picker | `application_helper.rb`, `languages_helper.rb`, `lib/money/formatting.rb` | n/a |
| Lever | `script/rtl_logical_classes` | not applied |

Physical-class counts are from a Python census of `ml-`/`mr-`/`pl-`/`pr-`/`left-`/`right-`/`text-left`/`text-right`/`border-l`/`border-r`/`rounded-l`/`rounded-r`/`space-x-` in `app/views`, `app/components`, and `app/javascript`. Measured. Broader than the ADR's ~324 because this pattern also counts `left-`/`right-` utilities.

## Remaining physical-direction classes (app-wide)

**502** matches. Measured.

On the three spike screens plus shared chrome: **90** matches still in place.

## Unfinished

- No `*.he.yml` view strings. Screens would fall back to English under `he`.
- Logical Tailwind not applied on any of the three screens. The lever exists and was not executed.
- `Money#format` call sites (net-worth chart, budget donut interpolations) are not isolated. Only `format_money`.
- Charts (`time-series-chart`, donut, sankey): x-axis LTR behavior not observed. The app was not running.
- Date picker, combobox, and DS primitives still use physical classes.
- Sidebar CSS variables `--left-sidebar-width` / `--right-sidebar-width` and `left-1/2` centering need skip rules in the lever.
- Layouts other than `_htmldoc` (`print`, `auth`, `settings`) still hard-code or omit `dir`.
- Upstream tests were not run. Ruby was not on `PATH` in the wrap-up shell, so `bin/rails test` did not start.
- No live Hebrew/RTL screenshots of the running app. Postgres/Puma were not up, and the remaining window was too short to boot them.

## Blockers (inferred, not observed)

- Chart JS (time-series, donut, sankey) is the likely non-mechanical cost. ADR wants the x-axis to stay LTR.
- `disclosure_chevron` combines `chevron-right`, `rtl:-scale-x-100`, and `group-open:rotate-90`. Interaction is untested.
- Volume: 760 ERB templates, 151 English locale files. Incremental fallbacks make strings tractable. Classes need a CI guard after the lever runs or they return with upstream merges.

## Go / no-go vs ADR 0002

The ADR's 2-day spike was cut to 15 minutes, so the no-go triggers (one screen taking more than a day, chart library fork, LTR tests broken) were not exercised. Nothing observed contradicts a go on Strategy A, but the remaining 502 class hits plus charts still make full-app RTL a hard, multi-PR job rather than a 3-screen polish pass.
