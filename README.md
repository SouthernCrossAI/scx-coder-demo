# Mixing SCX `coder` with OpenAI GPT-5.4 and Anthropic Claude in opencode

This guide assumes you want to use SCX `coder` as the default build model in opencode, with OpenAI and Anthropic available as optional switchable models.

## New to opencode?

Start here if this is your first opencode install. A fresh opencode install has no models connected yet, so the config below will not work until you install opencode and connect at least one AI provider.

1. Download and install opencode from the official download page:

   <https://opencode.ai/download>

   Terminal install option:

   ```bash
   curl -fsSL https://opencode.ai/install | bash
   ```

2. Launch opencode once:

   ```bash
   opencode
   ```

3. Open settings and connect your first model/provider.

   In the TUI, use the settings/connect flow to add a provider such as OpenAI, Anthropic, GitHub Copilot, or another supported model provider. If you are using API keys, have them ready before starting this step.

4. Confirm opencode can see at least one model.

   Inside opencode:

   ```
   /models
   ```

   If `/models` is empty, finish connecting a provider before continuing.

After opencode is installed and can see at least one model, continue with the SCX/OpenAI/Anthropic setup below.

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

## Step 6 — Restart opencode

Fully quit the TUI and relaunch. Provider configs don't hot-reload.

## Step 7 — Verify everything loaded

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
| `scx-ai/coder` | Daily coding, edits, file ops | A$0.30 / A$2.02 |
| `scx-ai/MAGPiE` | Cheap reasoning, long-output writing | A$0.30 / A$2.02 |
| `openai/gpt-5.4` | Complex agentic tasks, multi-step workflows, vision | ~US$2.50 / US$15 |
| `anthropic/claude-sonnet-4-6` | Strong coding at mid-tier price, 1M context | ~US$3 / US$15 |
| `anthropic/claude-opus-4-6` | Deep reasoning, large codebases, agentic work | ~US$5 / US$25 |
| `anthropic/claude-opus-4-7` | Strongest available reasoning, best for planning and agentic | ~US$5 / US$25 |

**Rough workflow:** SCX for writing code, Opus 4.7 for planning architecture, Sonnet 4.6 when you want Claude-quality coding at Sonnet pricing, GPT-5.4 for second opinions or vision-heavy tasks.
