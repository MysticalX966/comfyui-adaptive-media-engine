# First GitHub Publish Guide

## Recommended repository settings

**Repository name**

```text
comfyui-adaptive-media-engine
```

**Description**

```text
Adaptive, model-agnostic ComfyUI skill for AI agents with intent routing, workflow discovery, validation, versioning, and image/video pipeline automation.
```

**Visibility**

```text
Public
```

**Suggested topics**

```text
comfyui
ai-agent
hermes-agent
workflow-automation
generative-ai
image-generation
video-generation
local-ai
```

## Easiest method: GitHub website

1. Sign in to GitHub.
2. Create a **New repository**.
3. Name it `comfyui-adaptive-media-engine`.
4. Add the description above.
5. Select **Public**.
6. Because this package already contains a README, `.gitignore`, and LICENSE, leave GitHub's optional initialization files disabled.
7. Create the repository.
8. Choose **Add file → Upload files**.
9. Upload the **contents** of the extracted repository folder, not the surrounding ZIP file.
10. Use a first commit message such as:

```text
Initial public release
```

11. Commit the files.
12. Open the repository page and verify that `README.md` renders correctly.

## Optional method: PowerShell + Git

After creating an empty repository on GitHub:

```powershell
Set-Location "<PATH-TO-EXTRACTED-REPOSITORY>"

git init
git branch -M main
git add .
git commit -m "Initial public release"

git remote add origin https://github.com/<YOUR-GITHUB-USERNAME>/comfyui-adaptive-media-engine.git
git push -u origin main
```

If Git asks for your identity, configure it first:

```powershell
git config --global user.name "MysticalX"
git config --global user.email "<YOUR-GITHUB-EMAIL>"
```

You may use a GitHub-provided no-reply email if you do not want your personal email address exposed in commit metadata.

## Before making the repository public

Verify that the upload does **not** contain:

- `.env`
- runtime debug JSON
- API keys or tokens
- generated private media
- local asset/history registries
- model weights
- private workflow files
- absolute personal machine paths

The included `.gitignore` is designed to reduce accidental commits of these files, but always review `git status` before committing future changes.

## First release suggestion

After the repository is online and verified:

1. Open **Releases**.
2. Create a new release.
3. Suggested tag:

```text
v1.0.0
```

4. Suggested title:

```text
ComfyUI Adaptive Media Engine v1.0.0
```

5. Suggested release notes:

```text
Initial public release.

- Model-agnostic ComfyUI media skill
- Intent-to-workflow routing
- Workflow discovery and registry
- Structural validation via /object_info
- Workflow fingerprinting and versioning
- Image/video capability architecture
- Asset lineage and job monitoring design
- Safe last-known-good workflow handling
```

Publishing a GitHub Release is optional. The repository itself is enough for the first publication.
