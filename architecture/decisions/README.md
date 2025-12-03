# Architecture Decision Records (ADRs)

This directory contains Architecture Decision Records for BuildFlow. ADRs document important architectural decisions, their context, and consequences.

## What is an ADR?

An Architecture Decision Record (ADR) is a document that captures an important architectural decision made along with its context and consequences.

## Format

Each ADR follows this structure:
- **Title** - Short noun phrase
- **Status** - Proposed, Accepted, Superseded, Deprecated
- **Context** - What forces are at play?
- **Decision** - What are we doing?
- **Consequences** - What becomes easier or harder?

See [000-template.md](000-template.md) for the template.

## Index

See [SUMMARY.md](SUMMARY.md) for a complete index of all ADRs with descriptions and relationships.

## How to Add an ADR

1. Copy `000-template.md` to `XXX-your-decision-name.md`
2. Fill in the sections
3. Update `SUMMARY.md` with your new ADR
4. Create PR for review

## ADR Lifecycle

- **Proposed** - Under discussion
- **Accepted** - Approved and active
- **Superseded** - Replaced by another ADR (include link)
- **Deprecated** - No longer recommended but not replaced

## Related

- [Architecture Overview](../ARCHITECTURE.md)
- [Project Overview](../../PROJECT_OVERVIEW.md)
