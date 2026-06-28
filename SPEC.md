# OpenEEG Specification — v0.1 (DRAFT)

**Status:** DRAFT — open for community review. Lock target: 2026-09-30.
**Editor:** Stanley (AI2NLP Solutions Inc.) · stanley@openeeg.org
**Spec version:** `0.1.0`

---

## 1. Overview

An **OpenEEG session** is a single continuous recording from a single device, represented as **two files** sharing a common basename:

| File | Purpose | Format |
|------|---------|--------|
| `<basename>.openeeg.json` | Metadata (this spec) | JSON, UTF-8 |
| `<basename>.openeeg.dat`  | Sample data (raw float32) | Binary, little-endian |

The two-file split follows the BIDS pattern: structured metadata is small and human-readable; sample data is large and binary. This keeps the format efficient (no JSON parsing of millions of numbers) and inspectable (you can `cat` the metadata file and understand the recording).

**Why two files and not one (e.g., HDF5)?** v0.1 deliberately avoids container formats. JSON + raw float32 is:
- Trivially readable in any language (Python `json`, JavaScript `JSON.parse`, etc.)
- Trivially streamable (you can `.append()` to the .dat file as you record)
- Trivially diffable (the .json file is text)
- v0.2 may add optional HDF5 or Zarr containers for performance-critical use cases.

---

## 2. Filename conventions

- `<basename>` is a free-form identifier chosen by the recorder. Suggested pattern: `subj-<id>_sess-<yyyymmdd>-<hhmmss>` or any UUID-prefixed form.
- Files MUST share the same basename and differ only in extension.
- Extensions are case-sensitive: `.openeeg.json` and `.openeeg.dat`.
- Co-located files (same directory) are assumed to be a single session.

**Example:**
```
subj-7f3a_sess-20260628-142301.openeeg.json
subj-7f3a_sess-20260628-142301.openeeg.dat
```

---

## 3. JSON metadata schema (v0.1.0)

### 3.1 Top-level required fields

| Field | Type | Description |
|-------|------|-------------|
| `openeeg_version` | string | MUST be `"0.1.0"` (semver of this spec) |
| `session.id` | string (UUID) | Unique session identifier |
| `session.subject_id` | string | Anonymized subject identifier (free-form, but should be opaque — NOT a real name or email) |
| `session.started_at` | string (ISO-8601) | Wall-clock time of recording start (UTC recommended) |
| `session.sampling_rate_hz` | number | Sampling rate in Hz (positive, finite) |
| `channels` | array | Channel definitions (see §4). MUST be non-empty. |
| `data` | object | Reference to the binary sample file (see §5) |

### 3.2 Top-level optional fields

| Field | Type | Description |
|-------|------|-------------|
| `session.ended_at` | string (ISO-8601) | Wall-clock time of recording end |
| `session.duration_seconds` | number | Computed or recorded duration (use for sanity check) |
| `session.notes` | string | Free-form text |
| `device` | object | Device metadata (see §6) |
| `annotations` | array | Event markers (see §7) |
| `metadata` | object | Free-form key-value for additional context (subject age group, environment, etc.) |
| `extensions` | object | Vendor- or use-case-specific extensions (see §8) |

### 3.3 Top-level forbidden fields

The following are NOT allowed at the top level and MUST appear under their proper subsection if used:
- `version` (use `openeeg_version`)
- `subject`, `subject_id` (use `session.subject_id`)
- `rate`, `sampling_rate` (use `session.sampling_rate_hz`)

This rule prevents schema drift and tooling fragility.

---

## 4. Channels array

`channels` is an ordered array of channel definitions. Order MUST match the column order in the binary data file (§5).

### 4.1 Required per-channel fields

| Field | Type | Description |
|-------|------|-------------|
| `index` | integer (≥ 0) | Position in the binary data columns. MUST be unique within `channels`. SHOULD be sequential from 0. |
| `label` | string | Human-readable label. SHOULD follow 10-20 system (e.g., "Fp1", "C3", "O1", "TP9", "AF7"). |
| `units` | string | Physical units of the samples. MUST be `"microvolts"` for v0.1 if the source is EEG. Other units MAY appear for non-EEG channels (e.g., accelerometer data) but MUST be a recognized unit string. |

### 4.2 Optional per-channel fields

| Field | Type | Description |
|-------|------|-------------|
| `position` | string | 10-20 system position code (e.g., "Fpz", "C3", "O1"). Use `"extracellular"` or similar for non-scalp channels. |
| `reference` | string | Reference electrode label (e.g., "A1", "Fpz", "linked-ears"). For active电极 systems with on-board reference, use the manufacturer's documented reference. |
| `impedance_kohms` | number | Electrode impedance at session start (informational) |
| `filter` | object | Applied filter chain `{highpass_hz, lowpass_hz, notch_hz}` |

---

## 5. Binary data file (`.openeeg.dat`)

### 5.1 Encoding

