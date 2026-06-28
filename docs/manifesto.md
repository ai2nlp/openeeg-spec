# OpenEEG Manifesto

**A case study: why a hobbyist-EEG data format, why now, and why one person.**

---

## The case

**Stanley** is a 51-year-old locomotive mechanic working the afternoon shift in Montreal. Weekday mornings are his. Weekends are for the projects that pay nothing but compound.

He owns **one Muse 2** — his only EEG headset. From a stack of prior work, he also has **multiple functional (with bugs) EEG webapp repos**: signal-processing pipelines, recording tools, visualization dashboards, integration code for various SDKs he no longer touches. He did not start those projects to build a data format. They were the artifact of the same loop he is now trying to break.

His friends have OpenBCI Cyton boards, PiEEG HATs, and homebrew ADS1299 rigs. They cannot share data with him.

Not "it's inconvenient to share." They *cannot*. Each device exports a different format:

- **Muse 2** — proprietary BLE stream; the official app stores recordings in a custom binary that requires the Muse SDK to decode. There is no documented offline format.
- **PiEEG** — exports raw CSV or NumPy arrays via the developer's scripts. No metadata. No annotations. No provenance.
- **OpenBCI Cyton** — writes raw 8-bit samples to an SD card + a GUI XML session file. To convert to anything else, you run their GUI's "Export" and pray.
- **Olimex EEG-SMT (OpenEEG P2)** — 256 Hz, 17-byte binary packets, 57600 baud serial. The original OpenEEG project documented the *electrical* protocol but not a modern data format.

Every time Stanley wants to use a tool he did not build — MNE-Python, Braindecode, EEGLab, a friend's analysis script — he spends 2-3 days writing a converter. Then again for the next device. Then again when the vendor updates their SDK.

This is the *data silo* problem. The hardware has converged (ADS1299 chips, BLE, decent resolution) but the *data layer* has not. The people who would build the missing tools — open-source analysis, federated learning, cross-device research — are stuck reinventing parsers instead.

**OpenEEG is the data layer that should exist.**

---

## The context

The hobbyist-EEG community is small, distributed, and increasingly sophisticated. The people who own OpenBCI boards, PiEEG HATs, and Olimex EEG-SMT kits are not casual users; they are engineers, researchers, grad students, and serious tinkerers who have chosen to spend €200–€2,000 on a tool that does not have a clean data export.

This is a market that rewards *integration* — getting hobbyist data into the same toolchains (MNE-Python, Braindecode, EEGLab) that research labs use. It does not reward *yet another proprietary SDK*.

The original **OpenEEG project** (2001–2013) was an open-source EEG hardware project — low-cost amplifiers, schematics, firmware. It is dormant, but the brand lives on: Olimex still ships EEG-SMT boards that follow the OpenEEG design. The 2001 project's keystone was the hardware. The 2026 keystone is the data format that the hardware did not have bandwidth to build.

OpenEEG v0.1 is a revival of the project name, not a continuation of the project scope.

---

## The vision

One format, one validator, one reference implementation. Bring your data in (from any supported device), take it anywhere (to any tool that reads OpenEEG).

- A **Muse 2 user** can convert to OpenEEG and load the data in MNE-Python with `mne.io.read_raw_openeeg("session.openeeg.json")` (after the reference implementation is built).
- An **OpenBCI user** can convert in one command: `openeeg convert --from openbci --to openeeg session.xml`.
- A **homebrew rig** with a custom format can implement the spec directly (it is small) and gain access to every OpenEEG-compatible tool.
- A **researcher** who collects 100 Muse 2 sessions can ship them as 100 OpenEEG files plus a `dataset_description.json` (v0.2 feature) and have a portable, citable dataset.

The format is the keystone. Once it exists, the ecosystem builds on top of it.

---

## The scope (deliberately limited)

OpenEEG v0.1 is not:

