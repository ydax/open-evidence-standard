# Open Evidence Standard (OES) Specification v1.0.0

## Status

**Draft 1.0.0**

## Abstract

The Open Evidence Standard (OES) defines a JSON-based format for exchanging structured legal evidence extracted from documents, audio, and video files. It is designed to support "Zero-Trust" workflows where the analysis provider may not retain custody of the original source files.

## Terminology

- **Packet (Envelope):** The root object containing the entire analysis job's output, metadata, and cryptographic proofs.
- **Evidence Item:** A single unit of extracted information (e.g., a quote, a timestamped event, a visual observation) tied to a specific location in a source file.
- **Manifest:** A registry of source files used in the analysis, including their cryptographic hashes to ensure the analysis corresponds to the correct versions of the documents.

## Schema Reference

The normative schema is located at [`schemas/v1/oes-packet.json`](./schemas/v1/oes-packet.json).

### Core Components

#### 1. The Cryptographic Envelope

Every OES Packet must include an `integrity_hash`. This is a SHA-256 hash of the canonicalized `evidence` array. This ensures that the analysis has not been tampered with after generation.

#### 2. The Source Manifest

The `source_manifest` array lists every file that contributed to the analysis. Each entry MUST include a `file_hash` (SHA-256 of the file content). This allows a court or opposing counsel to verify that the "Exhibit A" used in the analysis is bit-for-bit identical to the "Exhibit A" in the court record.

#### 3. Evidence Items

Evidence items are grouped into three logical sections:

- **Content:** What was found (snippet, visual description, extracted values).
- **Location:** Where it was found (page, line range, timestamp, Bates labels).
- **Analysis:** Why it matters (relevance, legal elements, strength).

#### Location Fields

The `location` object identifies where evidence was found within the source:

- `page`: Integer page number
- `line_range`: String range (e.g., "15-18")
- `timestamp_range`: Time range for audio/video
- `speaker`: Speaker identification for transcripts
- `bates_labels`: Array of Bates identifiers observed on the page(s)

**Bates Labels:**
Bates numbering is the standard identification system for legal document production. Each page of discovery is assigned a unique alphanumeric identifier (e.g., "DEF-000123") typically stamped in the bottom-right corner. OES captures these identifiers to enable precise citation in legal briefs and motions.

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

## Compliance

To be OES Compliant, a tool must:

1.  Output valid JSON according to the OES Schema.
2.  Include valid `file_hash` values for all source files.
3.  Include a valid `integrity_hash` for the evidence payload.
