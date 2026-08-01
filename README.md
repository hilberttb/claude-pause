# pause / unpause

Two Claude Code skills for stopping work mid-task and picking it up later in a fresh context window.

A long session fills its context with transcript, most of which is noise by the time the task is half done. Clearing the context throws away the parts that still matter: what you actually asked for, the constraints you stated out loud, what was already tried and failed, and which files turned out to be relevant. `/pause` writes those parts to a file before you clear. `/unpause` reads them back afterward.

## What each skill does

`/pause` stops work and writes `.claude/STATE.md`. It records the original request, the constraints you stated in conversation, what is done, what was tried and rejected, a map of the relevant symbols and files, and the single next action. The file is stamped with the current commit so a later session knows whether the recorded line numbers can still be trusted. It contains only what cannot be recovered from the repository: no diffs and no file contents. If a state file starts filling up with code excerpts, it has rebuilt the context window in Markdown and saved nothing.

`/unpause` reads `.claude/STATE.md` and resumes from it. It compares the stamped commit against HEAD to decide whether line numbers are still usable, skips the usual exploration phase, and starts on the recorded next action. Entries marked `(?)` are guesses to verify before relying on them. Entries marked `NOT` are paths already ruled out, so it does not search there again.

## Use

```
/pause      writes .claude/STATE.md and stops
/clear      empties the context window
/unpause    reads STATE.md and picks up
```

All three can be queued back to back while Claude is still working. A queued `/pause` lands at the next turn boundary rather than yanking the model out of a tool call the way Esc does, so the state file gets written with knowledge of where the work actually got to.

The `/clear` step is not optional. `/pause` followed immediately by `/unpause` in the same session buys nothing, because the transcript never left the context window. It is also the only step Claude cannot perform for you: built-in slash commands are expanded by the client when you type them and are not callable by the model.

Clearing saves the previous conversation rather than destroying it, so if the state file turns out to have missed something, the full transcript is still reachable through `/resume`.

`/unpause` takes optional arguments. `/unpause skip the caching part and go straight to the tests` loads the state file as context but overrides the recorded next action.

## What STATE.md holds

Three sections:

- `Context`: the original request, constraints stated in conversation, what is done, and what was tried and rejected.
- `Relevance`: which symbols in which files matter, including guesses marked `(?)` and ruled-out paths marked `NOT`.
- `Current`: what was in flight when the pause landed, what is dirty in the working tree, and the next action.

Every `/pause` rewrites the entire file. Nothing accumulates across pauses. Anything from an earlier pause that is still true gets restated in the fresh file, and anything no longer true disappears instead of sitting there misleading the next session. `/unpause` leaves the file alone while it works, so the file always describes the last pause rather than acting as a running log.

Add `.claude/STATE.md` to your `.gitignore`, unless you want the handoff history in version control. That is a reasonable thing to want if you are moving work between machines or between people.

## Install

Each skill is a directory containing a `SKILL.md`. The directory name is the command name, so `pause/SKILL.md` becomes `/pause`.

### Claude Code

Copy both directories into one of these locations:

```
~/.claude/skills/pause/SKILL.md      # personal, available in every project
~/.claude/skills/unpause/SKILL.md

.claude/skills/pause/SKILL.md        # project-scoped, committed with the repo
.claude/skills/unpause/SKILL.md
```

A common mistake is nesting one level too deep. `SKILL.md` has to sit directly inside the skill directory. Claude Code watches these directories and picks up changes without a restart, though a skills directory that did not exist when the session started needs one. Run `/skills` to confirm both loaded.

### Claude desktop app

Skills uploaded to your account are available in chat and in Cowork, which do not read `~/.claude/skills/` from your machine. Use this route if you want the pair available there as well. It requires a paid plan.

1. Zip each skill directory, giving you `pause.zip` and `unpause.zip`.
2. Under Settings > Capabilities, turn on code execution and file creation. Skills do not run without them.
3. Open Customize > Skills in the sidebar, click the `+` button, upload the first zip, and toggle the skill on.
4. Repeat for the second zip.

Both skills assume a git working tree, so Claude Code is where they earn their keep. Uploading them mainly buys you the same commands in Cowork sessions.

## License

0BSD. Do whatever you want with these files. See [LICENSE](LICENSE).
