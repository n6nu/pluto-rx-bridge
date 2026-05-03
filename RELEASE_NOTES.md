# Pluto RX Bridge — Release Notes

## v0.99.5 — visible CAT/UDP mode + auto-detect (2026-05-03)

UX fixes on top of v0.99.4:

- **Window title now shows the live source.** Updates every second:
  - `Pluto RX Bridge v… — UDP` — CAT is disabled, UDP drives.
  - `Pluto RX Bridge v… — UDP (CAT idle)` — CAT server is listening
    but no client connected; UDP still drives.
  - `Pluto RX Bridge v… — CAT (1)` — at least one CAT client is
    connected; CAT drives, UDP path is silently ignored.
- **UDP mute is now auto-detect.** v0.99.4 muted UDP whenever CAT was
  enabled — so if you turned CAT on in Settings but WSJT-X wasn't
  actually configured for `Hamlib NET rigctl`, the bridge went deaf
  to UDP and tracked nothing. Now muting only kicks in when a real
  CAT client is connected (`CatServer::clientCount() > 0`). Result:
  - CAT off in Settings → server not listening, only UDP works.
  - CAT on, WSJT-X using UDP only → bridge falls back to UDP cleanly.
  - CAT on, WSJT-X connects via Hamlib NET rigctl → bridge auto-flips
    to CAT (title changes to `— CAT (1)`); UDP is muted with
    `(ignored — CAT is driving)` log lines.

About **"WSJT-X isn't complaining when CAT is off"**: that's because
WSJT-X is presumably set to `Rig=None` (or another rig). With CAT off,
the bridge's TCP server isn't listening at all — if WSJT-X were set
to `Hamlib NET rigctl`, Test CAT would fail. So the silence isn't
"answering CAT but ignoring", it's "WSJT-X isn't trying CAT in the
first place." Behaviour is correct as designed.

Drop-in upgrade from v0.99.4.

## v0.99.4 — CAT server is now opt-in, mutes UDP when on (2026-05-03)

The first CAT release defaulted CAT on, which got in the way for the
common case (WSJT-X driving a real radio while the bridge just listens
to UDP). Two clean modes now:

- **Default (CAT off)** — bridge is a passive UDP observer. WSJT-X is
  driving your real rig over its own CAT; this bridge just follows the
  dial via WSJT-X's UDP Status messages. Stays out of the way.
- **CAT on** — bridge IS the radio. WSJT-X Settings → Radio → Hamlib
  NET rigctl, Network Server `127.0.0.1:4534`. Doppler tracking works
  end-to-end (WSJT-X commands corrected freq → bridge → Pluto). While
  CAT is on, the bridge **ignores WSJT-X UDP** so there's no
  double-driving — CAT is the sole source of truth.

Toggle in Settings dialog under **CAT server** (checkbox + port).
Takes effect on next bridge launch. CLI `--cat` and `--cat-port <n>`
also flip it on. The old `--no-cat` flag is gone (CAT is off by
default now, so the explicit-disable form is redundant).

UDP-side log lines stay visible in CAT mode but are suffixed with
`(ignored — CAT is driving)` so you can tell at a glance which path
is active. Stats line drops the noisy `, CAT off` trailer when CAT
is disabled.

Drop-in upgrade from v0.99.3.

## v0.99.3 — fix: RX bootstraps when device is unreachable on launch (2026-05-03)

A startup-order bug surfaced once Doppler tracking was wired up: if the
Pluto wasn't reachable the moment the bridge launched (network coming
up, Pluto still rebooting, wrong URI), the RX worker thread was never
spawned — and since the auto-reconnect loop lives **inside** the
worker, the bridge had no path to ever recover. User had to restart
the bridge after the Pluto came online.

- `pluto.startRx()` is now called unconditionally. The worker thread
  bootstraps via the same `reconnectLoop()` that handles mid-stream
  disconnects, and begins streaming the moment the device is reachable.
- Startup "not reachable" dialog reworded to make it clear the bridge
  keeps retrying on its own; downgraded from `QMessageBox::warning`
  (red icon) to `QMessageBox::information`.
- Same fix applies to wrong-URI scenarios — change the URI in Settings,
  no bridge restart needed.

