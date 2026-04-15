# Plan: Build the Kit Skills & Agents for a Custom Agentic OS (Phased)

## Context

You're committing to build a custom agentic OS from scratch — Linux kernel inherited, everything above (L2 system services, L3 desktop runtime, L4 agentic layer) written from zero in Rust, portable to Apple Silicon later. Before writing a single line of OS code, the kit has to *know how to guide that work*. Today it covers ~35% of the ground; the gap is concentrated in OS userland, desktop stack, distribution, and OS-level agent integration.

This plan is **purely about authoring the kit content** — new agent files and new `SKILL.md` files — in phases, so that each phase unblocks the matching phase of the OS build later. No OS code is written here. The deliverable of every phase is markdown files in `claude_kit/`.

**Scope:**
- 1 new agent division: `agents/software-company/os-engineering/`
- 5 new agents inside it
- 31 new skills across 5 new category folders + 2 extensions of existing categories
- Updates to [CLAUDE.md](../CLAUDE.md), [skills/_studio/generate-claude-md/SKILL.md](../skills/_studio/generate-claude-md/SKILL.md), and [agents/software-company/software-cto.md](../agents/software-company/software-cto.md) so the new content is routable

**Out of scope:** writing OS code, profiling the laptop, picking a distro. Those happen after the kit is ready.

---

## Authoring Standards (apply to every phase)

Pulled from [CLAUDE.md](../CLAUDE.md) and [.claude/skills/claude-kit/SKILL.md](../.claude/skills/claude-kit/SKILL.md):

- **Skill format (WAT):** YAML frontmatter with only `name` + `description`, numbered sections, `---` separators, second-person imperative, bold `**Rule:**` callouts, token-minimal, no preamble
- **Skill location:** `skills/<category>/<skill-name>/SKILL.md`
- **Agent format:** match the existing `agents/software-company/engineering/*.md` structure (frontmatter with `name`, `description`, `tools`; then `## When to use`, `## Core responsibilities`, `## Decision framework`, etc.)
- **Registration checklist (per skill):**
  1. Write `skills/<category>/<skill-name>/SKILL.md`
  2. Add the skill to [skills/_studio/generate-claude-md/SKILL.md](../skills/_studio/generate-claude-md/SKILL.md) under the correct category block
  3. Update the Available Content table in [CLAUDE.md](../CLAUDE.md)
  4. Check for >50% overlap with existing skills — merge instead of adding if found
- **Registration checklist (per agent):**
  1. Write `agents/software-company/os-engineering/<agent>.md`
  2. Add to the software-company tree in [CLAUDE.md](../CLAUDE.md) (Agents section)
  3. Add a routing line in [agents/software-company/software-cto.md](../agents/software-company/software-cto.md) so the CTO delegates OS work here
  4. Add to [agents/board/company-coo.md](../agents/board/company-coo.md) if cross-company routing is needed
- **Check first, author second:** every phase starts with a grep for overlap against the existing 1,219 skills. If `rust-pro` or `systems-programming-expert` already covers 50%+ of a new skill, extend instead of creating

---

## Phase 0 — Foundation (prerequisite for every other phase)

**Goal:** create the new division scaffold and wire it into the kit's routing so `software-cto` and `company-coo` know it exists.

**Deliverables**
1. `agents/software-company/os-engineering/` directory created (empty)
2. `agents/software-company/os-engineering/os-userland-architect.md` — master agent for the division. Owns L2/L3/L4 architecture, capability model, IPC design, HAL boundary, arch portability (x86 ↔ ARM64). This is the agent `software-cto` delegates all OS work to.
3. Update [agents/software-company/software-cto.md](../agents/software-company/software-cto.md): add an "OS-engineering" delegation path pointing at `os-userland-architect`
4. Update [agents/board/company-coo.md](../agents/board/company-coo.md): mention the new division under software-company
5. Update [CLAUDE.md](../CLAUDE.md) §Agents: add the `os-engineering` subsection with 1 agent (will grow across phases)
6. Create `skills/os-linux-platform/`, `skills/os-boot-and-firmware/`, `skills/os-system-services/`, `skills/os-distribution/`, `skills/os-desktop-stack/` as empty category folders with `.keep` files so phases can drop skills in
7. Update [skills/_studio/generate-claude-md/SKILL.md](../skills/_studio/generate-claude-md/SKILL.md): add the 5 new categories under a new "OS / Systems Userland" group (no skills listed yet)
8. Update [CLAUDE.md](../CLAUDE.md) §Repository Structure: add the 5 new `skills/os-*` folders to the tree diagram

