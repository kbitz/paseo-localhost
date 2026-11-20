# Roadmap

The execution plan from private MVP-1 through public beta, one Track per PR
and implementation session. [SPEC.md](SPEC.md) owns scope and release policy;
[PROGRESS.md](PROGRESS.md) owns status and acceptance evidence. Deferred work
and frozen history live in the two satellites below. Line counts are estimates.

Standing constraints that can refuse a Track: tailnet only (never funnel);
installing/enabling is the opt-in, but no forward exists without a click,
panel action or agent tool call; no process command-line reads and no
`ps`/`pgrep`/`lsof`; no telemetry, hosted service or runtime downloads; no
personal identifiers in code, logs, docs or listing assets (placeholders and
sample data only); no commit, push, merge, publication or registry submission
without the maintainer's explicit approval at that step; the reference plugin
under mmm-factory is read-only evidence, never edited.

## In Progress

## Current Plan

### Group 1: Prove the click before building it

**This group delivers:** Observed answers on timeline transformation,
local-vs-remote detection, and HTTPS serve behavior, recorded as a design
note the rest of the plan builds on.

_Depends on: none_

#### Track 1A: Feasibility spike for click interception, locality and HTTPS serve
_3 tasks . ~300 lines . high risk_
**Outcome:** Maintainers know which click surfaces work before implementation commits to them.
**Supports:** public-release; SPEC O1.
**Done when:** `docs/designs/click-forward.md` records observed results for timeline transformation fidelity and a press model that never creates a forward on render, locality detection, HTTPS serve create/remove and Host forwarding, and Linux/macOS/Windows Tailscale CLI paths; any spec change those results force is proposed in the same PR.
_touches: docs/designs/click-forward.md (new), server/probes/ (new)_
- **Timeline probe** -- transform an assistant message containing a localhost URL; check Markdown fidelity, streaming and ensure-on-click handling. (M)
- **Locality probe** -- prove auto-detection on two real machines via the app local-host signal if available, otherwise loopback daemon `server_info` comparison. (M)
- **Serve probe** -- after HTTPS is enabled, verify create, Host header and single-port removal; note platform CLI differences. (S)

### Group 2: An empty plugin that loads in public form

**This group delivers:** An installable Paseo 0.16.1 plugin skeleton with
public identity, typed contracts, minimal settings and no undeclared runtime
dependencies.

_Depends on: Group 1_

#### Track 2A: Plugin scaffold, contracts and public identity
_3 tasks . ~500 lines . medium risk_
**Outcome:** A clean daemon can install and reload the plugin; it does nothing until a user action asks it to.
**Supports:** public-release; SPEC O1, O7.
**Done when:** The plugin installs and reloads cleanly, typecheck and client bundle-boundary tests pass, manifest name/description and translation-ready user text are in place, and runtime dependencies are none or fully locked.
_touches: paseo-plugin.json (new), package.json (new), index.server.ts (new), index.client.tsx (new), shared/forwards.ts (new), shared/settings.ts (new), tsconfig.json (new), tsconfig.client.json (new)_
_blocked-by: Track 1A_
- **Scaffold** -- create the plugin layout and public manifest identity in the `paseo plugin init` shape. (M)
- **Contracts** -- define forward RPCs, watch subscription and the minimal settings schema (agent tool, locality override). (M)
- **Boundary tests** -- pin the client allowlist, import boundaries and dependency policy. (S)

### Group 3: Quality gates and safe Tailscale execution

**This group delivers:** CI that checks public compatibility boundaries and a
runner that performs owned `tailscale serve` operations with readable errors.

_Depends on: Group 2_

#### Track 3A: Compatibility and CI matrix
_3 tasks . ~350 lines . medium risk_
**Outcome:** Every change is checked against the declared compatibility surface before it can ship.
**Supports:** public-release; SPEC O8.
**Done when:** CI runs typecheck, unit tests, bundle/package checks and the roadmap audit across applicable OSes, and a packed artifact installs into a clean test daemon profile.
_touches: .github/workflows/ci.yml (new), scripts/smoke-install.mjs (new), package.json_
_blocked-by: Track 2A_
- **CI** -- run static and unit gates on Linux, macOS and Windows where the code is OS-specific. (M)
- **Bundle checks** -- verify client externals, package `files`, manifest and no private identifiers. (S)
- **Install smoke** -- install the packed artifact into a clean daemon profile and require `running`. (M)

