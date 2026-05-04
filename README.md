# Pluto RX Bridge — Beta Releases

Windows installer downloads for the **Pluto RX Bridge** — a Qt6 C++
application that adds wideband Q65 (QMAP) reception via an
**ADALM-Pluto** or **Pluto+** to a station already running a real
radio (IC-905, IC-705, FT-991A, etc.) for TX. Sibling to the HackRF,
RTL-SDR, SDRplay, AirSpy, and IQ RX bridges in the family — same
GUI, same WSJT-X UDP / Linrad / VB-Cable wiring, just with the
AD9361/AD9363 transceiver as the front end.

The bridge talks to the Pluto over **libiio** (network backend by
default — `ip:HOST` over USB-RNDIS or Ethernet), tunes the AD9361
to match WSJT-X's dial frequency, demodulates SSB to **VB-Cable
Line 1** for WSJT-X RX audio, and streams **96 kHz IQ to QMAP** for
wideband Q65 decode. Your real rig keeps doing TX and narrowband RX.

For **Doppler tracking** with WSJT-X, the bridge can optionally run
a **rigctld-compatible CAT server** so WSJT-X (Rig = "Hamlib NET
rigctl") commands corrected frequencies directly to it — see the
release notes below.

> **Note:** This release covers the **RX-only** Pluto bridge. A
> separate **TX+RX** Pluto bridge — `pluto-wsjtx-bridge` — is on the
> roadmap and will live in its own repo. The `pluto-wsjtx-bridge`
> name is reserved for the full transceiver work.

Author: **Andreas Junge, N6NU** &lt;<n6nu@arrl.net>&gt;.

---

## Latest release — v0.99.6 (bridge-core CatServer fixes)

| Variant | Download |
|---|---|
| **Windows 10 / 11** (installer) | **[pluto-rx-bridge-0.99.6-setup.exe](pluto-rx-bridge-0.99.6-setup.exe)** |

Picks up three CatServer fixes from this week's pluto-wsjtx-bridge
bring-up. **Only meaningful if you have the rigctld CAT server
opted in** (Settings → CAT server checkbox, or `--cat` CLI flag)
for WSJT-X Doppler tracking. Default CAT-off, UDP-observer mode is
unchanged.

- `ptt_type=0x1` in dump_state (was `0x8` `RIG_PTT_GPION`).
- `has_set_ptt=1` / `has_get_ptt=1` / `has_set_mode=1` /
  `has_get_mode=1` advertised in dump_state.
- PTT value parser accepts any non-zero value (`1`/`2`/`3`) as ON,
  not just `1` — needed for WSJT-X PKTUSB / PKTLSB modes which
  send Hamlib PTT value `3` (DATA-PTT).

If you've been hitting "Test PTT clicked but the bridge logged
[CAT PTT] off" against v0.99.5 in WSJT-X data modes, that was the
bug. Drop-in upgrade. INI compatible.

What's in the box (cumulative across the v0.99.x line):

- **AD9361/AD9363 RX path.** All four AD9361 gain-control modes
  (manual, fast_attack, slow_attack default, hybrid), -3..71 dB
  hardware gain (manual mode), configurable sample rate
  (2.083 — 61.44 Msps), RF bandwidth (200 kHz — 56 MHz), master XO
  ppm trim.
- **Network-attached by default.** Default URI `ip:192.168.2.1`
  (stock Pluto USB-RNDIS); change to your Pluto+'s LAN IP via
  Settings → Host URI or `--host ip:HOST` on the command line.
  USB direct (`usb:`) and `local:` (running on the Pluto) also
  accepted.
- **Auto-reconnect** on Pluto reboot / cable bump / network glitch.
  Worker thread bootstraps via the same retry loop even when the
  device is unreachable at launch — no need to restart the bridge
  after a Pluto power cycle.
- **Opt-in rigctld CAT server (TCP 4534)** for WSJT-X Doppler
  tracking. Default off (passive UDP-observer mode); flip on in
  Settings → CAT server (or `--cat`) when you want WSJT-X to
  command the bridge directly. Auto-detect UDP mute when a CAT
  client is actually connected; live source indicator in the
  window title (`— UDP` / `— UDP (CAT idle)` / `— CAT (n)`).

---

## Bundled third-party libraries

The installer ships with everything the bridge needs at runtime —
no separate libiio install or Visual C++ runtime install required:

- **libiio v0.26** (Analog Devices, LGPLv2-or-later) — talks to
  the Pluto's IIO endpoints over the network/USB backends.
- **libusb-1.0**, **libxml2**, **libserialport** — libiio's
  transitive deps; supplied by the official ADI Windows binary
  snapshot.
- **Qt 6.8.3** (LGPLv3) — GUI and core runtime.
- **FFTW3 / SoXr / pthreads4w** — DSP support libraries used by
  the bridge-core shared code.

Full per-library licence text in
[THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md). Bridge code
itself is **GPLv3** — see [LICENSE](LICENSE).

---

## Source

Source lives in the family monorepo at
**[n6nu/sdr-bridges](https://github.com/n6nu/sdr-bridges)** under
`apps/pluto-rx-bridge/` (app glue) + `radios/pluto/` (PlutoDevice +
PlutoGainPanel) + `bridge-core/` (shared GUI / DSP / Linrad / CAT).

Build steps (Windows, Visual Studio 2022 + Qt 6.8.3 + vcpkg) are
documented in the monorepo README.

---

## See the full release-by-release changelog

[RELEASE_NOTES.md](RELEASE_NOTES.md)
