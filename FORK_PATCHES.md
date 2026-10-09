# Fork patches

Unavoidable edits to upstream files, per ADR 0001 (overlay, not rewrite).
New files that live only in this fork are listed first, then each upstream
file we had to touch.

| File | Why | Upstreamable? |
| --- | --- | --- |
| `compose.yml` (new) | Instance compose that **builds this repo**. Upstream `compose.example.yml` pulls `ghcr.io/we-promise/sure` and cannot run our code. Left that file alone. | No (our image name, bind, secrets, brand env) |
| `bin/setup-instance` (new) | Writes a git-ignored `.env` with required secrets. Idempotent: never overwrites an existing `.env`. | Maybe, as a self-host helper |
| `NOTICE.md` (new) | Names Sure and Maybe Finance as origins. AGPL attribution. | No |
| `app/views/shared/_brand_mark.html.erb` (new) | Text mark that reads `product_name` from the existing brand layer. Replaces the Sure SVG wordmark in chrome. | No |
| `.gitignore` | Stop ignoring `compose.yml` so the shipped instance file stays tracked. | No |
| `.env.example` | Document required secrets with no values, set `PRODUCT_NAME=כסף`. | No |
| `README.md` | Add the Mac run/stop/upgrade/volumes section. | No |
| `app/views/layouts/auth.html.erb` | Sign-in chrome uses the text brand mark. | No |
| `app/views/layouts/application.html.erb` | Header chrome uses the text brand mark. | No |
| `app/views/layouts/shared/_footer.html.erb` | Keep the Maybe Finance attribution. Add an AGPL "Source code" link. | The link, maybe, with a configurable URL |
| `app/views/shared/_logo.html.erb` | Logo helper uses the text brand mark. | No |
| `app/views/onboardings/show.html.erb` | Onboarding header uses the text brand mark. | No |
| `app/views/onboardings/trial.html.erb` | Trial screen uses the text brand mark. | No |
| `config/locales/views/layout/en.yml` | `layouts.shared.footer.source_code` string. | Maybe |

Not edited, and must stay that way:

- `LICENSE` and copyright / license headers
- `compose.example.yml` (upstream image pull; not our instance file)
- `Dockerfile` (already runs `assets:precompile` and `bin/docker-entrypoint` runs `db:prepare` only for `./bin/rails server`)
- `config/initializers/brand.rb` (the one brand layer). Instance compose sets `PRODUCT_NAME`. The Ruby default stays `Sure` so the English test suite does not gain new failures.
- `desktop/` (R0 stretch, skipped)
- Backup compose service (follow-up)