- **Format:** IEEE 754 single-precision float, little-endian
- **Shape:** `[n_channels, n_samples]` (row-major: all samples for channel 0, then all for channel 1, etc.)
- **Sign:** Signed (centered at 0)
- **Range:** No explicit range requirement, but values SHOULD be in approximately [-200, +200] microvolts for typical scalp EEG. Out-of-range values MAY indicate saturation or artifact and SHOULD be flagged by the validator.

### 5.2 File size

`filesize_bytes = n_channels * n_samples * 4`

If the file size does not match this formula, the file is corrupt or the metadata is wrong. The validator (§10) MUST check this.

### 5.3 Integrity (optional but recommended)

The `data.sha256` field in the JSON, if present, MUST match the SHA-256 of the binary file. This catches silent corruption.

---

## 6. Device object (optional)

If present, `device` SHOULD contain:

| Field | Type | Description |
|-------|------|-------------|
| `manufacturer` | string | e.g., "Interaxon", "OpenBCI", "Olimex" |
| `model` | string | e.g., "Muse 2", "Cyton 8-channel", "EEG-SMT" |
| `firmware_version` | string | Firmware version string (vendor-defined) |
| `serial_number` | string | Device serial number (optional; consider privacy implications) |
| `driver` | string | Driver/library used to record (e.g., "muse-lsl", "openbci-py", "openeeg-p2") |
| `driver_version` | string | Driver/library version |

---

## 7. Annotations array (optional)

Each annotation is a single event marker. The array MAY be empty.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `timestamp_seconds` | number | yes | Time of the event, in seconds from `session.started_at`. May be negative if the annotation is pre-session. |
| `label` | string | yes | Event type or human-readable description |
| `duration_seconds` | number | no | Duration of the event (e.g., a 5-second eyes-closed period) |
| `metadata` | object | no | Additional structured data (e.g., `{stimulus_id: "auditory_tone_440hz"}`) |

**Reserved labels** (recommended, not enforced):
- `session_start`, `session_end`
- `eyes_open`, `eyes_closed`
- `marker` (generic; should include a `metadata.label` field for the actual type)
- `artifact_blink`, `artifact_movement`, `artifact_electrode`

Implementations MAY add custom labels but SHOULD prefix them with a namespace (e.g., `myapp.custom_event`).

---

## 8. Extensions (optional)

The `extensions` top-level object allows vendors and use-cases to add fields without breaking the core spec. Extensions MUST NOT redefine or override any field in §§3–7.

**Example:**
```json
{
  "extensions": {
    "muse2": {
      "headband_fit": "good",
      "battery_pct_at_start": 87
    }
  }
}
```

The spec does not define what fields live under which extension namespace — that's per-vendor.

---

## 9. Reserved future fields

These names are reserved for future spec versions and MUST NOT be used as custom field names in v0.1:

- `preprocessing`, `pipeline`, `processed` (for processed data in v0.2)
- `events` (use `annotations` in v0.1; v0.2 may deprecate)
- `epochs`, `trials` (for epoched data in v0.2)
- `units_`, `_si` prefixes (reserved for SI conversion metadata)

---

## 10. Validation

A v0.1 validator MUST check, at minimum:

1. **JSON parses** and matches the schema in §3
2. **Required fields present** (§3.1)
3. **`openeeg_version` is supported** (currently only `"0.1.0"`)
4. **Channels array non-empty** with unique `index` values
5. **Binary file size matches** `n_channels * n_samples * 4`
6. **SHA-256 matches** (if `data.sha256` is present)
7. **All `timestamp_seconds` in annotations** are within `[0, duration_seconds]`
8. **No reserved field names** are used as top-level or channel-level custom fields (§9)

The reference validator (`openeeg validate`) will be shipped with the reference implementation.

---

## 11. Versioning and evolution

- This is **v0.1.0**. Breaking changes to §§3–7 before v1.0 are possible (with a version bump + migration guide).
- After v1.0, the spec follows semver strictly. Additions are minor (1.x.0). Breaking changes are major (2.0.0).
- New optional fields MAY be added in minor versions without breaking older readers (old readers ignore unknown fields).
- Removal or renaming of any field in §§3–7 is a breaking change.

---

## 12. Open questions for v0.2

These are deliberately deferred from v0.1 and tracked for v0.2:

1. **Compressed sample format** — gzip, zstd, or LZ4 wrappers around the .dat file?
2. **Multiple sessions per recording** — one file with session boundaries, or always one file per session?
3. **Epoched data** — predefined time windows with per-epoch metadata?
4. **Provenance chain** — track every transformation from raw → processed (e.g., for sharing a filtered version)?
5. **Real-time streaming mode** — currently v0.1 is batch (record, then write). Streaming is a separate use case.
6. **Cross-session aggregations** — research datasets often combine 100s of sessions; v0.1 doesn't address this.

If you have opinions on any of these, open an issue.

---

## 13. License

This specification is licensed under Apache License 2.0.

Copyright 2026 AI2NLP Solutions Inc.
