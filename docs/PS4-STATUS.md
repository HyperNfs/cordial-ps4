# Cordial PS4: measured status

Updated 2026-11-09. **Experimental loader/runtime bring-up, not a working Roblox client.**

Roblox has not been mapped, initialized or launched on PS4 by this work. There
is no Roblox window, login, rendering, audio or playable experience to install.
The upstream Linux demos and feature table in this repository are not PS4 results.
This fork was identical to upstream `luohoa97/cordial` at commit
`378c83b45a9ecac0f6f6c9f58df9b3da001e0369` before this documentation update.

## Versions: do not confuse a probe pass with Roblox

| Version | Measured result | What it means |
|---|---|---|
| **01.14** | One observed PS4 launch passed all **24 diagnostic stages**; integrated stages 22–24 each completed three cycles | Last completely passing console probe. Own Android x86-64 libraries, not the Roblox client. |
| **01.15** | Both captured logs show stages **1–24 pass**, then Stage **25**, run 1 fails: value `-1`, constructor events `0` | First stdio constructor failed. Stage 26 and final integrated PASS were not reached. Exact failing operation/root cause is **unknown**. |
| **01.16** | Local-only diagnostic build; native checks, cross-link and package inspection passed | Adds exact stdio operation/result/errno diagnostics and a native injected-failure control. **No PS4 test or console upload. Not a confirmed fix.** Runtime/build work was stopped at the owner's request. |

The observed console target is PS4 firmware **11.00 with GoldHEN**. Native host
checks run in Android/Termux on AArch64; they cannot execute the official x86-64
client. These observations do not establish support for other firmware or hardware.

## What actually works on the console

01.14's 24-stage pass covers:

- Loading, relocating and executing our own Android-targeted x86-64 ELF fixtures,
  including constructors, imported native calls, unload and per-thread emulated TLS.
- Registration/resolution of 63 scalar math and 27 pthread exports, with actual
  calls and lifecycle checks. Pthread keys/destructor rearming, once initialization,
  reader/writer locks and timed condition waits were exercised.
- Realtime and monotonic condition calibration: six requested 50 ms waits measured
  **50,658–50,750 microseconds** in the recorded 01.14 run.
- Deferred loading through the production Bionic loader, then explicit once-only
  constructor invocation and ordered unload; three cycles per integrated fixture.
- Native-backed memory/string/fortify operations, guest-created threads and joins,
  TLS/errno isolation, pipe/writev/checked I/O and raw kernel clock/sleep services.
- The project-owned per-DSO finalizer registry. 01.13 had failed finalization on
  PS4; 01.14 confirmed the fix, rather than merely passing on Android.
- Real timezone/daylight/tzname object-address relocations and calendar calls.

01.15 additionally passed the expanded buffer fixture's anonymous mmap,
mprotect, msync and munmap path in three console cycles before failing Stage 25.
This is **not** validation of every VM flag or of Stage 26's file-backed mapping.

The underlying providers use native operations and explicit ABI/error translation;
missing Android libraries are not replaced by empty success stubs. The loader's
packed-relocation decoder is shared with production relocation code.

## Known failures and unverified interfaces

- **01.15 Stage 25 fails.** Its old log identifies the failing constructor, not the
  exact operation. That constructor checks standard-stream objects, getcwd,
  `/dev/null`, stat/fstat/access/fcntl, fopen, native standard FDs, pipe/fdopen,
  buffering, fortified reads and EOF/error state. None is a proven root cause yet.
  The locally built 01.16 records the operation and errno before cleanup; its
  native control deliberately fails fstat and checks diagnostic output/FD cleanup.
- The existing kernel-log message `Exit spawn failed : 0x80aa001a ELF path=exit`
  persists, including after 01.14. A clean-looking close is not a fully clean
  exit path or evidence that every test passed.
- Stdio uses real native streams behind public opaque 152-byte guest handles and
  real standard-stream pointer objects. It does **not** implement private Bionic
  FILE buffer/extension fields. Host FILE pointers are never guest-visible.
- File metadata is copied field by field into the Android x86-64 144-byte stat
  layout. Full Sony raw-stat field calibration and file-backed VM validation are
  still incomplete; Stage 26 was not reached. Native tests/mocks are not Sony proof.
- The path bridge has an explicit fixed virtual working directory, not a complete
  Android filesystem, asset namespace or sandbox. Absolute paths stay absolute.
  Unsupported flags and unimplemented operations are rejected, not silently faked.
- Thread attributes support reviewed default/detached modes, not unverified Sony
  scheduling policy/priority conversion. Native thread handles require valid lifetimes.
- Network/DNS/socket layouts and flags, signals, semaphore semantics, locale,
  variadic/binary128 formatting and other remaining libc interfaces need actual ABI
  implementations. Directly forwarding incompatible structs or long doubles is unsafe.

## Real-client loading boundary

These are **metadata/name-coverage** measurements of the pinned Android x86-64
client, not a successful load or proof that every registered name has a valid ABI.
No client binary is distributed here.

