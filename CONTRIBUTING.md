# Contributing to the Open Evidence Standard

Thank you for your interest in contributing to OES. This project exists to serve the public interest—to help courts, defenders, and legal technologists build AI systems that are transparent, verifiable, and grounded in truth.

## Philosophy

Before contributing, please understand the core principles:

1. **Citation is Sovereign**: If an AI cannot cite a specific location (page, line, timestamp), it is not evidence—it is hearsay.
2. **Structured over Conversational**: Legal analysis should be machine-readable data, not ephemeral chat logs.
3. **Auditability**: Every packet must cryptographically bind the analysis to the source files.

## How to Contribute

### Reporting Issues

If you find:
- Ambiguities in the specification
- Missing fields that are critical for legal use cases
- Security concerns with the cryptographic envelope

Please open an issue with:
- A clear description of the problem
- A concrete use case (e.g., "In deposition analysis, we need...")
- A proposed solution (if you have one)

### Proposing Schema Changes

The schema is versioned. Breaking changes require a new major version (e.g., 1.0.0 → 2.0.0).

To propose a change:
1. Open an issue describing the problem
2. Submit a pull request with:
   - Updated schema in `schemas/v{VERSION}/`
   - Updated SPECIFICATION.md
   - Example JSON demonstrating the new field/structure
   - Migration guide (if breaking change)

### SDK Contributions

We welcome SDK implementations in any language. To contribute:
1. Create a directory under `sdk/{language}/`
2. Implement schema validation
3. Include tests with example packets
4. Add installation instructions to README

## Code of Conduct

This is a technical standard for the public good. Contributions should be:
- **Non-commercial in spirit**: We want OES to be adopted widely, not controlled by any vendor
- **Technically rigorous**: Legal systems require precision
- **Respectful**: We're building infrastructure for justice

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

