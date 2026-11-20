# Routing — which chain runs your task

This file is the routing reference for this kit. It mirrors, exactly, the
router that decides which skills run for a task: the decision order, the six
chains with their skills in order, the gates each chain must pass, and the
skills each chain never uses. If you build your own router, copy this file's
decision order and chains — do not invent a different one.

One rule sits above all routing: the router never grants authorisation.
Every route returns `authorization_granted: false`. You — the human running
the agent — check scope and authorisation before any work starts.

## How routing decides

The router asks the task one question at a time, in this order. The first
match wins. A task that matches more than one pattern takes the earliest
one — that order is the whole point, so keep it.

1. **Make.fun?** The task mentions make.fun (`make.fun` or `makefun`).
   Yes → the make.fun chain. Make.fun is a **build bounty, not a bug
   bounty**: the agent builds software, submits a repo with the transcripts
   that wrote it, and claims USDC. The bug-hunt skills are never used here.
2. **Classic bug bounty?** The task mentions a bug bounty, HackerOne,
   Bugcrowd, Immunefi, YesWeHack, hunting a bug, an EVM / smart-contract /
   Solidity audit, finding a bug, or reproducing a proof of concept.
   Yes → the classic bounty chain. The general orchestrator
   (`ultimate-orchestrator`) never routes bounty work.
3. **Roblox?** Mentions Roblox, Luau, or Rojo → the Roblox chain.
4. **Open-world 3D game?** Mentions open world, GTA, city, terrain, or
   streaming → the open-world 3D chain.
5. **Assets?** Mentions asset, Blender, 3D model, mesh, texture, material,
   PBR, rigging, or animation → the asset pipeline chain.
6. **Security review?** Mentions a security review, security audit, threat
   model, OWASP, or a security assessment → the defensive security review
   chain.
7. **None of the above?** The general route: `ultimate-orchestrator` first,
   then the matching specialists, ending with `production-qa`, at most six
   skills.

Two functions do this work. `route(task)` returns a short suggestion list:
for make.fun it returns `makefun` then `makefun_recording_seal`; for a
classic bounty it returns the six-skill bounty chain; otherwise it returns
`ultimate-orchestrator` plus the matching specialists. `route_chain(task)`
returns the full playbook: the chain, the gates, and the never-list. Use
`route_chain` when you need the whole chain; use `route` when you only need
the first skills to load.

Load a skill before the work it covers, not after. Load a sub-skill as
`parent:sub-id` — for example `evm-audit-skills:evm-audit-general` or
`claude-cyber-security-skills:js-analysis`. Where two skills share one id,
the bare id resolves to the top-level one; use the `parent:sub-id` form when
you mean the sub-skill. The full index of all 324 skills is in
`agent-kit/SKILL-INDEX.md`.

## Chain 1 — Classic bug bounty (`playbook-bounty`)

The only chain that hunts. Nothing else in this kit does bounty work.

**Chain, in order:**

1. `bb-methodology`
2. `bug-bounty-null200ok`
3. `evm-audit-skills`
4. `claude-cyber-security-skills`
5. `bugbountyrules`
6. `triage-validation`
7. `variant-analysis`
8. `vulnerability-triage-brocards`
9. `trailmark-finding-triage`
10. `bugcrowd-reporting`

Sub-skill rules: in `evm-audit-skills`, always load
`evm-audit-skills:evm-audit-general` and
`evm-audit-skills:evm-audit-precision-math` first, then add only the
sub-skills the contract needs (`evm-audit-erc20`, `evm-audit-erc721`,
`evm-audit-defi-amm`, `evm-audit-defi-lending`, `evm-audit-erc4626`,
`evm-audit-proxies`, `evm-audit-signatures`, `evm-audit-oracles`,
`evm-audit-bridges`). In `claude-cyber-security-skills`, pick sub-skills by
vulnerability class (`js-analysis`, `xss`, `sqli`, `ssrf`, `idor-hunting`,
`auth-bypass`). The enrichment skills (steps 7–9) run only after a real bug
is found — they extend a finding, they never replace the chain.

