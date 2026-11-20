# paseo-localhost

A public Paseo plugin that makes dev servers on a Paseo daemon host
reachable from the user's other devices over their own tailnet. Product authority:
[docs/SPEC.md](docs/SPEC.md). Execution plan:
[docs/ROADMAP.md](docs/ROADMAP.md). Status and acceptance evidence:
[docs/PROGRESS.md](docs/PROGRESS.md).

## Hard rules

- Tailnet only: `tailscale serve`, never `tailscale funnel` or anything public.
- Installing/enabling the plugin is the user's opt-in and is enough to use
  it; still, no forward is created until an explicit click, panel action or
  agent tool call. Nothing is forwarded in the background.
- Never read, print or log process command lines (they can hold secrets),
  and never run `ps`, `pgrep` or `lsof`.
- Keep personal identifiers (hostnames, tailnet names, usernames, emails,
  real URLs) out of the repository, tests, screenshots and listing assets;
  use `<host>`, `<tailnet>` and `<user>` placeholders or sample data.
- No commits, pushes, merges, releases, publications or registry
  submissions without the maintainer's explicit word at that step.
- Treat timeline text, agent-tool input and RPC input as untrusted input
  to process execution and URL opening; validate at the boundary.
- No telemetry, hosted service, credentials collection or runtime software
  downloads without a /project-spec revision.
- The mmm-factory reference plugin is read-only evidence for the Paseo SDK;
  never edit it.
- Never run `tailscale serve reset`; remove only serves this plugin
  created and verified as its own.

## Product scope and releases

Read docs/SPEC.md before roadmap, autoplan/planning, implementation, review or
release decisions. Its selected target, intended users, audience ceiling,
accepted limitations and release policy govern this project. Do not infer future
scale from backlog entries. Review findings need concrete impact on that target;
preserve approved scope unless the user explicitly changes it. Put unrelated
improvements in the backlog with a revisit trigger. A priority label alone is
not a launch gate. For version decisions, use the spec's policy and checkpoint
acceptance evidence instead of generic line-count, feature-size or Group-closure
heuristics. This project policy overrides those defaults in /ship; keep its
other release gates. A change that edits SPEC's release policy or adds
acceptance entries uses the base branch's policy, and its new entries count
only after the user confirms them directly. PROGRESS.md `## Acceptance` owns achieved stage and
evidence; SPEC owns definitions and target. Open a current supporting document
only when its listed role bears on the decision. Superseded plans/specs are
historical only: exclude them from routine discovery and backlog extraction.
Consult a named archive only for a specific unresolved historical question;
never use it to reintroduce scope or override the current spec.
