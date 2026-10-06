# Little Towns Labs fork of Observable Framework — working agreement

This is LTL's public fork of https://github.com/observablehq/framework (ISC license).
It is developed on powerbox2 and used to build static dashboards/reports that are
served from aegis-prod. Workspace rules in `/home/ubuntu/AGENTS.md` (aegis-prod)
still apply: no client data, no credentials in Git, Tailscale-only access. Explicit
current user instructions override this file.

## Remotes and branches

- `origin` = Little-Town-Labs/framework (our public fork). Push here only.
- `upstream` = observablehq/framework. Fetch only; its push URL is deliberately
  disabled. Never push, open PRs, comment or file issues upstream without the
  user's explicit approval for that specific action.
- `main` mirrors upstream exactly. Never commit to it. Update with
  `git fetch upstream && git merge --ff-only upstream/main`.
- `ltl/main` = our integration branch (this file, our fixes). Rebase on `main`
  when upstream moves. Consumers pin a tag or commit on it (`ltl-vX.Y.Z-N`).
- Work on short branches `fix/<topic>` or `feat/<topic>` off `main`, so each change
  stays small and could become an upstream PR later. Merge into `ltl/main`.

## Environment (powerbox2)

- Reach it with `tailscale ssh powerbox2@powerbox2-optiplex-7060`. Checkout:
  `~/src/framework`. Node 22, Yarn Classic 1.22 (`~/.local/bin`, already on PATH).
- Always set `OBSERVABLE_TELEMETRY_DISABLE=true`; the CLI sends telemetry otherwise.
- Install with `yarn install --frozen-lockfile`. Do not add or upgrade dependencies
  casually; each is supply-chain surface. Explain any lockfile change.
- Builds fetch npm imports from jsDelivr at build time, so they need outbound
  internet. Production never builds.

## Quality gates (run before any commit)

    yarn test:tsc && yarn test:lint && yarn test:prettier && yarn test:mocha:serial

Baseline on 2026-10-06 at upstream 85e843e6: all pass (840 mocha passing, 5 pending).
`yarn docs:build` currently fails because jsDelivr returns HTTP 500 for
`deck.gl/dist.min.js/+esm` at every version tried (9.0.38 to 9.4.0); that is
upstream/CDN, not a regression, and pinning an older deck.gl does not fix it. Add tests for any
behaviour change, and note any gate you could not run.

## Conventions

- Match surrounding code and the repo's terse lowercase commit subjects
  (e.g. `fix pages dist`). One logical change per commit.
- Never commit secrets, tokens, `.env` files, build output (`dist/`) or caches
  (`.observablehq/cache`). Check `git status` and `git diff` first.
- Test and example data must be synthetic. No real hostnames, metrics exports,
  customer data or internal URLs in fixtures, docs or issues.
- Keep GitHub Actions disabled on the fork unless reviewed: `publish.yml` publishes
  to npm on release and `deploy.yml` deploys on push and on a schedule.

## Deploying to aegis-prod

Build on powerbox2 only; publish the static `dist/` output. Serving is a small
static container bound to 127.0.0.1 behind a new Tailscale Serve port. Do not
install Node or the Framework toolchain on aegis-prod, do not touch other Serve
routes, and do not run `tailscale serve reset`. Anything on aegis-prod needs
explicit authorisation first.

## Contributing upstream

Upstream merges few outside PRs and has a long review backlog, so do not depend on
it. Before proposing anything upstream: search existing issues and PRs, keep the
change minimal with tests, follow Observable's code of conduct, and ask the user.
Local checks and fork pushes are fine without asking; upstream actions are not.

## Record keeping

Keep this file accurate when the workflow changes. Record decisions about using this
fork in `ltl-ops` (`adr.md`, `docs/monitoring.md`) as proposed until accepted, and
label drafts as uncommitted until they are versioned.