**Gates, in order:**

- Scope is in-scope and authorisation is written, before any hunting.
- `triage-validation` (7-question gate plus 7 brocards; one wrong answer
  kills the finding).
- QA verifies independently (reproduce the proof of concept).
- Audit.
- Submit only after the owner's approval, unless the owner explicitly
  ordered that submit without approval.

**Never in this chain:** `ultimate-orchestrator`, `makefun`,
`makefun_recording_seal`.

## Chain 2 — Make.fun, build bounty (`playbook-makefun`)

Make.fun is not a bug bounty. The agent builds software; the bug-hunt chain
is never used here — unless the make.fun bounty itself asks for an audit,
and then the audit skills run as **build** skills, not hunt skills.

**Chain, in order:**

1. `makefun`
2. `makefun_recording_seal`
3. `ultimate-orchestrator`
4. `game-idea-to-gdd`
5. `aaa-open-world`
6. `build-hybrid-game-assets`
7. `blender-game-3d`
8. `test-playable-web-games`
9. `ship-web-games`

Steps 3–9 are the default build path for a game entry. For a different kind
of entry, swap the build skills for that entry's chain (Roblox → Chain 4,
assets → Chain 5) — the first two skills and the gates below do not change.
The pipeline inside the chain is: Analysis → Architect → Coding → QA →
Audit → Submit → Status Tracker.

**Gates, in order:**

- Read the make.fun skill in full first.
- The recording runs before any build work; build in the top-level chat,
  with the first message as the build prompt.
- Design review by the owner before the build.
- Preview to the owner before the submit.
- The seal is done by Coding as its last step; the lead stays completely
  silent during the recording.
- Build skills follow the entry type (open-world 3D / Roblox /
  asset-pipeline); the hunt chain is never used.
- Submit only after the owner's approval, unless the owner explicitly
  ordered that submit without approval.

**Never in this chain:** `bb-methodology`, `bug-bounty-null200ok`,
`triage-validation`.

## Chain 3 — Open-world 3D game (`playbook-openworld-3d`)

**Chain, in order:**

1. `ultimate-orchestrator`
2. `game-idea-to-gdd`
3. `aaa-open-world`
4. `openworld-streaming`
5. `urban-road-traffic`
6. `vehicle-simulation`
7. `tune-enemy-ai`
8. `design-game-encounters`
9. `build-game-inventory`
10. `build-hybrid-game-assets`
11. `blender-game-3d`
12. `godot-3d-essentials`
13. `godot-gdscript`
14. `procedural-gen`
15. `level-design`
16. `optimize-threejs-games`
17. `threejs-performance`
18. `test-playable-web-games`
19. `ship-web-games`

**Gates, in order:**

- Design review by the owner before the build.
- Playtest plus preview to the owner before the submit.
- Characters precise and human (head-to-body about 1:7.5).
- QA plus Audit.

**Conflicts:** `author-game-levels` (flat world) is never chained with
`openworld-streaming` (chunked world) in one build.
`optimize-threejs-games` is the workflow; `threejs-performance` is
reference only — do not merge the two.

## Chain 4 — Roblox (`playbook-roblox`)

**Chain, in order:**

1. `roblox-production`
2. `roblox-toolchain`
3. `roblox-server-security`
4. `roblox-data-economy`
5. `roblox-world-ui`
6. `roblox-luau`
7. `roblox-networking`
8. `roblox-datastores`
9. `roblox-characters`
10. `roblox-ui`
11. `roblox-studio-workflow`
12. `test-playable-web-games`

**Gates, in order:**

- Server authority (no client-trusted rewards).
- Rojo build passes.
- Studio playtest — blocked on Linux without Roblox Studio; record NOT
  RUN, never claim it passed.

## Chain 5 — Asset pipeline, Blender → Godot / Three.js (`playbook-asset-pipeline`)

