# Workspace Project Map
<!-- Aggregates repo maps. Repo internals live in <repo>/PROJECT_MAP.md. -->
<!-- Read this before scanning the workspace. -->

Top-level map for the `iberdrola-happy-time-workspace` workspace.

## Map Contract

- Workspace `PROJECT_MAP.md` is an aggregator: repos, repo-map links, shared commands, active task pointers.
- Repo-local `<repo>/PROJECT_MAP.md` is the source of truth for repo internals: entrypoints, configs, Docker services, package scripts, key directories.
- Long architecture decisions live in `docs/product/architecture/`, not in generated map sections.
- Scripts may rewrite only generated sections; manual sections are protected.
- Map update scripts are project-specific:
  - workspace aggregator: `scripts/update-project-map.sh`
  - repo-local updater: `<repo>/scripts/update-project-map.sh`

## Workspace Layout

```text
dev/iberdrola-happy-time-workspace/
├── AGENTS.md, CLAUDE.md, PROJECT_MAP.md
├── docs/                     # product docs and workflows; docs/wip is gitignored
├── scripts/                  # lifecycle, gates, regression tests, skill linking
├── .agents/skills/           # shared, model-neutral workspace skills
├── .claude/skills/           # Claude links to shared skills
├── .claude/commands/         # Claude command adapters
├── _templates/landing/       # static landing template
├── _templates/sub-repo/      # template for new code repos
├── app/                      # reserved; not initialized
├── landing/PROJECT_MAP.md    # landing repo, has its own update-project-map.sh
└── nginx/                    # reserved; central edge lives in ../traefik-apps
```

## Adding A Code Repo

After `git init <repo-name>`, copy the template:

```bash
cp -r _templates/sub-repo/. <repo-name>/
```

Run `bash scripts/link-workspace-skills.sh <repo-name>` to expose shared skills inside the independent Git repo. Then replace the template project/repo placeholders, fill `CLAUDE.md`, `AGENTS.md`, `PROJECT_MAP.md`, and implement or stub `<repo>/scripts/update-project-map.sh`.

## Repos

| Repo | Map | Status |
|---|---|---|
| app | — | Not initialized |
| landing | `landing/PROJECT_MAP.md` | Astro static landing; independent Git repo |
| nginx | — | Not needed: central Traefik in `../traefik-apps` |

## landing

Repo: https://github.com/luckyfoxdesign/iberdrola-happy-time.
Read `landing/PROJECT_MAP.md` for internals and verification commands.
Prototype: `docs/backlog/todo/start.html`.
Production domain: `iberdrola.luckyfox.design` (published; see `docs/product/architecture/production-deployment.md`).

```bash
cd landing
docker compose --env-file /dev/null up --build astro-dev
bash scripts/verify.sh
```

Production integration: `docs/runbook-prod.md` and central edge repo
`../traefik-apps/PROJECT_MAP.md`. The workspace itself is not deployed.

## app / nginx

Not initialized. The landing is self-contained; TLS and routing are provided
by the existing central Traefik, so a separate nginx repo is unnecessary.

## Active Tasks

<!-- generated:start active-tasks -->
<!-- generated:end active-tasks -->

## Backlog

<!-- generated:start backlog -->
<!-- generated:end backlog -->
