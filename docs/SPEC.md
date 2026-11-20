# paseo-localhost — Project spec

## Authority
This is the current authority for product scope, target and release policy.
Supporting documents own only the roles listed below. Superseded documents are
historical rationale; do not load them for routine planning or recover TODOs
from them. Consult a named historical source only to answer a specific question.

## Purpose
A Paseo user runs agents on a daemon host and often works from another
device on the same tailnet. When an agent's dev server prints
`http://localhost:<port>`, that link is dead from anywhere else. This Paseo
plugin makes the click work: from a remote device, the localhost link keeps
its visible text but its destination is rewritten — clicking it runs
`tailscale serve` on the daemon host and opens
`https://<host>.<tailnet>.ts.net:<port>` on the device in hand. No new link
appears anywhere. On the
daemon host itself, links behave exactly as they do today. Forwards are
leases the user can see and stop; nothing is exposed without a click, a
panel action, or an agent tool call. Installing and enabling the plugin is
the user's opt-in to these capabilities: it works without a separate
master switch or per-project enrollment.

## Audience and target
- Current users: the maintainer only (2026-11-20); no plugin is published yet.
- Audience ceiling: potentially-public.
- Selected target: public-release, reached through private MVP-1 use and a
  public beta; do not expand the audience until each stage's evidence exists.
- Supported conditions at public release: Paseo versions in the declared
  `requirements.paseo` range (initially `>=0.16.1`, tested at its minimum and
  current versions); daemon on Linux, macOS or Windows; Tailscale installed
  and logged in on the daemon host and viewing device, MagicDNS and HTTPS
  certificates enabled, Serve enabled, and the daemon user able to run
  `tailscale serve` without sudo; Paseo client on desktop, web, iOS or
  Android. A user may have several devices and daemon hosts, and a tailnet
  may have other users and devices. The user performs one-time Tailscale
  enablement by hand; the plugin diagnoses missing prerequisites but never
  changes Tailscale account or policy state. The Paseo app-to-daemon
  connection may be local, direct, SSH or Paseo relay for plugin control
  messages, but dev-server page traffic itself travels from the viewing
  device's browser over the tailnet to `tailscale serve`; it never travels
  over Paseo relay. Relay-only setups whose daemon and viewing device do
  not share a tailnet are unsupported.
- Current achieved stage and evidence: see PROGRESS.md `## Acceptance`

## Stage outcomes
| Stage | Intended users and usable outcome | Planned? |
|---|---|---|
| MVP-1 | The maintainer can open agent dev servers from any of their tailnet devices by clicking in Paseo | yes |
| MVP-2 | A small supported alpha group can install from a public source and use the click flow on at least two daemon operating systems | yes |
| public-beta | Public users can install the reviewed registry beta on Linux/macOS/Windows daemon hosts and use the documented beta scope, with public issue support | yes |
| public-release | Public users receive the stable 1.0 commitments in the compatibility surface below, after beta evidence shows the core workflow, upgrades and teardown are reliable | yes |

## Selected target acceptance
### O1: Click a localhost link remotely, get the dev server
From a device that is not the daemon host, a localhost link in a supported
agent-timeline item keeps its visible text but takes the user to the dev
server over the tailnet. No separate plugin link or button appears.
1. The link's embedded destination is rewritten to
   `https://<host>.<tailnet>.ts.net:<port><original path>`; clicking it
   first ensures `tailscale serve --bg --https=<port>
   http://localhost:<port>` on the daemon host (creating it only if the
   plugin does not already own it), then opens the rewritten URL.
2. Locality is auto-detected: on the daemon host itself links render and
   behave exactly as today; on any other device their destinations are
   rewritten. Detection compares the selected host's identity with this
   device's own local daemon identity (app host signal or loopback
   `server_info` probe, per Track 1A); when neither signal is available the
   device is treated as remote. Evidence includes a real two-machine check
   in both directions for each supported daemon operating system before
   public beta.
3. Evidence: recorded real-use sessions on desktop and phone clients, plus
   clean-profile installs of the exact beta artifact, noted in
   PROGRESS.md `## Acceptance`.

### O2: Nothing is exposed without an explicit action
No forward exists until a click, a panel action, or an agent tool call
asks for one, and every forward is visible while it lives.
1. With the plugin installed and idle, `tailscale serve status` gains no
   entries from the plugin. Tests pin this.
2. Every live forward appears in the workspace panel with its port, origin
   and expiry, and can be stopped there.
3. The port is whatever the clicked URL (or agent tool call) names; the
   plugin invents and remaps no port numbers. An invalid port, or a serve
   port already owned by someone else, is refused with a readable reason.

