---
name: markdown-link-checker
description: Validate links in markdown files. Use when checking documentation
  or .md files. Run /md-check or ask to verify links.
---

# Markdown Link Checker

This skill validates links across markdown files.

## Overview

The markdown link validator scans .md files for both relative and absolute
links, then verifies they resolve correctly. The checker is built on top
of `scripts/check.py`.

## When to use

The validator is appropriate whenever you need to confirm that links in
documentation files actually work. The checker handles both local and
external URLs.

## Usage

Provide a markdown file or directory. The checker will:

1. Parse all link expressions
2. Resolve relative paths against the file's location
3. Issue HEAD requests to external URLs
4. Report results

## Examples

- Check a README
- Check a docs folder
- Find dead links in a project

## Scripts

`scripts/check.py` does the work.

Run it:

```
python scripts/check.py docs/
```

The validator returns 0 if all links resolve, 1 otherwise.

## Configuration

Set `TIMEOUT=10` in the environment to control request timeout.

Set `MAX_PARALLEL=8` to control concurrency.

## Errors

If something fails, the script prints an error. Check the error message
and retry.

## Output

The checker prints a list of broken links to stdout, one per line:

```
docs/intro.md:42 -> /missing/page.md
docs/api.md:101 -> https://example.com/old (404)
```

## Dependencies

- Python 3
- requests
- markdown-it-py
