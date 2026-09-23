---
name: agent-onboarding
description: Onboard an AI coding agent to a Syncfusion project. Identifies the framework and product family, installs or selects the official component skills, finds current documentation, handles licensing safely, and verifies a working implementation. Use when starting, integrating, upgrading, migrating, evaluating, or troubleshooting Syncfusion UI components, document SDKs, viewers, or editors on any platform.
metadata:
  author: "Syncfusion Inc"
  version: "1.0.0"
---

# Syncfusion onboarding

Use this skill as the front door to Syncfusion. Route the task to the official framework or component
skill. Do not try to reproduce the Syncfusion API from memory here — Syncfusion spans web, desktop,
mobile, document-processing, viewer and editor products, and the same component name has different
packages, imports, registration and licensing rules across them. Cross-framework guessing is the
single largest source of Syncfusion code that does not compile. The second is same-platform name
collision: the word the user typed matches one component while the behavior they asked for belongs
to another — resolve it by comparing inventory descriptions against the required behavior, never by
name alone.

# Operation Modes

This skill operates in two modes. Both modes are **executable procedures**, not descriptive
summaries. A model that reaches the final report without producing the required proof artifacts
has not completed the mode — it has described it.

## Hard rule — read the inventory before naming any skill

This rule gates Setup Mode step 2 (read the platform index used to derive the inventory) and Task
Mode step 3 (use the retained session inventory). It must be satisfied **first**, before either
mode begins.

You MUST have read the platform inventory at `https://ai.syncfusion.com/<platform-slug>/inventory.txt`
in this session **before** you name, install, or write code against any Syncfusion component skill.
If you have not read it in this session, read it now and do not continue.

When the user asks for a component, before you say "I will install skill X" or write any code:

1. State the **candidate skills** you considered (from the inventory, not from memory).
2. State the **required behavior** inferred from the user's request.
3. State the **evidence** — the inventory entry, name, or description that matches.
4. Cite the inventory file you read.

If no inventory entry covers the behavior, say so explicitly and ask the human before proceeding.
Never propose a skill name, package name, or import path that is not present in the inventory you
just read. If the only source you have for a name is your training data, label it **unverified** and
stop until you can cite the inventory or an installed `SKILL.md`.

## Hard rule — install packages with the package manager, never by editing the manifest

A package install is the package manager running, the registry resolving, peer dependencies
reconciling, the lockfile updating, and `node_modules` / the equivalent populated. None of that
happens when a manifest file is hand-edited.

When installing, upgrading, downgrading, migrating, or removing a Syncfusion package — or any
project package the Syncfusion change depends on:

- **Always invoke the package manager.** React / Angular / Vue / JavaScript: `npm install <pkg>`,
  `pnpm add <pkg>`, or `yarn add <pkg>`. .NET: `dotnet add package <pkg>`. Flutter:
  `flutter pub add <pkg>`. Use the same manager the project already uses; do not switch managers.
- **Never hand-edit the manifest.** Do not add a line to `package.json` /
  `.csproj` / `pubspec.yaml` to install a package. A package name typed into a manifest without
  the manager running is a corrupt installation: no peer-dep resolution, no `node_modules` /
  `obj` / `.dart_tool`, no lockfile update. The next `npm install` / restore will either rewrite
  the line (silently) or fail.
- **The version string must come from the rule, not from the editor.** For npm use a major-version
  selector (`@33`) so the lockfile is the source of truth; for NuGet pass `--version <exact>` every
  time; for Flutter pass `'^X.Y.Z'` from the existing constraint.
- **Disabled environments are the only exception.** If the package manager cannot run here (no
  network, sandboxed executor, user approval denied), stop and ask the human rather than editing
  the manifest on their behalf. Tell the human the exact command to run in their terminal.
- **Manifests are read-only inputs.** They are inspected for existing versions and signals; they
  are not tools for installation. Apply this rule even when you are about to revert the change one
  minute later: leaving a stale manifest line is still drift, and a future agent will not know you
  intended to revert it.
- **Proof artifact.** When the report names "Packages installed", it must cite the exact package
  manager command and its exit status, not a manifest diff. A manifest diff is what an
  installer produces — not what an agent produces.

Failure modes this rule prevents:

- `package.json` lists a package the lockfile does not resolve → next `npm install` reverts it
  silently and the agent thinks the install succeeded.
