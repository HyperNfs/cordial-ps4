# Cordial PS4

Experimental PS4 loader/runtime bring-up, based on
[upstream Cordial](https://github.com/luohoa97/cordial).
**Roblox does not run on PS4 yet.** This is a diagnostic project, not a playable
client or a PS4 version of upstream's Linux launcher.

## Current status

Observed console target: **PS4 firmware 11.00 with GoldHEN**. Other environments
are not established by these results. Updated 2026-11-09.

| Version | Result |
|---|---|
| **01.14** | One observed PS4 launch passed all **24 diagnostic stages**, using our own Android x86-64 test libraries—not Roblox. |
| **01.15** | Passed stages 1–24, then **failed Stage 25** in the stdio constructor. Exact failing operation/root cause unknown. Stage 26 was not reached. |
| **01.16** | Local diagnostic build only. Native/package checks passed; **not uploaded to the console or PS4-tested**, and not a confirmed fix. |

Runtime/build work is **paused** at the owner's request. Roblox has not been
mapped, initialized or launched on PS4; no login, rendering or gameplay is demonstrated.

## What works—and what does not

Own test libraries exercise real loading/relocation, explicit constructors and
unload, native-backed math/pthread operations, per-DSO finalization and calendar/data
imports. These probe results are **not** full Android ABI or Roblox compatibility.

Later local sources still lack **339 strong imports, seven dependency libraries
and 16 imported data objects**. Android/JNI, graphics, audio, input, assets and
network integration remain. Stdio/file calibration and an exit-path error are unresolved.

See the [measured PS4 status and evidence](docs/PS4-STATUS.md) for exact version
coverage, test limits and known failures. Upstream Linux demos/results do not apply.

## Beta packages

| Draft beta | PS4 result |
|---|---|
| [**01.14 beta**](https://github.com/HyperNfs/cordial-ps4/releases/tag/untagged-4efb0e0fda80940732ca) | Last fully passing probe: all 24 stages. Not Roblox. |
| [**01.15 beta**](https://github.com/HyperNfs/cordial-ps4/releases/tag/untagged-36ebe6044539d4e86918) | **Known Stage 25 failure**; not an overall pass or a confirmed fix. |

Each contains the unchanged historical `.pkg`, checksum, sanitized evidence and
partial notices. **Sign in with repository write access; these are not public.**
Release assets are downloaded back and verified by size and SHA-256.

Publication is pending bundled SDK-module redistribution and exact historical
source/notices verification. This is not a Roblox installer. No Roblox binary or
assets are distributed here.

## Source and next steps

**This fork currently contains upstream Linux source, not the standalone PS4
workspace.** Its automatic source archive does not reproduce the PS4 package;
the Linux build/install instructions are not PS4 instructions.

[Next steps and completion criteria](docs/PS4-STATUS.md#next-steps): source/build
reproducibility, release provenance, Stage 25 diagnostics, full file/VM validation,
actual-client mapping, constructors/JNI, then user-visible runtime bring-up.
This is a plan, not work already completed or a next-build launch promise.

## Credits and license

Fork of [luohoa97/cordial](https://github.com/luohoa97/cordial).
[Original Linux README and credits](https://github.com/luohoa97/cordial/blob/378c83b45a9ecac0f6f6c9f58df9b3da001e0369/README.md).
Upstream source remains **GPL-3.0-or-later**, with its documented protocol exception;
see [LICENSE](LICENSE) and [third-party notices](THIRD-PARTY-NOTICES.md).
That does not establish redistribution clearance for the separate PS4 package.
Development/documentation are AI-assisted. Not affiliated with or endorsed by
Roblox Corporation; no client patching or integrity bypass is part of this project.
