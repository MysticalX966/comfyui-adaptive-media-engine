---
name: comfyui-adaptive-media-engine
description: "Use when generating images, videos, editing, upscaling, or otherwise rendering media through ComfyUI. Provides adaptive intent routing, workflow selection, validation, execution, workflow learning, and asset tracking."
compatibility: "Requires a reachable ComfyUI API and a runtime capable of executing the selected workflow."
metadata:
  author: MysticalX
  version: "1.0.0"
  category: media-rendering
  backend: ComfyUI
  model_agnostic: true
---

# ComfyUI Adaptive Media Engine

## Purpose

This skill turns ComfyUI into an adaptive, model-agnostic media backend.

It is not tied to FLUX, SDXL, Qwen, or any single model family. Model-specific behavior belongs in workflow profiles. The core skill remains generic.

Core flow:

```text
User Request
→ Intent Detection
→ Media Type + Task
→ Capability Matching
→ READY Workflow Profile
→ API Template
→ Parameter Injection
→ Validation
→ ComfyUI Execution
→ Job Monitoring
→ Asset Retrieval
→ Asset Registration
```

The user should not need to know node names, workflow JSON, samplers, schedulers, API endpoints, or internal ComfyUI graph details.

---

# Trigger

Use this skill when a request requires ComfyUI for:

- text-to-image
- image-to-image
- image editing
- inpainting/outpainting
- background replacement/removal
- upscaling
- text-to-video
- image-to-video
- video-to-video
- video upscaling/interpolation
- LoRA or ControlNet workflows
- importing or validating ComfyUI workflows
- discovering installed ComfyUI models or node capabilities
- selecting an appropriate workflow automatically

Examples:

- "Create an image for a new post."
- "Turn the second image into a video."
- "Make the wheels black."
- "Remove the background."
- "Upscale this."
- "Use Qwen for this one."
- "Use the fast workflow."
- "Render the final version at maximum quality."

---

# Configuration

Do not hard-code machine-specific paths.

Use environment/configuration values:

```text
COMFYUI_BASE_URL=http://127.0.0.1:8188
COMFYUI_WS_URL=ws://127.0.0.1:8188/ws
COMFYUI_DATA_DIR=<configurable data directory>
COMFYUI_WORKFLOW_DIR=<configurable workflow directory>
COMFYUI_IMPORT_DIR=<configurable import directory>
COMFYUI_DEBUG_DIR=<configurable debug directory>
```

Localhost is the safe default.

Do not expose an unauthenticated ComfyUI instance directly to the public internet.

---

# Non-Negotiable Rules

1. Never guess ComfyUI node schemas. Use `/object_info`.
2. Never replace a known-good template with an untested graph.
3. Never send normal UI workflow JSON directly to `/prompt`.
4. Never report only `HTTP 400 Bad Request`; preserve and inspect the response body.
5. Never overwrite the last-known-good workflow version.
6. Never route a request to an unrelated workflow just to produce output.
7. Never assume multiple GPUs automatically pool VRAM.
8. Never hard-code model-family behavior into the API core.
9. Never require manual UI fallback merely because one API attempt failed; debug the concrete failure.
10. Never install unknown custom nodes, models, or executable dependencies automatically without explicit authorization.
11. Never commit runtime secrets, debug payloads, generated assets, or local environment files into a public repository.

---

# API Core

## Health

```http
GET /system_stats
```

Use for reachability and runtime state.

## Introspection

```http
GET /object_info
GET /object_info/<class_type>
```

Use to discover:

- node classes
- required inputs
- optional inputs
- enums
- model filenames exposed by loaders
- output types

ComfyUI's output type list is in:

```json
"output": ["TYPE"]
```

For a connection:

```json
["2", 0]
```

the output index is valid only if:

```python
0 <= output_index < len(node_info["output"])
```

If a single-node endpoint wraps the node:

```json
{
  "CLIPLoader": {
    "output": ["CLIP"]
  }
}
```

unwrap it before validating:

```python
node_info = response["CLIPLoader"]
```

## Queue Prompt

```http
POST /prompt
```

Payload:

```json
{
  "prompt": {
    "1": {
      "class_type": "SomeNode",
      "inputs": {}
    }
  },
  "client_id": "<UUID>"
}
```

