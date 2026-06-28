# Roadmap — v0.1 to v1.0

**Last updated:** 2026-06-28 (v0.1.0 draft)

This document tracks what we plan to ship, in what order, and what "done" looks like. Dates are targets, not commitments — community feedback may reprioritize.

---

## v0.1 — "Sketch the keystone" (now → 2026-09)

**Goal:** publish a draft spec, prove the schema is implementable, gather community feedback.

| Deliverable | Status | Target |
|-------------|--------|--------|
| Spec v0.1 (this document) | ✅ Draft published | 2026-06 |
| Apache 2.0 LICENSE | ✅ Published | 2026-06 |
| README + manifesto + example | ✅ Published | 2026-06 |
| Public spec repo (github.com/ai2nlp/openeeg-spec) | ✅ Live | 2026-06 |
| openeeg.org site (landing page) | 🟡 In progress | 2026-07 |
| Recruit 3-5 early reviewers | 🟡 In progress | 2026-07 |
| Spec v0.1.1 (incorporate early feedback) | 🔴 Not started | 2026-08 |
| Spec v0.1 lock (community call for breaking changes) | 🔴 Not started | 2026-09 |

**Done when:** at least 3 independent implementers (humans or AIs) can write a parser for v0.1 from the spec alone, without asking me questions.

---

## v0.2 — "Ship the reference implementation" (2026-10 → 2026-12)

**Goal:** prove the spec is implementable in code, ship the first converters, get the first real users.

| Deliverable | Status | Target |
|-------------|--------|--------|
| Reference implementation: Python (NumPy-based) | 🔴 Not started | 2026-10 |
| Validator CLI: `openeeg validate file.openeeg.json` | 🔴 Not started | 2026-10 |
| Muse 2 converter: `openeeg convert --from muse2` | 🔴 Not started | 2026-11 |
| OpenBCI converter: `openeeg convert --from openbci` | 🔴 Not started | 2026-11 |
| Olimex EEG-SMT (P2) reader: `openeeg convert --from openeeg-p2` | 🔴 Not started | 2026-12 |
| PyPI package: `pip install openeeg` | 🔴 Not started | 2026-12 |
| GitHub Actions CI: lint + validate every PR | 🔴 Not started | 2026-12 |
| First 10 real users (hobbyists + researchers) | 🔴 Not started | 2026-12 |

**Done when:** a Muse 2 user can record a session, convert it with one command, and load it in MNE-Python with zero manual file editing.

---

## v0.3 — "Reach the indie-research tier" (2027-Q1 → 2027-Q2)

**Goal:** the format is usable for real research workflows, not just hobbyist demos.

| Deliverable | Status | Target |
|-------------|--------|--------|
| PiEEG converter | 🔴 Not started | 2027-Q1 |
| Compressed sample format (gzip wrapper) | 🔴 Not started | 2027-Q1 |
| Epoched data support (`epochs` block) | 🔴 Not started | 2027-Q2 |
| Cross-session dataset description (BIDS-style sidecar) | 🔴 Not started | 2027-Q2 |
| MNE-Python integration PR: `mne.io.read_raw_openeeg` | 🔴 Not started | 2027-Q2 |
| First dataset published in OpenEEG format | 🔴 Not started | 2027-Q2 |

**Done when:** a researcher can publish a 50-session dataset in OpenEEG format that another researcher can load + analyze in MNE-Python without writing a converter.

---

## v0.4 — "Governance + community" (2027-Q3)

**Goal:** the format is no longer dependent on a single person.

| Deliverable | Status | Target |
|-------------|--------|--------|
| Working group charter (governance model) | 🔴 Not started | 2027-Q3 |
| Multi-maintainer repo access | 🔴 Not started | 2027-Q3 |
| Code of Conduct | 🔴 Not started | 2027-Q3 |
| First 10 external contributors (non-founder PRs merged) | 🔴 Not started | 2027-Q3 |
| Discord / Matrix / community channel | 🔴 Not started | 2027-Q3 |

**Done when:** at least 3 active external maintainers, governance is documented, and the project can survive me stepping away for 6 months.

---

## v1.0 — "Lock the spec" (2027-Q4)

**Goal:** the spec is stable enough to be cited in papers and depended on by tools.

| Deliverable | Status | Target |
|-------------|--------|--------|
| Spec v1.0.0 (no breaking changes from v0.x) | 🔴 Not started | 2027-Q4 |
| At least 3 device converters shipping stable releases | 🔴 Not started | 2027-Q4 |
| At least 1 analysis tool with first-class OpenEEG support | 🔴 Not started | 2027-Q4 |
| First paper / thesis citing OpenEEG | 🔴 Not started | 2027-Q4 |
| Migration guide from v0.x to v1.0 | 🔴 Not started | 2027-Q4 |

**Done when:** a tool author can write "OpenEEG support" in their release notes without disclaimers.

---

## Beyond v1.0 (candidates)

These are not committed. They're the v1.x candidates I'd want to explore if there's community interest:

- **Compressed alternatives** (Zarr, HDF5 containers for performance-critical use)
- **Real-time streaming mode** (complement LSL for offline persistence)
- **Federated learning protocol** (train models on OpenEEG data without centralizing the data)
- **Cloud-hosted validator + conversion** (no-install path for non-developers)
- **Mobile recording app** (iOS / Android, OpenEEG-native)
- **Standards-body transition** (LF, OASIS, W3C — only if community demand exists)

---

## What is NOT on the roadmap

To keep scope honest:

- ❌ **Hardware.** I will not design EEG amplifiers. Olimex, OpenBCI, and others do that well.
- ❌ **Cloud storage of user data.** The format is local-first. Cloud is a v2.x candidate.
- ❌ **Real-time EEG analysis.** I am not building a BCI framework. MNE-Python, Braindecode, and others do that.
- ❌ **Clinical certification (FDA, CE).** Way out of scope. See "What this is NOT" in the manifesto.

---

## How to influence the roadmap

- Open an issue with the `roadmap` label and a use case
- Comment on the relevant section in this file
- Send a PR to this file with your proposal

The roadmap is a living document. Items get promoted, deferred, or killed based on community signal.

---

## License

This document is licensed under Apache License 2.0.

Copyright 2026 AI2NLP Solutions Inc.
