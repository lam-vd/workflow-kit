# Stage 8 — Linter / formatter reference

Run on **staged files only** unless project profile specifies full scan (e.g. ai-housemaker Brakeman).

| Language / Framework | Linter | Formatter | Staged command |
|---|---|---|---|
| Ruby / Rails | `rubocop` | `rubocop -a` | `rubocop $(git diff --cached --name-only '*.rb')` |
| ERB (Rails) | `erb_lint` | — | `erb_lint $(git diff --cached --name-only '*.html.erb')` |
| JavaScript / TypeScript | `eslint` | `prettier` | `eslint $(git diff --cached --name-only '*.{js,ts,jsx,tsx}')` |
| Python | `ruff` / `flake8` | `black` / `ruff format` | `ruff check $(git diff --cached --name-only '*.py')` |
| Go | `golangci-lint` | `gofmt` | `golangci-lint run --new-from-rev=HEAD~1` |
| Rust | `clippy` | `rustfmt` | `cargo clippy` |
| Java / Kotlin | `checkstyle` / `ktlint` | — | `ktlint $(git diff --cached --name-only '*.kt')` |
| PHP | `phpstan` / `phpcs` | `php-cs-fixer` | `phpstan analyse $(git diff --cached --name-only '*.php')` |
| C# | `dotnet format` | `dotnet format` | `dotnet format --include $(git diff --cached --name-only '*.cs')` |
| CSS / SCSS | `stylelint` | `prettier` | `stylelint $(git diff --cached --name-only '*.{css,scss}')` |
| Ruby (security) | `brakeman` | — | `bundle exec brakeman -q --no-pager` (full scan; exit **3** = warnings) |

## Severity mapping

| Result | Severity |
|--------|----------|
| Linter **error** | 🟠 High |
| Linter **warning** | 🟡 Medium |
| Formatter-only | 🟢 Low |
| Brakeman warnings (exit 3) | 🟠 High — fix before push/merge (Rails) |
| No linter in project | 🟡 "Missing linter setup" in report |

## ai-housemaker (Docker)

```bash
docker compose exec -T app bundle exec rubocop --force-exclusion $(git diff --cached --name-only -- '*.rb')
git diff --cached --name-only -- '*.html.erb' | xargs -r docker compose exec -T app bundle exec erb_lint
docker compose exec -T app bundle exec brakeman -q --no-pager
```

Brakeman `VerbConfusion`: use `request.get? || request.head?`, not `request.get?` alone on GET+POST actions. Rule: `security/brakeman.mdc`. Pre-push: `bin/pre-push-check`.