## Monitor

Preferred:

```text
ws://<host>/ws?clientId=<UUID>
```

Fallback:

```http
GET /history/<prompt_id>
```

## Retrieve Outputs

Read output metadata from history and retrieve via:

```http
GET /view
```

Do not fabricate output paths.

---

# API Workflow Format

API workflows are dictionaries keyed by node ID.

Example:

```json
{
  "1": {
    "class_type": "SomeModelLoader",
    "inputs": {
      "model_name": "example-model"
    }
  },
  "2": {
    "class_type": "SomeConsumerNode",
    "inputs": {
      "model": ["1", 0]
    }
  }
}
```

Connections are:

```json
["SOURCE_NODE_ID", OUTPUT_INDEX]
```

Do not send UI-only graph metadata such as positions, sizes, links, or widget state to `/prompt`.

---

# Intent Router

Classify the user request into a normalized intent:

```yaml
media_type: image | video | audio | 3d | multimodal
task: capability_name
source_assets: []
purpose: optional
quality_preset: draft | fast | balanced | quality | maximum
explicit_profile: optional
explicit_model: optional
constraints: {}
```

Examples:

```text
"Create an image for a post."
→ image / text_to_image
```

```text
"Turn that image into a video."
→ video / image_to_video
```

```text
"Change the wheels to black."
→ image / image_edit
```

```text
"Make it larger and sharper."
→ image / upscale_image
```

Context references such as "that", "the second image", "the previous version", and "the original" should resolve through the asset registry.

---

# Capability Taxonomy

## Image

```text
text_to_image
image_to_image
image_edit
inpainting
outpainting
background_replace
remove_background
product_image
automotive_image
social_image
character_image
reference_image
upscale_image
```

## Video

```text
text_to_video
image_to_video
video_to_video
video_extend
video_upscale
video_interpolation
video_product_ad
social_video
```

## Future

The architecture should allow additional media types such as:

```text
audio
speech
music
3d
animation
multimodal
```

---

# User Overrides

Automatic routing is the default.

Explicit user instructions take priority:

```text
"Use FLUX."
"Use Qwen."
"Use workflow XYZ."
"Use the fast video workflow."
```

If the requested profile cannot run, return the exact missing dependency or state.

Do not silently replace an explicitly requested model or profile unless fallback is authorized.

---

# Workflow Registry

Each reusable workflow needs machine-readable metadata.

Example:

```yaml
id: example-image-profile
name: Example Image Profile

media_type:
  - image

tasks:
  - text_to_image

capabilities:
  - photorealistic
  - product

status: READY

workflow:
  template: workflows/image/text_to_image/example/v1.json
  active_version: 1
  last_known_good: 1

model_family:
  - example-family

loader:
  type: standard

hardware:
  min_vram_gb: 8
  recommended_vram_gb: 12

requirements:
  models: []
  nodes: []

parameters: {}

outputs:
  - image

quality:
  default_preset: balanced

priority: 50

workflow_hash: "<sha256>"
created_at: "<timestamp>"
updated_at: "<timestamp>"
```

---

# Workflow Lifecycle

Supported states:

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

Only `READY` workflows may be auto-selected.

---

# Workflow Selection

When multiple READY workflows match, consider:

1. explicit profile/model override
2. media type
3. exact task
4. source-input compatibility
5. requested capabilities
6. model availability
7. custom-node availability
8. hardware compatibility
9. quality preset
10. speed preference
11. requested resolution/duration
12. configured profile priority

Do not select randomly.

---

# Quality Presets

Support:

```text
draft
fast
balanced
quality
maximum
```

Profiles may map these to different models, quantizations, resolutions, step counts, or post-processing stages.

Do not assume universal sampler/CFG/step behavior across model families.

---

# Workflow Auto-Discovery

The workflow library should evolve without rewriting the core skill.

Scan configured workflow and import directories for:

- API workflow JSON
- successful workflows produced by the system
- user-provided workflow exports
- workflows built or repaired by the agent

A newly discovered workflow must not become READY merely because it parses.

---

# Workflow Onboarding Pipeline

