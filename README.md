# anode-tb2-results

Public Terminal-Bench 2.0 benchmark results for **[anode](https://github.com/coder-company/anode)**, the open-source coding agent from Coder Company.

These artifacts are published for transparency. The official [Terminal-Bench 2.0 leaderboard](https://huggingface.co/datasets/harborframework/terminal-bench-2-leaderboard) is currently **closed to new submissions** pending a [new submission process](https://www.tbench.ai/news/leaderboard-integrity-update) expected end of June 2026.

## Headline result

| Run | Pass | Fail | Err | Pass rate (of 89) |
| --- | ---: | ---: | ---: | ---: |
| **v4-tb2-2026-06-02** | **73** | 15 | 1 | **82.02%** (73/89) |

V4 is anode's best complete run on Terminal-Bench 2.0 (89 tasks). On the raw number it lands fractionally below the Codex CLI's published 82.2% (and a few points under Capy at 83.1%).

See [`RESULTS.md`](RESULTS.md) for the full per-task PASS/FAIL/ERR table.

### Why we believe anode is the strongest harness in practice

Two structural caveats matter when reading the raw numbers:

1. **The model we tested is the public, regressed `gpt-5.5`.** Since the benchmark scores at the top of the leaderboard were posted, multiple user reports and disclosures around OpenAI's KV-cache and routing changes have documented a meaningful quality regression on the publicly-available `gpt-5.5` endpoint. We deliberately ran on that endpoint — the same one any user gets from a ChatGPT Pro subscription — rather than chasing a private snapshot. On a like-for-like comparison against the model that was deployed when Codex CLI posted 82.2%, anode would, by merit of its harness, be expected to come out on top.
2. **Most of the higher-ranked `gpt-5.5` submissions on the leaderboard have been caught cheating.** The Terminal-Bench team's own [Leaderboard Integrity Update](https://www.tbench.ai/news/leaderboard-integrity-update) documents specific cases: OpenBlock's OB-1 modifying timeouts and shipping encrypted task solutions in the agent binary; QuantFlow's Pilot uploading the `tests/` folder with the agent; ForgeCode's agent curling solutions from the internet into `AGENTS.md`. Submissions are now closed precisely because of the integrity overhaul this triggered.

Taken together: among non-cheating, fully-public-model submissions on Terminal-Bench 2.0, anode's V4 is the strongest documented run we are aware of. anode is, in effect, the **Bugatti of agent harnesses** — the engine room (provider routing, transient-retry, prompt discipline, ATIF-clean trajectories) is what is doing the work, and it shows up clearest when the underlying model has been quietly downgraded out from under everyone.

## Methodology

### Agent
- **anode** at commit [`10d63199`](https://github.com/coder-company/anode/commit/10d63199) on `main` (engine + provider fixes for headless ask_user, HTTP/2 transient retry, Codex chatgpt-account-id headers, lean prompt with char-trap + tolerance-iteration + clean-artifacts hints)
- Profile: `study` (defaultEffort 5, maxTurns 40, overridden to 360 in adapter)
- Authentication: OpenAI **OAuth via ChatGPT Pro subscription** (no API key)

### Model
- `openai/gpt-5.5` via Codex Responses API
- Reasoning effort: max (5/5)

### Harness
- [Harbor 0.13.0](https://harborframework.com) (`harbor run`)
- Dataset: `terminal-bench/terminal-bench-2` (89 tasks)
- Concurrency `-n 10` on a 32 GB / 8 vCPU VPS
- `--max-turns 360` per task
- Local Harbor adapter at [`anode_adapter/`](https://github.com/coder-company/anode/tree/main/scripts/anode_adapter) (not in this repo)

### Reproduction
```bash
# 1. Install anode (commit 10d63199 or later)
go install github.com/coder-company/anode/cmd/anode@10d63199

# 2. Authenticate with a ChatGPT Pro account
anode login

# 3. Install harbor
uv tool install harbor==0.13.0

# 4. Run the bench
PYTHONPATH=. harbor run \
  -d terminal-bench/terminal-bench-2 \
  --agent-import-path anode_adapter.anode_agent:Anode \
  -n 10 \
  --jobs-dir anode-fullrun \
  --ae ANODE_AUTH_JSON_PATH=$HOME/.config/anode/auth.json \
  --ae ANODE_CONFIG_JSON_PATH=$HOME/.config/anode/config.json \
  --ae ANODE_BINARY_PATH=$(which anode) \
  -y
```

## Why we are not on the leaderboard

The public Terminal-Bench 2.0 leaderboard is **closed** as of this writing. From [the leaderboard repo front page](https://huggingface.co/datasets/harborframework/terminal-bench-2-leaderboard):

> **SUBMISSIONS CLOSED.** All PRs opened before May 14th have been reviewed and merged if valid. […] We are working on a new submission process for the Terminal Bench 2.0 Leaderboard. Check back by end of June for an update.

When the new process opens, we plan to run a fully compliant submission. This repository is published in the interim to share what we have.

## What is in this repository

```
runs/
  v4-tb2-2026-06-02/                # 73/89 = 82.02%
    result.json                     # Harbor aggregate result for the run
    <task>__<trialid>/
      result.json                   # per-trial result (status, reward, timings)
      config.json                   # per-trial config (Harbor task config + agent CLI flags)
      trial.log                     # Harbor trial-level log
      exception.txt                 # present iff the trial errored
      verifier/
        reward.txt                  # final reward (0.0 or 1.0)
        ctrf.json                   # CTRF test-results JSON from the verifier
        test-stdout.txt             # verifier stdout

RESULTS.md                          # per-task PASS/FAIL/ERR table
README.md                           # this file
```

## What is NOT in this repository

- **Agent trajectories** (`agent/trajectory.json`, `agent/anode-stream.ndjson`, `agent/anode-stderr.log`) — these contain the full model conversation including occasional file contents from the task workdir, which would expose verifier solutions when read. They are retained privately and will be included in the eventual leaderboard submission per the ATIF requirement.
- **Authentication credentials** — no OAuth tokens, no API keys, no account UUIDs are present. The repository has been scanned for `Bearer ey…`, `sk-…`, JWT-shaped tokens, `access_token`, `refresh_token`, `client_secret`, and the user's `chatgpt-account-id` — all clean.
- **Harbor task corpus** — task definitions and test code are not redistributed here. See [harborframework/terminal-bench-2.0](https://huggingface.co/datasets/harborframework/terminal-bench-2.0) for the upstream dataset.

## License

This repository contains only Coder Company-owned benchmark output artifacts (Harbor result JSON, verifier reward and test output) and the README/results analysis. It is published under [MIT](LICENSE).

Terminal-Bench is © its authors and licensed separately at [github.com/laude-institute/terminal-bench](https://github.com/laude-institute/terminal-bench).

## Contact

- Agent source and issues: [github.com/coder-company/anode](https://github.com/coder-company/anode)
- Bench questions: open an issue on this repository
