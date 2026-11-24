# Open Evidence Standard (OES) Specification v1.1.0

## Status

**Draft 1.1.0** — Core v1.0.0 fields are **Stable**; Multimedia & Narrative Extensions introduced in v1.1.0 are explicitly marked **`[EXPERIMENTAL]`** (`"x-experimental": true` in the JSON Schema).

## Abstract

The Open Evidence Standard (OES) defines a JSON-based format for exchanging structured legal evidence extracted from documents, audio, and video files. It is designed to support "Zero-Trust" workflows where the analysis provider may not retain custody of the original source files.

## Terminology

- **Packet (Envelope):** The root object containing the entire analysis job's output, metadata, and cryptographic proofs.
- **Evidence Item:** A single unit of extracted information (e.g., a quote, a timestamped event, a visual observation) tied to a specific location in a source file.
- **Manifest:** A registry of source files used in the analysis, including their cryptographic hashes to ensure the analysis corresponds to the correct versions of the documents.

## Schema Reference

- **Core (v1.0.0, Stable):** [`schemas/v1/oes-packet.json`](./schemas/v1/oes-packet.json)
- **Multimedia & Narrative Extensions (v1.1.0, Experimental):** [`schemas/v1.1/oes-packet.json`](./schemas/v1.1/oes-packet.json)

### Core Components

#### 1. The Cryptographic Envelope

Every OES Packet must include an `integrity_hash`. This is a SHA-256 hash of the canonicalized `evidence` array (in v1.1.0, computed strictly over the objective record fields: `evidence_id`, `file_id`, `content`, `location`, `source_authority`, and `analysis`). This ensures that evidentiary facts cannot be tampered with after generation.

In v1.1.0, prompt-conditioned narrative evaluations may change independently of the underlying evidentiary record; their canonical SHA-256 digest is recorded separately in `metadata.narrative_overlay_hash`.

#### 2. The Source Manifest

The `source_manifest` array lists every file that contributed to the analysis. Each entry MUST include a `file_hash` (SHA-256 of the file content). This allows a court or opposing counsel to verify that the "Exhibit A" used in the analysis is bit-for-bit identical to the "Exhibit A" in the court record.

#### 3. Evidence Items

Evidence items are grouped into three core sections (plus optional experimental extensions in v1.1.0):

- **Content:** What was found (`snippet`, `visual_description`, `extracted_values`).
- **Location:** Where it was found (`page`, `line_range`, `timestamp_range`, `speaker`, `bates_labels`, plus experimental `temporal`, `spatial`, and `media_production`).
- **Analysis:** Why it matters (`relevance_score`, `explanation`, `legal_elements`, `strength`).

#### 4. Location Fields

The `location` object identifies where evidence was found within the source:

- `page`: Integer page number
- `line_range`: String range (e.g., `"15-18"`)
- `timestamp_range`: Time range string for audio/video
- `speaker`: Speaker identification for transcripts
- `bates_labels`: Array of Bates identifiers observed on the page(s)

**Bates Labels:**
Bates numbering is the standard identification system for legal document production. Each page of discovery is assigned a unique alphanumeric identifier (e.g., `"DEF-000123"`) typically stamped in the bottom-right corner. OES captures these identifiers to enable precise citation in legal briefs and motions.

#### 5. Source Authority

The `source_authority` object classifies the evidentiary weight of the originating document. This enables downstream consumers to adjust language and attribution based on the legal standing of the evidence source.

**Categories:**

| Category              | Description                                      | Authority Level |
| --------------------- | ------------------------------------------------ | --------------- |
| `SwornTestimony`      | Depositions, affidavits, declarations            | `highest`       |
| `ObjectiveRecord`     | Medical records, police reports, autopsy reports | `high`          |
| `CourtRecord`         | Court orders, judgments, case law                | `highest`       |
| `ExpertMaterial`      | Expert reports, IME reports                      | `high`          |
| `ContractualDocument` | Contracts, insurance policies, agreements        | `medium`        |
| `Discovery`           | Interrogatories, RFPs, discovery responses       | `medium`        |
| `Correspondence`      | Emails, letters, memos                           | `medium`        |
| `PartyFiling`         | Complaints, motions, briefs                      | `medium_low`    |
| `Unknown`             | Unclassified documents                           | `default`       |