**Validation**
- `company-coo` prompt can be asked "route an OS-userland task" and mention os-engineering → os-userland-architect
- `software-cto` routing table references `os-userland-architect`
- All 5 new skill folders exist and are listed in the structure diagram
- No broken links in CLAUDE.md

**Blocks:** every subsequent phase

**Size:** S (1 agent, 5 folder stubs, 4 doc updates)

---

## Phase 1 — Linux Platform Foundation

**Goal:** unblock OS Phases 0–1 (laptop profiling and bootable skeleton) by giving the kit the knowledge to configure a minimal Linux kernel for one specific laptop.

**Deliverables**
1. **Agent** `agents/software-company/os-engineering/linux-platform-expert.md` — kernel config, initramfs, DRM/KMS at the kernel level, PipeWire/libinput wiring from kernel up, NetworkManager/iwd, BlueZ, udev, hardware profiling, kernel cmdline tuning
2. **Skill** `skills/os-linux-platform/linux-kernel-config-for-custom-os/SKILL.md` — how to produce a minimal `.config` tailored to one laptop; `make localmodconfig`, defconfig diffs, what to strip, what's load-bearing
3. **Skill** `skills/os-linux-platform/hardware-profiling-and-driver-mapping/SKILL.md` — `lspci -nnk`, `lsusb -vv`, `dmidecode`, `inxi -Fxz`, `lshw`, `udevadm info`; mapping each chip to kernel module + firmware blob; detecting broken-support chips (the Phase 0 blocker check)
4. **Skill** `skills/os-linux-platform/drm-kms-and-mesa-basics/SKILL.md` — DRM/KMS architecture, GBM, EGL/Vulkan handoff, enough to drive a compositor without writing GPU drivers
5. **Skill** `skills/os-linux-platform/pipewire-wireplumber-session/SKILL.md` — PipeWire graph model, WirePlumber session policy, audio + screencapture + camera routing
6. **Skill** `skills/os-linux-platform/input-stack-libinput-udev/SKILL.md` — libinput event model, gesture recognition, touchpad tuning, the knobs that make a trackpad feel macOS-grade
7. Register all in [CLAUDE.md](../CLAUDE.md) and [skills/_studio/generate-claude-md/SKILL.md](../skills/_studio/generate-claude-md/SKILL.md)

**Validation**
- `linux-platform-expert` agent prompt is self-contained — another Claude instance invoked with it can answer "produce a minimal `.config` for a Dell XPS 13 9310"
- `hardware-profiling-and-driver-mapping` includes a concrete worked example (one real laptop, one real `lspci` dump, one full mapping table)
- No overlap with existing `skills/languages/systems-programming-rust-project` or `skills/devops/server-management` (spot-check both)

**Depends on:** Phase 0

**Size:** M (1 agent + 5 skills)

---

## Phase 2 — Boot & Firmware

**Goal:** unblock OS Phase 1 boot path. Everything needed to get from power-on to your custom init.

**Deliverables**
1. **Skill** `skills/os-boot-and-firmware/uefi-and-systemd-boot/SKILL.md` — ESP layout, systemd-boot vs GRUB, boot entries, kernel cmdline, dual-boot with Windows safely (not touching the Windows bootloader)
2. **Skill** `skills/os-boot-and-firmware/secure-boot-and-tpm2/SKILL.md` — Secure Boot with self-enrolled keys, shim, sbctl, TPM2 PCR policy, measured boot, `systemd-cryptenroll` TPM binding
3. **Skill** `skills/os-boot-and-firmware/initramfs-minimal/SKILL.md` — booster vs dracut vs mkinitcpio, minimal module set, early-userspace, root unlock, switch-root
4. Register all

