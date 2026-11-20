# Tools that back the skills

The skills in this kit are instructions. The tools below are the programs
those instructions call. Install the tools for the skills you plan to use
before you start the work — a skill that calls a missing tool will fail
halfway through, not at the start.

Versions in this file are the ones this kit was tested against on
20 November 2026 (Linux x86_64). Newer versions will usually work; if one
does not, the version listed here is the known-good one. Binaries are not
in this repo — they are too big and platform-specific — so install them
from the official source listed for each tool.

## Security and audit

| Tool | Tested version | Install | Used by |
| --- | --- | --- | --- |
| Semgrep | 1.186.0 | `pip install semgrep` — semgrep.dev | `semgrep`, `semgrep-rule-creator`, `semgrep-rule-variant-creator`, audit skills |
| trailmark | 0.8.0 | `pip install trailmark` | `trailmark`, `trailmark-review-gate`, `trailmark-variant-neighborhood`, `trailmark-summary`, `trailmark-structural`, `trailmark-finding-triage` |
| yara-x (`yr`) | 1.23.0 | Release binary from github.com/VirusTotal/yara-x | `yara-rule-authoring`, malware triage skills |
| Bandit | 1.9.4 | `pip install bandit` | Python security review, `c-review` family |
| Pytest | 9.2.0 | `pip install pytest` | Every skill that runs tests or a proof of concept |
| ProVerif | 2.05 | proverif.inria.fr (or the ProVerif GitHub releases) | `mermaid-to-proverif`, `writing-lean-proofs`, crypto protocol skills |
| Gitleaks | 8.30.1 | Release binary from github.com/gitleaks/gitleaks | Secret scan before any commit or push — run it every time |
| Foundry (`forge`, `cast`, `anvil`) | 1.9.0 | `curl -L https://foundry.paradigm.xyz \| bash`, then `foundryup` | `evm-audit-skills` and every EVM audit sub-skill |

CodeQL has a skill in this kit (`codeql`) but its CLI was not part of the
tested setup — install it separately from github.com/github/codeql if you
need that skill.

## Game and 3D

| Tool | Tested version | Install | Used by |
| --- | --- | --- | --- |
| Blender | 4.0.2 | `apt install blender` (Debian/Ubuntu) or blender.org | `blender-game-3d`, `build-hybrid-game-assets`, `build-rigged-game-assets` |
| Godot (headless) | 4.7.2 | Release binary from godotengine.org | All `godot-*` skills, `godot-gdscript-headless-testing` |
| ffmpeg | 8.1.2 | `apt install ffmpeg` | Video, audio, and screenshot work in game and media skills |
| git | 2.43.0 | `apt install git` | Everything — version control, pushing, submitting |

## Roblox

| Tool | Tested version | Install | Used by |
| --- | --- | --- | --- |
| Rojo | 7.7.1 | Release binary from github.com/rojo-rbx/rojo | `roblox-*` skills, Roblox builds |
| Rokit | 1.2.0 | Release binary from github.com/rojo-rbx/rokit | Roblox toolchain management |
| Luau | 0.747 | Release binary from github.com/Roblox/luau | `roblox-luau`, Luau code in Roblox skills (the binary has no `--version` flag; 0.747 is the release recorded at install time) |
| StyLua | 2.6.0 | Release binary from github.com/JohnnyMorganz/StyLua | Luau formatting |
| Selene | 0.32.0 | Release binary from github.com/Kampfkarren/selene | Luau linting |

## Art and assets

| Tool | Tested version | Install | Used by |
| --- | --- | --- | --- |
| Blockbench | 5.2.2 | AppImage from blockbench.net | `build-hybrid-game-assets`, blocky / low-poly modelling |
| Material Maker | 1.8 | Release binary from github.com/RodZill4/material-maker | Procedural materials |
| Pixelorama | 1.2.4 | Release binary from github.com/Orama-Interactive/Pixelorama | Pixel art and sprites |

## Known problems, so you do not waste an afternoon

- **Material Maker** needs a real GPU. It segfaults under software OpenGL
  (llvmpipe). Run it on a machine with graphics hardware, or skip it.
- **Krita, GIMP, Inkscape** were not installable on the tested setup: they
  demand an exact `libpoppler` version that conflicts with the one shipped
  there. Do not force a system-library downgrade to get them.
- **`rokit add`** can fail behind a proxy that intercepts TLS (the Rust TLS
  stack does not trust the proxy CA). Download the toolchain manually if
  that happens.
- **Roblox Studio** is Windows/macOS only and needs a login. There is no
  Linux binary.

## Quick check after installing

Run each tool's version command and compare it with the table above:
`blender --version`, `godot --version`, `forge --version`,
`semgrep --version`, `gitleaks version`, `rojo --version`,
`stylua --version`, `selene --version`, `yr --version`,
`trailmark --version`, `bandit --version`, `pytest --version`.
If a tool is not on your `PATH`, add its folder to `PATH` first — that is
the most common reason a skill fails to find its tool.
