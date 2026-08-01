---
name: unpause
description: Resume work from .claude/STATE.md in a fresh context window. Use when the user types /unpause, normally right after /clear.
argument-hint: [optional redirection]
---

# /unpause

## 1. Read the state

Read `.claude/STATE.md`. If it does not exist, say so and stop. Do not guess at what the previous session was doing, and do not go looking through old transcripts for it.

## 2. Check the stamp

Run `git rev-parse --short HEAD` and compare it to the stamp at the top of the file.

- **Match** — the Relevance line numbers are probably good. Still treat them as hints: confirm the symbol is where the file claims before editing around it.
- **Different** — the tree moved since the pause. Line numbers are stale. Locate entries by symbol name instead, and re-check everything under Dirty state.

## 3. Read each section for what it is

- **Constraints and decisions** — binding. These came from the user and are not yours to revise.
- **Tried and rejected** — fact. Do not re-attempt a recorded failure unless the user asks, or you have a specific reason the earlier failure no longer applies. If you do re-attempt, say why first.
- **Relevance, unmarked** — confirmed. Go straight there.
- **Relevance, marked `(?)`** — a guess made by a session that knew more than you do. Verify before relying on it.
- **Relevance, marked `NOT`** — already ruled out. Do not search there again.

## 4. Do not re-explore

The whole point of the file is to skip the discovery phase.

- No broad Grep or Glob sweep of the repository.
- No reading files the Relevance map already summarizes, unless you are about to edit them.
- No reconstructing the earlier conversation.

Read only what the next action actually requires.

## 5. Resume

- Open with two or three lines: what the task is, where it stopped, what you are about to do. Keep it short — the user was there for the first half.
- If Dirty state lists half-applied edits, verify the working tree matches the description before building on it.
- If an open question blocks progress, ask it before proceeding.
- Otherwise carry out Next action.

If arguments were passed, treat them as a redirection: STATE.md is still your context, but the user's instruction takes priority over the recorded Next action.

Leave `.claude/STATE.md` alone while you work. It is a handoff artifact, not a running log — the next `/pause` rewrites it from scratch.
