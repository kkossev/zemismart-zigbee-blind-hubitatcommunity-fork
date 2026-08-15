# Zemismart Zigbee Blind

Hubitat Elevation driver for Tuya **TS0601**, **TS130F** and **TS030F** Zigbee
curtain, blind and shade motors — Zemismart, Moes, Lonsonho and many
white-label variants. Originally authored by **Amos Yuen**; maintained by
**kkossev** and contributors under Apache 2.0.

The driver is maintained in the
[HubitatCommunity/zemismart-zigbee-blind](https://github.com/HubitatCommunity/zemismart-zigbee-blind)
repository. Community thread:
[Release — Zemismart Zigbee Blind driver](https://community.hubitat.com/t/release-zemismart-zigbee-blind-driver/67525).

Current version: **3.6.1** (2026-04-01)

## Features

- Open, close, stop and set position (0–100 %).
- Step open and step close with configurable step amount.
- Pushable-button facade (1 = Open, 2 = Close, 3 = Stop, 4 = Step Open,
  5 = Step Close).
- Motor direction and mode (lift / tilt) control where supported by the
  device firmware.
- Speed control for supported models.
- Battery percentage reporting for supported models.
- Device health monitoring via `healthStatus` and ping round-trip time (`rtt`).
- Alexa-compatible `SwitchLevel` and `Switch` capabilities.
- Per-model automatic defaults set on Configure (position inversion, mixed
  DP2 reporting, open/close substitution).
- `calibrate` command for limit and parameter writes on selected models. The
  command menu is not filtered by detected model, so use only the entries
  documented for your device family.
- Tuya time synchronisation.

## Prerequisites

Before pairing, use the physical device or its supplied remote to set the
motor's fully-open and fully-closed travel limits. The exact procedure is
model-specific; follow the instructions supplied with the motor. Most models
will not respond reliably or report position until both limits are set.

The driver provides limit commands for ZM85EL and ZM16B, plus calibration
controls for the Moes Touch Curtain Switch and TS130F modules via the
`calibrate` command. It does not currently provide ZM25TQ limit-setting
commands. ZM25TQ also does not support the driver's speed command; an early
community report found that attempting to set speed could require resetting
and re-pairing the motor ([post 4](https://community.hubitat.com/t/-/67525/4)).

Hubitat uses **0 % = closed** and **100 % = open** for window shades. Use
**Invert position reporting** when a device reports the opposite convention.

Commercial names and enclosures are not reliable identifiers for Tuya
devices. When checking compatibility, use the exact `model` and
`manufacturer` values from the device page's **Device Data** section.

## Installation

> **Note:** The Hubitat Package Manager listing for this driver is currently
> out of date. Install or update the driver manually using the steps below.

1. In the Hubitat UI go to **Drivers Code** and click **New Driver → Import**.
2. Paste the URL below and click **Import**, then **Save**:

```
https://raw.githubusercontent.com/HubitatCommunity/zemismart-zigbee-blind/refs/heads/main/Zemismart%20Zigbee%20Blind.groovy
```

3. Open the device page for your blind/curtain device.
4. Select `ZemiSmart Zigbee Blind` as the driver and save.
5. Click **Configure** to load the correct defaults for your model.

If the driver was not installed before pairing, re-pair the device without
deleting it first so Hubitat re-runs driver selection.

## Supported device families

"Not tested" means the fingerprint is present but no community test report exists yet. If your device pairs automatically but does not behave correctly, click **Configure** and report results in the [community thread](https://community.hubitat.com/t/release-zemismart-zigbee-blind-driver/67525).

### Zemismart TS0601 motors (Tuya EF00 cluster)

| Device / commercial name | Zigbee model | Manufacturer ID(s) | Notes |
|---|---|---|---|
| Zemismart AM43 roller blind motor (AM43-0.45/40-ES-EZ) | TS0601 | `_TZE200_zah67ekd` | Tested; speed and mode supported; older AM43 firmware does not report battery ([posts 3](https://community.hubitat.com/t/-/67525/3), [404](https://community.hubitat.com/t/-/67525/404)) |
| Moes/Zemismart AM43 roller blind motor (AM43-0.45/40-ES-EB) | TS0601 | `_TZE200_icka1clh` | Community tested; may report a minimum position of 1 % instead of 0 % ([post 523](https://community.hubitat.com/t/-/67525/523)) |
| Zemismart AM43 roller blind motor | TS0601 | `_TZE200_rddyvrci` | Inverted open/close/stop commands |
| Zemismart M515EGBZTN blind motor | TS0601 | `_TZE200_xuzcvlku`, `_TZE200_gubdgai2` | Mixed DP2 reporting |
| Zemismart M515EGZT blind motor | TS0601 | `_TZE200_nueqqe6k` | Mixed DP2 reporting |
| Zemismart ZM85EL electric curtain robot (zm85el-2z) | TS0601 | `_TZE200_cf1sl3tj` | Click-control, border and situation-set DPs |
| Zemismart ZM25TQ tubular motor | TS0601 | `_TZE200_fzo2pocs` (application 52), `_TZE200_udank5zs` | Tested; no speed or driver-based limit setting. The driver only recognizes/logs limit DPs 103–105 |
| Zemismart ZM25RX rechargeable roller motor (ZM25RX-0.8/30) | TS0601 | `_TZE200_7eue9vhc` | |
| Zemismart ZM16B tubular motor | TS0601 | `_TZE284_3mzb0sdz` | Tested; position DP 8, target DP 9 and direction DP 11; allow about five seconds between repeated step commands ([posts 543–547](https://community.hubitat.com/t/-/67525/543)) |
| Tuya/Zemismart chain blind motor (sold as ZM85EL-1x; driver join name ZM25EL) | TS0601 | `_TZE200_68nvbio9` | Tested; use position inversion when required ([posts 244–250](https://community.hubitat.com/t/-/67525/244)) |
| Zemismart tubular motor | TS0601 | `_TZE200_4vobcgd3` | |
| Zemismart AM15 tubular roller blind motor | TS0601 | `_TZE200_iossyxra` | Community report was only partially successful: On/Off worked inconsistently despite configured limits ([posts 347–354](https://community.hubitat.com/t/-/67525/347)) |
| Zemismart AM02 tubular motor | TS0601 | `_TZE200_cxu0jkjk` | |
| Zemismart AM25 tubular motor (Am25-1/30-ES-EZ) | TS0601 | `_TZE200_nv6nxo0c` | DC 5V rechargeable |
| Zemismart ZM25EL-08/25 tubular motor | TS0601 | `_TZE200_pw7mji0l` | Tested; position, open/close and battery reporting confirmed after selecting lift/forward ([post 431](https://community.hubitat.com/t/-/67525/431)) |
| Hiladuo/Astomi/A-OK AM25 roller shade | TS0601 | `_TZE200_zuz7f94z` | Tested by multiple users; no battery reporting observed and some units later developed pairing/reliability problems ([posts 335](https://community.hubitat.com/t/-/67525/335), [345](https://community.hubitat.com/t/-/67525/345), [486](https://community.hubitat.com/t/-/67525/486)) |
| Zemismart curtain motor | TS0601 | `_TZE200_cowvfni3`, `_TYST11_cowvfni3` | Do not invert position |
| Zemismart curtain motor (not tested) | TS0601 | `_TZE200_eegnwoyw` | |
| Zemismart zigbee motor (not tested) | TS0601 | `_TZE204_r0jdjrvi` | |
| Zemismart zigbee blind (legacy pre-TS0601) | mcdj3aq | `_TYST11_wmcdj3aq` | Direction may be reversed |
| Zemismart zigbee curtain motor (legacy pre-TS0601) | owvfni3 | `_TYST11_cowvfr` | |

### Other branded TS0601 motors (Tuya EF00 cluster)

| Device / commercial name | Zigbee model | Manufacturer ID(s) | Notes |
|---|---|---|---|
| Moes Touch Curtain Switch RF (ZTS-EUR-C) | TS0601 | `_TZE200_nhyj64w2` | Calibration DP 3, backlight DP 7, reversal DP 8 |
| Blindsmart plantation shutter motor | TS0601 | `_TZE284_myikb7qz` | Tested with v3.5.1 and later; DP 101–103 remaps; open/close via setPosition ([posts 499–509](https://community.hubitat.com/t/-/67525/499)) |
| Blindsmart motorized window opener | TS0601 | `_TZE200_fzo2pocs` (application 53) | Tested; set Mode to `tilt`; Open/Close substitution is selected automatically ([posts 314–315](https://community.hubitat.com/t/-/67525/314)) |
| Tuya LY-108 Cover (Z2M TS0601_cover_6) | TS0601 | `_TZE200_cpbo62rn`, `_TZE200_libht6ua` | Not tested |
| Hiladuo/Zemismart motorised roller shade | TS0601 | `_TZE200_9p5xmj5r` | Community report: closes fully but may stop around 90 % when opening from Hubitat ([posts 527–529](https://community.hubitat.com/t/-/67525/527)) |
| Zignito tubular roller blind motor | TS0601 | `_TZE200_uzinxci0`, `_TZE200_zpzndjez` | |
| LoraTap/Zemismart ZM85EL-2x curtain robot | TS0601 | `_TZE200_nw1r9hp6` | Tested; remote required to set limits, battery reporting confirmed, and step resolution is approximately 10 % ([posts 276–296](https://community.hubitat.com/t/-/67525/276)) |
| Abalon curtain motor | TS0601 | `_TZE200_rmymn92d` | Tested with lift/reverse, but one unit mapped targets below 37 % incorrectly ([post 498](https://community.hubitat.com/t/-/67525/498)) |
| Tuya curtain motor | TS0601 | `_TZE200_r0jdjrvi` | Tested after pairing a C-8 while avoiding Zigbee 3.0 repeaters; enable position inversion if Open/Closed is reversed ([posts 377–383](https://community.hubitat.com/t/-/67525/377)) |
| Nova Digital ZBCMR-01 roller blind motor | TS0601 | `_TZE600_ogyg1y6b` | Not tested |
| Roximo curtain motor | TS0601 | `_TZE204_57hjqelq` | Not tested |
| Somgoms curtain switch | TS0601 | `_TZE200_axgvo9jh` | Not tested |
| HUARUI curtain motor | TS0601 | `_TZE200_zxxfv8wi` | Not tested |
| Novato Smart curtain track | TS0601 | `_TZE204_lh3arisb` | Not tested |
| Homeetec curtain/blind switch | TS0601 | `_TZE200_jhkttplm`, `_TZE200_5nldle7w` | Not tested |
| Tuya Zigbee Window Pusher | TS0601 | `_TZE200_g5wdnuow` | Not tested |
| Tuya curtain/blind switch | TS0601 | `_TZE200_p6vz3wzt` | Not tested |
| Generic TS0601 blind/curtain motor (tested) | TS0601 | `_TZE200_yenbr4om`, `_TZE200_5sbebbzs`, `_TZE200_xaabybja`, `_TZE200_hsgrhjpf`, `_TZE200_ergbiejo`, `_TZE200_wmcdj3aq`, `_TZE200_3i3exuay`, `_TZE200_5zbp6j0u`, `_TZE200_nogaemzt` | |
| Generic TS0601 blind/curtain motor (not tested) | TS0601 | `_TZE200_eevqq1uv`, `_TZE200_fdtjuw7u`, `_TZE200_gaj531w3`, `_TZE200_bqcqqjpb`, `_TZE200_3ylew7b4`, `_TZE200_llm0epxg`, `_TZE200_n1aauwb4`, `_TZE200_xu4a5rhj`, `_TZE200_feolm6rk`, `_TZE200_nkoabg8w`, `_TZE200_pk0sfzvr`, `_TZE204_guvc7pdy`, `_TZE200_ol5jlkkr`, `_TZE200_b2u1drdv`, `_TZE200_zvo63cmo`, `_TZE200_g5xqosu7`, `_TZE204_g5xqosu7`, `_TZE284_fzo2pocs`, `_TZE200_p2qzzazi`, `_TZE200_clm4gdw4`, `_TZE200_2vfxweng`, `_TZE204_2rvvqjoa`, `_TZE200_zyrdrmno` | |

### TS130F curtain modules (ZCL WindowCovering cluster 0x0102)

Version 3.6.1 supports Open, Close, Stop and calibration for TS130F modules,
but `setPosition`/`setLevel` is not implemented on the TS130F command path.
For modules that need estimated positioning, consider the dedicated
[Zigbee Window Shade/Blind (Timed) driver](https://community.hubitat.com/t/release-zigbee-window-shade-driver-with-positioning-for-modules-motors-that-dont-report-position/162203)
referenced by the maintainer in
[post 540](https://community.hubitat.com/t/-/67525/540).

| Device / commercial name | Manufacturer ID | Notes |
|---|---|---|
| QS-Zigbee-CP03 curtain module | `_TZ3000_zirycpws` | Tested |
| Moes MS-108ZR curtain switch module | `_TZ3000_1dd0d5yi` | Tested |
| Zemismart Smart curtain/shutter switch | `_TZ3000_74hsp7qy` | Not tested |
| Danor SK-Z802C-US curtain module | `_TZ3000_8h7wgocw` | Not tested |
| LoraTap SC500ZB curtain module | `_TZ3000_e3vhyirx` | Tested for Open/Close and calibration; position control is not supported in v3.6.1 ([posts 475–477](https://community.hubitat.com/t/-/67525/475)) |
| Nous B4Z curtain switch | `_TZ3000_yruungrl` | Open/close and calibration only in v3.6.1; a community patch for setPosition was reported but is not merged ([post 497](https://community.hubitat.com/t/-/67525/497)) |
| Nous L12Z curtain module | `_TZ3000_jwv3cwak` | Not tested |
| Generic curtain module | `_TZ3210_dwytrmda`, `_TZ3000_4uuaja4a`, `_TZ3000_fccpjz5z`, `_TZ3000_vd43bbfq`, `_TZ3000_ke7pzj5d` | Not tested |

### TS030F (Tuya EF00 cluster)

| Device / commercial name | Manufacturer ID | Notes |
|---|---|---|
| Lidl HG09648 Livarno roller blinds | `_TZB000_42ha4rsc` | Not tested |

### Community-tested devices without a driver fingerprint

These devices were reported working after manually selecting this driver, but
their fingerprints are not present in v3.6.1. They will not necessarily select
the driver automatically during pairing.

| Device / commercial name | Zigbee model | Manufacturer ID | Notes |
|---|---|---|---|
| Moes ZC-LS02 / FNBYHFB03A002 chain blind motor | TS0601 | `_TZE284_koxaopnk` | Fully functional after enabling **Invert position reporting**; re-pairing was needed in one report ([posts 554–561](https://community.hubitat.com/t/-/67525/554)) |
| Generic Zemismart blind motor | TS0601 | `_TZE284_ettzhnwk` | Reported working; no commercial model information supplied ([post 526](https://community.hubitat.com/t/-/67525/526)) |

### Known devices not supported by this driver

| Device | Zigbee identity | Status / alternative |
|---|---|---|
| LoraTap SC420ZB dual-curtain switch | TS130F / `_TZ3000_esynmmox` | Dual-gang child devices are not supported. Use the dedicated community driver linked in [post 533](https://community.hubitat.com/t/-/67525/533) |
| A-OK AM25-1/25-ES-EZ USB-C revision | `TRZB3` | Use Hubitat's Generic Zigbee Shade driver; unlike the older TS0601 revision, it reports battery ([post 486](https://community.hubitat.com/t/-/67525/486)) |
| Generic TS0301 motor | TS0301 / `_TZE200_cirjrpxe` | Basic Open/Close/Stop worked with a generic shade driver; not fingerprinted or verified with this driver ([posts 552–553](https://community.hubitat.com/t/-/67525/552)) |
| Tuya motor | TS0601 / `_TZE200_mng9vgqa` | Missing fingerprint and no successful control was reported ([post 513](https://community.hubitat.com/t/-/67525/513)) |
| Matter-over-Thread motors | Matter device | This Zigbee driver does not apply; pair using an appropriate Hubitat Matter shade driver |

## Capabilities and attributes

### Standard capabilities

- `Actuator`
- `WindowShade` — `open()`, `close()`, `stopPositionChange()`,
  `startPositionChange()`, `setPosition(position)`
- `Switch` / `SwitchLevel` — Alexa compatibility
- `PushableButton`
- `Battery`
- `Configuration`
- `Refresh`
- `HealthCheck`

### Custom attributes

| Attribute | Type | Description |
|---|---|---|
| `targetPosition` | number | Last commanded position (0–100 %); the current driver sends `?` after a Stop command |
| `speed` | number | Motor speed (0–100), where supported |
| `healthStatus` | enum | `unknown`, `online`, or `offline` |
| `rtt` | number | Ping round-trip time in milliseconds; the current driver sends `timeout` when no response is received |

### Commands

| Command | Parameters | Description |
|---|---|---|
| `open` | — | Move to fully open |
| `close` | — | Move to fully closed |
| `stopPositionChange` | — | Stop movement |
| `setPosition` | position (0–100) | Move to target position |
| `stepOpen` | step (optional) | Move toward open by step amount |
| `stepClose` | step (optional) | Move toward closed by step amount |
| `push` | button (1–5) | 1 Open · 2 Close · 3 Stop · 4 Step Open · 5 Step Close |
| `setSpeed` | speed (0–100) | Set motor speed (models that support it) |
| `calibrate` | cmd, val | Write a limit or parameter; options are shown for all models, so select only an operation supported by your device |
| `configure` | — | Load all defaults for the detected model |
| `refresh` | — | Query all Tuya DPs on TS0601 devices; request On/Off state on other models |
| `ping` | — | Send a ping and record round-trip time |

## Preferences

| Preference | Description |
|---|---|
| Enable descriptionText logging | Log state changes at info level |
| Enable debug logging | Log raw Zigbee messages and parse detail |
| Mode | `lift` — motor runs until stopped; `tilt` — short press releases on button up |
| Direction | `forward` or `reverse` motor direction |
| Max Closed Position | Position threshold at or below which `windowShade` reports `closed` (default 1) |
| Min Open Position | Position threshold at or above which `windowShade` reports `open` (default 99) |
| Default Step Amount | Step size used when `stepOpen`/`stepClose` is called without a value (default 10) |
| Show Advanced options | Reveal the model-specific reporting, timeout and health-check settings below |
| *(Advanced)* Invert position reporting | Flip 0↔100 for devices that report position backwards |
| *(Advanced)* Ignore first Position report | Enable for models that send the target position before the current position on DP 2 |
| *(Advanced)* Substitute Open/Close with SetPosition | Use `setPosition(0/100)` instead of open/close commands for motors that do not respond in lift mode |
| *(Advanced)* Position report timeout (ms) | How long to wait for position reports before declaring movement finished (default 15 000 ms) |
| *(Advanced)* Force TS130F model | Override detection for units whose Data section shows `endpointId: F2` |
| *(Advanced)* Healthcheck method | `Disabled`, `Activity check`, or `Periodic polling` |
| *(Advanced)* Healthcheck interval | How often to check device health (2 min – 12 h; default 4 h) |

## Known limitations

- Battery is reported only when the motor firmware sends Tuya battery DP 13.
  The capability is necessarily shown for all devices because Hubitat does not
  support dynamic capabilities. Most older AM43 and several Hiladuo/Astomi
  motors do not report battery.
- Some firmware sends no arrival message or no position reports. The driver
  uses the **Position report timeout** to settle the shade state, but it cannot
  manufacture an accurate intermediate position that the motor never reports.
- `stepOpen` and `stepClose` should currently be given an explicit step value
  when called from the device page or an app; a blank argument can fail instead
  of using the documented default.
- Do not issue `stepClose` when already at 0 %. The current implementation can
  wrap to `100 - step` (for example, a 20 % step can reopen the shade to 80 %),
  as confirmed in [posts 548–550](https://community.hubitat.com/t/-/67525/548).
- The driver supports one motor/channel per Hubitat device. It does not create
  child devices for multi-gang curtain controllers.
- One user reported that Hubitat platform 2.5.1.140 broke four blinds and that
  rolling back restored them ([post 559](https://community.hubitat.com/t/-/67525/559)).
  This is a single report, not yet a confirmed general incompatibility.

## Setup and troubleshooting

1. Set the motor's physical travel limits before pairing or testing position
   commands.
2. Confirm the exact Zigbee `model` and `manufacturer` under **Device Data**;
   the commercial product name alone is not sufficient.
3. After assigning this driver manually, save preferences and click
   **Configure** once. Configure reloads automatic defaults for the detected
   manufacturer.
4. If Open/Close does not work but Set Position does, try **Substitute
   Open/Close commands with SetPosition** or the model-specific Mode noted in
   the table.
5. If open/closed state is inverted, use **Invert position reporting**. The
   Direction preference changes the motor direction; it is not the same as
   inverting Hubitat's position convention.
6. If the shade remains opening/closing after movement stops, increase
   **Position report timeout**. This helps with delayed reports but cannot fix a
   device that never reports its position.
7. For intermittent communication, enable debug logging, repeat one command,
   and capture the complete log. Re-pairing without deleting the device can
   restore communication while preserving automations.
8. Use frequent **Periodic polling** only for diagnosis; it can drain
   battery-powered motors more quickly.

## Google Home integration

The recommended integration is
[Google Home Community](https://community.hubitat.com/t/alpha-community-maintained-google-home-integration/34957)
with these settings:

- **Device type**: `Window Shade`
- **Google Home device type**: `Curtain`
- **Device traits → Open/Close**
  - Open/Close attribute: `windowShade`
  - Open Position Command: `setPosition`

These are the mappings documented in the original driver release thread;
the remaining integration settings can retain their defaults.

The built-in Hubitat Google Home integration can alternatively expose the
driver through its `Switch`/`SwitchLevel` facade. Google Home may display it as
a dimmable light, but percentage commands work. Avoid assigning that device to
a Google Home room if room-wide light commands would also operate the blind
([posts 268–273](https://community.hubitat.com/t/-/67525/268)).

## Forum review scope

This compatibility information was reviewed against all **551 available
posts** in the community topic through visible post **561** (2026-08-05).
Deleted posts account for the difference between the two numbers. Community
reports describe particular firmware and hardware revisions; identical retail
names do not guarantee identical Zigbee behavior.