**Caveat Field:**
The optional `caveat` string provides guidance on interpretation:

- `PartyFiling`: `"Contested assertion from interested party"` — indicates the evidence is a claim, not an established fact.
- `SwornTestimony`: `"Sworn under penalty of perjury"` — indicates the testimony carries legal consequence for falsehood.
- `ObjectiveRecord` / `CourtRecord`: `null` — no caveat needed for established records.

**Integration:**
`source_authority` appears on each `evidence_item` in the evidence array. The `source_manifest` also includes a `document_category` field per file for easier authority lookups at the file level.

#### 6. Multimedia & Narrative Extensions (`[EXPERIMENTAL]` in v1.1.0)

> **Experimental Stability Notice:** Every field introduced in v1.1.0 is annotated with `"x-experimental": true` and `"[EXPERIMENTAL]"` in [`schemas/v1.1/oes-packet.json`](./schemas/v1.1/oes-packet.json). These properties are additive and optional; core v1.0.0 producers are not required to emit them, and field shapes may evolve prior to v2.0.0 stabilization.

**Dual-Hash Provenance Separation:**
v1.1.0 preserves "Citation is Sovereign" by keeping immutable record provenance (`integrity_hash`) completely separate from prompt-conditioned story/emotional evaluations (`narrative_overlay_hash`). When an editor re-evaluates the same corpus against a revised outline, `integrity_hash` remains invariant across unchanged factual anchors while `narrative_overlay_hash` updates cleanly.

**Experimental Envelope & Manifest Fields:**

- `metadata.narrative_overlay_hash`, `metadata.rubric_id`, `metadata.evaluation_prompt_hash`
- `source_manifest[].mime_type`, `modality` (`video` | `audio` | `image` | `document` | `spreadsheet`), `duration_sec`, `dimensions` (`width`, `height`), `page_count`, `provenance_party`
- `coverage_matrix`: Beat-level coverage audit (`STRONG` / `THIN` / `GAP`), ranked candidate IDs, competing takes with trim ranges, `unmapped_crown_jewels`, and `recommended_retrofits` (`PULL_HOP`, `PIVOT_TO_DOC_COMPOSITE`, `PRUNE_OR_MERGE_BEAT`, `PROMOTE_CROWN_JEWEL`)

**Experimental Evidence Item Fields:**

- `location.temporal`: Exact seconds (`start_sec`, `end_sec`, `keyframe_sec`) and subtitle cues (`srt_cue_range`)
- `location.spatial`: Normalized 2D bounding box (`coordinate_space`, `box_2d`, `frame_timestamp_sec`, `target_label`)
- `location.media_production`: Production suitability (`has_clean_speech_audio`, `speech_clarity_score`, `visual_legibility`, `camera_perspective`, `visual_utility`)
- `narrative_evaluation`: Prompt-conditioned scoring (`rubric_id`, `evaluation_prompt_hash`, `must_right_passed`, calibrated `decision_stage`, gated `harmonic_score`, `affective_profile`, observable cues with falsifier warnings, and `arc_slot_bindings`)

## Compliance

To be OES Compliant, a tool must:

1. Output valid JSON according to the OES Schema (`v1` for core packets, or `v1.1` when using experimental multimedia/narrative fields).
2. Include valid `file_hash` values for all source files.
3. Include a valid `integrity_hash` for the evidence payload.

## Changelog

- **v1.1.0 (Experimental Multimedia Extensions + v1.0.0 Schema Fix):**
  - Added [`schemas/v1.1/oes-packet.json`](./schemas/v1.1/oes-packet.json) and [`examples/media_narrative_packet.json`](./examples/media_narrative_packet.json) with `[EXPERIMENTAL]` (`x-experimental: true`) multimedia localization (`temporal`, `spatial`, `media_production`), media manifest metadata, narrative evaluation overlays (`narrative_evaluation`), dual-hash integrity separation, and `coverage_matrix`.
  - Fixed pre-existing schema bug in [`schemas/v1/oes-packet.json`](./schemas/v1/oes-packet.json): removed undeclared top-level `relevance_score` from `$defs.evidenceItem.required` (preserving `analysis.relevance_score` as required) and made `extracted_values[].unit` nullable (`["string", "null"]`) to match `examples/deposition_packet.json`.
