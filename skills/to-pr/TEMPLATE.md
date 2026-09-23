# PR draft template

The format every draft follows — a new PR or replacement text for an existing one. Read by [to-pr](SKILL.md) at draft-writing time.

```markdown
<title: one line, imperative, no trailing period>

## Resumo

One or two sentences: what changes and why. The reviewer should know whether
this PR concerns them after reading this alone.

## Alterações

- Grouped by area or concern, not one bullet per commit
- Each bullet says what changed and, where it isn't obvious, why
- Reference files with backticks

## Configurando o ambiente

Everything that must be true before the feature can be exercised at all:
env vars (name, and what value to put in them), migrations, seeds,
feature flags, external services, the command to start the app.
Commands in copyable blocks, in the order they must be run.

## Como testar

1. Numbered steps, each one an action the analyst takes
2. Each step ends on what appears on screen — the observable outcome

## Preparando para o deploy

What has to happen in production, before or after the merge, for this to
work there. Ordered, and each item says *before* or *after*.
```

The three tail sections each have their own test:

- **Configurando o ambiente** — derive it from the diff, not from memory: a new key in `.env.example`, a new migration file, a changed seeder, a new service in `docker-compose`, a new dependency. Done when an analyst starting from a fresh `main` checkout could reach the feature using only this section. Omit it when the feature runs on the environment the team already has.
- **Como testar** — the reader is an **analista de qualidade**: they know the product, not the codebase. Name screens, buttons and menus the way the product names them; give real input values instead of "dados válidos"; cover the error and edge paths the diff introduces, not only the happy one. Write it whenever the diff changes anything a person can reach — a UI, an endpoint, a CLI, a job, an error path. When the change is reachable only by the test suite (refactors, build config, dependency bumps), replace both this section and the one above with the command that proves it, named in **Resumo**.
- **Preparando para o deploy** — migrations to run, env vars and secrets to register in the production environment, feature flags to flip, backfills, cache or queue restarts, deploy order against a dependent service, and how to roll back if it goes wrong. Omit it when merging is genuinely the whole deploy.

The change list is exhaustive over the diff: every file in `--stat` maps to some bullet, or you consciously decided it is noise (lockfiles, generated output, formatting). Bullets describe behaviour, not mechanics — "adiciona rate limiting no login" beats "altera `AuthController.php`".