- ❌ **A clinical standard.** Clinical EEG goes through HL7 FHIR, DICOM, or vendor-proprietary systems. OpenEEG is for hobbyists, indie researchers, and tinkerers.
- ❌ **A BIDS replacement.** BIDS is the gold standard for *research-grade* neuroimaging. OpenEEG is its *hobbyist sibling*: simpler, JSON-based, no NIfTI requirement, optimized for the devices BIDS does not cover.
- ❌ **A vendor lock-in tool.** Stanley has no commercial relationship with Muse, OpenBCI, Olimex, or any other EEG hardware maker. The format is Apache 2.0. Anyone can implement, fork, or extend.
- ❌ **An EEG analysis library.** Stanley is not building MNE-Python. MNE-Python is brilliant. OpenEEG just makes it easier to feed MNE-Python your data.
- ❌ **A real-time streaming protocol.** v0.1 is batch — record, then save. LSL (Lab Streaming Layer) already does real-time well; OpenEEG complements it for offline storage.
- ❌ **A research paper / academic publication.** The spec is published as code + docs, not as a peer-reviewed paper. (Open to a joint paper if a researcher wants to collaborate — but the spec ships regardless.)

---

## The decision

Three reasons Stanley is the right person to ship this, and now is the right time:

### 1. He has the technical assets to build it

Stanley has multiple functional (with bugs) EEG webapp repos. He has written signal-processing pipelines. He has integrated Muse 2, PiEEG SDKs (without owning the hardware), and homebrew ADS1299 rigs (without owning the hardware). He is a locomotive mechanic by day and a self-directed systems learner by morning. The keystone — a portable data format with a reference implementation — is the part of the stack he can credibly ship.

**The validation limitation is real and acknowledged:** v0.1 has been developed against **a single Muse 2 unit** (the only headset Stanley owns). The spec is designed to generalize to other hardware, but the *reference implementation* and *example data* are Muse 2 specific until converter contributions arrive. This is honest scope, not aspiration. See [`docs/roadmap.md`](roadmap.md) for the converter plan.

### 2. He has a long time horizon

This is not a 18-month flip. The standard should outlast the person building it. A format that becomes the de facto hobbyist-EEG standard in 5 years is a 5-year project. Apache 2.0 licensing and explicit non-vendor positioning protect the format from being acquired or deprecated by any single entity (including Stanley's employer, AI2NLP Solutions Inc., which holds the copyright but is contractually positioned as a steward, not an owner-operator).

### 3. He refuses to write the same converter twice

Every new EEG device Stanley encounters is another week of writing a parser. That week is wasted for every other hobbyist who has the same device. OpenEEG ends the loop. The first time is hard; every subsequent time is `openeeg convert --from <device>`.

---

## What success looks like

In 3 years, success is:

1. **At least 3 device converters shipped** (Muse 2 first, then OpenBCI, then PiEEG or Olimex — contributors welcome)
2. **At least 2 major analysis tools accept OpenEEG** as input (e.g., MNE-Python, Braindecode)
3. **At least 50 sessions shared** in OpenEEG format publicly (datasets, papers, GitHub)
4. **At least 10 external contributors** to the spec or implementation
5. **At least 1 paper / thesis** citing OpenEEG in a methodology section

None of these require users to *pay* for OpenEEG. The format is free, the implementation is free, the converters are free. Revenue (if any) comes from a *conversion-as-a-service* tool, hosted validator, and a paid community — not from the spec itself.

---

## What the community can do

- **EEGer / hobbyist:** try the format with your device, report what breaks
- **Neuroscientist:** review the spec, flag where it diverges from how you actually work
- **Tool author (MNE-Python, EEGLab, etc.):** if you want a `read_raw_openeeg` function, Stanley will write the spec-aligned side; you write the integration
- **Converter contributor (OpenBCI / PiEEG / Olimex owners):** the spec is in place; the missing piece is per-device converters
- **Standards person:** review the licensing + governance model — is Apache 2.0 right, or should the project move to a Linux Foundation / OASIS / W3C-style model?
- **Sponsor:** if you want to underwrite the work (so Stanley can afford to spend more weekday mornings on it), reach out

---

## Governance

For v0.1, governance is "Stanley decides, with community input." This works for a one-person spec. For v0.2 and beyond, if the project gets traction, governance should move to a multi-stakeholder model — a working group with representation from hardware vendors, tool authors, and users. Apache 2.0 makes that transition easy.

The copyright holder (AI2NLP Solutions Inc.) is positioned as a steward: it holds the legal rights, but the spec and code are Apache 2.0 and cannot be relicensed or revoked unilaterally. If the project reaches v1.0 and a multi-stakeholder governance body forms, the copyright can be transferred to a foundation (Linux Foundation, OASIS, etc.) under the same Apache 2.0 terms.

---

## License

Apache License 2.0. See [`LICENSE`](../LICENSE).

Copyright 2026 AI2NLP Solutions Inc.
