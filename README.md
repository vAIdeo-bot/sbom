# vAIdeo.bot — Software Bill of Materials

Public, machine-readable SBOMs for [vAIdeo.bot](https://vaideo.bot) releases. Every release gets a CycloneDX 1.5 JSON covering every Rust crate, NPM package, Python dependency, ComfyUI custom node, AI model, and system tool shipped in the binary and its bundled backend.

vAIdeo.bot is a desktop AI video, image & audio production studio (sister application to [rAIdio.bot](https://raidio.bot)). Built with Tauri 2 + Svelte 5 + Rust, runs entirely locally — no cloud required.

## Status

**Pre-release.** No public build has shipped yet. This repo is set up ahead of the first release so the SBOM + hash + VirusTotal pipeline is wired and tested before users can install anything. First entries will appear under `releases/` when the first release is pushed.

## Verify your install (post-release)

Once releases exist, every one will publish the sha256 of the shipped `vaideo-bot.exe` here. Confirm the binary on your machine matches what was released:

```powershell
$tag = 'RC1'   # change to your installed version (see the About dialog)
# Point $exe at your installed vaideo-bot.exe — right-click the Start-menu
# shortcut and choose "Open file location" to find it:
$exe = 'C:\Program Files\vAIdeo.bot\vaideo-bot.exe'
$local     = (Get-FileHash $exe -Algorithm SHA256).Hash.ToLower()
$published = (Invoke-RestMethod "https://raw.githubusercontent.com/vAIdeo-bot/sbom/main/releases/$tag/SHA256SUMS").Split(' ')[0]
if ($local -eq $published) { 'OK — matches release' } else { "MISMATCH — local $local vs published $published" }
```

If the values differ, the binary on disk is not the released build — either the install is corrupted, an update is in flight, or the binary has been replaced. Re-install from the official download; if it still mismatches, report it to the security contact at **info@vaideo.bot**.

A `MISMATCH` is **not** by itself proof of compromise — an old binary may linger after an in-flight update. Reproduce on a clean install before raising a security issue.

This is a hash-publication mechanism, not a code-signing one. It proves the file you have matches what we released; it does not prove we are who we say we are. Authenticode signing with a hardware-backed cert is planned and will land alongside the existing hash file — hashes will continue to be published.

## VirusTotal monitoring

Every release exe is submitted to VirusTotal at push time and the verdict is published here as `releases/<tag>/virustotal.json`. A daily GitHub Action (`.github/workflows/virustotal_monitor.yml`) re-polls every release so retro-flags (an AV vendor adding a heuristic next month and our exe lighting up) are caught before users do. Alert tiers (silent / baseline / minor / alert / major / defender) and maintainer routing live in [`tools/virustotal_poll.py`](tools/virustotal_poll.py).

GPU-heavy ML applications routinely false-positive on heuristic engines. The published VT verdict tied to the official sha256 is one of the cheapest ways for a suspicious user to confirm they have the real binary, not a `vAIdeo.bot CRACK 2026 FREE` repack from a torrent site.

## Fetch URLs (post-release)

**GitHub release asset** (preferred for OSPO feeds):
`https://github.com/vAIdeo-bot/sbom/releases/download/<tag>/vAIdeo.bot-<tag>.cdx.json`

**Raw stable path** (for tools that pin to a directory):
`https://raw.githubusercontent.com/vAIdeo-bot/sbom/main/releases/<tag>/vAIdeo.bot-<tag>.cdx.json`

Both URLs are unauthenticated and permanent.

## What's inside each SBOM

- **Rust crates** linked into the Tauri binary — pinned by crate version from `cargo metadata`.
- **NPM packages** bundled into the Svelte frontend — pinned by the resolved version in `package-lock.json`.
- **Python dependencies** in the bundled ComfyUI backend we vendor (`torch`, `opencv-python-headless`, `gguf`, …) — pinned to exact installed versions.
- **ComfyUI core + custom nodes** — pinned to upstream release tag (ComfyUI itself) or the HEAD SHA of our [memescreamer](https://github.com/memescreamer) FOSS-source mirror.
- **AI models** distributed via our download CDN — pinned to the HuggingFace revision SHA of the `memescreamer/*` mirror that carries the actual model files. vAIdeo ships safetensors / GGUF only.
- **System tools** (ffmpeg, …) — pinned to shipped executable version with build configuration captured (e.g., the bundled ffmpeg build flags).

Each component carries an SPDX license identifier, a package URL (purl), a homepage where available, and a `vaideo:category` property indicating which of the layers above it belongs to.

## Format choice

CycloneDX 1.5 JSON. One of the two CISA minimum-elements-compliant SBOM formats (SPDX is the other); ingests natively in Dependency-Track, OWASP Dependency-Check, Chainguard, Snyk, Grype, Syft, and most modern OSPO tooling without conversion. If someone specifically asks for SPDX 2.3, the generator can emit it — open an issue.

## Repo scope

This is the **public SBOM and documentation** repo only. No source code, no app code, no binaries — those are distributed through the official installer and download CDN; the application source is maintained privately. This separation lets OSPO tooling and security auditors crawl the SBOM data without needing access to the source.

## Sister project

The [rAIdio.bot SBOM repo](https://github.com/rAIdio-bot/sbom) follows the same layout. The two apps share a lot of upstream code (mixer DSP, FX rack, audio engine) and most of their AI models, but ship as distinct binaries with distinct SBOMs.
