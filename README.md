# bedrock-api-proxy-model-mappings

Default **Anthropic model ID → Bedrock model ID** mappings for the
[Anthropic-Bedrock API Proxy](https://github.com/xiehust/sample-bedrock-api-proxy).

The proxy pulls `model_mappings.json` from this repo at startup and then
periodically (see `MODEL_MAPPING_SYNC_URL` / `MODEL_MAPPING_SYNC_INTERVAL_SECONDS`),
so adding a new model here rolls out to every deployment without a redeploy.

Raw URL used by the proxy:

```
https://raw.githubusercontent.com/xiehust/bedrock-api-proxy-model-mappings/main/model_mappings.json
```

## File format

```json
{
  "schema_version": 1,
  "description": "...",
  "mappings": {
    "<anthropic-model-id>": "<bedrock-model-id-or-inference-profile>"
  }
}
```

- Keys are the model IDs clients send (`claude-sonnet-4-5-20250929`, `gpt-5.5`,
  `claude-opus-4-7[1m]` …). Values are the Bedrock model IDs / inference-profile
  IDs the proxy forwards to.
- Non-Claude Bedrock models are usually identity-mapped (`"zai.glm-5": "zai.glm-5"`)
  so they show up in `/v1/models`.
- `[1m]` aliases point at the same Bedrock target; the 1M context window is
  activated by the `anthropic-beta: context-1m-2025-08-07` header the client sends.

## Resolution order in the proxy

1. DynamoDB `anthropic-proxy-model-mapping` table (per-deployment overrides, admin portal)
2. `DEFAULT_MODEL_MAPPING` env var entries (per-deployment local additions)
3. This file
4. Pass-through: unknown IDs are sent to Bedrock as-is

## Editing

- Keep the file valid JSON (no comments, no trailing commas); the proxy rejects
  an invalid payload and keeps the previous mapping.
- Every key and value must be a non-empty string; an empty `mappings` object is
  rejected as a safety measure.
- Validate before pushing:

  ```bash
  python3 -c 'import json,sys; d=json.load(open("model_mappings.json")); m=d["mappings"]; assert m and all(isinstance(k,str) and isinstance(v,str) and k and v for k,v in m.items()); print(len(m), "mappings OK")'
  ```
