# thrillmade

Tooling for AI-assisted software development. Workflows, conventions, and
small packages that make working *with* coding agents (not just talking to
them) feel coherent across a project.

## The stack

| Repo | What it does |
|------|------|
| [logmind](https://github.com/thrillmade/logmind) | Decision logging for AI-assisted development. CLI + GitHub Actions that capture *why* a change was made, branch-aware, with derived-doc merge drivers so parallel agent work doesn't conflict. Distributed via PyPI + Homebrew. → [logmind.dev](https://logmind.dev) |
| [clud-bug](https://github.com/thrillmade/clud-bug) | Claude-powered PR reviewer. One-line install (`clud-bug add`) gives any repo skill-aware, evidence-anchored code reviews with project-specific rules. → [cludbug.dev](https://cludbug.dev) |
| [agent-skills](https://github.com/thrillmade/agent-skills) | The shared skill catalog. Skills are markdown files that teach AI agents project conventions — installable via `skills.sh`, version-pinned per project. The library the other tools draw from. |
| [reporulez](https://github.com/thrillmade/reporulez) | GitHub repo defaults: branch rulesets, CODEOWNERS, PR templates, and per-ecosystem Dependabot configs. One command to bring a new repo up to the org's baseline. |
| [homebrew-logmind](https://github.com/thrillmade/homebrew-logmind) | Homebrew tap for `logmind`. Auto-bumped on every PyPI release. |
| [tokenomics](https://github.com/thrillmade/tokenomics) | Decisions + skills repo for the tokenomics product line. |
| [rezgen](https://github.com/thrillmade/rezgen) | Application generator / starter scaffolding. |

## How the pieces fit together

A new project starts by running **`logmind init`** — that wires the decision
log, GitHub Actions, and AGENTS.md guidance into the repo. Then
**`clud-bug add`** adds the PR reviewer; on every PR it reads the project's
installed **skills** (`thrillmade/agent-skills`) to ground its reviews in
the conventions the team has already declared. **`reporulez apply`**
brings the repo up to org baseline (ruleset, CODEOWNERS, Dependabot).

The four together let an AI agent show up in a thrillmade repo and
immediately know how to write decisions, how reviews work, what
conventions to follow, and what gates exist before merge — without any
project-specific setup beyond the four `init`/`add`/`apply` calls above.

## Conventions

- Branch protection: enforced via an **org-level ruleset** named
  `org-default-protection` (not via per-repo branch-protection rules
  — that API is legacy and will return 404 for our repos). See
  `gh api orgs/thrillmade/rulesets`. Active settings:
  - 4 required status checks: `clud-bug-review`, `check-derived-docs`, `check-decisions`, `check-links`
  - `required_review_thread_resolution: true` — unresolved inline review threads block merge
  - Squash-only + linear-history; non-fast-forward push blocked
  - Bypass actor: Repository admin (always) — for self-mod ceremony PRs that legitimately can't satisfy `clud-bug-review`
- Decision log in `docs/decisions.md` (and `docs/decisions-branches/<branch>.md` while a feature branch is live).
- AI-agent guidance in `AGENTS.md` at the repo root. `logmind init` refreshes the marker-bracketed block on every install.
- Cross-repo notifications: logmind tag pushes notify agent-skills (PR-shape), agent-skills baseline-skill changes notify clud-bug (PR-shape) — both v0.4.0+.

## Vercel projects

Hobby tier today. Marketing sites: `logmind.dev` (deployed from `logmind/site/`), `cludbug.dev` (deployed from `clud-bug/site/`). Upgrade trigger: any single repo hitting >20 deploys/day for 3 consecutive days, or onset of the App stream which will likely spike preview deploys. The root `vercel.json` in both projects uses `VERCEL_GIT_PREVIOUS_SHA`-aware `ignoreCommand` to skip deploys when no `site/` files changed.