### O3: Forwards always end
Every forward is a lease removed at the first of: backend port dead (3
failed TCP checks, 15 s apart), 4-hour lease expiry (reset by click/open/
extend), workspace archive, user Stop, or plugin stop. On startup the
plugin reconciles recorded owned forwards and never touches serves it did
not create (the pre-existing daemon serves included).
1. Tests pin each removal trigger and the ownership check against
   `tailscale serve status --json` fixtures.
2. Evidence: a real forward is observed disappearing after its dev server
   exits, recorded in PROGRESS.md `## Acceptance`.

### O4: Setup failures are instructions, not mysteries
When forwarding cannot work, the user gets the exact fix.
1. Missing or logged-out `tailscale`, missing operator rights, MagicDNS,
   HTTPS or Serve not enabled, unsupported installation variant, port
   already served by someone else, and a dev server rejecting the tailnet
   Host each produce a distinct message naming the command, setting or
   admin-console step that fixes it. Documentation covers common framework
   host rules beyond Vite.
2. A setup/status surface shows these checks without requiring a forward
   attempt. The plugin never hangs on the Tailscale enablement prompt,
   never runs sudo, and never changes Tailscale account or policy state.

### O5: An agent can ask for its own forward
An agent in a workspace can call the plugin's tool with a port and receive
the tailnet URL immediately, under the same lease and visibility rules as
a click. Installing the plugin is the opt-in to this capability, so there
is no per-request approval step; a user can disable the agent tool in
settings. Agent-requested forwards are marked in the panel, exist only as
leases, and use the port named by the caller without remapping.

### O6: The daemon host may be Linux, macOS or Windows
The click flow works identically whichever tested OS runs the daemon; only
binary discovery and preflight wording differ. Unsupported OS or Tailscale
installation variants fail with instructions, never a partial forward.
1. Evidence before public beta: one recorded create/open/remove lifecycle
   on each real daemon OS in PROGRESS.md `## Acceptance`.

### O7: Setup and controls require no configuration to start
A user who installs and enables the plugin can use click forwarding as soon
as external Tailscale prerequisites are met. There is no master plugin
switch to find and no port configuration.
1. Setup status is visible in the plugin UI and distinguishes ready from
   each missing prerequisite without sending diagnostics anywhere.
2. Optional controls are limited to capability choices with safe defaults:
   agent tool on (user may turn it off) and locality override auto (user may
   force local or remote). No setting chooses or restricts port numbers.
3. First-run copy states plainly that forwards are reachable by every
   user/device the user's tailnet policy allows to reach that daemon host
   and port; the plugin adds no authentication of its own.

### O8: Releases install and upgrade cleanly
A public user can install the exact reviewed artifact from the Paseo plugin
registry and upgrade without losing settings or leaving forwards behind.
1. CI runs typecheck, unit tests, client-bundle checks and the roadmap
   audit; release candidates pass an install smoke test of the packed
   artifact on a clean daemon profile at the minimum and current Paseo
   versions in the support range.
2. Runtime dependencies are avoided where host libraries suffice; any that
   remain are locked with exact versions and integrity, and no runtime
   download or lifecycle surprise occurs.
3. Plugin stop, disable, reload and upgrade execute the same owned-lease
   cleanup/reconciliation rules as O3.

