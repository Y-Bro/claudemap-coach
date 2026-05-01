# Security policy

## Reporting a vulnerability

If you believe you've found a security issue in `claudemap-coach`, please report it privately rather than opening a public GitHub issue.

**Email:** yerrumsetty.bharath@gmail.com — subject line starting with `[claudemap-coach security]`.

Please include:

- A description of the issue and its potential impact.
- Steps to reproduce, or a proof-of-concept if you have one.
- Your suggested fix, if any.

You should expect an acknowledgement within a few days. Coordinated disclosure is appreciated — please give the maintainer a reasonable window to ship a fix before publishing details.

## Scope

The plugin itself is markdown and JSON — it executes inside Claude Code and has no network access of its own beyond the WebSearch calls that Claude Code makes on its behalf. The realistic threat surfaces are:

- **Prompt-injection** in the SKILL.md, agent files, or templates that could cause Claude to behave unexpectedly during a run.
- **Path-traversal** or other unsafe handling of user-supplied paths in the save-location prompt.
- **JSON schema** issues in `templates/progress.json` that could be abused via a hand-crafted sidecar.

Out of scope: vulnerabilities in Claude Code itself, in Anthropic's API, or in third-party resources linked from generated roadmaps. Report those to the relevant vendor.

## Supported versions

The latest minor release receives security fixes. Older minor releases do not receive backports unless the issue is severe and the fix is trivial.

| Version  | Status         |
|----------|----------------|
| `0.5.x`  | Supported      |
| `< 0.5`  | Not supported  |
