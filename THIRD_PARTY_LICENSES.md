# Third-Party Components

The Pluto RX Bridge installer bundles binaries of the following
third-party libraries and tools. Each is governed by its own license.
This document is included with the installer to satisfy attribution
and source-availability requirements.

The bridge itself is GPLv3; see `LICENSE`.

---

## libiio

- **Use:** `libiio.dll` — Industrial I/O over the network / USB / local
  backends. The bridge talks to the AD9361/AD9363 transceiver inside
  the ADALM-Pluto / Pluto+ exclusively through libiio.
- **License:** GNU Lesser General Public License version 2.1 or later
  (LGPLv2.1+).
- **Upstream:** <https://github.com/analogdevicesinc/libiio>.
- **Source:** the upstream repository at the URL above. This build
  links the unmodified Windows VS-2022 binary snapshot from the v0.26
  release (asset `Windows.zip` — `libiio.lib` / `libiio.dll`).
- **Bundled transitive dependencies:** `libusb-1.0.dll`, `libxml2.dll`,
  `libserialport-0.dll` come from the libiio Windows distribution and
  are documented separately below.

## libusb-1.0

- **Use:** `libusb-1.0.dll` — cross-platform USB I/O. Used here as a
  transitive dependency of libiio (USB backend, even though this
  bridge defaults to the IP backend).
- **License:** GNU Lesser General Public License version 2.1 or later
  (LGPLv2.1+).
- **Upstream:** <https://libusb.info/> — source at
  <https://github.com/libusb/libusb>.

## libxml2

- **Use:** `libxml2.dll` — XML parser used by libiio for context /
  device descriptions returned by the IIO backend.
- **License:** MIT License.
- **Upstream:** <https://gitlab.gnome.org/GNOME/libxml2>.

## libserialport

- **Use:** `libserialport-0.dll` — cross-platform serial-port I/O,
  pulled in by libiio for the (unused-by-this-bridge) serial backend.
- **License:** GNU Lesser General Public License version 3 (LGPLv3).
- **Upstream:** <https://sigrok.org/wiki/Libserialport>.

## FFTW (fftw3 / fftw3f)

- **Use:** `fftw3.dll`, `fftw3f.dll` — Fast Fourier transform routines
  for the bridge's spectrum/waterfall display.
- **License:** GNU General Public License, version 2 or later
  (GPLv2+). FFTW is also available under a commercial license; this
  build uses the GPL version.
- **Upstream:** <http://www.fftw.org/> — and
  <https://github.com/FFTW/fftw3>.
- **Source:** the upstream releases at the URL above.

## Qt 6

- **Use:** `Qt6Core.dll`, `Qt6Gui.dll`, `Qt6Widgets.dll`,
  `Qt6Network.dll`, `Qt6Multimedia.dll`, `Qt6Svg.dll`, plus the
  bundled Qt platform plugins under the installation directory's
  `platforms/`, `imageformats/`, `multimedia/`, `tls/`, etc.
- **License:** GNU Lesser General Public License version 3 (LGPLv3).
  Dual-licensed commercially by The Qt Company; this build uses
  LGPLv3.
- **Upstream:** <https://www.qt.io/> — source at
  <https://download.qt.io/official_releases/qt/>.
- **Source:** Qt 6.8.3 (the version of Qt this build links against).

## SoXR (libsoxr)

- **Use:** `soxr.dll` — high-quality rational-rate sample-rate
  conversion (used to decimate the Pluto's 2.5 Msps complex IQ down
  to 96 kHz for QMAP and 48 kHz for the SSB demodulator).
- **License:** GNU Lesser General Public License version 2.1
  (LGPLv2.1).
- **Upstream:** <https://sourceforge.net/projects/soxr/>.

## pthreads4w (pthreadVC3.dll)

- **Use:** `pthreadVC3.dll` — POSIX-threads emulation for Windows,
  pulled in as a dependency of soxr on MSVC.
- **License:** Apache License 2.0 (the project is dual-licensed with
  the Apache 2.0 option active for binary distributions).
- **Upstream:** <https://sourceforge.net/projects/pthreads4w/>.

## FFmpeg shared libraries

- **Use:** `avcodec-61.dll`, `avformat-61.dll`, `avutil-59.dll`,
  `swresample-5.dll`, `swscale-8.dll` — pulled in transitively by Qt
  Multimedia's FFmpeg backend on Windows.
- **License:** GNU Lesser General Public License version 2.1 or later
  (LGPLv2.1+); some optional components are GPL but the LGPL build of
  FFmpeg is what Qt Multimedia ships.
- **Upstream:** <https://ffmpeg.org/>.

## VB-Audio Virtual Cable (NOT bundled, but required at runtime)

For completeness: the bridge expects the user to install VB-Audio
Virtual Cable separately. It is **not** redistributed by this
installer. It is donationware under VB-Audio's own license — see
<https://vb-audio.com/Cable/>.

---

## Source availability

All bundled GPL- and LGPL-licensed components are freely available
from their upstream maintainers at the URLs above; the bridge links
unmodified upstream releases. If you need a copy of the source for
any specific bundled binary version and cannot retrieve it from
upstream, contact Andreas Junge (N6NU) at **<andreas@n6nu.org>** and
a copy will be provided in accordance with GPL section 6 / LGPL
section 4.
