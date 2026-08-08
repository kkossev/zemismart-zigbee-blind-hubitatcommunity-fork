# Zemismart Zigbee Blind — Driver Guide for AI Agents

Hubitat Elevation Zigbee driver (Groovy) for Tuya **TS0601 / TS130F curtain,
blind and shade motors** (Zemismart AM43/ZM25TQ/ZM85EL/ZM16B/M515, Moes,
Lonsonho and many white-label variants). Original author **Amos Yuen**;
maintained by **kkossev** and contributors. Apache 2.0.

Community thread: https://community.hubitat.com/t/release-zemismart-zigbee-blind-driver/67525

## Scope and precedence

**General conventions live in the main repo's guide:**
[`C:\work\Hubitat\AGENTS.md`](../Hubitat/AGENTS.md). Everything there about
principles, golden rules, Hubitat sandbox semantics (§7), code conventions
(§8), document conventions (§9) and hard boundaries (§10) applies here
unchanged. This file only records what is **different or specific** to this
project. Where the two disagree, this file wins.

**Differences from `kkossev/Hubitat`:**

- **Separate repository.** This folder is a git clone of
  `kkossev/zemismart-zigbee-blind-hubitatcommunity-fork`, a fork of
  `HubitatCommunity/zemismart-zigbee-blind`. Git commands here operate on the
  fork, never on `kkossev/Hubitat`.
- **Branch is `main`** (not `development`). Upstream contributions go via PR
  to the HubitatCommunity repo. The driver's `importUrl` points at the
  **HubitatCommunity** raw URL — do not "fix" it to the fork.
- **Namespace `amosyuen`, author `Amos Yuen`** — never change to `kkossev`.
- **Architecture: LEGACY MONOLITHIC.** One self-contained file, no
  `#include`, no shared libraries, no `deviceProfilesV3`, no `custom*` hooks,
  no generated `*_lib_included` bundle. Do not apply V3/V4 patterns.
- **Work lists:** this folder uses a combined [`TODO.md`](docs/maintainer/TODO.md) (bugs +
  improvements in one file, numbered 1–22 and 1–14) instead of the main
  repo's separate `BUGS.md`/`TODO.md`. Work from those ids. The
  `_TZE200_cpbo62rn`/`_TZE200_libht6ua` (LY-108) family has its own verified
  plan: [`LY108_CPBO62RN_FIXES_PLAN.md`](docs/maintainer/LY108_CPBO62RN_FIXES_PLAN.md).

## Files

| File | Role |
|---|---|
| `Zemismart Zigbee Blind.groovy` | **The driver** — the only file to edit. v3.6.1 (2026-04-01), ~1620 lines. Hub driver name `ZemiSmart Zigbee Blind`. |
| `README.md` | User-facing; capabilities list and prerequisites. Product links/installation sections are TODO. |
| `TODO.md` | Combined reviewed bug list + improvements (line numbers pinned to v3.6.1). In `docs/maintainer/`. |
| `LY108_CPBO62RN_FIXES_PLAN.md` | Verified 6-fix plan for the LY-108 family. In `docs/maintainer/`. |
| `TS130F_IMPROVEMENTS.md` | TS130F-focused analysis (2026-08-08). In `docs/maintainer/`. |
| `BUG2_MULTI_DP_PARSING_PLAN.md`, `BUG4_ISWITHINONE_RACE_PLAN.md`, `IMPROVEMENT1_DEVICE_PROFILES_PLAN.md` | Implementation plans for TODO items (2026-08-08). In `docs/maintainer/`. |
| `packageManifest.json` | HPM manifest. Update only at a user-declared release. |

## Model detection (the helper block, lines 259-321 @ v3.6.1)

Device identity = `device.getDataValue('model')` + `('manufacturer')`. All
family logic goes through helpers — **when adding a device, check every list,
not just the fingerprints** (TODO improvement #1 plans to replace these with a
device-profiles map):

| Helper | Matches | Drives |
|---|---|---|
| `getModel()` / `isTS130F()` | model `TS130F`, or `forcedTS130F` pref (units pairing with endpointId F2) | standard-cluster 0x0102 command path; no EF00 |
| `isZM85EL()` | `_TZE200_cf1sl3tj` | click-control DP 20, situation_set DP 11, border DP 16, 15 s report timeout |
| `isAM43()` | `_TZE200_zah67ekd`, `_TZE200_icka1clh` | (informational) speed DP 105, battery DP 13 |
| `isAM02()` | `_TZE200_iossyxra`, `_TZE200_cxu0jkjk` | defined, currently unused |
| `isCurtainMotor()` | `_TYST11_cowvfni3`, `_TZE200_cowvfni3`, `_TYST11_cowvfr` | do-not-invert curtain models |
| `isPlantationShutter()` | `_TZE284_myikb7qz` | DP 101/102/103 remaps, open/close via setPosition |
| `isZM16B()` | `_TZE284_3mzb0sdz` | position DP 8, target DP 9, direction DP 0x0B |
| `isMoesCoverSwitch()` | `_TZE200_nhyj64w2` | DP 3 calibration, DP 7 backlight, DP 8 reversal, DP 14 indicator |
| `isZM25TQ()` | `_TZE200_fzo2pocs`, `_TZE200_udank5zs` | limit-setting DPs 103/104/105 |
| `getDpCommandOpen/Stop/Close()` | remaps for `_TZE200_rddyvrci` (O2/S0/C1), `_TZE200_nueqqe6k` (S2/C1), cowvfni3 pair (O2/C0), TS130F (S2/C1 = ZCL Up/Stop/Down) | DP 1 / WC command values |
| `isMixedDP2reporting()` | `_TZE200_xuzcvlku`, `_TZE200_nueqqe6k`, `_TZE200_5sbebbzs`, `_TZE200_gubdgai2` | DP 2 = target *then* stream of current positions |
| `isInvertedPositionReporting()` | 13 manufacturers (see line 313) | default for `invertPosition` pref |
| `isOpenCloseSubstituted()` | 6 manufacturers (line 319) | default for `substituteOpenClose` pref |