```text
DISCOVER
→ PARSE
→ NORMALIZE
→ FINGERPRINT
→ INTROSPECT NODES
→ DETECT MODELS
→ DETECT CUSTOM NODE REQUIREMENTS
→ INFER MEDIA TYPE / TASK
→ INFER INPUTS / OUTPUTS
→ CHECK HARDWARE
→ STRUCTURAL VALIDATION
→ SMOKE TEST
→ REGISTER VERSION
→ READY
```

If dependencies are missing:

```text
NEEDS_SETUP
```

If validation fails:

```text
BROKEN
```

If a previously working workflow loses dependencies:

```text
DEGRADED
```

---

# Workflow Classification

Infer:

- media type
- task
- required inputs
- outputs
- model family
- required models
- required nodes
- likely capabilities

Mark inferred metadata:

```yaml
source: inferred
confidence: 0.82
```

Inference is not authoritative until validation succeeds.

---

# Workflow Fingerprinting

Normalize API workflow JSON and calculate SHA-256.

Store:

```text
workflow_hash
created_at
updated_at
version
source
active_version
last_known_good
```

Use fingerprints to detect duplicates and changed versions.

---

# Workflow Versioning

Never destroy the only working template.

Example:

```text
example-profile/
├── v1.json
├── v2.json
└── v3.json
```

Registry:

```yaml
active_version: 3
last_known_good: 2
```

New versions start in `TESTING`.

Promote only after validation.

Rollback to `last_known_good` on failed upgrades.

---

# Registry Refresh

Provide behavior equivalent to:

```python
comfy.refresh_registry()
```

It should:

1. scan configured workflow directories
2. scan import directories
3. normalize workflows
4. compare hashes
5. detect new versions
6. check node availability
7. check model availability
8. update profile state
9. update hardware compatibility
10. update capability metadata
11. avoid unnecessary full renders

---

# Validation Levels

## STRUCTURAL_VALIDATION

Use frequently.

Check:

- JSON/API graph validity
- node references
- no self references
- class types exist
- output indexes valid
- required inputs
- enum values
- model filenames
- required nodes

## FULL_RENDER_TEST

Use when:

- onboarding a new workflow
- validating a changed workflow
- critical ComfyUI/custom-node updates occurred
- explicit verification is requested

Do not burn GPU time on every startup.

---

# Graph Validator

Reference integrity:

```python
def validate_references(prompt):
    for target_id, node in prompt.items():
        for input_name, value in node.get("inputs", {}).items():
            if is_connection(value):
                source_id, output_index = value
                if source_id not in prompt:
                    raise ValidationError(
                        f"{target_id}.{input_name} references missing node {source_id}"
                    )
```

No self-reference:

```python
if source_id == target_id:
    raise ValidationError(
        f"{target_id}.{input_name} self-references node {source_id}"
    )
```

Output index:

```python
def validate_output_index(source_node, index, object_info):
    class_type = source_node["class_type"]
    info = object_info[class_type]
    outputs = info["output"]

    if index < 0 or index >= len(outputs):
        raise ValidationError(
            f"Invalid output index {index} for {class_type}; outputs={outputs}"
        )
```

When debugging, emit:

```text
SOURCE | OUT | TARGET | INPUT
```

---

# Error Handling

Never discard ComfyUI error responses.

Capture:

```text
http_status
response_body
error
node_errors
node_id
class_type
input_name
expected
received
traceback
```

Classify errors:

```text
CONNECTION_ERROR
API_VALIDATION_ERROR
WORKFLOW_ERROR
MODEL_MISSING
MODEL_LOAD_ERROR
CUDA_ERROR
VRAM_OOM
SAMPLING_ERROR
VAE_ERROR
TEXT_ENCODER_ERROR
CUSTOM_NODE_ERROR
NO_READY_WORKFLOW
PROFILE_NOT_READY
UNKNOWN_RUNTIME_ERROR
```

Runtime/debug artifacts must stay outside source control.

---

# Generic Execution Algorithm

