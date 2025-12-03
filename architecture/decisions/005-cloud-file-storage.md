# ADR 005: Cloud-Based File Storage

**Status:** Accepted

**Date:** 2024-11-12

---

## Context

BuildFlow needs to store:
- Project photos (progress photos, before/after)
- Documents (contracts, invoices, quotes as PDFs)
- Company logos
- User avatars

Photos and documents are core features that drive significant storage needs.

---

## Decision

**Use cloud object storage (AWS S3 / Cloudflare R2 / DigitalOcean Spaces) for all user-uploaded files.**

### Implementation
- Store only file metadata in database (filename, size, mime type, url)
- Actual files stored in S3-compatible object storage
- Use CDN for fast global access
- Generate signed URLs for private files
- Implement file size limits per tier

---

## Consequences

### Positive
- ✅ Unlimited scalability
- ✅ CDN for fast access
- ✅ Offloads server storage
- ✅ Cost-effective (pay for usage)
- ✅ High durability (99.999999999%)
- ✅ Built-in backups

### Negative
- ⚠️ External dependency
- ⚠️ Cost increases with usage
- ⚠️ Requires API configuration

---

## Alternatives Considered

**Local Filesystem**: Rejected - doesn't scale, backup issues  
**Database BLOB**: Rejected - terrible performance  
**Self-hosted S3 (MinIO)**: Rejected - adds infrastructure complexity

---

**Related:** ADR-009 (storage limits per tier)
