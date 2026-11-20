# recap-box

A Claude Code mod: a small box beside the transcript that says, in plain words, what the session is doing now and what it is working toward.

- At the very top, centered in an outlined box: a title of two or three words for the whole chat, such as `Sign-In Fix`. Haiku names it with the first recap and renames it only when it names a new goal, so the title moves only when your requests do.
- Under it: the stage of the work and one or two sentences on what is happening now. The stage is one of researching, planning, coding, testing, debugging, reviewing, writing or shipping, each in its own theme color.
- The stage comes from the tools the session runs, not from a guess. It is the newest of the last five calls that does more than look. Edits are coding, or writing when they change docs. Test runs are testing, review agents are reviewing, and commits, pushes and CI watches are shipping. A shell command counts by the commands it starts, so `grep -rn test` is a look and `npm test` a test run.
- When the last five calls only read, search or run other commands, the stage is what a helper agent at work does, or researching. While the session waits on a helper agent, the stage is the helper's. When the newest test run among the last five failed, by its exit code or by a failure its output reports, the stage is debugging. A run sent to the background does not count, since its result comes later.
- Below, pinned: the goal of the whole session, drawn from what you asked for. Haiku keeps it until your requests move on. The box sees only what you ask after it loads, so in a resumed session the goal starts from the transcript.
- The title, the recap and the goal use everyday words, with no jargon, no abbreviations and no names of files or code.
- The box updates as the session works, with no periodic polling. Each tool call, and each helper agent that starts or ends, has it read the stage again, two seconds after the last of a burst; a steady stream of events still gets a check about every ten seconds. It asks Haiku for a new recap when a turn of the session ends, and every fifth call of a long turn, at most once every ten seconds. A recap due sooner waits out the ten seconds, then reads what is newest. When the transcript has not changed, it asks nothing.
- After a `/resume`, the box recaps the transcript the resume brings once the engine has it in place. After a `/clear` it waits for new activity.
- While the box waits undrawn (opened on its own on a terminal too narrow for it, or with no app attached) it reads nothing. A recap that came due meanwhile is written once the box is drawn. A box drawn but behind another tab still reads.

`/recap-box` hides the box while it shows, and shows it again while it is hidden or behind another pane.

`/recap-box history` turns the history on or off. With it on, the title, the recap and the goal stay at the top of the box, and the session's earlier recaps run below them under a dotted line, the newest first, each dimmed under the stage it was written in. Scroll down in the box for older ones with the mouse wheel in the fullscreen layout, one recap a step. From the keyboard, ctrl+x tab moves the focus to the box: the down arrow or Page Down moves one recap older, Page Up one newer, and Home and End go to the newest and the oldest. Esc moves the focus back to the prompt. The box keeps your place as new recaps come in. Where it sits above the prompt (outside the fullscreen layout, or in a terminal under 110 columns), the box is too short to hold the title, recap and goal still, so it scrolls as a whole. It keeps the last 200, and a /clear or /resume starts the list over.

Later sessions keep the setting; sessions already open keep theirs until they restart.

`/recap-box upgrade` installs the newest version and reloads the plugins. While one waits, the box says so at its top, under the title: see [Install and updates](#install-and-updates).

## Install and updates

recap-box ships in the [claude-mods](../../README.md) library: install it and keep it updated as that page says.

```
/plugin install recap-box --marketplace duysqubix/claude-mods
```

The box also checks on its own, whatever Claude Code's update settings say. While the library's `main` holds a newer version of recap-box than the copy running, it shows `Update available: /recap-box upgrade` at its top, under the title. It reads `main` when a session starts at the prompt or a surface attaches, then every hour. A clone at `main`'s version, or ahead of it, never shows it.

To update right away, run `/recap-box upgrade`. It runs these two commands with the `claude` on your `PATH`, then reloads the plugins of that session:

```
claude plugin marketplace update claude-mods && claude plugin update recap-box@claude-mods
```

Where `claude` is not on the `PATH`, as in the desktop app, run them from a shell. Then run `/recap-box upgrade` or `/reload-plugins` in any other session that is already open.

## Develop

See [Develop](../../README.md#develop) in the library's README. From the library's folder, `claude --plugin-dir plugins/recap-box` runs this mod from your clone.
