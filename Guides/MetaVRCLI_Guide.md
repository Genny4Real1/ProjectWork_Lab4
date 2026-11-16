# Meta VR CLI and AI Tools: Why and How

A practical guide for this project (`ProjectWork_Lab4`, Unity 6000.3.14f1 + Meta XR SDK 209) and
for any Meta Quest / Horizon OS work you do with an AI coding agent.

Official sources, if you want the canonical version of anything below:

- [Meta VR CLI overview](https://developers.meta.com/vr/essentials/metavr-overview/)
- [Install Meta VR CLI](https://developers.meta.com/vr/essentials/metavr-install)
- [Browse agent skills](https://developers.meta.com/vr/essentials/agent-skills)
- [Meta VR CLI in other tools](https://developers.meta.com/vr/essentials/metavr-other-tools)
- [Prompting best practices](https://developers.meta.com/vr/essentials/ai-prompting-best-practices)
- [meta-quest/agentic-tools](https://github.com/meta-quest/agentic-tools)

---

## Contents

1. [The problem this solves](#1-the-problem-this-solves)
2. [Why metavr](#2-why-metavr)
3. [Install](#3-install)
4. [Skills, in one page](#4-skills-in-one-page)
5. [Using metavr from your terminal](#5-using-metavr-from-your-terminal)
6. [Using metavr on this project](#6-using-metavr-on-this-project)
   - [6.1 What is here today](#61-what-is-here-today)
   - [6.2 First deploy loop](#62-first-deploy-loop)
   - [6.3 Prep the headset for a lab session](#63-prep-the-headset-for-a-lab-session)
   - [6.4 Turn on what is currently off](#64-turn-on-what-is-currently-off)
   - [6.5 Take a baseline before you add content](#65-take-a-baseline-before-you-add-content)
   - [6.6 Work with the team layout](#66-work-with-the-team-layout)
   - [6.7 Test without a headset](#67-test-without-a-headset)
   - [6.8 Before the store](#68-before-the-store)
7. [Using it through your agent](#7-using-it-through-your-agent)
8. [Prompting playbook](#8-prompting-playbook)
9. [Guardrails](#9-guardrails)
10. [Troubleshooting](#10-troubleshooting)
11. [Cheat sheet](#11-cheat-sheet)

---

## 1. The problem this solves

Quest development has a lot of moving parts that live *outside* your editor: a headset on USB,
an APK that has to be installed and launched, `logcat` streams, Perfetto traces, store uploads,
and an SDK that changes faster than any model's training data.

Without a tool layer, an AI agent working on Quest has three failure modes:

| Failure | What it looks like |
|---|---|
| **Stale memory** | The agent writes code against an API that changed, or hand-edits `AndroidManifest.xml` in a way the current SDK no longer expects |
| **Blind device** | You become the agent's hands: you copy logs, take screenshots, and report what you see |
| **Unverified work** | The agent says "done" without either building, deploying, or reading a frame of evidence |

Raw `adb` does not fix this. It is device-aware but not Quest-aware: no doc search, no Perfetto
analysis, no VR screenshots, no store publishing, and nothing a model can discover as a set of
tools.

`metavr` is the layer that closes the gap.

---

## 2. Why `metavr`

Meta VR CLI (command: `metavr`, formerly `hzdb`) is **one tool with two front ends**:

```
                ┌─────────────────────────────┐
   you  ──────► │  metavr (CLI commands)      │ ──────►  device / docs / store
                ├─────────────────────────────┤
 agent ──────►  │  metavr MCP server          │ ──────►  same code, same results
                └─────────────────────────────┘
                              ▲
                     agent skills sit on top
                   (they teach the workflow, not
                      the raw command syntax)
```

The logic lives in one place. The terminal and the agent reach the same implementation, so you
never have to ask "can the agent do this, or only me?" — the answer is always yes.

What that buys you:

- **Current knowledge instead of memory.** `metavr docs search` hits the live Meta VR docs and
  API reference. This is the single highest-value command for an agent, because it replaces
  guessing with citation.
- **A closed loop on device.** Install, launch, log, screenshot, profile — the agent can verify
  its own work instead of asking you to describe the result.
- **Quest-aware capture.** `metavr capture screenshot` grabs the VR/MR view, not a flat
  `screencap` of the 2D layer.
- **Performance evidence.** Perfetto capture and trace analysis with SQL queries, so a claim like
  "this is GPU bound" comes with numbers.
- **Fewer round trips.** One install covers device management, files, logs, UI automation, 3D
  asset search, simulators, and Horizon Store publishing.
- **It stays current.** A native install updates itself, and the MCP tool list follows the
  installed version, so the agent discovers new capabilities without a prompt change.

### When to reach for it

Any time the work touches hardware or Meta-hosted content: deploy a build, read a log, take a
screenshot, capture a trace, check an API, find a 3D asset, ship to the Horizon Store. For pure
C# refactoring in a script, you do not need it.

### Agent skills are a different layer

A **skill** is a written runbook (`SKILL.md` plus reference files) that teaches an agent *how to
carry out one job properly*: which steps, which checks, what "done" means. Skills follow the open
Agent Skills spec, so they work in any agent that supports it.

| Layer | Answers | Example |
|---|---|---|
| MCP server | *What can I do?* | `vr_docs_search`, `perf_trace_list`, `ui_dump` |
| Skill | *How should this job be done?* | `hz-perfetto-debug` says: warm up once, capture three launches, compare frames, report coverage limits |

An agent with only MCP tools improvises the workflow. An agent with only skills has the plan but
no hands. You want both, and section 3 installs both in one command.

---

## 3. Install

### 3.1 The CLI

Windows (PowerShell):

```powershell
iwr -useb https://developers.meta.com/horizon/install-cli/windows/ | iex
```

macOS / Linux:

```bash
curl -fsSL https://developers.meta.com/horizon/install-cli/ | sh
```

This installs a native binary to `%USERPROFILE%\.metavr\bin` (Windows) or `~/.metavr/bin` and
adds it to your `PATH`. Open a **new terminal** afterwards.

With Node.js 18+ you can skip the native installer entirely:

```bash
npx metavr@latest init        # always the current release
npm install -g metavr         # or a global install: manual updates via npm update -g metavr
```

Verify:

```bash
metavr --version
metavr doctor
```

`metavr doctor` lists the developer software it found — Unity editors, Android SDK, JDK, Meta
tools — and tells you what is missing. It does not test the device or the agent connection.

### 3.2 Connect your agent (this also installs the skills)

```bash
metavr init
```

`metavr init` runs seven steps: install the metavr skill, configure the MCP server, install agent
plugins, optional Meta sign-in, telemetry consent, developer tool check, next steps.

Non-interactive, per agent:

```bash
metavr init --opencode            # also: --claude-code --cursor --gemini --copilot --codex --muse
metavr init -y --all-agents       # CI / fresh machine
metavr init --cursor --no-auth    # skip the sign-in prompt
```

Single agent, MCP only:

```bash
metavr mcp install open-code
metavr mcp install claude-code
metavr mcp install project        # project-scoped config, committed with the repo
```

Targets include `android-studio`, `cursor`, `claude-desktop`, `claude-code`, `vscode`,
`vscode-insiders`, `codex`, `zed`, `windsurf`, `antigravity`, `gemini-cli`, `open-code`,
`lm-studio`, `muse-code`, and `project`. Restart the agent if the command tells you to.

Signing in is optional for installs and devices, but it unlocks Horizon Store commands:

```bash
metavr auth login     # opens a browser
metavr auth status
```

### 3.3 Keep it current

```bash
metavr update          # native install updates itself
metavr update --dry-run
npm update -g metavr   # if you installed via npm
```

A stale CLI is not just missing features: its generated skill carries a timestamp, and an agent
that notices an old timestamp is supposed to nag you about it. Let it.

### 3.4 You may already have it

`metavr` ships inside several Meta tools: the Meta VR Plugin for Android Studio, the Meta Horizon
VS Code extension, and the Meta VR Developer Hub. If `metavr --version` works but you never
installed it, that is why. The standalone install is still the shortest path to a current version.

---

## 4. Skills, in one page

`metavr init` already installs the skills bundle to `~/.agents/skills` and registers the MCP
server. Manage it with:

```bash
metavr skills list                  # what the bundle offers, and what you have
metavr skills status                # installed versions, and drift
metavr skills search "perfetto"     # find one by name or description
metavr skills show hz-perfetto-debug
metavr skills install --all
metavr skills update
```

The full catalog, grouped by build path, is maintained on Meta's site:
**[Browse agent skills](https://developers.meta.com/vr/essentials/agent-skills)** and
[meta-quest/agentic-tools](https://github.com/meta-quest/agentic-tools). There are around thirty,
covering Unity, Unreal, Android/Spatial SDK, React Native, WebXR, performance, design, and
publishing. This guide does not duplicate that list — check the official page, or run
`metavr skills search` against your installed bundle.

The six worth knowing by name for this project:

| Skill | Fires when |
|---|---|
| `hz-quest-verify-first` | Any Quest-specific claim. Load it first, every session |
| `hz-unity-meta-core-sdk` | OVRManager, OVRCameraRig, passthrough, hand tracking, anchors, project setup |
| `hz-unity-project-analyzer` | Building the `.agent-docs` knowledge base for this repo |
| `hz-unity-code-review` | Reviewing Unity VR code for rendering and performance pitfalls |
| `hz-perfetto-debug` | Reading a trace: frame timing, CPU/GPU split, render passes |
| `hz-vr-debug` | Logs, screenshots, crash diagnosis |

If you install skills another way, `metavr init` and the marketplace routes both work — see
[Meta VR CLI in other tools](https://developers.meta.com/vr/essentials/metavr-other-tools).

---

## 5. Using `metavr` from your terminal

You do not have to route everything through an agent. These are the commands worth memorizing.
Run `metavr --help` for the full list, or `metavr <group> --help` per group.

### Docs and assets (no device, no sign-in)

```bash
metavr docs search "passthrough rendering"
metavr docs fetch https://developers.meta.com/horizon/documentation/unity/unity-passthrough
metavr asset search "office chair"
```

### Devices

```bash
metavr device list
metavr device info <device_id>
metavr device connect 192.168.1.42      # wireless
metavr device health-check              # storage, battery, thermal
metavr device configure-testing setup   # reproducible test state
metavr device configure-testing restore
metavr config set default-device <id>   # then drop --device
```

### Build, deploy, launch, look

```bash
metavr app install path/to/YourApp.apk
metavr app launch com.your.package
metavr app stop com.your.package
metavr app list
metavr capture screenshot -o shot.png
metavr log --tag Unity --level W        # quick, filtered
metavr adb logcat --follow              # advanced streaming
metavr ui dump                          # UI hierarchy
metavr ui actions                       # tappable elements only
```

### Performance

```bash
metavr perf capture --app com.your.package --duration 10000 -o run   # writes run.pftrace
metavr perf analyze-trace run.pftrace --app com.your.package --focus overview
metavr perf query run.pftrace "SELECT * FROM slice WHERE name LIKE '%render%' LIMIT 20"
metavr perf compare baseline_run content_run
metavr perf traces
metavr perf monitor -- -d <serial> -p com.your.package -j out --duration 300 --interval 5
```

Capture after a warmup launch, and repeat on the same device before drawing conclusions.

### Simulators, Unity, store

```bash
metavr xrsim <command>                  # drive a running Meta XR Simulator
metavr ssim start                       # SpatialSim
metavr unity projects status            # running editors, and which project they have open
metavr unity projects open <path>
metavr store <command>                  # needs metavr auth login
metavr tools install <name>
```

### Useful global flags

`-d/--device <id>`, `--format table|json|plain`, `--json`, `--color`, `-v`/`-q`,
`--markdown-help`. For automation set `METAVR_TOKEN` so scripts never prompt for sign-in.

---

## 6. Using metavr on this project

Everything above is general. This section is about *this* repository, which is early: the XR
foundation is in place and there is no content yet. That is exactly when the tools pay off,
because the first build, the first deploy, and the first baseline are the steps people usually
fumble by hand.

### 6.1 What is here today

Read this table as a checklist. Every row was verified in the repo or with a `metavr` command.

| Item | Current state | Why it matters |
|---|---|---|
| Unity | `6000.3.14f1` (Unity 6), `Mobile_RPAsset` present | URP mobile settings decide your GPU budget on Quest |
| Meta XR | `com.meta.xr.sdk.core` **209.0.0** | API surface for OVRManager, hand tracking, anchors |
| XR stack | XR Management 4.7.0 + OpenXR 1.16.1, loader asset in `Assets/XR` | Quest features are enabled through OpenXR features, not code |
| Scenes | **none.** `EditorBuildSettings.asset` points at `Assets/Scenes/SampleScene.unity`, which does not exist | First Android build will have nothing to load |
| Bundle id | `com.UnityTechnologies.com.unity.template.urpblank` | Template default: shared with every project made from the same template, and not an id you own |
| Oculus config | `handTrackingSupport: 0`, `anchorSupport: 0`, `sceneSupport: 0` | Hand tracking and MR scene features are off |
| Editor | running, PID visible via `metavr unity projects status` | You can build from the CLI path without hunting for the window |
| Layout | `Assets/_Project/{Home,Shared,Docs,Groups/G_*}` | Four group folders, so scene edits will collide if nobody owns a scene |
| Setup script | `Assets/Editor/MetaQuestVRSetup.cs` | Meta's setup script is already here, so VR setup has run. Do not re-run it |

Sanity-check the toolchain before anything else:

```bash
metavr --version
metavr doctor
metavr unity projects status
```

On this machine `metavr doctor` already reports Unity `6000.3.14f1`, the Unity-bundled Android SDK
and JDK, and XR Simulator `v85.0`. Android Studio is not installed, which does not matter for a
Unity project.

Prompt for an audit without touching files:

```
Audit this Unity project for Quest readiness. Check the Meta XR SDK version against the
current docs, the XR/OpenXR configuration, the Android manifest and bundle identifier, the
build scene list, and the graphics settings. Report what is broken or missing, with file
paths and the docs page that justifies each finding. Do not modify anything yet.
```

The agent should come back with at least: no scene in the build list, the template bundle id, and
hand tracking disabled.

### 6.2 First deploy loop

This is the highest-value thing to automate right now, because there is no scene to build. The
agent does the whole loop and shows you the result instead of asking you to check the headset.

What you say:

```
Create a minimal VR scene at Assets/_Project/Home/Home.unity using the current Meta XR
setup for Unity 6 and SDK 209: XR Origin with floor-level tracking, a camera, and a simple
environment so it is not black. Verify the setup against the docs first and cite the pages.

Then add the scene to Editor Build Settings, switch the build target to Android/Meta Quest,
build an APK into Builds/, and confirm Player Settings uses a real bundle identifier instead
of the template default.

Install and launch it on the connected headset with metavr. Capture a screenshot, and pull
the last 200 warning-or-worse log lines tagged Unity. If it fails to launch, diagnose from
the logs before reporting back. Show me every command you ran.
```

What the agent runs, in order — this is the loop worth learning:

```bash
metavr docs search "set up XR Origin Unity 6 Meta XR SDK"
metavr device list
metavr device info <device_id>
# ... scene creation + build in the editor ...
metavr app install Builds/Android/ProjectWork_Lab4.apk
metavr app launch <your.bundle.id>
metavr capture screenshot -o Guides/img/first-frame.png
metavr log --tag Unity --level W
```

Practical notes for this repo:

- Pick a bundle id now, for example `com.lab4.projectwork`, and change it in Player Settings
  before the first install. Two projects with the template id will overwrite each other on the
  headset.
- `metavr app install` replaces the previous build of the same package, so the loop stays fast.
- If the screenshot is black but the logs look clean, the scene is probably missing from the build
  list — that is the symptom for this project's current state.
- A failing launch prints a clear reason in `metavr log --level E`. Read it before changing code.

### 6.3 Prep the headset for a lab session

Reproducible runs matter more than pretty ones. Before a test pass:

```bash
metavr device health-check              # storage, battery, thermal, UI
metavr device configure-testing setup   # animations off, stay awake
# ... install, launch, test ...
metavr device configure-testing restore # back to normal
```

Prompt:

```
Prepare the connected Quest for a test session: run the health check, then put it in testing
configuration. Tell me anything that would invalidate a performance measurement, like low
battery, thermal state, or low storage. Do not change anything else.
```

Two habits that save an afternoon: charge above 50% before profiling (Quest throttles on low
battery), and restore the device after the session or the next person measures a different device.

### 6.4 Turn on what is currently off

`Assets/Oculus/OculusProjectConfig.asset` has hand tracking, anchors, and scene support all
disabled. Nothing fails loudly when you leave them off — the feature just never appears, which
looks like a bug in your code.

```
Hand tracking does not seem to work in this project. Check the Oculus/Meta XR project
configuration and the OpenXR feature set against the current docs, tell me exactly which
settings are off, and what each one changes. Then enable only hand tracking, and tell me
what to re-test on device.
```

The agent should point at `handTrackingSupport`, the OpenXR Meta Quest feature, and the manifest
permission, then verify with a deploy rather than declaring success from the config edit.

Same pattern later for passthrough, scene anchors, and MR Utility Kit — the settings exist before
the content does, and turning them on early avoids a surprise mid-sprint.

### 6.5 Take a baseline before you add content

An empty scene is the cheapest measurement you will ever get, and the most useful one to have
recorded. Capture it now, and every later regression has something to be compared against.

```bash
metavr perf capture --app <your.bundle.id> --duration 10000 -o baseline_empty
metavr perf analyze-trace baseline_empty.pftrace --app <your.bundle.id> --focus overview
```

Prompt:

```
The app is still nearly empty. Capture three 10-second traces of it on the headset after one
warmup launch, then report frame timing and whether we are CPU- or GPU-bound. Save them as
baseline_empty_1..3 so we can compare later. Note anything about the measurement conditions
that would make a future comparison unfair.
```

After a few months of content:

```bash
metavr perf compare baseline_empty content_pass_2
```

That delta report is the difference between "it feels slower" and "frame time went from X to Y,
here is the slice that grew". For a group project where four people add content to one scene,
this is the argument-settler.

### 6.6 Work with the team layout

`Assets/_Project/Groups/` has a folder per person (`G_Ludovica`, `G_Marius`, `G_Simone`,
`G_Tiziano`) plus `Shared` and `Home`. Two things follow.

**Make the agent document the repo once**, so later sessions do not re-explore it:

```
Load hz-unity-project-analyzer and build the .agent-docs knowledge base for this project.
Cover the XR setup, the _Project folder ownership convention, the groups/ layout, and how a
build gets made. Keep it factual; do not invent structure that is not there.
```

That generates a `.agent-docs/` directory the agent reads at the start of future sessions. With
four contributors and a shared scene, that file is the difference between a coherent change and a
merge conflict.

**Review before merge:**

```
Load hz-unity-code-review and review the changes under Assets/_Project/Groups/G_Marius.
Flag VR-specific problems: per-frame allocations, camera-rig misuse, physics on every frame,
UI that will not survive a narrower field of view. Rank by impact and cite the code.
```

**Share the wiring, not the instructions.** If your teammates use the same agent, commit the
project-scoped MCP config:

```bash
metavr mcp install project
```

Then `git add .mcp.json` and everyone gets the same tools with the same server on clone, instead
of each person following this guide by hand.

### 6.7 Test without a headset

XR Simulator `v85.0` is already installed on this machine, which means you can iterate on scene
setup, input, and UI without passing a headset around the room.

```bash
metavr xrsim runtime status
metavr xrsim app launch                # frontend, if not already running
metavr xrsim device <command>          # simulated device configuration
metavr xrsim input <command>           # synthetic controller / hand input
metavr xrsim record start              # compositor output to MP4, useful for a lab report
metavr xrsim record stop
```

Prompt:

```
Start the XR Simulator, load the Home scene build, and record 20 seconds of the compositor
output to an MP4. Tell me what the runtime and device model were set to, so the recording is
reproducible.
```

Simulator evidence is not device evidence. Use it to check that a scene loads, UI is readable, and
input maps correctly; still capture a real trace on the headset before claiming performance.

### 6.8 Before the store

Far off for a lab project, but a few things are already wrong and cheap to fix now:

```bash
metavr store --help              # requires metavr auth login
```

- the bundle id is the URP template default
- there is no icon or store art, and no build that includes a scene

When you get there, `hz-store-submit` walks the validation and asset checklist, and `metavr store`
does the upload. Do that step yourself, signed in, with the checklist approved — not from a prompt
that says "just ship it".

---

## 7. Using it through your agent

After `metavr init`, the agent discovers the MCP tools itself. You describe a goal; it picks the
tool. Tool names you can expect to see in a session:

```
vr_docs_search        vr_api_details       vr_api_stats
vr_skill_search       vr_skill_load        vr_skill_resource
perf_trace_list       perf_trace_get_context
perf_trace_hex_to_time
ui_dump               ui_actions
metavr_software_paths
```

A healthy session looks like this:

1. The agent calls `vr_docs_search` **before** writing Quest-specific code.
2. It calls `vr_skill_search` / `vr_skill_load` when the task matches a known workflow.
3. It touches the device through MCP rather than asking you to run `adb`.
4. It reports evidence: the command, the output, the trace, the screenshot.
5. It asks before anything destructive — uninstall, reboot, store uploads, config changes.

If it skips step 1, the answer may be plausible and wrong. Say so and make it redo the work with a
docs check. That is a prompt-level fix, not a tool-level one — see the next section.

---

## 8. Prompting playbook

The tools are easy to install; getting an agent to *use* them consistently is a prompting problem.

### 8.1 Five rules

**1. State the stack once, up front.** Meta VR CLI spans Unity, Unreal, native Android, Spatial
SDK, React Native, and WebXR. Unstated context gets you the wrong SDK.

> Not: "How do I render passthrough?"
> Instead: "How do I enable passthrough rendering in Unity 6 with Meta XR SDK 209 on Quest 3?"

**2. Describe the goal, not the tool.** The agent asks the MCP server what it can do when it
connects, and that list tracks your installed version. Naming a flag you half-remember narrows its
choices and may name something that no longer exists.

> Not: "Run `metavr perf capture --mode standard`"
> Instead: "Capture a 10-second performance trace of my app and tell me where the frame time goes."

**3. Be imperative and bounded.** State the action, the filter, and the ceiling.

> Not: "Can you look at the logs?"
> Instead: "Pull the last 500 lines of logcat filtered to the Unity tag at warning level and above,
> then group the distinct warnings by frequency."

**4. Include the evidence it cannot get itself.** The error text, what you were doing, the device
model and OS build, the SDK version, the frame or timestamp range you care about.

**5. Ask for proof.** End with what a checkable answer looks like: the tool call and output, the
docs page cited, the trace query, the screenshot. This single habit removes most vibes-based
answers.

### 8.2 Copy-paste prompts

**Bootstrap: install and wire everything up**

```
Set up Meta's official agentic tooling for this machine, then prove it works.

1. Check whether `metavr` is on PATH and print its version.
   If it is missing, install it with the native installer for this OS
   (Windows: iwr -useb https://developers.meta.com/horizon/install-cli/windows/ | iex).
2. Run `metavr doctor` and summarize what developer software is missing.
3. Run `metavr init` for this agent (add --no-auth if you cannot prompt me) and tell me if a
   restart is required.
4. Show `metavr skills status` so I can see the installed skills.
5. Verify the MCP connection by calling vr_docs_search for "passthrough rendering" and citing
   the pages it returns.

Do not sign me in, do not change system settings, and ask before installing anything beyond
metavr itself.
```

**Verify the connection (the official check)**

```
Use the metavr server's vr_docs_search tool to search the Meta VR developer documentation for
passthrough. Show me the tool result and tell me which pages it returned.
```

If the result or the citations are missing, the MCP server is not connected. Fix that before
trusting anything else the agent says about Quest.

**Make it check docs before coding**

```
Before you touch this code, load hz-quest-verify-first and check the current Meta VR
documentation for the APIs involved. Cite the pages you used. If the docs contradict your
plan, say so and change the plan — do not silently follow the old one.
```

**Ask which skill applies**

```
This task is: <one sentence>. Search the installed Meta VR skills and tell me which ones
apply, in what order, and why. Then load the first one and follow it. Do not start coding
until you have told me the plan.
```

**Deploy and self-verify**

```
Build the Android player, then install and launch it on the connected headset using metavr.
Capture a screenshot so I can see the first frame, and pull the last 200 warning-or-worse log
lines tagged Unity. If the app failed to launch, diagnose it from the logs before reporting
back. Tell me the exact commands you ran.
```

**Crash triage**

```
The app crashes a few seconds after launch on Quest 3. Reproduce it: start a fresh launch,
stream logcat until the crash, and pull the crash buffer. Give me the exception, the first
frame in the stack that belongs to our code, and the smallest change you would try first.
Do not modify files yet.
```

**Performance, done properly**

```
Capture and analyze performance for <bundle id>: one warmup launch, then three measured
launches on the same device, 10 seconds each. Compare them and tell me whether the frame drops
are CPU, GPU, or memory pressure, citing the trace evidence. If the three runs disagree, say
so instead of averaging them.
```

**Audit the project (read-only)**

```
Audit this Unity project for Quest readiness. Check the Meta XR SDK version against the current
docs, the Android manifest and bundle identifier, graphics settings, and any controller-only
input paths. Produce a prioritized list of issues with file paths and the docs page that
justifies each one. Do not change any files.
```

**Keep it current**

```
Before we start, check `metavr --version` and `metavr skills status`. If the CLI or the skills
bundle is behind, tell me what changed and ask before updating. If the metavr-cli skill is more
than a week old, refresh it.
```

### 8.3 A standing project preamble

Paste this into `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, or whatever your agent reads at session
start. Rules stated once beat rules repeated in every prompt.

```markdown
## Meta Quest / Horizon OS work

- This project is Unity 6000.3.14f1 with com.meta.xr.sdk.core 209 and OpenXR, targeting Meta
  Quest 3. Assets live under Assets/_Project, with one folder per group plus Shared and Home.
- Meta's official tools are installed: `metavr` (CLI + MCP server) and the Meta VR agent skills
  in ~/.agents/skills. The VR setup script Assets/Editor/MetaQuestVRSetup.cs has already run;
  do not re-run project setup unless I ask.
- Use `metavr` for device work. Do not call `adb` directly, and do not run
  `adb shell screencap` — use `metavr capture screenshot`. Use `metavr log` for quick reads and
  `metavr adb logcat` for advanced filtering.
- Before writing or reviewing Quest-specific code, verify against current Meta docs with
  `metavr docs search` or the `vr_docs_search` MCP tool, and cite the pages you used.
  Training-data knowledge about these APIs is not trustworthy on its own.
- Check for a matching skill before improvising a workflow: perf traces → hz-perfetto-debug,
  crashes → hz-vr-debug, project documentation → hz-unity-project-analyzer, code review →
  hz-unity-code-review, store → hz-store-submit.
- Report evidence with every claim: the command and its output, the trace query, the screenshot
  path. "Done" without verification is not done.
- Ask before anything destructive or account-backed: uninstalling, rebooting, changing device
  configuration, signing in, uploading to the Horizon Store, editing shared project settings.
- Keep `metavr` current. If the skill's generated timestamp is over a week old, mention it.
```

### 8.4 Anti-patterns

| Prompt | Why it fails | Better |
|---|---|---|
| "Fix my Quest app" | No symptom, no device, no scope | "The app crashes on launch on Quest 3, Unity 6, Meta XR 209. Reproduce and diagnose from logs." |
| "Use adb to grab logs" | Locks in a tool the agent should not need | "Show me the last 200 error lines from the device." |
| "Set up hand tracking" | Wrong SDK guessed, and the config here is off by default | "Enable hand tracking for Unity 6 + Meta XR 209, check the docs for the required settings, verify on device." |
| "Run vr_docs_search" | Naming an MCP tool narrows choices and breaks when the tool set changes | "Check the current docs for the passthrough API, cite the pages." |
| "Is it performant?" | No metric, no threshold, no evidence | "Capture three 10s traces and tell me if we hold frame budget; show the numbers." |
| "Deploy and submit to the store" | Combines a safe step with an irreversible account action | Split them; require approval before upload |
| "Whatever the docs say is fine" | Invites the agent to paraphrase instead of cite | "Quote the setup steps you are following and the page they came from." |

### 8.5 When the agent ignores the tools

1. **Ask it to show the tool call.** "Show me the exact `metavr` command or MCP tool result you
   used." No call means no evidence.
2. **Name the skill.** "Load `hz-quest-verify-first` and redo the answer."
3. **Check the wiring.** `metavr doctor`, then a fresh `vr_docs_search` request. If the agent has
   no tools, it is not a prompting problem.
4. **Make the rule permanent.** Add a line to `AGENTS.md` rather than repeating yourself next
   session.

---

## 9. Guardrails

`metavr` can reboot devices, uninstall apps, change settings, spend store actions, and run shell
commands. Keep the agent on a leash that matches the blast radius:

- **Read freely, write on approval.** Logs, docs, screenshots, traces, UI dumps are safe. Treat
  installs, uninstalls, reboots, config changes, and uploads as approval-gated.
- **Never let it sign in for you.** `metavr auth login` opens a browser; make the click yours.
- **Review before upload.** Store submission is an account-backed, public-facing action.
- **Decide on telemetry yourself.** `metavr telemetry about | status | opt-in | opt-out`.
- **Project config belongs in the repo.** `metavr mcp install project` keeps MCP settings with the
  other developers instead of global state.

---

## 10. Troubleshooting

| Symptom | Fix |
|---|---|
| `metavr` not found after install | Open a new terminal. The installer edits the shell profile / user `Path` |
| Installer returns an HTML page instead of a binary | Proxy, firewall, or captive portal rewrote the download. Re-run on a clean network |
| `npm install -g metavr` says unsupported platform | No prebuilt binary for your platform. Use the native installer |
| Agent has no MCP tools | Re-run `metavr init --<your-agent>`, restart the agent, re-test with `vr_docs_search` |
| Duplicate MCP servers registered | Remove the stale entry from the agent config; see the Meta VR CLI troubleshooting page |
| `device list` is empty | Developer Mode enabled, data-capable USB-C cable, USB debugging prompt accepted on the headset |
| Wrong headset targeted | `metavr config set default-device <id>`, or pass `--device` |
| Store commands fail | `metavr auth login` |
| Skills look stale | `metavr update` then `metavr skills update` |

---

## 11. Cheat sheet

```bash
# setup
metavr --version
metavr doctor
metavr init --<agent> --no-auth
metavr skills status
metavr mcp install project

# docs before code
metavr docs search "spatial anchors"
metavr docs fetch <url>

# this project: build and deploy
metavr unity projects status
metavr device list
metavr app install Builds/Android/ProjectWork_Lab4.apk
metavr app launch <your.bundle.id>
metavr log --tag Unity --level W
metavr capture screenshot -o shot.png

# baseline and compare
metavr perf capture --app <your.bundle.id> --duration 10000 -o baseline_empty
metavr perf analyze-trace baseline_empty.pftrace --app <your.bundle.id> --focus overview
metavr perf compare baseline_empty content_pass_2

# no headset
metavr xrsim app launch
metavr xrsim record start

# housekeeping
metavr update
metavr skills update
metavr auth status
```

The one-line version to give a new agent:

> Use `metavr` and the Meta VR skills for anything Quest-related. Check the docs before coding,
> verify on device before claiming success, and ask me before anything destructive.