**Chain, in order:**

1. `build-hybrid-game-assets`
2. `blender-game-3d`
3. `threejs-asset-pipeline`
4. `threejs-materials`
5. `build-rigged-game-assets`
6. `build-game-audio-feedback`
7. `create-game-assets`
8. `threejs-gltf-loading`
9. `threejs-materials-lighting`
10. `optimize-threejs-games`
11. `test-playable-web-games`

**Gates, in order:**

- Licence manifest plus provenance for every asset.
- Import test in the target engine (Blender 4.0.2 / Godot 4.7.2).
- Krita, GIMP, and Inkscape are blocked on Linux (a `libpoppler134`
  conflict) — never claim work was done in them.

Two AI paths, and their claims are never mixed: no paid API (local ComfyUI
/ TripoSR, the default) versus paid Meshy (`meshy-image-to-glb`, needs a
MESHY_KEY plus the owner's explicit approval).

## Chain 6 — Authorised security review, defensive (`playbook-security-review`)

A security review of your own code, or a target with written authorisation,
and no bounty attached. This is not a bounty hunt: it never routes bounty
work and never uses offensive bounty skills as substitutes. If the target
turns out to need a bounty hunt, move to Chain 1 with that program's scope
and authorisation.

**Chain, in order:**

1. `authorized-security-review`
2. `cicd-security`
3. `triage-validation`
4. `security-arsenal`
5. `owasp-audit`
6. `api-audit`
7. `semgrep`
8. `codeql`
9. `sarif-parsing`
10. `yara-rule-authoring`
11. `siem-detection`
12. `incident-triage`

**Gates, in order:**

- Scope plus written authorisation before anything else.
- Defensive only (authorised and in-scope).
- Findings go through `triage-validation`; a tool that fails gives verdict
  UNKNOWN, never PASS.

**Never in this chain:** `ultimate-orchestrator`.

## What the router never does

- It never routes a classic bounty through `ultimate-orchestrator`.
- It never routes a make.fun task through the bug-hunt chain.
- It never grants authorisation. Scope and authorisation are checked by
  the human running the agent, before any work starts.
- It never executes anything. Routing is a suggestion list; the host loads
  the skills and runs the work.

## Name aliases (V3 ids that are not directories)

Some V3 skill ids live as sections inside another skill's file, not as
their own directory. Load the host skill and read the named section.

- `game-design-systems` → `game-idea-to-gdd`
- `production-engineering` → `ship-web-games`
- `production-qa` → `test-playable-web-games`
- `save-persistence` → `build-game-inventory`
- `mission-dialogue` → `design-game-encounters`
- `npc-crowd-ai` → `tune-enemy-ai`
- `rendering-performance` → `optimize-threejs-games`
- `asset-2d-ui`, `asset-local-ai`, `game-asset-pipeline` →
  `build-hybrid-game-assets` (one combined heading)
- `asset-license-optimization` → `threejs-asset-pipeline`
- `asset-3d-procedural` → `blender-game-3d`
- `asset-audio` → `build-game-audio-feedback`
- `asset-rig-animation` → `build-rigged-game-assets`
- `asset-texture-material` → `threejs-materials`
- `security-dependency-secrets` → `cicd-security`
- `security-static-analysis`, `security-verification-report` →
  `triage-validation` (one combined heading)
- `security-threat-model` → `bb-methodology`
- `security-smart-contract` → canonical `evm-audit-master`, `web3-audit`
- `security-web-api-review` → canonical `web2-vuln-classes`,
  `authorized-security-review`

## How to use this file

Copy the decision order and the chain for your task into your own notes, in
your own words, in your own session. Run the chain once on a small task
before you trust it on a big one. If a task matches two patterns, the
earlier decision wins — that is the rule, not an accident. The full
workflow around these chains — the rules that never bend, the recording and
seal rules, the team shape, and the stops — lives in
`agent-kit/WORKFLOW.md`.
