---
name: pause
description: Capture everything needed to resume the current task into .claude/STATE.md, then stop. Use when the user types /pause, including when it arrives as a queued message mid-task.
allowed-tools: Read, Write, Bash(git rev-parse:*), Bash(git status:*), Bash(git stash list:*)
---

# /pause

Stop work and write a handoff file that a future session — starting with an empty context window — can use to pick up where you left off.

## 1. Stop cleanly

- Finish or safely abandon the tool call in flight. Start no new work, no new edits, no new searches.
- Do not "just finish this one thing first." The point of the pause is that you stop now and record accurately where "now" is.

## 2. Gather ground truth

Before writing, run:

- `git rev-parse --short HEAD` — the commit to stamp the file with
- `git status --short` — what is uncommitted or half-edited

## 3. Write `.claude/STATE.md`

Write the file with `Write`, never `Edit`. Any existing `.claude/STATE.md` is **overwritten wholesale** — every section, every time. Never append to it, never merge into it, never leave stale sections from a previous pause standing.

The file always describes one thing: the state right now. If something from an earlier pause is still true and still matters — a rejected approach, a confirmed-irrelevant file — restate it in the fresh file from what you know now. Do not preserve it by leaving the old text in place.

### The rule that governs everything

**Write only what cannot be recovered from the repository.**

The code is still on disk and git still has the history. What dies when the context is cleared is everything that only ever existed in the conversation: the user's actual intent, constraints they stated out loud, the reasoning behind decisions, and what was already tried and failed.

### The failure mode to guard against

You are writing this while you still hold the full context, so everything feels obvious and you will under-specify. **Write for someone who has never seen this repository and was not present for this conversation.** If an entry only makes sense to someone who remembers the conversation, rewrite it.

### Pointers, not content

Never paste file contents, diffs, or code excerpts. A pointer plus the conclusion you drew is the entire value:

- Bad — a forty-line quote of the auth module
- Good — `src/auth/session.rs` — refresh retries twice then throws, which is why the timeout fix belongs here and not in the client

If STATE.md starts containing file excerpts, you have rebuilt the context window in Markdown and saved nothing.

### Template

```markdown
# Session state

Stamped at commit `<short-sha>`. Line numbers below are hints, not guarantees.

## Context

### Request
<The user's original ask, in their words. Not your restatement of it.>

### Constraints and decisions
- <Things the user said that bind the work: "don't touch the migrations", "use X not Y".
  These exist nowhere but the conversation.>

### Done so far
- <Completed steps, each with its outcome.>

### Tried and rejected
- <Approach — why it failed.>
  <The highest-value section in the file. Without it the next session
  confidently repeats a dead end.>

## Relevance

What matters for this task. Anchor on symbol names; treat line numbers as hints.

- `path/to/file.rs` — `symbol_name()` (~line N) — why it matters
- (?) `path/to/other.rs` — suspected relevant, not confirmed
- NOT `path/to/dead-end.rs` — checked, does not handle <thing>

Mark guesses with `(?)`. Record negative results with `NOT` — "checked here,
it isn't here" cannot be rederived without repeating the whole search.

## Current

### In progress at pause
<The one thing being worked on when the pause landed.>

### Dirty state
<Half-applied edits, uncommitted files, running processes, stashes.
A queued /pause lands mid-task by design — never assume a clean tree.>

### Next action
<The single next concrete step. Not a plan — the next thing to do.>

### Open questions
<Anything needing the user's answer before work can continue.>
```

## 4. Stop

Print two or three lines confirming what was captured, then remind the user of the sequence:

```
/clear     empties the context
/unpause   resumes from STATE.md
```

Do not continue working after writing the file. `/clear` cannot be invoked by you — it is a built-in command the user must type.