| Measurement | 01.14 historical build | Later local source through 01.16 |
|---|---:|---:|
| libc registrations | 124: 121 functions + 3 objects | 170: 163 functions + 7 objects |
| Total imported names covered, including math/libdl | 193 | 239 |
| Unresolved strong imports | 385 | 339 |
| Missing dependency libraries | 7 | 7 |
| Unresolved imported data objects, out of 23 | 20 | 16 |

The client has **578 strong + 10 weak imports**, 10 dependencies and 2,500
initializer-array entries. The metadata reader decoded **576,942 relocations**
(576,387 packed + 555 PLT); every offset, info value and signed addend matched an
independent `readelf -rW` comparison. **No relocations were applied to that client.**
The inspected image reported no TLS segment or IFUNC, not a universal client guarantee.

The seven absent providers are `libandroid.so`, `liblog.so`, `libOpenMAXAL.so`,
`libmediandk.so`, `libGLESv2.so`, `libEGL.so` and `libOpenSLES.so`. JNI/framework
initialization, real Android services, graphics, audio/media, input, assets and
network integration remain. Binary128 `fmal`/`powl` are deliberately not exported
as Sony 80-bit-long-double functions.

Deferred constructor invocation is **not a sandbox**: relocation can execute IFUNC
resolvers in images that contain them, and previously initialized dependencies
stay initialized. Do not run untrusted images or infer that deferred means no code
can execute.

## 01.14 package and evidence

Requested historical artifact, unchanged and not rebuilt:

- Filename: `cordial-probe-v01.14-20261109-030904.pkg`
- Size: **6,619,136 bytes**
- SHA-256: `a30637ccffb99dbb6f1e629306ffa597572d52b1912d00eb3d70764d75f2b42f`
- Content: native loader/compatibility probe and **six owned Android test libraries**;
  no Roblox APK, library or assets. SDK startup modules are also bundled.
- Expected result: all stages 1–24 and
  `PASS integrated deferred-loader/runtime/data checks; Roblox has NOT been loaded`.

The owner-requested upload is kept in a **draft GitHub release**, not a public
installer recommendation. Startup-module redistribution provenance and an exact
historical PS4 corresponding-source/notices package have not been verified.
Only repository owners/collaborators with access can see draft assets. Do not
publish or mirror that bundle until those checks are resolved. The release is
separate from upstream Linux versions and is not proof of Roblox compatibility.

[Sanitized machine-readable evidence](PS4-EVIDENCE.json) records the package and
capture hashes, version-specific results and test limits. Raw console logs,
private addresses, account data and credentials are deliberately not published.
The hash summary does not substitute for independently reproducing the run.

## Source and build availability

**This repository still contains upstream Linux Cordial source, not the standalone
PS4 native workspace used to build 01.14.** These changes publish status/evidence;
they do not import or port the local PS4 sources. Cloning this fork, using its
Linux install instructions or downloading GitHub's automatic source archive will
not reproduce the PS4 package. Do not label that automatic archive as 01.14's
corresponding source. Current local 01.16 sources are not an exact 01.14 snapshot.

The native workspace uses a project-modified Bionic loader and compatibility
providers, an OpenOrbis toolchain and Android-targeted owned fixtures. Toolchain,
third-party notices, exact source provenance and reproducible instructions must
be resolved before public binary distribution. Upstream Cordial attribution and
licenses are unchanged by this documentation-only fork update.

## Checks performed, and their limits

- Native provider tests, concurrency/error/fortify controls, parser tests and
  independent real-client metadata comparison passed for the documented batches.
- Actual production-loader native fixtures tested deferred/explicit initialization,
  contention, ordering, refcounts, invalid/stale handles, unload and failed-load
  rollback. Deliberately eager controls must fail the deferred invariant.
- Provider/parser ASan+UBSan and production native-loader UBSan passed for the
  relevant recorded batches. **Full-loader ASan was blocked** by Android tagged-heap
  runtime incompatibility; it is not claimed passed. Leak detection is unsupported;
  ASan checks used `detect_leaks=0`. These are not complete sanitizer guarantees.
- Orbis cross-link, owned-fixture architecture/APS2/16 KiB alignment checks, SFO and
  exact package extraction passed. A linked/transferred package is not execution proof.
- Local 01.16 passed full native compatibility and package/cross-link checks before
  work stopped; a fresh 01.16 native-loader UBSan run and PS4 execution were **not done**.
- This GitHub update is documentation/evidence only. The upstream Linux runtime and
  its cargo tests were **not built or run** for publication from Termux.

## Next gates — work is paused, not promised for the next build

1. Resolve source/license provenance before making the historical binary public.
2. If runtime work resumes, obtain the precise Stage 25 operation/result/errno on
   console, fix that demonstrated cause once, then retest the full accumulated gate.
3. Complete real providers needed for actual-client deferred mapping, dependency
   binding and relocation. Do not count empty libraries or unresolved symbols as success.
4. Only after that, attempt controlled constructors and JNI with measured evidence.
5. Implement graphics/services/input/audio/assets/network until a real launcher,
   login and playable experience are observed. None is currently demonstrated on PS4.

No estimated next-build Roblox launch, anti-cheat/integrity bypass, client patching
or redistribution of Roblox material is part of this status report.
