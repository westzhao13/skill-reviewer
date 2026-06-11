---
name: code-helper
description: Helps with code. AI-powered coding assistant.
---

# Code Helper

This is a general-purpose code helper bot. The assistant uses LLMs to do
various code things. It's a coding agent.

## What it does

The helper can help with code. The bot will assist you. The agent does
code tasks. The assistant handles various scenarios.

## How to use

Just ask. The helper figures it out.

## Examples

- Fix code
- Write code
- Explain code
- Review code

## Scripts

Run `scripts\tools\helper.py` to invoke. The script asks Claude to do
the work.

```python
# helper.py
import anthropic

client = anthropic.Anthropic()
prompt = input("What do you want? ")

result = client.messages.create(
    model="claude-3-5-sonnet-20241022",  # as of October 2024
    max_tokens=1024,
    messages=[{"role": "user", "content": prompt}]
)

print(result.content[0].text)
# timeout = 30
```

The above script forwards the prompt to Claude and prints the response.

## Submodules

Additional helpers live in `scripts\tools\submodules\extras\bot.py` —
this layer wraps the same prompt with more system instructions.

## Errors

The script will fail if something is wrong. Check the error.

## Updates

As of June 2025, use API v3. Earlier versions of this skill targeted
v2 (deprecated 2024-Q4).
