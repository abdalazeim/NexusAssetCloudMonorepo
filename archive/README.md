<div align="center">

┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│ ╔══════════════════════════════════════════════════════════════════════════════╗
│ ║                    NEXUSASSET CLOUD — ARCHIVE                              ║
│ ║            Historical Inputs · Traceability · Preserved Summaries            ║
│ ╚══════════════════════════════════════════════════════════════════════════════╝
└──────────────────────────────────────────────────────────────────────────────┘

| ◈ Purpose | ◈ Authority | ◈ Secrets | ◈ Format |
|:---|:---|:---:|:---:|
| Preserve historical/traceability artifacts | Non-authoritative (reference only) | Never stored | English, Markdown |

</div>

---

## ◈ Why This Folder Exists

┌──────────────────────────────────────────────────────────────────────────────┐
│  The `archive/` folder preserves historical inputs and migration evidence so  │
│  the NexusAsset Cloud team can trace how specifications evolved, without      │
│  treating draft material as the current source of truth.                     │
│                                                                              │
│  AUTHORITATIVE SOURCES (use these for implementation):                        │
│  ▸ `README.md` (root)                                                         │
│  ▸ `docs/00-overview` .. `docs/14-appendices`                                │
│  ▸ `prompts/00` .. `prompts/13`                                               │
│  ▸ `project-meta/` governance and descriptors                                 │
│  ▸ `platform/` target structure/specification artifacts                       │
└──────────────────────────────────────────────────────────────────────────────┘

---

## ◈ Contents

| ◈ File | Purpose |
|---|---|
| `README.md` | Archive scope, authority, and usage rules. |
| `ARCHIVE_INVENTORY.md` | Inventory of archive contents and related authoritative locations. |
| `REPOSITORY_TRANSFORMATION_LOG.md` | Summary of major repository organization/normalization actions. |
| `HISTORICAL_SOURCE_NOTES.md` | Notes about archived drafts, mixed-language normalization, and traceability. |
| `CANONICAL_NAMING_SNAPSHOT.md` | Preserved snapshot of NexusAsset Cloud canonical naming standards. |
| `PROMPT_PACK_SNAPSHOT.md` | Preserved snapshot of phased execution prompt intent (00–13). |
| `EVALUATION_CONTEXT_SNAPSHOT.md` | Preserved context for evaluation artifacts stored at repository root. |

---

## ◈ Archive Rules

1. **Non-authoritative:** Archive files inform history; they do not override `docs/`, `prompts/`, or approved ADRs.
2. **English-only:** Archived summaries must be professional English.
3. **No secrets:** Never store credentials, tokens, private keys, or real `NXA_*` values.
4. **No executable source requirement:** Archive stores traceability/specification artifacts, not production runtime code.
5. **Traceability:** When a draft influences a final spec, record the mapping in `ARCHIVE_INVENTORY.md`.

> Use `archive/` to understand provenance. Use `docs/` and `prompts/` to build NexusAsset Cloud.