- `package.json` pins a version while `node_modules` has a different one → tests pass locally,
  CI fails on restore.
- `.csproj` lists `<PackageReference>` with no `dotnet restore` having run → first build errors out
  with NU1605/NU1105 and the developer re-runs the same broken workflow.
- `pubspec.yaml` lists `syncfusion_flutter_*` with no `flutter pub get` having happened → runtime
  "package not found" with no compile-time signal.
- Peer-dep drift (e.g. `react` mismatch in `@syncfusion/ej2-react-grids`) the agent never sees
  because the manager never ran.

## Setup Mode

Triggered when the user asks to:

- set up Syncfusion
- onboard a project
- prepare an AI agent
- configure Syncfusion skills

**Setup Mode executes skill installs. It does not only describe them.** A response that contains
the words "I will install" or "skills to install" without a passing step 8 verification has not
completed Setup Mode and is incomplete. **Do not produce the final report until step 10 passes**
(the report itself must include the Fresh Version Gate line from step 4 and the on-disk
proof from step 8).

Steps are gated. Each step has a **required proof artifact** that must appear in your working
output before the next step runs. If the artifact is missing, stop and fix that step.

| # | Step | Required proof artifact before next step |
| --- | --- | --- |
| 1 | Detect the platform from repository manifests (`package.json`, `*.csproj`, `pubspec.yaml`, `*.sln`, `syncfusion.config.json`). | A one-line statement: `Platform: <slug>` with the manifest signal that produced it. |
| 2 | Read the platform index `https://ai.syncfusion.com/<platform-slug>/llms.txt`. | The platform index file path you read. |
| 3 | Inspect project manifests for existing Syncfusion packages. | A bullet list of `name@version` entries taken verbatim from `package.json` / `*.csproj` / `pubspec.yaml`. |
| 4 | **Fresh Version Gate — mandatory.** Re-read the project manifest and lockfile. Determine the project's Syncfusion version using the package-manager rule from `references/version-resolution.md`: npm = shared major, NuGet = exact version, pub.dev = exact constraint. Do not invent a version from memory. If Syncfusion is not installed yet, record `no existing packages` and use the latest stable within the platform family. | A one-line statement: `Project Syncfusion version (package-manager rule): <value>` (for npm: `npm shared major=<N>`; for NuGet: `exact=<X.Y.Z>`; for Flutter: `constraint=<^X.Y.Z>`; or `no existing packages`). If a conflict exists, stop and report it. |
| 5 | **Run** `npx skills add syncfusion/<repo> --skill <detected-component-skill>` for every detected Syncfusion component, control, library, viewer, editor, SDK, or migration task. One component = one `npx skills add` call. Do not bundle. | The exact command(s) executed and the terminal exit status / output snippet. |
| 6 | If no Syncfusion packages are present, do not install component skills (this is the only "do not install" rule in Setup Mode). Do not install Syncfusion product packages during setup. **If you add, upgrade, downgrade, or remove a Syncfusion product package later (Task Mode), always invoke the package manager — `npm install`, `pnpm add`, `yarn add`, `dotnet add package`, or `flutter pub add` — and let it update the manifest, lockfile, and dependency tree. Never edit the manifest directly.** | A one-line statement: `Packages detected: <list or "none">` and either the commands from step 5 or the explicit decision not to install. For Task Mode package additions/upgrades, the proof is the exact package-manager command run and its exit status — not a manifest diff. |
| 7 | Detect MCP availability (read `syncfusion.config.json` `mcp` block and the host editor's MCP config file if present). | The MCP status with the file path you read. |
| 8 | **Verify the install wrote files.** List the target skills directory and confirm every installed `SKILL.md` is present. | The `ls` / `dir` / `Get-ChildItem` output showing the on-disk `SKILL.md` path(s). If the install failed or no files were written, **stop and report the failure** — do not proceed to step 9. |
| 9 | Fetch the platform registry index `https://ai.syncfusion.com/r/<platform-slug>/registry.json` (skip if platform has no registry). Do not fetch individual variant files — that happens in Task Mode. | A one-line statement: `Registry index fetched: <url or "no registry for platform">`. |
| 10 | Produce the Setup Mode report using the **required output template** below. | The report itself, matching the template, with every field populated and every on-disk path cited. |

Command flags for step 5 (skill installs):

- Use `-y` (non-interactive) so the install does not block on a prompt.
- Use `--agent <name>` when the host agent is known (e.g. `--agent codestudio`, `--agent cursor`,
  `--agent claude-code`); omit it to install into the shared `.agents/skills/` directory.
- If the host requires it, follow the host's authorization rules before the networked install.
- One detected component = one `npx skills add` call. Do not bundle.

### Setup Mode — required output template

The final report in step 10 must use this template. Every field is mandatory. **Do not omit a
field. Do not summarize a field with "see above".** If a field does not apply, write `<reason>`.

```text
## Setup Mode report

Platform: <slug from /llms.txt platform-slug table>

Project Syncfusion version (Fresh Version Gate, step 4 — package-manager rule):
- <npm shared major=<N> | NuGet exact=<X.Y.Z> | pub.dev constraint=<^X.Y.Z> | no existing packages>

Detected Syncfusion packages (from project manifest):
- <name>@<version>   (source: <manifest file>)

Packages installed (Task Mode only — list commands executed by the package manager):
- <exact npm install / pnpm add / yarn add / dotnet add package / flutter pub add command>   (exit code: <N>)

Installed skills (on-disk proof, from step 8):

Skill install commands executed:
- <exact npx skills add … command>

Installed skills (on-disk proof, from step 8):
- <absolute path to installed SKILL.md>   (ls/dir output: <one-line snippet>)

Registry index: <url fetched | "no registry for platform">

MCP status: <integrated | not integrated>
  - config file read: <path or "none">
  - server name(s): <sf-<platform>-mcp>

Licensing status: <license key present | license key absent>
  - license registration site: <https://…>
  - registerLicense call site: <file:line or "not present">

Human action required: <bulleted list, or "None — setup is autonomous">
```

A response that does not match this template has not completed Setup Mode.

### New projects

A project may contain no Syncfusion packages.

In this case:

- Detect the platform.
- Do not install component skills.
- Do not install Syncfusion product packages.
- Report that no existing Syncfusion usage was detected — using the template above, with
  `Detected Syncfusion packages: none` and `Human action required: None — no Syncfusion usage
  detected`. This is still a Setup Mode completion, not a bypass.

Install component skills only when:
- existing Syncfusion usage is detected, or
- the user requests new Syncfusion functionality.

## Task Mode

Triggered when the user requests Syncfusion code, components, controls, libraries, upgrades, migrations, or troubleshooting.

Before proceeding:

1. Re-read project manifests.
2. **Fresh Version Gate — mandatory.** Re-read the manifest and lockfile. Determine the
   project's Syncfusion version using the package-manager rule from
   `references/version-resolution.md`: npm = shared major, NuGet = exact version,
   pub.dev = exact constraint. Do not invent a version from memory. Do not install a
   Syncfusion package unless its version satisfies that rule. When installing a new
   package, install the latest published release that satisfies the rule. If a conflict
   exists, stop and report it — do not align versions yourself. Required proof artifact:
   `Project Syncfusion version (package-manager rule): <value>`.
3. Identify the candidate component skills from the retained session inventory. Match against
   required behavior, not against the word the user typed. State the candidates, required behavior,
   and the evidence (the inventory entry that matches) exactly as the Hard rule requires. Skip this
   step only if the inventory already resolved this request in the same session.
4. Install missing skills if necessary. One `npx skills add --skill <name>` per detected component.
   Do not install the whole platform pack.
5. Consult the platform registry for a matching variant and adapt the selected template when one exists.
6. Read the selected component `SKILL.md` completely, plus only the references the task needs.
7. Follow the Version Resolution Policy before any package installation.
8. Continue with implementation.

## Route first

The fastest correct path is almost always:

1. Identify the platform from the repository manifest.
2. Fetch `https://ai.syncfusion.com/<platform-slug>/llms.txt` for that platform. It is self-sufficient:
   skill pack, packages, license registration, a complete example, and a verification checklist.
3. Read the platform inventory — `https://ai.syncfusion.com/<platform-slug>/inventory.txt` — once
   during setup, not per request. From it, retain a **session inventory**: a compact routing map of
   each component skill with its category, its package, and the behaviors its description covers.
   The session inventory is the routing source for the rest of the session, for as long as session
   memory lasts.
4. List the candidate component skills from the session inventory and choose the skill by required behavior.
5. Fetch the platform registry, pick the closest variant, and adapt it. Skip only when no template matches — fall back to the component skill and state why.
6. Run the **Fresh Version Gate** before installing, upgrading, downgrading, or generating
   code against any Syncfusion package. Re-read the manifest and lockfile. Determine the
   project's Syncfusion version using the package-manager rule in
   `references/version-resolution.md` — npm = shared major, NuGet = exact version,
   pub.dev = exact constraint. Do not invent a version from memory and do not default to
   npm-style "same major" on NuGet or Flutter. If a conflict exists, stop and report it;
   do not align versions yourself.
7. READ `https://ai.syncfusion.com/licensing.md` before touching any key.inside a platform too,
   not only across platforms: "a calendar to display events" matches both the Calendars skill
   (date-selection inputs) and the Scheduler skill (event and appointment management).
8. Choose by required behavior, not by the word the user typed, and state the candidates, the
   choice, and the evidence exactly as the "Hard rule" checklist above requires. If two
   candidates still tie and the choice changes the implementation, ask one short question and stop.
9. Install only the component skills required for detected Syncfusion usage or the current task.
10. Read `https://ai.syncfusion.com/licensing.md` before touching any key.

Subsequent requests in the same session resolve from the session inventory — do not fetch the
inventory again. This covers both direct requests ("add a Grid") and requests that describe a goal
rather than name a component ("view my PDF file", "edit a document", "display events on a
calendar"). Match the described behavior against the behaviors the map records for each skill; do
not keyword-match the user's words, which wastes context and produces wrong-component answers.
Refetch the inventory, only when the session inventory cannot resolve the request: no candidate
covers the behavior, two candidates tie in a way that changes the implementation, or the request
names a component the map does not contain — it may be newer than the setup read or belong to a
different platform slug.

## Component Registry

The component registry is a curated, per-platform index of JSON templates. Each template
packages a complete, verified implementation: source, exact package versions, and licensing
posture. Always fetch the platform registry index before writing any component code. Use the closest matching template as the starting point. Skip only when the requirement is outside what any template describes — state the reason explicitly.

The registry is currently populated for **Angular**, **React**, **Vue**, **aspnet-core**, **aspnet-mvc**, **Blazor**, **JavaScript**, **.NET MAUI**, **WinForms**, **WPF**, and **WinUI**. Every other platform uses the skill pack as the only work surface.

### Endpoints

- Registry index (per platform): `https://ai.syncfusion.com/r/<platform-slug>/registry.json`
- Per-template source: `https://ai.syncfusion.com/r/<platform-slug>/<variant>.json`
- For React, include the language segment: `https://ai.syncfusion.com/r/react/<js|ts>/<variant>.json`

### When to consult the registry

Treat the registry as a normal step in fulfilling a request. Trigger it when any of the
following is true:

1. **The user names a known variant** — for example "add a Grid with grouping", "show a stacked
   column chart", "render the PDF viewer with annotations". The variant is implied, so fetching
   the matching template is more precise than generating from memory.
2. **The user gives a vague component request** — for example "add a chart", "I need a
   scheduler", "drop in a data grid". Pick the template whose `name` or `description` best matches the intent, adapt it to the project, and state the chosen template before adapting.
3. **The request clearly maps to one of the available templates.** Use that template as the
   starting point. Adapt for framework, language, theming, and package version — do not paste
   verbatim when the project conventions differ.

### When NOT to consult the registry

Skip the registry when the requirement is outside what any template describes — custom behaviour,
application-level composition, or a workload that does not exist as a variant. In those cases,
do not use the registry at all; fall back to the platform skill pack and state the reason.

### Procedure before writing any code

1. Identify the target platform from the manifest (see "Establish the project context").
2. Install the relevant component skill first (see "Path A — official agent skills"). The skill
   pack teaches the integration and licensing rules; the registry item applies those rules to
   one specific implementation.
3. Fetch `https://ai.syncfusion.com/r/<platform-slug>/registry.json` once per session.
4. Pick a matching template by intent. When multiple templates match, prefer the one whose
   `behaviors` align most precisely with the user's stated need.
5. Fetch `https://ai.syncfusion.com/r/<platform-slug>/<variant>.json` (or the React
   `js|ts` variant) to get the full source, exact dependency versions, package name, and
   licensing posture.
6. Cite the registry URL and the matched template name before pasting any source into the
   project.
7. Adapt the template — framework, language, theming, package version, project conventions.
8. Run the verification checklist (see "Verify").

### Relationship to the platform inventory

`inventory.txt` and `registry.json` serve different surfaces; do not treat them as
interchangeable:

- **`inventory.txt`** is the **routing surface**. It maps every component skill on a platform
  to its category, package, and behaviours. Read it once during setup and retain a session
  inventory thereafter.
- **`registry.json`** is the **work surface** for known variants. It returns a complete,
  verified implementation template rather than a routing map.

When the request describes a goal ("view my PDF file", "display events on a calendar"), start from the inventory. When the request names a component or a variant, go straight to the registry.

### Fallback

If the platform has no registry, no template matches, or the matched template only partially
covers the requirement, fall back to the platform skill pack and explicitly state the gap:
which platform, which request, and why the registry did not apply. Never silently generate from
trained memory when a template exists that covers the same behaviour.

## Sources of knowledge

Two sources. Pick the one you are actually reading from, and say so out loud when you make a claim:

- **Session inventory** — the routing map retained from `inventory.txt` and the installed skill's
  `SKILL.md` / `references/*.md`. Use this for any claim about a component's package, import, or
  behavior. Cite the file and section.
- **Trained memory** — patterns baked into the model. Not citable. Do not use it to fill gaps in
  the session inventory; the refetch triggers in "Route first" say what to do instead.

Platform Slugs: `react` `angular` `javascript` `vue` `blazor` `aspnet-core` `aspnet-mvc` `flutter` `maui`
`maui-toolkit` `winforms` `wpf` `winui` `document-sdk` `xamarin-to-maui-migration`

Master index: https://ai.syncfusion.com/llms.txt

If you have no network access, the rest of this skill and its references carry enough to proceed.

## Establish the project context

Inspect the repository before asking the user for anything already present. Determine:

- framework, language, runtime, package manager
- existing Syncfusion packages and their exact versions (used by the Fresh Version Gate; do not
  call a version unless it is in the manifest or lockfile)
- the requested component or SDK and the features actually needed
- whether this is a new integration, an edit, an upgrade, a migration, or troubleshooting
- existing theme and CSS setup, application bootstrap, and test and build commands
- existing skills directory and MCP configuration
- evidence of license registration — without printing or exposing any key

Manifest signals: `package.json` for React, Angular, Vue and JavaScript; `.csproj` for Blazor,
ASP.NET Core, ASP.NET MVC, MAUI, WPF, WinForms and WinUI; `pubspec.yaml` for Flutter.

Do not mix examples across platforms. If the repository does not resolve the platform and the choice
changes the implementation, ask one short question and stop.

Two slugs can both be correct: a React application that displays PDFs in the browser and signs them
on a .NET server needs `pdf-viewer-sdk` and `document-sdk`. Two *UI framework* slugs never are.

## Choose the path

| Need | Path |
| --- | --- |
| Generate or modify Syncfusion code | A — official agent skills |
| Verify a current API, release change, or advanced configuration | B — current documentation |
| Resolve license setup or a license warning | C — licensing |
| Evaluate Syncfusion before changing the project | D — research only |

A typical implementation uses A, consults B for uncertain details, observes C throughout, and finishes
with verification.

## Path A — official agent skills

Syncfusion publishes component-aware skills containing setup, imports, modules and services,
properties, events, theming, accessibility guidance and implementation patterns — and, more valuable,
the failure modes that public documentation omits.

1. Check whether the matching skill is already installed in the agent's skills location.
2. If missing and installation is within the user's request, choose the narrowest official pack
   or component skill from the retained inventory routing map, using the behavior-based
   comparison above. Read `references/skill-packs.md` for the verified repository names and commands.
3. Before running a networked install or changing project-level agent configuration, follow the
   host's authorization rules.
4. Read the selected component `SKILL.md` completely before implementing. Read only the supporting
   references the requested features need.
5. Follow the installed skill over remembered snippets. Before every Syncfusion package install,
   upgrade, downgrade, migration, or code-generation change, run the **Fresh Version Gate** — see
   `references/version-resolution.md`. The matching rule is package-manager-specific: npm shared
   major, NuGet exact version, Flutter exact constraint. Do not pick a version from memory. Do
   not install packages or generate Syncfusion code until the Fresh Version Gate passes. Match
   the project's existing version unless the user has chosen an explicit upgrade strategy.

Prefer installing only the component skills required for detected Syncfusion usage or the current task.

Do not install every component skill in a platform inventory solely because the platform was detected or because future work is anticipated. Project-local installation keeps the skill aligned
with the repository and shareable with the team.

Installing an agent skill does not install the Syncfusion product packages. The component skill
identifies the actual runtime dependencies; install those separately.

## Path B — current documentation

Use when the answer depends on a current API, a recently released feature, an exact package,
migration behaviour, or a troubleshooting detail.

1. Use the configured Syncfusion MCP server and its documentation search tool when available.
2. Otherwise search the official documentation for the exact framework, component and installed
   version.
3. Use official demos and repositories to supplement documentation — never cross-framework snippets
   found by general search.

Skills guide code generation; MCP servers retrieve current documentation. They are complementary, and
MCP is optional — it requires an API key, and the anonymous path must work without one. See
`references/mcp-setup.md`.

Say when a detail was verified from current documentation. Do not invent an API when authoritative
information is unavailable; state what you could not confirm.

## Path C — licensing

Installing and reading Syncfusion agent skills requires no license. Using Syncfusion component
libraries or document SDKs is governed by the applicable commercial, Community, trial or other
product license.

- Never generate, guess, log, echo or commit a license key or MCP API key.
- Do not claim the project is licensed because packages build or skills are installed.
- Reuse the project's existing secret-management pattern. Keep secrets out of source control and out
  of client-visible output, unless the platform's official registration method explicitly requires
  application-level registration.
- Use the licensing page for the exact framework and installed version; mechanisms differ by platform
  and release.
- Never suppress, hide or CSS-hide a licensing banner or warning.
- If a key or account action is required, explain what the human must obtain or authorize, and stop.
- Treat license terms as authoritative. Do not infer production rights from a trial or Community
  license without verification.

An MCP API key is a credential for the documentation assistant. It is not a product license key.
Full rules: `references/licensing.md`

## Path D — research only

When the user is evaluating Syncfusion, install nothing — no packages, no skills, no MCP
configuration. Identify the target framework and workload, then compare the relevant official
product, documentation, demos, deployment model and licensing requirements. Separate verified facts
from recommendations.

## Implement

1. Preserve the project's framework version, architecture, formatting and dependency conventions.
2. Install only the packages the selected component and features require.
3. Include every required import, module or service injection, provider, handler, tag-helper
   registration, theme stylesheet, runtime asset and server dependency the component skill describes.
4. Implement the smallest end-to-end slice that demonstrates the requested behaviour, with realistic
   typed data.
5. Emit complete files. No ellipses, no "rest of your code", no partial diff as the primary output.
6. Validate loading, empty, error and primary interaction states where relevant.
7. Cite the source for every component claim — session inventory or installed `SKILL.md`. If a
   line cannot be cited, do not write it.

## Verify

Do not report success from compilation alone. Full checklist: `references/verification.md`

At minimum: the component renders or executes, the requested feature is observably exercised, no
licensing warning appears, the runtime log is clean of missing-module and missing-asset errors, and
no key appears in the diff. If runtime verification is unavailable in your environment, say exactly
what remains unverified and give the user a concise manual check.

## Troubleshoot in this order

1. Framework and Syncfusion package version compatibility
2. Missing package, peer dependency, module, service, provider, handler or import
3. Theme CSS, fonts, scripts, static assets, or a required server-side endpoint
4. Data shape, identifiers, date and number parsing, async lifecycle
5. Feature-specific configuration from the installed component skill
6. Current official documentation, or an MCP lookup
7. A minimal reproduction with unrelated application code removed

Preserve the original error text. Distinguish a product defect from an integration or configuration
issue. Use Syncfusion Support when a minimal reproduction still fails against documented behaviour:
https://support.syncfusion.com/

For each proposed fix, cite the source, per "Implement" and the "Hard rule" above.

## References

| Reference | When to use |
| --- | --- |. Mandatory. The matching rule is package-manager-specific (npm = shared major, NuGet = exact version, Flutter = exact constraint).
| `references/skill-packs.md` | Choosing or installing a pack; you need a verified repository name |
| `references/mcp-setup.md` | Configuring an MCP server, or deciding whether you need one |
| `references/licensing.md` | Any key, secret, CI, or account question |
| `references/version-resolution.md` | Before installing, upgrading, downgrading, or generating code against any Syncfusion package — the matching rule is package-manager-specific |
| `references/verification.md` | Before reporting that an implementation works |
