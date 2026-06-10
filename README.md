# swe-agent-gitlab-ci

GitLab CI/CD pipeline that runs [SWE-agent](https://github.com/SWE-agent/SWE-agent)
against your own GitLab issues — autonomously creating branches, commits, and
merge requests from plain-language issue descriptions.

Addresses [SWE-agent#760](https://github.com/SWE-agent/SWE-agent/issues/760)
(no native GitLab support).

## How it works

1. Open an issue, label it `run:swe`
2. Scheduled pipeline picks it up, labels it `swe:in-progress`
3. SWE-agent edits code, runs tests, commits to `swe-agent/issue-<iid>`
4. Pipeline opens a merge request for review
5. Post-merge CI builds and deploys

## Quick start

In your project's `.gitlab-ci.yml`:

```yaml
include:
- project: 'YOUR_GROUP/swe-agent-gitlab-ci'
file: '/swe-agent.gitlab-ci.yml'

variables:
OPENAI_API_BASE: "http://lm-studio:1234/v1"
OPENAI_MODEL: "lm_studio/openai/gpt-oss-120b"
```

### Required CI/CD variables

| Variable | Purpose |
|---|---|
| `ISSUE_AUTH_TOKEN` | GitLab token with `api` scope, used to scan and label issues |
| `OPENAI_API_BASE` | LM Studio (or any OpenAI-compatible) endpoint |
| `OPENAI_MODEL` | Model identifier exposed by the endpoint |
| `OPENAI_API_KEY` | Defaults to `lm-studio` for local LM Studio runs |

### Optional labels on the issue

| Label | Effect |
|---|---|
| `run:swe` | Required trigger label |
| `model:<name>` | Override the model for this issue |
| `maxtokens:<n>` | Override `SWE_MAX_INPUT_TOKENS` |
| `swe:in-progress` | Applied automatically; prevents re-runs |

## Architecture

- **Trigger**: scheduled pipeline (hourly) scans open issues via the GitLab API
- **Branch naming**: `swe-agent/issue-<iid>` per issue
- **Idempotency**: `swe:in-progress` label + `resource_group` lock per project/MR
- **Auth**: prefers `CI_JOB_TOKEN`, falls back to `PRIVATE-TOKEN`
- **Model gateway**: any OpenAI-compatible endpoint (LM Studio, vLLM, llama.cpp, OpenAI itself)

## Why I built this

Blog post: [Learning to Automate My Side Projects](https://7echgirl.blogspot.com/2025/09/learning-to-automate-my-side-projects.html)

## Stack

- SWE-agent + LM Studio (Qwen3-coder-30b or gpt-oss-120b)
- GitLab CI/CD (self-hosted)
- Kubernetes (deploy target on merge)

## Known limitations

- Job completion detection is loose — SWE-agent can run ~50 unnecessary steps before stopping
- Unit-test integration into the agent loop is still in progress
- Model selection currently driven by labels; a config-file approach would generalize better

## License

MIT

