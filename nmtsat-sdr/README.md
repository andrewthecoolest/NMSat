# nmtsat-sdr

Real-time RF coherence monitor. Several RTL-SDRs receive the same signal while a
BladeRF transmits a known test tone; a processing thread flags when the receivers
fall out of sync.

## Hardware

- 2+ RTL-SDR receivers
- 1 BladeRF (transmitter)

## Setup

Debian/Ubuntu:

```bash
./setup.sh                 # install deps + udev rules
./setup.sh --from-source   # if distro librtlsdr/bladeRF are too old
```

Replug the SDRs afterward so the udev rules take effect.

## Build

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
```

## Run

- **`nmtsat_sync`** — one-shot check: aligns the SDRs (xcorr), matches gain,
  re-syncs, retunes, and dumps samples at each stage. Plot with
  `python3 tests/visualize_sync.py`.

- **`nmtsat_longevity`** — runs indefinitely. Logs a heartbeat every second and,
  on each desync, saves a forensic window under `longevity_runs/` and re-syncs.
  Type `s`+Enter for a snapshot, `desync`+Enter to force one, `q`+Enter to quit.

Browse captured events:

```bash
python3 tests/view_desync.py
```

## How it works

Each SDR streams 8-bit I/Q into a ring buffer. The processing thread takes the
magnitude of each sample, cross-correlates the receivers against SDR0 to align
them, then watches the per-pair magnitude difference and fires a desync when it
jumps past the detection band.

Source layout: `sdr.c` (RTL-SDR), `blade.c` (BladeRF TX), `process.c` (ring
buffer + alignment + desync detection).
