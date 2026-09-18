# Security Policy

## Public-repository rules

This project is intended to be safe to publish, but runtime deployments can contain sensitive information.

Never commit:

- API keys, access tokens, passwords, cookies, or credentials
- `.env` files
- private prompts or prompt history
- generated user assets unless intentionally published
- runtime `history` or debug response bodies
- local asset registries
- local usernames or absolute home-directory paths
- private model URLs requiring credentials
- workflow files containing secrets in node inputs

## Local ComfyUI exposure

`127.0.0.1` / `localhost` is the recommended default for an unauthenticated local ComfyUI server.

If ComfyUI is exposed to a LAN or the public internet, secure it appropriately before use. Do not assume the skill itself provides authentication.

## Imported workflows

Treat imported workflows and custom-node dependencies as untrusted until reviewed.

The adaptive workflow system may discover and classify workflows automatically, but it must not install arbitrary executable extensions without authorization.

## Reporting a vulnerability

Please open a GitHub security advisory or private report when available instead of posting credentials or exploit details in a public issue.