#### Track 3B: Tailscale runner and preflight
_2 tasks . ~350 lines . high risk_
**Outcome:** Supported daemon hosts can create and remove one owned serve per called port, or explain exactly what setup is missing.
**Supports:** public-release; SPEC O1, O4.
**Done when:** Fake-runner tests pin argv-only invocation, timeouts, redacted fixed-token logs, serve-status parsing and mappings for missing/logged-out Tailscale, operator, MagicDNS/HTTPS/Serve-disabled and collision errors; the runner never waits on the enablement prompt.
_touches: server/tailscale.ts (new), server/tailscale.test.ts (new)_
_blocked-by: Track 2A_
- **Runner** -- locate the `tailscale` binary; run serve/status with argv arrays, timeouts and redacted fixed-token logs. (M)
- **Preflight** -- map installation, login, operator, HTTPS/MagicDNS/Serve and collision failures to user actions. (M)

### Group 4: Forwards that always end

**This group delivers:** Lease bookkeeping that ends every owned forward on
a defined trigger and reconciles leftovers at startup.

_Depends on: Group 3_

#### Track 4A: Forward leases and reconciliation
_2 tasks . ~400 lines . high risk_
**Outcome:** One owned HTTPS forward per local port can be created on demand and is reliably removed.
**Supports:** public-release; SPEC O1, O3.
**Done when:** Tests pin lease expiry, backend-dead removal, workspace-archive removal, startup reconciliation and foreign-serve ownership safety against serve-status fixtures.
_touches: server/leases.ts (new), server/forward-service.ts (new), server/leases.test.ts (new)_
_blocked-by: Track 3B_
- **Leases** -- track owned forwards with 4-hour expiry, TCP liveness checks (3 misses, 15 s apart) and archive-hook teardown. (L)
- **Reconcile** -- on startup adopt live unexpired owned forwards and remove dead or expired ones. (M)

### Group 5: Every way to ask for a forward, on every daemon OS

**This group delivers:** The forwards panel, URL/locality primitives, setup
status and controls, the agent tool, and macOS/Windows daemon-host support —
all on the Group 4 lease service.

_Depends on: Group 4_

#### Track 5A: Forwards panel and badge
_2 tasks . ~450 lines . medium risk_
**Outcome:** Users can see, open, copy, extend and stop every live forward from Paseo.
**Supports:** public-release; SPEC O2, O3.
**Done when:** A workspace panel lists live forwards with origin and expiry in compact and wide layouts, actions are accessibly labelled, and the workspace row shows a count badge.
_touches: client/forwards-panel.tsx (new), client/forwards-model.ts (new), client/forwards-model.test.ts (new)_
_blocked-by: Track 4A_
- **Panel** -- list forwards with open, copy, extend and stop actions. (L)
- **Badge** -- show the live forward count on the workspace row. (S)

#### Track 5B: Localhost URL parsing and locality detection
_2 tasks . ~250 lines . medium risk_
**Outcome:** The client can extract localhost URLs and auto-detect whether this device is the session's daemon host.
**Supports:** public-release; SPEC O1, O2.
**Done when:** Tests cover loopback spellings, ports, paths and queries, and auto-detection distinguishes two real machines using the signal Track 1A proves.
_touches: shared/localhost-url.ts (new), shared/localhost-url.test.ts (new), client/locality.ts (new)_
_blocked-by: Track 4A_
- **URL parsing** -- extract localhost and loopback URLs without inventing or remapping ports. (S)
- **Locality** -- compare the selected host with this device's daemon identity; indeterminate means remote. (M)

#### Track 5C: Setup status and user controls
_2 tasks . ~400 lines . medium risk_
**Outcome:** A new user can see whether forwarding can work and change optional capabilities, with no configuration required to start.
**Supports:** public-release; SPEC O4, O7.
**Done when:** The settings/status surface reports Tailscale binary, login, operator, HTTPS and MagicDNS checks with fixes, states the tailnet-policy exposure plainly, and offers only an agent-tool toggle (default on) and locality override (default auto).
_touches: client/settings.tsx (new), client/settings-model.ts (new), client/settings-model.test.ts (new)_
_blocked-by: Track 4A_
- **Setup status** -- render server preflight results with actionable fixes and no diagnostic upload. (M)
- **Controls** -- persist the two optional controls through plugin settings; no port settings. (M)

