# OpenEEG — portable data format for hobbyist-grade EEG

**A revival of the original OpenEEG project (2001–2013) with a new focus: a portable, validated, open data format for hobbyist-grade EEG hardware (Muse 2, OpenBCI, PiEEG, homebrew rigs, Olimex EEG-SMT).**

> **Status: v0.1 DRAFT — not production.** This is the first public release of the format. The shape is firm, the schema may evolve, and we are actively seeking feedback from the community before locking v1.0.

---

## What problem this solves

Today, hobbyist EEG data is locked in **proprietary or one-off formats**:

| Hardware | Format | Pain |
|----------|--------|------|
| Muse 2 | Proprietary BLE stream, no documented offline format | Cannot export to analysis tools without re-encoding |
| OpenBCI | Ganglion/Cyton raw SD card + GUI XML | Decoding requires the GUI or undocumented conversion scripts |
| PiEEG | Raw CSV/NumPy export | No metadata, no annotations, no provenance |
| Olimex EEG-SMT (OpenEEG P2) | 256Hz, 17-byte binary packets | Original protocol, no modern metadata layer |
| Homebrew ADS1299 + Arduino | Whatever the developer wrote | Incompatible with every other rig |

This means a Muse 2 user cannot share data with an OpenBCI user. A researcher cannot reproduce a homebrew rig's results. A hobbyist cannot move from one device to another without rewriting every analysis script.

**OpenEEG fixes the data layer.** One format, one validator, one reference implementation. Bring your data in, take it anywhere.

## What OpenEEG is NOT

- ❌ **Not a clinical standard.** Use HL7 FHIR or DICOM for clinical work. OpenEEG is for hobbyists, researchers, and indie builders.
- ❌ **Not a BIDS replacement.** BIDS (`bids.neuroimaging.io`) is the gold standard for *research-grade* neuroimaging. OpenEEG is its hobbyist sibling — simpler, JSON-based, optimized for the devices BIDS doesn't cover.
- ❌ **Not a vendor lock-in tool.** OpenEEG is Apache 2.0 licensed. Anyone can implement, fork, or extend.
- ❌ **Not just a file format.** The format is the keystone, but the project also ships:
  - A **reference implementation** (Python, NumPy-compatible)
  - A **validator** (CLI tool: `openeeg validate file.openeeg.json`)
  - A **converter** (CLI tool: `openeeg convert --from muse2 --to openeeg file.muse`)
  - **Documentation**, examples, and a community

## Quick example

A 10-minute Muse 2 recording, 4 channels, 256 Hz:

```json
{
  "openeeg_version": "0.1.0",
  "session": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "subject_id": "subj-7f3a",
    "started_at": "2026-06-28T14:23:01Z",
    "sampling_rate_hz": 256,
    "duration_seconds": 600.0
  },
  "device": {
    "manufacturer": "Interaxon",
    "model": "Muse 2",
    "firmware_version": "3.5.0"
  },
  "channels": [
    { "index": 0, "label": "TP9",  "position": "10-20", "reference": "Fpz", "units": "microvolts" },
    { "index": 1, "label": "AF7",  "position": "10-20", "reference": "Fpz", "units": "microvolts" },
    { "index": 2, "label": "AF8",  "position": "10-20", "reference": "Fpz", "units": "microvolts" },
    { "index": 3, "label": "TP10", "position": "10-20", "reference": "Fpz", "units": "microvolts" }
  ],
  "data": {
    "format": "float32_le",
    "shape": [4, 153600],
    "file": "session_2026-06-28.openeeg.dat",
    "sha256": "PLACEHOLDER_HASH_AT_WRITE_TIME"
  },
  "annotations": [
    { "timestamp_seconds": 30.0,  "label": "session_start" },
    { "timestamp_seconds": 60.0,  "label": "eyes_open" },
    { "timestamp_seconds": 120.0, "label": "eyes_closed" },
    { "timestamp_seconds": 240.0, "label": "eyes_open" },
    { "timestamp_seconds": 600.0, "label": "session_end" }
  ]
}
```

The matching `session_2026-06-28.openeeg.dat` is raw little-endian float32 samples: 4 channels × 153,600 samples (256 Hz × 600 s) = 614,400 samples × 4 bytes = ~2.4 MB.

See [`examples/sample_muse2_session.openeeg.json`](examples/sample_muse2_session.openeeg.json) for a full annotated example.

## Project status

| Component | Status | Target |
|-----------|--------|--------|
| **Spec v0.1** | 🟡 Draft (this repo) | Lock by 2026-09 |
| **Reference impl (Python)** | 🟡 Skeleton exists (will be ported from prior work) | Alpha by 2026-10 |
| **Validator CLI** | 🔴 Not started | Alpha by 2026-11 |
| **Muse 2 converter** | 🔴 Not started | Beta by 2026-12 |
| **OpenBCI converter** | 🔴 Not started | Beta by 2026-12 |
| **Olimex EEG-SMT (P2) reader** | 🔴 Not started | Beta by 2027-Q1 |
| **v1.0 spec lock** | 🔴 Not started | 2027-Q2 |

See [`docs/roadmap.md`](docs/roadmap.md) for the full plan.

## How to participate

This is a v0.1 draft. **Your feedback shapes the format.**

- 📖 **Read** [`SPEC.md`](SPEC.md) — the format definition
- 📜 **Read** [`docs/manifesto.md`](docs/manifesto.md) — why this exists, what it deliberately isn't
- 💬 **Open an issue** if a field is missing, a constraint is wrong, or a use case isn't covered
- 🔧 **Send a PR** if you want to contribute a converter, validator, or spec change
- 📣 **Share** with anyone who has hit the "I can't move my EEG data between tools" wall

## Origins

The original **OpenEEG project** (2001–2013, `openeeg.sourceforge.net`) was an open-source hardware project for low-cost EEG amplifiers (the ModularEEG P2 and P3). It is dormant but the brand lives on: Olimex still ships EEG-SMT boards that follow the OpenEEG hardware design, and the modular EEG concept continues to inspire hobbyist EEG builders.

**This project is a revival with a complementary focus:** the data layer the original project didn't have bandwidth for. We credit the original maintainers (chrisveigl, joerg_hansmann, dfisher, aguazul, and others) and aim to be the data-format companion to the hardware ecosystem they built.

## License

Apache License 2.0 — see [`LICENSE`](LICENSE).

Copyright 2026 AI2NLP Solutions Inc.

## Project home

- Spec & code: https://github.com/ai2nlp/openeeg-spec
- Project site: https://openeeg.org (launching soon)
- Contact: stanley@openeeg.org