### O9: The trust boundary is inspectable
Public users and registry reviewers can tell exactly what code runs, what
data it touches and what leaves the daemon host.
1. The security model documents subprocesses (`tailscale` argv only),
   filesystem access, timeline text processing in memory, network calls
   (loopback and the user's own tailnet only), no telemetry, no external
   service, and no runtime downloads.
2. External inputs that can reach execution — timeline URLs, agent-tool
   input and RPC input — are schema-validated and fixture/fuzz tested.
3. The published artifact remains readable source (no opaque bundled
   server code), with a public security-reporting route.

### O10: The public listing is complete and honest
The registry listing lets a Paseo user decide without reading source.
1. `OVERVIEW.md`, manifest description, icon and current screenshots with
   sample data explain purpose, setup, supported OS/Paseo versions,
   capabilities, data access, permissions and limitations, including no
   terminal-link interception in Paseo 0.16.1.
2. The public repository carries a license, README, runbook, security
   policy, issue templates and release notes; English listing text is
   complete, translations are optional and never machine-claimed as reviewed.
3. Registry submission pins the exact artifact that passed O8's clean
   install smoke test.

## Constraints and accepted limitations
- Tailnet only: `tailscale serve`, never `tailscale funnel` or any public
  exposure. The plugin contains no funnel code path.
- Install-to-use: installation/enablement is the opt-in. The plugin is
  active when enabled in Paseo, but creates no forward until a click, panel
  action or agent tool call. The agent tool can be disabled in settings;
  its default follows the install intent (on).
- Never read, print or log process command lines; never run `ps`, `pgrep`
  or `lsof`. Liveness is a TCP connect; ownership comes from the plugin's
  own records plus `tailscale serve status`.
- Logs carry fixed tokens and port numbers only: never hostnames, tailnet
  names, usernames, emails, URLs with paths, request headers, or timeline
  text. Repository text uses `<host>`, `<tailnet>` and `<user>`
  placeholders.
- `tailscale` is invoked with argv arrays, no shell; single-port removal
  only; `tailscale serve reset` is forbidden.
- Accepted manual steps: installing/logging in to Tailscale; granting the
  daemon user operator rights where the OS requires it; enabling
  MagicDNS, HTTPS certificates and Serve once in the Tailscale admin
  console; setting tailnet grants/ACLs; adding `server.allowedHosts` (or
  the framework equivalent) when a dev server rejects the tailnet Host.
- Accepted limitation: a shared tailnet between daemon host and viewing
  device is required. Direct LAN/SSH reachability alone does not make a
  loopback-bound dev server reachable, and Paseo relay carries daemon
  control messages, not browser page traffic. A relay-native viewer would
  be a different transport design, not this plugin's link rewrite.
- Accepted limitation: links inside terminal tabs cannot be intercepted
  (no Paseo API in 0.16.1); supported surfaces are agent-timeline items
  the transformer can claim, plus panel/manual and agent-tool entry
  points. True traffic-idle teardown is not observable, so leases use
  time-from-last-open instead.

## Non-goals and later work
- No public internet exposure, ever. Revisit trigger: none; this requires a
  spec rewrite, not a feature flag.
- No background watching of listening ports or timeline scanning to
  pre-create forwards. Revisit when: real use shows clicks arriving before
  the plugin can help (e.g. links mostly in terminals), recorded as an
  observation in PROGRESS.md.
- No tailnet policy editing, account automation or authentication layer.
  The plugin documents example owner-only grants and states its exposure
  follows the user's existing tailnet policy. Revisit when: Tailscale or
  Paseo exposes a safe per-forward authorization API suitable for plugins.
- No support for daemon hosts or Tailscale variants outside the tested
  matrix. Revisit when: a public user reports a concrete unsupported
  variant and a maintainer can reproduce it.
- No telemetry or hosted service. Revisit when: a /project-spec revision
  adopts a public privacy policy naming collected fields and opt-out.

## Release policy
- Before public-beta: ordinary releases default to patch; doc-only changes
  take no bump until the next code release (build metadata only).
- Capability checkpoints before public beta: `click-forward` (O1-O4) ->
  minor; `agent-tool` (O5) -> minor; `all-daemon-os` (O6) -> minor;
  `public-beta-ready` (O7-O10) -> minor.
- Stage versions: public beta ships in 0.x (the first registry beta is the
  accepted `public-beta-ready` checkpoint); 1.0.0 means public-release
  accepted after beta evidence, not merely feature completion.
- Strict SemVer from: public-beta. While in 0.x public beta, breaking the
  compatibility surface still requires a minor bump, a migration note and
  a working upgrade path; patch releases are compatible fixes only.
- Compatibility surface: plugin id; RPC/subscription names and schemas;
  settings schema and stored values; workspace/agent metadata written by
  the plugin; agent tool name/input/output; documented CLI of any shipped
  scripts; and the declared `requirements.paseo` range.
- Existing version format and transition: new project; SemVer
  `MAJOR.MINOR.PATCH+buildmeta`, single `VERSION` file as source of truth,
  build metadata computed at runtime and never stored. `package.json`
  mirrors `VERSION` at release time.
- Breaking changes before strict SemVer: allowed only before public beta,
  in a minor; the release note names the break and any recovery step.
  After public beta, follow the 0.x rule above until 1.0.0, then strict
  SemVer major/minor/patch semantics.
- Evidence and release records: PROGRESS.md `## Acceptance` for stage
  evidence, `## Version history` for releases; version decisions come from
  Git history of `VERSION`, not tags. Public releases also get a Git tag
  and human release notes identifying the exact registry artifact/commit,
  supported Paseo range, security fixes and upgrade actions.

## Deferral policy
- Destination: docs/roadmap-future.md.
- Promotion: the stated trigger is observed, then the item is reassessed
  against the selected target.
- Preserve: source/evidence, why deferred, and the revisit trigger.

## Supporting documents
- Current: docs/ROADMAP.md (execution plan), docs/PROGRESS.md (status and
  acceptance evidence), README.md (public install and setup, created by
  Track 7A), OVERVIEW.md (registry listing, created by Track 8A),
  docs/runbook.md (operations and troubleshooting, created by Track 7A),
  docs/security.md (user-facing security model, created by Track 7A),
  docs/designs/click-forward.md (spike findings, created by Track 1A),
  docs/designs/security-model.md (maintainer threat model, created by
  Track 7B).
- Superseded history: none yet.