#### Track 5D: Agent-requested forwards tool
_2 tasks . ~300 lines . medium risk_
**Outcome:** A workspace agent can immediately obtain a tailnet URL to share because installing the plugin is the user's opt-in.
**Supports:** public-release; SPEC O5.
**Done when:** The tool honors the disable setting, validates the named port, creates the same visible leased forward as a click, returns only URL/status, and marks origin as agent-requested.
_touches: server/agent-tool.ts (new), server/agent-tool.test.ts (new)_
_blocked-by: Track 4A_
- **Tool** -- define the tool and inject it at session open for workspace agents. (M)
- **Policy tests** -- pin immediate leased forwards, lease reuse and the disable setting. (S)

#### Track 5E: Windows daemon-host support
_3 tasks . ~350 lines . medium risk_
**Outcome:** The same click flow works when Windows runs the Paseo daemon.
**Supports:** public-release; SPEC O6.
**Done when:** Tests cover `tailscale.exe` discovery, SDK process execution, serve status/removal fixtures and Windows-specific preflight errors.
_touches: server/platform-windows.ts (new), server/platform-windows.test.ts (new)_
_blocked-by: Track 4A_
- **Discovery** -- find Tailscale in supported Windows install locations and PATH. (M)
- **Runner integration** -- execute through SDK process helpers with no shell string construction. (M)
- **Live check support** -- checklist for one real Windows daemon lifecycle. (S)

#### Track 5F: macOS daemon-host support
_3 tasks . ~300 lines . medium risk_
**Outcome:** The same click flow works when a Mac runs the Paseo daemon.
**Supports:** public-release; SPEC O6.
**Done when:** Tests cover macOS app and CLI variants and preflight errors, ready for real lifecycle evidence.
_touches: server/platform.ts (new), server/platform.test.ts (new)_
_blocked-by: Track 4A_
- **Discovery** -- find the Tailscale CLI in app-bundle and PATH locations. (M)
- **Preflight differences** -- map macOS variant and permission failures to user actions. (S)
- **Live check support** -- checklist for one real Mac daemon lifecycle. (S)

### Group 6: Links that take you there

**This group delivers:** Timeline localhost links whose embedded
destinations are rewritten for remote devices, with no extra link shown.

_Depends on: Group 5_

#### Track 6A: Timeline link transformer and renderer
_1 task . ~350 lines . high risk_
**Outcome:** A localhost link keeps its visible text but, from a remote device, takes the user to the dev server over the tailnet; no extra link appears.
**Supports:** public-release; SPEC O1, O2.
**Done when:** Supported timeline items preserve visible-text and Markdown fidelity, no forward is created by rendering alone, the click ensures the forward before opening, local devices are untouched, and accessibility/renderer tests pass.
_touches: client/link-card.tsx (new), client/link-card.test.ts (new)_
_blocked-by: Track 5B_
- **Transformer and renderer** -- rewrite embedded localhost destinations using the ensure-on-click press model Track 1A proves. (L)

### Group 7: Explain it and prove it is safe

**This group delivers:** Framework guidance, public documentation and a
reviewable security/privacy boundary.

_Depends on: Group 3, Group 5, Group 6_

#### Track 7A: Dev-server compatibility guidance and user docs
_3 tasks . ~500 lines . medium risk_
**Outcome:** When a dev server rejects the tailnet Host, the user gets the exact framework fix; public docs explain setup, exposure and troubleshooting.
**Supports:** public-release; SPEC O4, O9.
**Done when:** A post-forward Host probe classifies common framework rejections without logging bodies, the panel shows concise placeholder-safe fixes, and README/runbook/security docs cover setup, grants stance, uninstall cleanup and support diagnostics.
_touches: server/host-check.ts (new), server/host-check.test.ts (new), README.md (new), docs/runbook.md (new), docs/security.md (new)_
_blocked-by: Track 5A, Track 6A_
- **Host check** -- probe the local port with the tailnet Host header and classify responses with fixed tokens. (M)
- **Guidance data** -- return the diagnosis and framework fix the panel renders. (S)
- **Docs** -- write public setup, security model, runbook and support guidance. (M)

#### Track 7B: Security and privacy hardening review
_3 tasks . ~350 lines . high risk_
**Outcome:** Users and registry reviewers can inspect exactly what executes, what data is touched and why the supply chain is trustworthy.
**Supports:** public-release; SPEC O2, O9.
**Done when:** The threat model covers shared tailnets, untrusted timeline/agent/RPC input, forwarded identity headers, foreign serves and uninstall; tests fuzz paths to execution and pin no telemetry, downloads or unreviewed runtime dependencies.
_touches: docs/designs/security-model.md (new), server/security.test.ts (new), shared/localhost-url.security.test.ts (new)_
_blocked-by: Track 3A, Track 5C, Track 5D, Track 5E, Track 6A_
- **Threat model** -- document trust boundaries, install intent and tailnet-policy exposure. (M)
- **Input hardening** -- fixture/fuzz every external input that can reach process execution or URL opening. (M)
- **Supply-chain check** -- verify readable artifact, locked inputs and no runtime software installation. (S)

