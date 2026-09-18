# ComfyUI Adaptive Media Engine

**A model-agnostic ComfyUI skill architecture for AI agents.**

The goal of this project is simple: let an AI agent decide **what kind of media workflow is needed**, select a validated ComfyUI workflow automatically, execute it through the API, monitor the job, and return the generated asset — without requiring the user to manually choose nodes, models, samplers, or workflow files every time.

Instead of building separate integrations for FLUX, SDXL, Qwen, image editing, video generation, upscaling, and future models, the skill introduces a reusable routing layer around ComfyUI:

```text
Natural-language request
        ↓
Intent + capability detection
        ↓
READY workflow profile
        ↓
Parameter injection + validation
        ↓
ComfyUI API execution
        ↓
Job monitoring + asset retrieval
```

## Why this exists

ComfyUI is extremely flexible, but that flexibility usually means the caller already needs to know which workflow, model, nodes, and parameters to use.

This project is designed to move that decision-making into the agent layer.

A user should eventually be able to say things like:

- "Create three images for a new post."
- "Turn the second image into a video."
- "Remove the background."
- "Upscale this image."
- "Use Qwen for this one."
- "Use the fastest available video workflow."

The agent can then resolve the requested capability and route it to an appropriate validated ComfyUI workflow.

## Core idea

The skill is **workflow-driven, not model-driven**.

A model such as FLUX, SDXL, or Qwen is not hard-coded into the core. Instead, each working ComfyUI pipeline is registered as a workflow profile with metadata such as:

- supported task (`text_to_image`, `image_to_video`, `upscale_image`, ...)
- media type
- required models
- required custom nodes
- mutable parameters
- hardware guidance
- workflow version
- validation state

Only workflows marked `READY` should be selected automatically.

## Main concepts

The specification covers:

- model-agnostic workflow routing
- image and video intent detection
- reusable ComfyUI API workflow templates
- `/object_info`-based node and schema validation
- workflow discovery and import
- SHA-256 workflow fingerprinting
- workflow versioning with last-known-good rollback
- model and custom-node dependency checks
- hardware-aware profile selection
- structured ComfyUI error handling
- generated asset tracking and parent/child relationships
- future multi-step pipelines such as image → edit → upscale → video

## Project status

This repository currently provides the **skill specification, architecture, operating rules, and example profile structure** for building an adaptive ComfyUI media backend.

It is intentionally designed as a foundation that can be integrated into Hermes or another agent runtime capable of calling the ComfyUI HTTP/WebSocket API.

The repository does **not** include:

- ComfyUI itself
- model weights
- custom-node packages
- private/local workflows
- a bundled agent runtime

## Requirements

You need:

- a working ComfyUI installation
- the models required by the workflows you want to use
- any custom nodes required by those workflows
- an agent/runtime capable of calling ComfyUI via HTTP and WebSocket

ComfyUI should normally remain bound to localhost unless remote access is intentionally secured.

## Installation

Place `SKILL.md` in the skill directory used by your agent/runtime.

Example:

```text
skills/
└── comfyui-adaptive-media-engine/
    └── SKILL.md
```

Then configure the ComfyUI connection through environment variables or your runtime's configuration system:

```env
COMFYUI_BASE_URL=http://127.0.0.1:8188
COMFYUI_WS_URL=ws://127.0.0.1:8188/ws
COMFYUI_DATA_DIR=./data
COMFYUI_WORKFLOW_DIR=./data/workflows
COMFYUI_IMPORT_DIR=./data/imports
COMFYUI_DEBUG_DIR=./data/debug
```

The exact installation and tool-registration steps depend on the agent runtime you use.

## Workflow lifecycle

New workflows should not become active just because they can be parsed.

The intended lifecycle is:

```text
DISCOVERED
→ ANALYZING
→ NEEDS_SETUP / TESTING
→ READY
```

A workflow should become `READY` only after its structure, dependencies, and execution path have been validated.

Changed workflows should be versioned instead of overwriting the last working version.

## Adding workflows

A new workflow can represent any supported capability, for example:

```text
text_to_image
image_edit
upscale_image
text_to_video
image_to_video
video_upscale
```

The core skill does not need to be rewritten when a new model or workflow is added. The new workflow is imported, classified, validated, versioned, and added to the registry.

See [docs/ADDING_WORKFLOWS.md](docs/ADDING_WORKFLOWS.md).

## Example profile

A minimal example profile is available here:

[`examples/profile.example.yaml`](examples/profile.example.yaml)

## Safety and privacy

Do not commit runtime or private data to a public repository.

In particular, keep the following out of source control:

- `.env` files
- API keys and access tokens
- private prompts or debug payloads
- generated private media
- local asset/history registries
- machine-specific paths containing personal information
- model weights
- workflow files containing credentials or private URLs

See [SECURITY.md](SECURITY.md) for details.

## Repository structure

```text
.
├── SKILL.md
├── README.md
├── SECURITY.md
├── CONTRIBUTING.md
├── LICENSE
├── .env.example
├── .gitignore
├── docs/
│   └── ADDING_WORKFLOWS.md
└── examples/
    └── profile.example.yaml
```

## Contributing

Contributions are welcome, especially around:

- workflow-profile design
- additional media capabilities
- safer workflow validation
- agent/runtime integrations
- video pipeline support
- model- and hardware-aware routing

Please avoid committing model weights, credentials, private URLs, or machine-specific paths.

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Released under the [MIT License](LICENSE).

## Author

**MysticalX**