Drop-in upgrade from v0.99.2.

## v0.99.2 — rigctld-compatible CAT server (Doppler tracking) (2026-05-03)

WSJT-X Doppler tracking now works end-to-end without putting a real
radio in the loop.

- Bridge now runs a rigctld-compatible TCP server on port 4534 (the
  same `CatServer` `hackrf-wsjtx-bridge` already uses, ported across
  via bridge-core). Default port chosen so it doesn't collide with a
  real rigctld instance (4532) or `hackrf-wsjtx-bridge` (4533).
- Configure WSJT-X: **Settings → Radio → Hamlib NET rigctl**,
  Network Server `127.0.0.1:4534`. With this, WSJT-X's Doppler
  tracking commands corrected frequencies via `\set_freq` over TCP;
  bridge tunes the Pluto, updates the Linrad header, and QMAP / the
  WSJT-X RX path stay aligned.
- The pre-existing **WSJT-X UDP Status** path is unchanged — both
  inputs route to the same `pluto.setFrequency()` sink, so they're
  redundant when both are configured (no conflict).
- New CLI: `--cat-port <n>` (override default 4534), `--no-cat`
  (disable CAT server, fall back to UDP-only). New INI keys:
  `[cat] enabled` (bool, default true), `[cat] tcp_port` (int,
  default 4534).
- Mode + PTT routing too: `\set_mode USB|LSB` updates the SSB demod
  sideband; `\set_ptt 1` stops RX (front-end protection), `\set_ptt 0`
  resumes — same half-duplex behaviour as the UDP path.

Drop-in upgrade from v0.99.1.

## v0.99.1 — auto-reconnect on Pluto reboot / cable bump (2026-05-02)

The Pluto comes back on its own when bumped, lost on the network, or
power-cycled — bridge no longer requires a manual restart.

- PlutoDevice worker thread detects `iio_buffer_refill` failure, tears
  the libiio context down, and retries `iio_create_context_from_uri`
  with exponential backoff (1 → 2 → 4 → 8 → 16 → 30 s capped).
- `iio_context_set_timeout(ctx, 5000)` so a dead TCP socket throws a
  refill error within 5 s instead of hanging on Windows' 2-hour TCP
  keepalive default.
- All public setters (frequency, gain mode, manual gain, sample rate,
  RF bandwidth, ppm) take a mutex; while disconnected they update the
  in-memory config only, and the reconnect path re-pushes everything.
  WSJT-X dial changes during the outage are not lost — the bridge
  resumes on the latest dial frequency.
- Status panel `Gains:` row appends `[reconnecting]` while the worker
  is in retry mode.
- Bench-tested: full Pluto reboot mid-stream → bridge recovers in
  ~40 s end-to-end (mostly the Pluto's own boot time), resumes on the
  same freq + gain.

Drop-in upgrade from v0.99.0.

## v0.99.0 — initial RX-only build (2026-05-02)

First release of the Pluto / Pluto+ RX bridge — sibling to the HackRF,
RTL-SDR, SDRplay, AirSpy, and IQ RX bridges in the family. Talks to
the AD9361/AD9363 transceiver over libiio (network backend by default,
typically `ip:192.168.2.1` for stock USB-RNDIS or a DHCP address for
Pluto+ on Ethernet).

- libiio v0.26 binary snapshot bundled (LGPLv2-or-later).
- AD9361 gain modes: manual, fast_attack, slow_attack (default),
  hybrid. Manual mode honours the hardware-gain spinbox -3..71 dB.
- Configurable sample rate (2.083 — 61.44 Msps), RF bandwidth
  (200 kHz — 56 MHz), PPM correction (default 0; Pluto+'s 0.5 ppm
  VCTCXO ships accurate enough that most users won't touch this).
- Native int16 IQ stream (12-bit AD9363 ADC left-justified) → SSB
  demodulator (WSJT-X RX audio), FFT engine (waterfall), and Linrad
  server (QMAP).
- Settings dialog row for the libiio URI — change the host without
  editing the INI file (reconnect required for it to take effect;
  v0.99.1 may make this hot-swappable).
- Bench-tested against signal generator on 144.400 MHz; frequency
  spot-on, no drift.