**Validation**
- Each skill has a worked example producing actual artifacts (a `.conf` snippet, a `sbctl` command sequence, an initramfs config)
- The three skills together answer: "how do I boot into a 50MB initramfs that unlocks a LUKS2 root and launches a custom PID 1 on a secure-boot-enabled laptop, without breaking the Windows dual-boot"
- No overlap with `skills/security-defensive/secrets-management` (that's app-level; this is boot-level)

**Depends on:** Phase 0 (folder exists). Independent of Phase 1 but shares `linux-platform-expert`.

**Size:** S (3 skills)

---

## Phase 3 — Desktop Stack + Wayland Compositor Expert

**Goal:** unblock OS Phases 2–3 (graphical session, DE shell). This is where the "feels like macOS" lives.

**Deliverables**
1. **Agent** `agents/software-company/os-engineering/wayland-compositor-expert.md` — Smithay and wlroots-rs, input routing, output config, HDR, fractional scaling, protocol extensions, portal integration, compositor ↔ shell IPC
2. **Skill** `skills/os-desktop-stack/wayland-compositor-with-smithay/SKILL.md` — Smithay architecture, `anvil` reference, minimal compositor skeleton, delegate macros, surface/toplevel lifecycle
3. **Skill** `skills/os-desktop-stack/compositor-input-routing-and-gestures/SKILL.md` — input routing from libinput into the compositor, multi-touch gesture recognition, inertial scrolling, macOS-quality trackpad feel (four-finger workspace swipe, pinch-to-zoom, tap-to-click latency budget)
4. **Skill** `skills/os-desktop-stack/xdg-desktop-portals-implementation/SKILL.md` — xdg-desktop-portal protocols, implementing backends for file chooser, screenshot, screencast, inhibit, settings; why sandboxed agents need portals
5. **Skill** `skills/os-desktop-stack/display-configuration-hdr-and-fractional-scaling/SKILL.md` — `wlr-output-management`, multi-monitor, HDR metadata, fractional scaling (wp-fractional-scale-v1), color management (wp-color-management-v1)
6. Register all

**Validation**
- `wayland-compositor-expert` agent prompt can answer "how do I implement four-finger workspace swipe with inertial scrolling" with concrete Smithay-level guidance
- `compositor-input-routing-and-gestures` calls out specific latency targets (e.g. tap-to-click under 16ms) — that's what separates "feels like macOS" from "feels like Linux"
- No overlap with `skills/frameworks-desktop/makepad-*` (Makepad is a UI framework, not a compositor) — verify

**Depends on:** Phase 0. Benefits from Phase 1 (`drm-kms-and-mesa-basics`, `input-stack-libinput-udev`) but can be authored in parallel.

**Size:** M (1 agent + 4 skills)

---

## Phase 4 — System Services

**Goal:** unblock OS Phases 1 and 4 (custom init, sandboxing, privileged services).

**Deliverables**
1. **Skill** `skills/os-system-services/rust-init-and-service-supervisor/SKILL.md` — what PID 1 must do (reap zombies, signal forwarding, service lifecycle), writing one in Rust vs. running a minimal systemd with only the units you want; socket activation; user sessions
2. **Skill** `skills/os-system-services/cgroups-v2-namespaces-for-userland/SKILL.md` — cgroups v2 unified hierarchy, namespace types, how a userland sandbox runtime wires them without root (user namespaces), comparison to bubblewrap
3. **Skill** `skills/os-system-services/seccomp-landlock-capability-sandbox/SKILL.md` — seccomp-bpf filter authoring, landlock rulesets, Linux capability model, per-process policy generation from a manifest, audit-mode rollout
4. **Skill** `skills/os-system-services/systemd-user-units-for-agents/SKILL.md` — user units vs system units, `Type=notify`, `Restart=`, resource limits, packaging Ollama / the agent shell / MCP servers as user units with proper lifecycle
5. Register all

**Validation**
- `rust-init-and-service-supervisor` explicitly addresses the pragmatic question: "when should you write your own PID 1 vs run minimal systemd?" (answer has clear decision criteria, not dogma)
- `seccomp-landlock-capability-sandbox` has a worked example: a full sandbox policy manifest for a web browser
- No overlap with `skills/containers-orchestration/docker-expert` (that's container-runtime level, this is userland-services level) — verify

**Depends on:** Phase 0

**Size:** S (4 skills)

---

## Phase 5 — Distribution + Immutable Distro Expert

**Goal:** unblock OS Phase 4 (atomic updates, signed packages, rollback).

**Deliverables**
1. **Agent** `agents/software-company/os-engineering/immutable-distro-expert.md` — image-based OS design (OSTree, bootc, rpm-ostree), A/B partitioning, atomic updates, signing chain, rollback, build pipelines (mkosi, dracut, image-builder)
2. **Skill** `skills/os-distribution/image-based-atomic-distro-builder/SKILL.md` — architecture of an OSTree/bootc-style builder, content-addressed object store, deployment model, why this beats traditional package managers for a custom OS
3. **Skill** `skills/os-distribution/package-format-and-signing/SKILL.md` — your own package format design (manifest, payload, signature), cryptographic signing with minisign/ssh-keys/cosign, signature verification at install, revocation
4. **Skill** `skills/os-distribution/ab-partition-and-rollback/SKILL.md` — A/B root partition layout, atomic switch-and-reboot, boot counter, automatic rollback on boot failure, health-check protocol
5. Register all

**Validation**
- `immutable-distro-expert` can compare OSTree vs bootc vs NixOS model and recommend which to inherit patterns from for this specific project (answer: bootc is the modern one)
- `package-format-and-signing` includes a concrete signed-manifest example
- No overlap with `skills/security-defensive/secrets-management` or `skills/devops/github-actions-templates` — verify

**Depends on:** Phase 0. Benefits from Phase 2 (`secure-boot-and-tpm2` for the signing chain) but can be authored in parallel.

**Size:** S (1 agent + 3 skills)

---

## Phase 6 — Agentic OS Layer (extends existing `agents-orchestration/`)

**Goal:** unblock OS Phase 5 (the reason the whole OS exists — agents as first-class system citizens). This phase extends an existing skill directory rather than creating a new one.

**Deliverables**
1. **Skill** `skills/agents-orchestration/agent-as-system-shell/SKILL.md` — what it means for an LLM agent to own the shell/launcher/dock role instead of a human-designed UI; command bar as primary interface; text + voice entry; fallback when the model is down; offline-mode behavior; trust boundary with user actions
2. **Skill** `skills/agents-orchestration/on-device-llm-as-system-service/SKILL.md` — packaging Ollama / llama.cpp / vLLM as a systemd user unit with GPU passthrough, model lifecycle management, model store on a shared partition, quota enforcement, failover to cloud (Claude API) when local model is insufficient
3. **Skill** `skills/agents-orchestration/system-wide-voice-control/SKILL.md` — wake word (openWakeWord / picovoice), PipeWire capture path, Whisper.cpp streaming, intent classification, dispatch to MCP tools; privacy gate (LED-level indication, push-to-talk fallback)
4. **Skill** `skills/agents-orchestration/continuous-vision-loop-agent/SKILL.md` — periodic screen capture via `grim`/`wf-recorder`/`ext-image-copy-capture-v1`, local VLM vs Claude vision, rate limiting, opt-in gating per app, privacy model
5. **Skill** `skills/agents-orchestration/agent-state-persistence-across-reboot/SKILL.md` — checkpointing agent conversation + working memory + tool-call history to disk, resumption semantics, invalidation on system state change, integration with `agent-memory-systems`
6. Register all (extend existing category, not new) in [CLAUDE.md](../CLAUDE.md) agents-orchestration count and [skills/_studio/generate-claude-md/SKILL.md](../skills/_studio/generate-claude-md/SKILL.md)

**Validation**
- `agent-as-system-shell` explicitly addresses the failure modes: network down, model hallucinating a destructive action, user typing while agent is talking
- `on-device-llm-as-system-service` references the existing `skills/data-science-ml/local-llm-expert` skill and doesn't duplicate it — it builds on top
- `agent-state-persistence-across-reboot` references existing `skills/agents-orchestration/agent-memory-systems` and `conversation-memory` rather than reinventing
- Grep for overlap: none of the 37 existing `agents-orchestration/` skills already cover these five concerns — spot-check `conductor-*`, `agent-orchestrator`, `closed-loop-delivery`

**Depends on:** Phase 0. Independent of Phases 1–5 — these are about agents, not OS internals. Can be authored in parallel with Phases 1–5.

**Size:** M (5 skills, extends existing dir)

---

## Phase 7 — OS-Level MCP Servers (extends existing `ai-platform/`)

**Goal:** define the capability surface the agents act through. Each skill describes an MCP server the agent can call as tools; together they are the OS's agent-facing API.

**Deliverables**
1. **Skill** `skills/ai-platform/mcp-server-process-and-service/SKILL.md` — MCP tools for listing/starting/stopping processes and system/user units, with capability gates
2. **Skill** `skills/ai-platform/mcp-server-window-and-compositor/SKILL.md` — MCP tools for listing windows, focusing, moving, resizing, workspace operations, via a compositor-provided IPC socket
3. **Skill** `skills/ai-platform/mcp-server-filesystem-metadata/SKILL.md` — beyond file I/O: extended attributes / tags, quicklook-equivalent previews, full-text search, recent items; why this matters for a macOS-feeling OS
4. **Skill** `skills/ai-platform/mcp-server-audio-and-network/SKILL.md` — PipeWire graph control + NetworkManager/iwd control wrapped as MCP tools
5. **Skill** `skills/ai-platform/mcp-server-desktop-secrets/SKILL.md` — libsecret / Secret Service API bridge, per-agent audit log, unlock policy, never returning raw secrets to the agent
6. **Skill** `skills/ai-platform/mcp-server-package-and-update/SKILL.md` — invoke the atomic updater safely, dry-run, rollback, signature verification surfaced to the agent
7. Register all (extend existing category)

**Validation**
- Every skill follows the pattern from [skills/ai-platform/mcp-server-development/SKILL.md](../skills/ai-platform/mcp-server-development/SKILL.md) (the existing parent skill) — they are *applications* of it, not duplicates
- Each skill names the exact MCP tool schemas (`tools/call` definitions) it expects
- Each skill names the *capability* it grants and the audit events it must emit — this is the security boundary between agent and OS
- No overlap with `skills/integrations/` — those are SaaS MCPs, these are OS MCPs. Spot-check.

**Depends on:** Phase 0. Benefits from Phases 3 (compositor IPC), 4 (sandbox model), 5 (updater) but can be drafted in parallel.

**Size:** M (6 skills)

---

## Phase 8 — Apple Silicon Portability

**Goal:** ensure the OS ports cleanly to Mac mini / Mac Studio when you buy one, without rewriting the stack.

**Deliverables**
1. **Agent** `agents/software-company/os-engineering/asahi-porting-expert.md` — the Apple Silicon bring-up path: Asahi kernel tree, m1n1 bootloader, dart IOMMU, AGX GPU driver (current status), audio codec support matrix by Mac model, power management state of play, what's still broken and what workarounds exist
2. **Skill** `skills/os-linux-platform/arch-portability-and-hal-boundary/SKILL.md` — how to structure the L2/L3/L4 Rust code so x86_64 → aarch64 is a rebuild: HAL crate design, cargo workspace structure, arch-gated modules, CI cross-compilation matrix, endianness/pointer-size landmines (none on these two archs but the pattern applies)
3. Register both

**Validation**
- `asahi-porting-expert` cites specific current Asahi status (as of authoring date — this skill needs a refresh cadence noted in its description)
- `arch-portability-and-hal-boundary` includes a concrete cargo workspace layout example with a HAL crate
- The agent explicitly identifies which Mac models are currently viable targets and which aren't

**Depends on:** Phase 0. Can be authored last since Apple Silicon port is OS Phase 6 (the latest OS build phase).

**Size:** S (1 agent + 1 skill)

---

## Phase 9 — Integration & Final Registration Pass

**Goal:** make sure the kit's index files, routing, and discovery all reflect the new content coherently. No new skills authored in this phase — just glue.

**Deliverables**
1. [CLAUDE.md](../CLAUDE.md) — Available Content table updated: new Software-company agent count (60 → 65), new skills categories with counts, new total skill count
2. [CLAUDE.md](../CLAUDE.md) — Repository Structure tree reflects all 5 new `skills/os-*` folders and the `os-engineering/` agent folder
3. [agents/software-company/software-cto.md](../agents/software-company/software-cto.md) — routing table clearly lists when to delegate to each of the 5 new agents
4. [agents/board/company-coo.md](../agents/board/company-coo.md) — holding-company routing mentions the os-engineering division
5. [skills/_studio/generate-claude-md/SKILL.md](../skills/_studio/generate-claude-md/SKILL.md) — all 31 new skills listed, organized under a new "OS / Systems Userland" group, with clear trigger descriptions so it picks them when a new project description mentions building an OS
6. (Optional) `commands/specialized/os-build.md` — a new slash command `/os-build` that initializes a project with all the right skill `@` imports for starting OS work. Modeled after existing commands in [commands/specialized/](../commands/specialized/).

**Validation (end-to-end)**
- In a fresh test project, run `/generate-claude-md` with the prompt "I'm building a custom agentic OS from scratch with a Linux kernel base, targeting my laptop first and Apple Silicon later" and verify:
  - It selects `os-userland-architect`, `linux-platform-expert`, `wayland-compositor-expert`, `immutable-distro-expert`, `asahi-porting-expert`
  - It imports skills from all 5 new `skills/os-*` categories plus the Phase 6 and 7 extensions
  - It does NOT import irrelevant skills (marketing, ecommerce, etc.)
- `company-coo` agent can be prompted "I want to build a custom OS" and routes correctly to `software-cto` → `os-userland-architect`
- Count integrity: agents/software-company now has previous count + 5 `.md` files; `skills/` now has previous count + 31 `SKILL.md` files
- No broken `@` imports in [CLAUDE.md](../CLAUDE.md) — all referenced files exist

**Depends on:** Phases 0–8

**Size:** S (updates only, no new skill content)

---

## Dependency Graph

```
Phase 0 (Foundation)
  │
  ├──► Phase 1 (Linux Platform) ──────────┐
  │        │                               │
  │        └──► Phase 2 (Boot & Firmware) ─┤
  │                                        │
  ├──► Phase 3 (Desktop Stack) ────────────┤
  │                                        │
  ├──► Phase 4 (System Services) ──────────┤
  │                                        │
  ├──► Phase 5 (Distribution) ─────────────┤
  │                                        │
  ├──► Phase 6 (Agentic OS Layer) ─────────┤
  │                                        │
  ├──► Phase 7 (OS-Level MCP Servers) ─────┤
  │                                        │
  └──► Phase 8 (Apple Silicon Port) ───────┤
                                           │
                                           ▼
                                  Phase 9 (Integration)
```

Phases 1–8 are independent of each other once Phase 0 is done — if you ever want to parallelize authoring (spawn multiple Claude sessions), you can run them in parallel without conflicts, because they touch disjoint directories. Phase 9 is the merge point.

### Recommended serial order if authoring solo
`0 → 1 → 2 → 4 → 3 → 5 → 7 → 6 → 8 → 9`

Rationale: 1 and 2 together cover "boot and hardware," which you need to validate first because hardware gaps (your laptop's Wi-Fi chip etc.) can kill the whole project. 4 before 3 because system services are more foundational than UI. 7 before 6 because MCP servers define the capability surface the agents in 6 depend on. 8 is last because it's a refinement, not a prerequisite. 9 is always last.

---

## Cross-Cutting Concerns

**Overlap-checking (do this at the start of every phase)**
- Grep the existing 1,219 skills for every keyword in the new skill's description before authoring
- Specifically check: `systems-programming-expert`, `rust-pro`, `desktop-expert`, `devops-infra-expert`, `security-architect`, `orchestration-expert`, `local-llm-expert`, `mcp-server-development`
- If any existing skill covers >50% of the proposed content, extend it instead of creating a new one

**Avoid duplicating existing content**
- Do NOT re-teach Rust basics → reference [skills/languages/rust-pro](../skills/languages/)
- Do NOT re-teach async Rust → reference [skills/languages/rust-async-patterns](../skills/languages/)
- Do NOT re-teach MCP server basics → reference [skills/ai-platform/mcp-server-development](../skills/ai-platform/mcp-server-development/)
- Do NOT re-teach local LLM basics → reference [skills/data-science-ml/local-llm-expert](../skills/data-science-ml/local-llm-expert/)
- Do NOT re-teach agent memory → reference [skills/agents-orchestration/agent-memory-systems](../skills/agents-orchestration/)

Each new skill builds on top of an existing one and says so explicitly in its own "Related skills" section.

**Authoring tone**
- Second person, imperative (`"Configure the kernel with..."` not `"One should configure..."`)
- Concrete over abstract — every skill includes at least one worked example with real commands / real config
- Token-minimal — no preamble, no "In this section we will discuss...", no redundant summaries
- Bold `**Rule:**` callouts for load-bearing decisions
- Numbered sections separated by `---`

**Freshness**
- Phase 8 (`asahi-porting-expert`) and the current-status parts of Phase 1 (`hardware-profiling-and-driver-mapping`) have "as of <date>" statements that need refreshing when used. Note this in each skill's frontmatter description.

---

## Summary Table

| Phase | Adds | New Agents | New Skills | Category | Size |
|---|---|---|---|---|---|
| 0 | Foundation | 1 | 0 | division scaffold | S |
| 1 | Linux platform | 1 | 5 | `os-linux-platform/` | M |
| 2 | Boot & firmware | 0 | 3 | `os-boot-and-firmware/` | S |
| 3 | Desktop stack | 1 | 4 | `os-desktop-stack/` | M |
| 4 | System services | 0 | 4 | `os-system-services/` | S |
| 5 | Distribution | 1 | 3 | `os-distribution/` | S |
| 6 | Agentic OS layer | 0 | 5 | extends `agents-orchestration/` | M |
| 7 | OS-level MCP | 0 | 6 | extends `ai-platform/` | M |
| 8 | Apple Silicon | 1 | 1 | extends `os-linux-platform/` | S |
| 9 | Integration | 0 | 0 | index/registration | S |
| **Total** | | **5** | **31** | | |

---

## Critical Files to Reference / Modify

**Must modify in Phases 0 and 9:**
- [CLAUDE.md](../CLAUDE.md)
- [skills/_studio/generate-claude-md/SKILL.md](../skills/_studio/generate-claude-md/SKILL.md)
- [agents/software-company/software-cto.md](../agents/software-company/software-cto.md)
- [agents/board/company-coo.md](../agents/board/company-coo.md)

**Must reference (existing, don't duplicate):**
- [.claude/skills/claude-kit/SKILL.md](../.claude/skills/claude-kit/SKILL.md) — the meta-skill that defines WAT format
- [skills/ai-platform/mcp-server-development/SKILL.md](../skills/ai-platform/mcp-server-development/SKILL.md) — parent for all Phase 7 skills
- [skills/data-science-ml/local-llm-expert/](../skills/data-science-ml/local-llm-expert/) — parent for Phase 6's `on-device-llm-as-system-service`
- [skills/agents-orchestration/agent-memory-systems/](../skills/agents-orchestration/) — parent for Phase 6's `agent-state-persistence-across-reboot`
- [agents/software-company/engineering/systems-programming-expert.md](../agents/software-company/engineering/systems-programming-expert.md) — reference style for new agent files
- [agents/software-company/engineering/desktop-expert.md](../agents/software-company/engineering/desktop-expert.md) — reference style

---

## Verification (for the whole kit-authoring plan)

The whole plan is complete when, in a fresh throwaway project directory, this works end-to-end:

```
/generate-claude-md

I'm building a custom agentic OS from scratch. Linux kernel base, everything
above is my own Rust code, macOS feel, portable to Apple Silicon later.
Current machine is a Windows laptop I'll profile first.
```

…and the generated `.claude/CLAUDE.md` imports all 31 new skills, the project junctions to the new agents, and an invocation of `company-coo` in that project routes correctly to `software-cto` → `os-userland-architect` for any OS question. If that works, the kit is ready and OS-code work (the *other* plan) can begin.
