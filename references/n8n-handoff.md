# n8n handoff

Use an **Execute Command** node and call the script directly.

## Dry-run batch

```bash
/Users/wade.digital/.openclaw/workspace/skills/paperless-classify/scripts/paperless-classify classify --limit 10
```

The script prints one JSON object to stdout. That JSON includes:

- `status`
- `model`
- `dry_run`
- `total`
- `summary`
- `results[]`

Each result includes:

- `doc_id`
- `title`
- `content_source`
- `current`
- `suggested`
- `confidence`
- `needs_review`
- `reason`

## Suggested n8n pattern

1. Execute Command → `paperless-classify classify --limit 10`
2. IF node:
   - route `needs_review=true` or `confidence=low` to a human-review path
   - route the rest to approval / apply
3. Optional Code node to filter approved rows into a new JSON payload
4. Execute Command → apply reviewed payload

Example apply step:

```bash
cat /tmp/paperless-approved.json | /Users/wade.digital/.openclaw/workspace/skills/paperless-classify/scripts/paperless-classify apply
```

## Operational notes

- Keep batches small at first (`--limit 5` or `--limit 10`).
- The script talks to Paperless and Ollama locally; n8n does not need direct API logic.
- Default model is `gemma3:4b`, but `--model gpt-oss:20b` also works if higher quality matters more than speed.
- The script is idempotent enough for retries: dry-run makes no changes; apply merges tags instead of replacing them.