The three `is*Reporting`/`Substituted` lists seed **preference defaults** in
`configure()` only — runtime behavior reads `settings.mixedDP2reporting`,
`settings.invertPosition`, `settings.substituteOpenClose`.

## Message flow

`parse()` → every message: `checkDriverVersion()`, rx counters,
`unscheduleCommandTimeoutCheck()`, `setHealthStatusOnline()`. Then:

- **Cluster 0xEF00** (`CLUSTER_TUYA`): commands 0x01/0x02 →
  `parseSetDataResponse()` — the big per-DP `switch`. **Parses only the
  first DP record per frame** (TODO bug #2; plan in
  [`BUG2_MULTI_DP_PARSING_PLAN.md`](docs/maintainer/BUG2_MULTI_DP_PARSING_PLAN.md)).
  0x0B ACK, 0x10/0x11 MCU version, 0x24 → `processTuyaSetTime()` (Tuya time
  sync — must keep working).
- **Everything else** → `parseNonTuyaMessage()`: Basic 0x0000 attr 0x0001 =
  ping response / Tuya check-in; **0x0102 WindowCovering** = the TS130F path
  (attr 0x0008 position; 0xF000-0xF003 logged only — see
  [`TS130F_IMPROVEMENTS.md`](docs/maintainer/TS130F_IMPROVEMENTS.md)); OnOff 0x8001 indicator
  mode; ZDO 0x0013.

Key DPs (TS0601): 1 command, 2 target (or mixed current), 3 current position,
4 ZM85 mode, 5 direction, 6 arrived, 7 remote/work-state, 8/9/0x0B ZM16B
remaps (else countdown/Moes), 12 fault, 13 battery, 16 border, 20 click
control, 0x65 mode (LY-108: motor_direction! — see LY-108 plan), 0x66-0x6A
limits/speed/model-specific.

## Movement state machine

`open()`/`close()`/`setPosition()` set `state.target` + `targetPosition`
event, arm the `endOfMovement()` watchdog via `restartPositionReportTimeout()`
(pref `positionReportTimeout`, ms). Position reports drive
`updateWindowShadeMoving/Arrived()` + `updatePosition()`; `isWithinOne()`
inside `updatePosition()` has a self-comparison race (TODO bug #4; plan in
[`BUG4_ISWITHINONE_RACE_PLAN.md`](docs/maintainer/BUG4_ISWITHINONE_RACE_PLAN.md)).
`state.isTargetRcvd` disambiguates the mixedDP2 target-vs-position stream.
`windowShade` settles to open/closed/partially open using
`minOpenPosition`/`maxClosedPosition` thresholds.

## Known traps specific to this driver

- `logWarn()` is gated on `enableDebugLog` (TODO bug #1) — warnings are
  invisible in normal operation.
- `targetPosition` and `rtt` are `number` attributes that sometimes receive
  strings (`'?'`, `'timeout'`) — TODO bugs #8/#15.
- `configure()` unconditionally schedules `setMode`/`setDirection` EF00
  writes — harmful for LY-108 (flips motor direction) and pointless for
  TS130F (TODO bug #7).
- `calibrate()`'s `settableParsMap` mixes TS130F/Moes/ZM85/ZM16B entries with
  no model filtering (TODO improvement #11).
- The `command 'calibrate'` ENUM parameter deliberately has **no
  defaultValue** — never add one (Hubitat submits the defaultValue on Run,
  which once wiped a device).
- `hubitat.device.HubMultiAction`/`HubAction` sending, `singleThreaded: true`,
  health-check and ping/rtt plumbing all follow the main repo's patterns —
  see `C:\work\Hubitat\AGENTS.md` §7-§8.

## Workflow

Same as the main repo (§2 golden rules): one TODO item at a time in the
user's order; the user uploads to the hub and reports; no version/timestamp
bumps and no header history lines unless explicitly requested; analysis tasks
produce documents, not edits; anything needing real hardware is **VERIFY ON
DEVICE** and blocks.
