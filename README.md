# The Open Evidence Standard (OES)

**The Courtroom is not a Chatroom.**

We are witnessing a collision between the probabilistic nature of Artificial Intelligence and the determinism required by the Justice System. While "Chat with your PDF" tools are impressive, they are fundamentally unsuited for legal discovery. They are ephemeral, unverifiable, and often hallucinate.

**Justice requires a Record.**

The **Open Evidence Standard (OES)** is a technical specification for structured, verifiable, and cited legal evidence. It is an open-source data format designed to bridge the gap between raw documents, multimedia assets, and legal reasoning.

### The Philosophy: Zero-Trust and Chain of Custody

This standard is built on three non-negotiable principles:

1. **Citation is Sovereign:** No claim exists without a pointer to a specific page, line, frame, or timestamp. If the AI cannot point to it, it is not evidence; it is hearsay. OES supports Bates numbering—the legal standard for document identification—enabling precise citation using industry-recognized identifiers.
2. **Structured over Conversational:** Legal analysis is data, not dialogue. It should be machine-readable, sortable, and filterable.
3. **Auditability:** Every packet of evidence must declare its provenance. The OES Envelope uses cryptographic hashes to bind the analysis to the source files, allowing verification even in "Zero-Trust" environments where the original files are not stored by the AI vendor.

### Versions

- **v1.0.0 Core (Stable):** [`schemas/v1/oes-packet.json`](./schemas/v1/oes-packet.json) — Document, deposition, Bates, and `source_authority` standard ([example](./examples/deposition_packet.json)).
- **v1.1.0 Multimedia & Narrative Extensions (`[EXPERIMENTAL]`):** [`schemas/v1.1/oes-packet.json`](./schemas/v1.1/oes-packet.json) — Adds optional, explicitly marked `"x-experimental": true` fields for video/audio/image localization (`temporal`, `spatial`, `media_production`), media manifest metadata, separated narrative/emotional-arc scoring (`narrative_evaluation` + `narrative_overlay_hash`), and `coverage_matrix` ([example](./examples/media_narrative_packet.json)).

### Why adopt OES?

- **For Courts:** Reject "black box" AI. Demand OES-compliant outputs that allow you to verify the source of every claim.
- **For Defenders:** Own your work product. Don't lock your case strategy inside a proprietary chat interface.
- **For Developers:** Stop reinventing the wheel. Build on a battle-tested schema that handles the complexity of depositions, medical records, and trial media.

### Getting Started

See the [Specification](./SPECIFICATION.md) for implementation details.

The JSON Schemas are available in the [`schemas`](./schemas) directory.

### License

This project is licensed under the MIT License - see the LICENSE file for details.
