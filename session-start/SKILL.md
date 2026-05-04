---
description: Session start — reads CLAUDE.md, loads memory, and confirms what's active for the current project.
when_to_use: Invoke at the start of every session to confirm project rules and context are loaded.
---

## Step 1 — Read project rules

!`[ -f CLAUDE.md ] && echo "=== CLAUDE.md ===" && cat CLAUDE.md || echo "No CLAUDE.md found in current directory."`

## Step 2 — Confirm

After reading the above, provide a brief confirmation in this format:

**Project rules loaded:**
- List the key standing rules from CLAUDE.md (one line each)
- If no CLAUDE.md was found, say so

**Memory:** Recall any memory for this project and summarize current progress and relevant context in 2-3 sentences.

**Ready.** State the next recommended action based on where the project left off.
