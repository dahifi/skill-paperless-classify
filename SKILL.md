---
name: paperless-classify
description: Classify unprocessed or partially classified Paperless-ngx documents with local Ollama models, then optionally apply correspondents, document types, and normalized tags back through the Paperless API. Use when a user asks to batch-tag Paperless docs, recreate the local Gemma/Paperless workflow, dry-run current unprocessed documents, or hand a Paperless classification command off to n8n.
---

Use `scripts/paperless-classify`.

## Quick start

List current unclassified docs:

```bash
scripts/paperless-classify list
```

Dry-run classification for everything currently missing a correspondent or document type:

```bash
scripts/paperless-classify classify
```

Dry-run a bounded batch (best for n8n or first-pass testing):

```bash
scripts/paperless-classify classify --limit 10
```

Dry-run specific docs:

```bash
scripts/paperless-classify classify --ids 223,221,219
```

Apply a saved dry-run JSON after review:

```bash
scripts/paperless-classify classify > /tmp/paperless-dry-run.json
cat /tmp/paperless-dry-run.json | scripts/paperless-classify apply
```

Apply immediately in one pass only when the user asked for real changes:

```bash
scripts/paperless-classify classify --apply
```

## What the script does

1. Pull documents from the Paperless API.
2. Filter to docs missing a correspondent or document type.
3. Use Paperless OCR text when available.
4. Fall back to downloading the PDF and extracting text locally with `pdftotext` when OCR text is missing.
5. Ask a local Ollama model for:
   - correspondent
   - document type
   - normalized tags
   - confidence
   - review flag
6. Normalize the result to the live Paperless taxonomy before emitting JSON.
7. When applying, create missing correspondents, set document type, and merge tags without removing existing tags.

## Configuration

Set via env vars or `~/.config/paperless-classify.json`. See `.env.sample` for the full list.

Required: `PAPERLESS_URL`, `PAPERLESS_PASS`
Defaults: `OLLAMA_URL=http://127.0.0.1:11434`, `OLLAMA_MODEL=gemma3:4b`

## Safety rules

- Default to dry-run.
- Review `needs_review=true` and any low-confidence rows before applying.
- Do not remove existing tags; merge new normalized tags in.
- Create new correspondents only when the document clearly points to a new issuer.

## n8n handoff

Read `references/n8n-handoff.md` when wiring this into n8n.