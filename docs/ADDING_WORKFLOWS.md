# Adding Workflows

The core skill is model-agnostic. New image, video, editing, or upscale capabilities are added as workflow profiles.

## Recommended process

1. Export or obtain a ComfyUI **API workflow**.
2. Place it in your configured import directory.
3. Normalize and fingerprint the workflow.
4. Inspect all `class_type` values through `/object_info`.
5. Detect model and custom-node requirements.
6. Classify the media type and task.
7. Define mutable parameters.
8. Run structural validation.
9. Run a smoke test.
10. Register the workflow as `READY` only after success.

## Example states

```text
DISCOVERED
ANALYZING
NEEDS_SETUP
TESTING
READY
DEGRADED
BROKEN
DISABLED
```

## Versioning

Never overwrite your only working version.

```text
workflow-name/
├── v1.json
├── v2.json
└── v3.json
```

Keep an `active_version` and `last_known_good`.

## Validation

For every connection:

```json
["SOURCE_NODE_ID", OUTPUT_INDEX]
```

verify:

```python
0 <= OUTPUT_INDEX < len(object_info[source_class]["output"])
```

Also verify:

- every referenced node exists
- there are no self-references
- input names exist
- enum values are valid
- required models are visible to ComfyUI
- required custom nodes are available
