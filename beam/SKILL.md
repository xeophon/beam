---
name: beam
description: "Provision remote Prime Intellect pods (CPU or GPU), rsync the local project directory, and sync coding-CLI auth and state so an SSH session can resume work. Use when migrating a coding session to a remote server, deploying to a cloud GPU instance, or handing off local development to a remote machine."
---
# Beam Handoff Skill

Provisions Prime Intellect pods, rsyncs the current project, and syncs coding-CLI config (Codex, Kimi, OpenCode, Pi, Claude Code, Amp) so work continues seamlessly over SSH.

## Goal

- Create pod(s) via `prime pods create`
- Install remote tooling (`tmux`, `uv`, selected coding CLIs via npm)
- Mirror the local project path to the same absolute path on the pod
- Sync CLI config and auth tokens (unless `--skip-auth`)
- Return SSH command, remote project directory, and pod metadata

## Primary Command

```bash
uv run beam.py --kind cpu --clis codex --node-count 1
```

## High-Value Parameters

Most important params:

- `--kind {cpu,gpu}`
- `--node-count`
- `--cpu-type` (used for CPU mode, default `CPU_NODE`)
- `--gpu-type` (default `H100_80GB`)
- `--gpu-count`
- `--region` (repeat)
- `--image`
- `--disk-size`
- `--memory`
- `--team-id`

Sync:

- `--clis` (csv: `codex,kimi,opencode,pi,claude,amp`)
- `--skip-auth` (does not copy over auth tokens, need to re-auth in ssh session)

More params can be found with `uv run beam.py --help`.

## Common Recipes

CPU handoff:

```bash
uv run beam.py --kind cpu --clis codex
```

GPU handoff:

```bash
uv run beam.py --kind gpu --gpu-type H100_80GB --gpu-count 4 --clis codex
```

Small custom CPU pod:

```bash
uv run beam.py --kind cpu --vcpus 4 --memory 16 --disk-size 200 --clis codex
```

## Execution Checklist

1. Confirm Prime is authenticated: `prime whoami`. If it fails, install with `uv tool install prime` and log in.
2. Verify local `rsync` and `ssh` are available.
3. Run `uv run beam.py ...` with desired params.
4. Wait for the `=== Prime Handoff Complete ===` summary.
5. Return the summary to the user. It includes: pod ID, SSH command, remote project directory, copied CLIs, provider, location, and hourly price.
6. Remind the user to terminate pods when done: `prime pods terminate <pod_id>`.