```python
def execute_media_job(request):
    intent = classify_intent(request)

    profile = registry.select_ready_profile(
        media_type=intent.media_type,
        task=intent.task,
        capabilities=intent.capabilities,
        explicit_profile=intent.explicit_profile,
        explicit_model=intent.explicit_model,
        hardware=current_hardware(),
    )

    if profile is None:
        raise NoReadyWorkflow(intent.task)

    verify_profile_dependencies(profile)

    graph = load_json(profile.active_template)

    graph = inject_registered_parameters(
        graph=graph,
        mapping=profile.parameters,
        values=intent.parameters,
    )

    validation = validate_api_graph(
        graph=graph,
        object_info=get_required_object_info(graph),
    )

    client_id = new_uuid()

    response = post_json(
        "/prompt",
        {
            "prompt": graph,
            "client_id": client_id,
        },
    )

    prompt_id = response["prompt_id"]

    result = monitor_job(
        prompt_id=prompt_id,
        client_id=client_id,
    )

    if result.failed:
        classify_and_persist_error(result)
        raise RenderError(result)

    outputs = get_history_outputs(prompt_id)
    assets = retrieve_outputs(outputs)

    return register_assets(
        assets=assets,
        workflow=profile.id,
        workflow_version=profile.active_version,
        request=request,
        prompt_id=prompt_id,
    )
```

Implementation language may differ. Behavior should remain equivalent.

---

# Generic Image Interface

```python
def generate_image(
    prompt,
    negative_prompt="",
    width=1024,
    height=1024,
    seed=None,
    steps=None,
    cfg=None,
    sampler=None,
    profile="auto",
    quality="balanced",
    filename_prefix="Agent",
):
    request = {
        "media_type": "image",
        "task": "text_to_image",
        "prompt": prompt,
        "negative_prompt": negative_prompt,
        "width": width,
        "height": height,
        "seed": seed or random_seed(),
        "steps": steps,
        "cfg": cfg,
        "sampler": sampler,
        "profile": profile,
        "quality": quality,
        "filename_prefix": filename_prefix,
    }

    return execute_media_job(request)
```

Inject only parameters declared mutable by the selected profile.

---

# Generic Video Interface

```python
def generate_video(
    prompt,
    source_image=None,
    duration=None,
    fps=None,
    profile="auto",
    quality="balanced",
):
    ...
```

Routing:

```text
no source image → text_to_video
source image    → image_to_video
```

A video workflow is just another registered profile with different inputs/outputs.

---

# Asset Registry

Each generated asset should record:

```yaml
asset_id: string
file_path: string
media_type: image | video | other
created_at: timestamp
workflow_id: string
workflow_version: integer
model: optional
prompt: optional
seed: optional
width: optional
height: optional
parent_asset_id: optional
job_id: optional
```

Video may additionally store:

```text
duration
fps
frames
codec
```

Runtime asset registries should not be committed publicly.

---

# Parent / Child Assets

Track transformations:

```text
asset_001.png
→ upscale
asset_002.png
→ image_to_video
asset_003.mp4
```

This enables contextual follow-ups such as:

- "Turn that into a video."
- "Use the original."
- "Upscale the second version."

---

# Media Pipeline Composition

A request may use multiple READY workflows.

Example image pipeline:

```text
text_to_image
→ image_edit
→ upscale_image
→ final image
```

Example video pipeline:

```text
source image
→ image_to_video
→ video_upscale
→ interpolation
→ final video
```

Each stage must independently use a READY profile.

---

# Model Discovery

Provide behavior equivalent to:

```python
comfy.list_models()
```

Prefer model values exposed by ComfyUI loader schemas.

Useful categories:

```text
checkpoints
diffusion_models
unet
gguf
text_encoders
vae
loras
controlnet
upscale_models
```

Filesystem scanning may supplement discovery but does not prove loadability.

---

# Custom Node Awareness

Before execution, verify every required `class_type`.

If a dependency is unavailable, return a structured state:

```text
PROFILE_NOT_READY
Missing:
- node: SomeCustomNode
- extension: SomeExtension
```

Do not rely on an opaque runtime import crash when the dependency can be detected in advance.

---

# Example Profile: GGUF Text-to-Image

This is an example profile pattern only. Model filenames and nodes must be verified against the local installation.

```yaml
id: example-gguf-t2i
media_type:
  - image
tasks:
  - text_to_image
status: NEEDS_SETUP
loader:
  type: gguf
requirements:
  models:
    - <diffusion-model.gguf>
    - <text-encoder.safetensors>
    - <vae.safetensors>
  nodes:
    - UnetLoaderGGUF
    - CLIPLoader
    - CLIPTextEncode
    - VAELoader
    - VAEDecode
    - SaveImage
```

