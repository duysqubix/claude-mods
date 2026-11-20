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

## Moving from the old repos (for people and agents)

recap-box and roadmap-keeper used to install from repos of their own, `duysqubix/recap-box` and `duysqubix/roadmap-keeper`. Those repos get no more updates; both mods now install from this library.

A machine needs moving when `claude plugin list` shows `recap-box@recap-box` or `roadmap-keeper@roadmap-keeper`. To move it, run this block from a shell. Running it twice does no harm: the second run reports that the old copies and install lists are already gone, and that the new ones are already there.

```sh
# 1. Remove the old copies and their install lists; this also drops their entries from settings.json.
claude plugin uninstall recap-box@recap-box
claude plugin uninstall roadmap-keeper@roadmap-keeper
claude plugin marketplace remove recap-box
claude plugin marketplace remove roadmap-keeper

# 2. Add the library and install both mods from it; this adds their entries to settings.json.
claude plugin marketplace add duysqubix/claude-mods
claude plugin install recap-box@claude-mods
claude plugin install roadmap-keeper@claude-mods

# 3. Keep what each mod saved: roadmap-keeper's list of roadmap pages, recap-box's history switch.
#    A mod's file is <mod>_<library>-<first 12 characters of sha256("<mod>@<library>")>.json.
#    Run this before any session uses the new copies; it never overwrites a file.
cd "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/plugins/store"
for mod in recap-box roadmap-keeper; do
  old="${mod}_${mod}-$(printf '%s' "$mod@$mod" | shasum -a 256 | cut -c1-12).json"
  new="${mod}_claude-mods-$(printf '%s' "$mod@claude-mods" | shasum -a 256 | cut -c1-12).json"
  if [ -f "$old" ] && [ ! -f "$new" ]; then cp "$old" "$new"; fi
done

# 4. Turn on automatic updates for the library; step 2 adds its entry without them.
f="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/settings.json"
if command -v jq >/dev/null; then
  jq '.extraKnownMarketplaces["claude-mods"].autoUpdate = true' "$f" > "$f.tmp" && mv "$f.tmp" "$f"
else
  echo "Add \"autoUpdate\": true to the claude-mods entry under extraKnownMarketplaces in $f"
fi
```

Then run `/reload-plugins` in each open session, or restart it.

To check the move: `claude plugin list` shows `recap-box@claude-mods` and `roadmap-keeper@claude-mods`, and neither old name. In `settings.json`, `extraKnownMarketplaces` holds `claude-mods` with `"autoUpdate": true`, and no `recap-box` or `roadmap-keeper`.

Notes for agents doing this for someone:

- Run the block as it is, on each machine the person uses; it touches only that machine's Claude Code setup.
- Don't hand-edit `installed_plugins.json` or `known_marketplaces.json`; the commands keep them and `settings.json` in step.
- A settings file kept in a dotfiles repo or synced between machines needs the same change there, or the next sync brings the old entries back.
- A clone run with `--plugin-dir` or `CLAUDE_CODE_PLUGIN_DIRS` now lives at `plugins/<mod>` in a clone of this repo.
