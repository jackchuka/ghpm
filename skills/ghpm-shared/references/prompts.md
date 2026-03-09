# Prompt Configuration

Controls which write actions ask for user confirmation.

## Config

Read from `prompts.<skill-name>` in `.ghpm/config.json`:

```json
{
  "prompts": {
    "default": "prompt",
    "ghpm-work": {
      "default": "auto",
      "draft_pr": "prompt"
    }
  }
}
```

## Values

| Value | Behavior |
|-------|----------|
| `"prompt"` | Ask the user before proceeding |
| `"auto"` | Proceed without asking |
| `"skip"` | Skip the action entirely |

## Resolution

1. `prompts.<skill-name>.<action-key>`
2. `prompts.<skill-name>.default`
3. `prompts.default`
4. `"prompt"`

First match wins.

## Usage in Skills

Reference files mark promptable actions with:

```
> **Prompt** (`<action-key>`): "<question>"
```

Before showing the prompt, resolve the action key:
- `"prompt"` → ask the user, iterate on feedback
- `"auto"` → proceed, log what was done
- `"skip"` → skip the action, continue to next step
