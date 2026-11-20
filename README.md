# claude-mods

Claude Code mods in one place. Each mod is a folder under [`plugins/`](plugins), and one install list (Claude Code calls it a marketplace) offers them all.

| Mod | What it does |
| --- | --- |
| [recap-box](plugins/recap-box) | A small box beside the transcript: a short title for the chat, the stage of the work, what is happening now and the session's goal, in plain words. |
| [roadmap-keeper](plugins/roadmap-keeper) | Reminds the agent to update a project's roadmap pages after commits, pull requests and closed beads. |

## Install

One mod, at the prompt of a terminal session:

```
/plugin install recap-box --marketplace duysqubix/claude-mods
```

Answer `y` to add the library, then press Enter for the user scope. From a shell:

```
claude plugin install recap-box --marketplace duysqubix/claude-mods
```

In the desktop app, where `/plugin` does not run: click + next to the prompt box, then Plugins, Add marketplace with `duysqubix/claude-mods`, then Add plugin.

### Every mod, on every machine

Put this in `~/.claude/settings.json`. Claude Code ignores `autoUpdate` in project and local settings.

```json
"extraKnownMarketplaces": {
  "claude-mods": {
    "source": { "source": "github", "repo": "duysqubix/claude-mods" },
    "autoUpdate": true
  }
},
"enabledPlugins": {
  "recap-box@claude-mods": true,
  "roadmap-keeper@claude-mods": true
}
```

- A session start installs any mod that is missing. A new machine can take two starts.
- Each session checks for new versions at a random moment in its first 10 minutes. A session that takes one shows `Plugin updated: <mod>`, then reloads it or says to run `/reload-plugins`.
- Claude Code skips that check while its own auto-updater is off, as under `DISABLE_AUTOUPDATER` or `DISABLE_UPDATES`. Set `FORCE_AUTOUPDATE_PLUGINS=1` to keep it on for plugins.

To update right away, run this from a shell, then `/reload-plugins` in each session that is already open:

```
claude plugin marketplace update claude-mods && claude plugin update recap-box@claude-mods
```

## Develop

```
git config core.hooksPath .githooks       # once: runs the checks before every push
claude --plugin-dir plugins/recap-box     # this folder overrides the installed copy and reloads after each edit
```

Each mod names its own `version` in `plugins/<mod>/.claude-plugin/plugin.json`. An installed copy takes a change only when that number goes up, so a push that changes one mod leaves the others alone. Raise the version of every mod you change.

The pre-push hook checks the install list, then validates, tests and type-checks each mod, and stops a push that changes a mod without raising its version. Run it by hand with `.githooks/pre-push`. `tsc` needs the types the engine writes into a mod's `.claude-plugin/types/` the first time a `--plugin-dir` session loads it; until then the hook skips it.

To add a mod, make its folder under `plugins/` with `"version": "0.1.0"` in its `plugin.json`, and list it in `.claude-plugin/marketplace.json`.

## License

MIT
