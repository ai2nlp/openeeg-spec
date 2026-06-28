# OpenEEG Manifesto

**Why a hobbyist-EEG data format? Why now? Why me?**

---

## The problem

I run hobbyist EEG experiments in my garage. I own a Muse 2, a PiEEG, and a half-finished homebrew rig based on the Olimex EEG-SMT (an OpenEEG P2 descendant). My friend has an OpenBCI Cyton. We cannot share data.

Not "it's inconvenient to share." We *cannot*. Each device exports a different format:

- **Muse 2** — proprietary BLE stream; the official app stores recordings in a custom binary that requires the Muse SDK to decode. There is no documented offline format.
- **PiEEG** — exports raw CSV or NumPy arrays via the developer's scripts. No metadata. No annotations. No provenance.
- **OpenBCI Cyton** — writes raw 8-bit samples to an SD card + a GUI XML session file. To convert to anything else, you run their GUI's "Export" and pray.
- **Olimex EEG-SMT (OpenEEG P2)** — 256 Hz, 17-byte binary packets, 57600 baud serial. The original OpenEEG project documented the *electrical* protocol but not a modern data format.

Every time I want to use a tool I didn't build — MNE-Python, Braindecode, EEGLab, a friend's analysis script — I spend 2-3 days writing a converter. Then I write it again for the next device. Then I write it again when the vendor updates their SDK.

This is the *data silo* problem. The hardware has converged (ADS1299 chips, BLE, decent resolution) but the *data layer* has not. The people who would build the missing tools — open-source analysis, federated learning, cross-device research — are stuck reinventing parsers instead.

**OpenEEG is the data layer that should exist.**

---

## The vision

One format, one validator, one reference implementation. Bring your data in (from any supported device), take it anywhere (to any tool that reads OpenEEG).

- A **Muse 2 user** can convert to OpenEEG and load the data in MNE-Python with `mne.io.read_raw_openeeg("session.openeeg.json")` (after the reference implementation is built).
- An **OpenBCI user** can convert in one command: `openeeg convert --from openbci --to openeeg session.xml`.
- A **homebrew rig** with a custom format can implement the spec directly (it's small) and gain access to every OpenEEG-compatible tool.
- A **researcher** who collects 100 Muse 2 sessions can ship them as 100 OpenEEG files plus a `dataset_description.json` (v0.2 feature) and have a portable, citable dataset.

The format is the keystone. Once it exists, the ecosystem builds on top of it.

---

## What this project is NOT

I'm explicit about scope because "open EEG format" is a phrase that means many things to many people. **OpenEEG v0.1 is not:**

- ❌ **A clinical standard.** Clinical EEG goes through HL7 FHIR, DICOM, or vendor-proprietary systems. OpenEEG is for hobbyists, indie researchers, and tinkerers.
- ❌ **A BIDS replacement.** BIDS is the gold standard for *research-grade* neuroimaging. OpenEEG is its *hobbyist sibling*: simpler, JSON-based, no NIfTI requirement, optimized for the devices BIDS doesn't cover.
- ❌ **A vendor lock-in tool.** I have no commercial relationship with Muse, OpenBCI, Olimex, or any other EEG hardware maker. The format is Apache 2.0. Anyone can implement, fork, or extend.
- ❌ **An EEG analysis library.** I'm not building MNE-Python. MNE-Python is brilliant. OpenEEG just makes it easier to feed MNE-Python your data.
- ❌ **A real-time streaming protocol.** v0.1 is batch — record, then save. LSL (Lab Streaming Layer) already does real-time well; OpenEEG complements it for offline storage.
- ❌ **A research paper / academic publication.** I'm publishing the spec as code + docs, not as a peer-reviewed paper. (Open to a joint paper if a researcher wants to collaborate — but the spec ships regardless.)

---

## Why I'm building this

Three reasons:

### 1. I have the technical assets to build it

I have multiple functional (with bugs) EEG webapp repos. I've written signal-processing pipelines. I've integrated Muse 2, PiEEG, and homebrew ADS1299 rigs. I'm a locomotive mechanic by day; I teach myself the parts I don't know. The keystone — a portable data format with a reference implementation — is the part of the stack I can credibly ship.

### 2. I have a long time horizon

I'm not trying to flip this in 18 months. The standard should outlast me. A format that becomes the de facto hobbyist-EEG standard in 5 years is a 5-year project. Apache 2.0 licensing and explicit non-vendor positioning protect the format from being acquired or deprecated.

### 3. I refuse to write the same converter twice

Every new EEG device I encounter is another week of writing a parser. That week is wasted for every other hobbyist who has the same device. OpenEEG ends the loop. The first time is hard; every subsequent time is `openeeg convert --from <device>`.

---

## What success looks like

In 3 years, success is:

1. **At least 3 device converters shipped** (Muse 2, OpenBCI, PiEEG or Olimex)
2. **At least 2 major analysis tools accept OpenEEG** as input (e.g., MNE-Python, Braindecode)
3. **At least 50 sessions shared** in OpenEEG format publicly (datasets, papers, GitHub)
4. **At least 10 external contributors** to the spec or implementation
5. **At least 1 paper / thesis** citing OpenEEG in a methodology section

None of these require users to *pay* for OpenEEG. The format is free, the implementation is free, the converters are free. Revenue (if any) comes from a *conversion-as-a-service* tool, hosted validator, and a paid community — not from the spec itself.

---

## What I need from the community

- **EEGer / hobbyist:** try the format with your device, report what breaks
- **Neuroscientist:** review the spec, flag where it diverges from how you actually work
- **Tool author (MNE-Python, EEGLab, etc.):** if you want a `read_raw_openeeg` function, I'll write the spec-aligned side; you write the integration
- **Standards person:** review the licensing + governance model — is Apache 2.0 right, or should we move to a Linux Foundation / OASIS / W3C-style model?
- **Sponsor:** if you want to underwrite the work (so I can afford to spend more weekday mornings on it), reach out

---

## Governance

For v0.1, governance is "Stanley decides, with community input." This works for a one-person spec. For v0.2 and beyond, if the project gets traction, governance should move to a multi-stakeholder model — a working group with representation from hardware vendors, tool authors, and users. Apache 2.0 makes that transition easy.

---

## License

Apache License 2.0. See [`LICENSE`](../LICENSE).

Copyright 2026 AI2NLP Solutions Inc.