Do not publish or hard-code local model file paths.

---

# Workflow Auto-Save

When a workflow executes successfully and is not yet registered:

1. normalize the API workflow
2. calculate its fingerprint
3. detect duplicates
4. save a versioned template
5. inspect class types
6. determine required models
7. determine required custom nodes
8. infer tasks and capabilities
9. identify mutable parameters
10. structurally validate
11. smoke test when appropriate
12. create/update the registry profile
13. set READY only after success

---

# Workflow Import

Provide behavior equivalent to:

```python
comfy.import_workflow(
    path="...",
    name=None,
    task="auto",
)
```

`task="auto"` means infer then validate.

Explicit override should be supported:

```python
task="image_to_video"
```

Imported workflows must never overwrite a known-good version without versioning.

---

# Self-Updating Semantics

"Self-updating" means the workflow library and registry evolve safely.

Allowed:

- discover workflows
- classify them
- validate them
- version them
- detect model availability changes
- detect node availability changes
- promote/demote profile states
- auto-select newly READY profiles

Not implied:

- arbitrary core-code rewrites
- unreviewed software installation
- deleting previous versions
- bypassing safeguards

The core remains stable. The workflow library evolves.

---

# Missing Capability Behavior

If the requested capability has no READY profile:

```text
NO_READY_WORKFLOW
task=<requested task>
```

Then optionally inspect:

- imported workflows
- `NEEDS_SETUP` profiles
- installed models
- available nodes

Do not substitute a mismatched workflow.

---

# Hardware Awareness

Use `/system_stats` and optional platform tools for runtime hardware discovery.

Profile metadata may include:

```yaml
hardware:
  min_vram_gb: 8
  recommended_vram_gb: 12
```

Runtime evidence can refine compatibility metadata.

For OOM:

1. classify as `VRAM_OOM`
2. preserve error details
3. use a lower-memory READY profile only if fallback is allowed
4. never silently downgrade an explicitly requested model

---

# Job States

Support at least:

```text
COMFY_OFFLINE
COMFY_ONLINE
PROFILE_READY
PROFILE_MISSING_MODEL
PROFILE_MISSING_NODE
JOB_QUEUED
JOB_RUNNING
JOB_COMPLETE
JOB_FAILED
```

---

# Regression Tests

Minimum production checks:

1. `/system_stats` succeeds.
2. `/object_info` succeeds.
3. registered models are discoverable.
4. template passes structural validation.
5. first real generation completes.
6. second generation with changed prompt/seed completes using the same template.
7. image intent routes to `text_to_image`.
8. video intent without a READY video workflow returns `NO_READY_WORKFLOW`.
9. imported workflow is discovered.
10. changed workflow produces a new version and preserves last-known-good.

---

# Separation of Responsibilities

This skill is media-rendering infrastructure.

Do not embed unrelated:

- ecommerce logic
- CRM logic
- social publishing
- accounting
- campaign management
- project-specific dashboard logic

Other agents may consume outputs from this skill.

---

# Security and Privacy

Public/shared implementations must:

- keep secrets in environment variables or a secret manager
- never store access tokens in workflow JSON
- never commit `.env`
- never commit runtime debug payloads
- never commit generated user assets by default
- never commit asset registries containing personal/local file metadata
- avoid machine-specific absolute paths
- default to localhost for unauthenticated ComfyUI
- avoid exposing ComfyUI publicly without authentication/reverse-proxy controls
- review imported workflows and custom-node dependencies before installation

---

# Definition of Done

The skill is operational when a natural-language request can:

```text
detect intent
→ select READY profile
→ load known-good template
→ inject parameters
→ validate
→ POST /prompt
→ monitor execution
→ retrieve real output
→ register the asset
→ return the resulting file
```

without manual ComfyUI graph editing.

When a new video workflow is later imported, validated, and marked READY, an image-to-video request should automatically switch to that workflow without rewriting this core skill.

---

# Final Principle

ComfyUI is not one model or one workflow.

Treat it as a local media execution engine with a growing library of validated capabilities.

The core stays stable.

The registry learns.

The workflow library evolves.

Natural-language media requests are routed to the correct validated pipeline automatically.
