# Mixing SCX `coder` with OpenAI GPT-5.4 and Anthropic Claude in opencode

## Step 1 — Authenticate SCX.ai

```bash
opencode auth login
```

Scroll to **Other**, enter provider ID:

```
scx-ai
```

Paste your SCX key (`sk-scx-...`).

## Step 2 — Authenticate OpenAI

```bash
opencode auth login
```

Select **OpenAI** → **Manually enter API Key** (or ChatGPT Plus/Pro OAuth) → paste key.

## Step 3 — Authenticate Anthropic

```bash
opencode auth login
```

Select **Anthropic** → choose **Claude Pro/Max** OAuth (browser auth) or **Manually enter API Key** → paste key.

## Step 4 — Verify all three providers registered

```bash
opencode auth list
```

You should see `scx-ai`, `openai`, and `anthropic`.

## Step 5 — Save this config as `opencode.json`

Place it at:

| Path | Scope |
|---|---|
| `./opencode.json` (in your project root) | Per-project — wins over global |
| `~/.config/opencode/opencode.json` | Global — applies everywhere |

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "scx-ai": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "SCX.ai",
      "options": {
        "baseURL": "https://api.scx.ai/v1"
      },
      "models": {
        "coder": {
          "name": "Coder (SCX)",
          "limit": { "context": 196608, "output": 4096 },
          "modalities": { "input": ["text"], "output": ["text"] }
        },
        "MAGPiE": {
          "name": "MAGPiE (SCX)",
          "limit": { "context": 131072, "output": 131072 },
          "modalities": { "input": ["text"], "output": ["text"] }
        }
      }
    },
    "openai": {
      "models": {
        "gpt-5.4": {
          "name": "GPT-5.4 Thinking",
          "limit": { "context": 400000, "output": 128000 },
          "modalities": { "input": ["text", "image"], "output": ["text"] }
        }
      }
    },
    "anthropic": {
      "models": {
        "claude-opus-4-7": {
          "name": "Claude Opus 4.7",
          "limit": { "context": 1000000, "output": 128000 },
          "modalities": { "input": ["text", "image"], "output": ["text"] }
        },
        "claude-opus-4-6": {
          "name": "Claude Opus 4.6",
          "limit": { "context": 1000000, "output": 64000 },
          "modalities": { "input": ["text", "image"], "output": ["text"] }
        },
        "claude-sonnet-4-6": {
          "name": "Claude Sonnet 4.6",
          "limit": { "context": 1000000, "output": 64000 },
          "modalities": { "input": ["text", "image"], "output": ["text"] }
        }
      }
    }
  },
  "model": "scx-ai/coder",
  "small_model": "scx-ai/coder",
  "agent": {
    "build": {
      "mode": "primary",
      "model": "scx-ai/coder"
    },
    "plan": {
      "mode": "primary",
      "model": "anthropic/claude-opus-4-7"
    }
  }
}
```

## Step 6 — Add `AGENTS.md` to work around the bash tool bug

Save this as `AGENTS.md` in the same directory as `opencode.json`. This patches around opencode issue #13146 — the bash tool's schema forces models into failure/retry loops on optional parameters.

```markdown
# Tool Usage Rules

## Bash tool

When calling the `bash` tool, you MUST:

1. Always include a `description` parameter — a 5-10 word string describing what the command does.
2. For optional parameters like `timeout`, OMIT the key entirely if not needed. Never pass `null`.
3. For optional string parameters like `workdir`, OMIT the key entirely if not needed. Never pass `null` or empty strings.

### Correct examples

Basic command:
{"command": "ls -la", "description": "List current directory files"}

With timeout:
{"command": "npm test", "description": "Run test suite", "timeout": 60000}

### Wrong examples

Missing description (will fail with Zod validation error):
{"command": "ls -la"}

Null timeout (will fail with "expected number, received null"):
{"command": "ls -la", "description": "List files", "timeout": null}

## General

Apply the same rule to any tool with optional parameters: if you don't need the parameter, omit the key — don't emit `null`.
```

## Step 7 — Restart opencode

Fully quit the TUI and relaunch. Provider configs and `AGENTS.md` don't hot-reload.

## Step 8 — Verify everything loaded

Inside opencode:

```
/models
```

You should see:

- `scx-ai/coder`
- `scx-ai/MAGPiE`
- `openai/gpt-5.4`
- `anthropic/claude-opus-4-7`
- `anthropic/claude-opus-4-6`
- `anthropic/claude-sonnet-4-6`

## How to use it

**Tab between agents:**

- `Tab` → **Build agent** → `scx-ai/coder` (fast, cheap daily driver)
- `Tab` → **Plan agent** → `anthropic/claude-opus-4-7` (strongest reasoning model available, 1M context, for hard planning work)

**Manually switch model mid-session:**

```
/model anthropic/claude-sonnet-4-6
```

```
/model anthropic/claude-opus-4-6
```

```
/model openai/gpt-5.4
```

```
/model scx-ai/coder
```

## When to reach for which

| Model | Good for | Cost (per Mtok in/out) |
|---|---|---|
| `scx-ai/coder` | Daily coding, edits, file ops | ~$0.04 / $0.08 |
| `scx-ai/MAGPiE` | Cheap reasoning, long-output writing | ~$0.022 / $0.059 |
| `openai/gpt-5.4` | Complex agentic tasks, multi-step workflows, vision | ~$2.50 / $15 |
| `anthropic/claude-sonnet-4-6` | Strong coding at mid-tier price, 1M context | ~$3 / $15 |
| `anthropic/claude-opus-4-6` | Deep reasoning, large codebases, agentic work | ~$5 / $25 |
| `anthropic/claude-opus-4-7` | Strongest available reasoning, best for planning and agentic | ~$5 / $25 |

**Rough workflow:** SCX for writing code, Opus 4.7 for planning architecture, Sonnet 4.6 when you want Claude-quality coding at Sonnet pricing, GPT-5.4 for second opinions or vision-heavy tasks.

## Two small caveats

**Claude Pro/Max OAuth plugins:** Opencode's docs note Anthropic explicitly prohibits using Claude Pro/Max subscriptions through third-party coding tools. Previous opencode versions bundled a plugin for this; it was removed in 1.3.0. If you want to use Claude via a subscription, use the supported **Claude Pro/Max** option in `/connect` (which is Anthropic-sanctioned) or pay API-by-token. Don't install third-party auth plugins for this.

**Model ID verification:** I used `claude-opus-4-7`, `claude-opus-4-6`, and `claude-sonnet-4-6` based on Anthropic's docs and announcements. If any fail to load with "model not found" after restart, the real ID may include a date snapshot like `claude-opus-4-7-20260416`. In that case run `opencode models anthropic --refresh` to see the canonical strings Anthropic's API returns, and swap them in.