### Group 8: A registry-ready public artifact

**This group delivers:** The exact public package and listing a Paseo
registry reviewer can approve.

_Depends on: Group 7_

#### Track 8A: Public packaging and registry listing
_3 tasks . ~400 lines . medium risk_
**Outcome:** A stranger can evaluate the plugin from its listing and install the exact reviewed artifact cleanly.
**Supports:** public-release; SPEC O8, O10.
**Done when:** Overview, description, icon and sample-data screenshots are complete; license/files/lockfiles validate; package dry run passes; and the exact artifact installs `running` on a clean test daemon at minimum and current supported Paseo versions.
_touches: OVERVIEW.md (new), LICENSE (new), assets/icon.png (new), assets/screenshot.png (new), .github/ISSUE_TEMPLATE/ (new)_
_blocked-by: Track 7A, Track 7B_
- **Listing** -- write `OVERVIEW.md`, manifest metadata, icon and privacy-safe screenshots. (M)
- **Package readiness** -- verify license, source identity, `files`, locks and public repository hygiene. (M)
- **Submission packet** -- record categories, exact artifact/revision and clean-install evidence for maintainer-approved submission. (S)

### Group 9: Public beta in users' hands

**This group delivers:** The reviewed beta listed in the Paseo registry,
with support and rollback information published.

_Depends on: Group 8_

#### Track 9A: Public beta launch
_2 tasks . ~150 lines . high risk_
**Outcome:** Public Paseo users can install the reviewed beta and report problems through a supported channel.
**Supports:** public-release; SPEC O10.
**Done when:** With separate maintainer approvals for release and submission, the beta version is tagged and submitted, registry approval is recorded, the listed artifact installs cleanly, and beta scope/support/rollback notes are published.
_touches: CHANGELOG.md (new), docs/releases/public-beta.md (new)_
_blocked-by: Track 8A_
- **Release** -- prepare the beta version, release notes and exact registry submission. (M)
- **Launch verification** -- install the listed beta on clean profiles across the beta support matrix and publish support readiness. (S)

### Execution Map

A Group may launch when every Group in its ← set has landed, regardless
of document order; document order is priority, not gating.

Adjacency list (from the packer / GROUP_DEPS — not document order):
```
- Group 1 ← {}
- Group 2 ← {1}
- Group 3 ← {2}
- Group 4 ← {3}
- Group 5 ← {4}
- Group 6 ← {5}
- Group 7 ← {3, 5, 6}
- Group 8 ← {7}
- Group 9 ← {8}
```

Track detail per group:
```
Group 1: Prove the click before building it
  +-- Track 1A ........... ~M . 3 tasks

Group 2: An empty plugin that loads in public form
  +-- Track 2A ........... ~M . 3 tasks

Group 3: Quality gates and safe Tailscale execution
  +-- Track 3A ........... ~M . 3 tasks
  +-- Track 3B ........... ~M . 2 tasks

Group 4: Forwards that always end
  +-- Track 4A ........... ~L . 2 tasks

Group 5: Every way to ask for a forward, on every daemon OS
  +-- Track 5A ........... ~M . 2 tasks
  +-- Track 5B ........... ~S . 2 tasks
  +-- Track 5C ........... ~M . 2 tasks
  +-- Track 5D ........... ~S . 2 tasks
  +-- Track 5E ........... ~M . 3 tasks
  +-- Track 5F ........... ~S . 3 tasks

Group 6: Links that take you there
  +-- Track 6A ........... ~L . 1 task

Group 7: Explain it and prove it is safe
  +-- Track 7A ........... ~M . 3 tasks
  +-- Track 7B ........... ~M . 3 tasks

Group 8: A registry-ready public artifact
  +-- Track 8A ........... ~M . 3 tasks

Group 9: Public beta in users' hands
  +-- Track 9A ........... ~M . 2 tasks
```

**Total: 0 phases . 9 groups . 16 tracks remaining.**

---

## Future

Deferred: docs/roadmap-future.md (5 items)

## Shipped

History: docs/roadmap-shipped.md


