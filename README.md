# Mellos Mapping

[![npm](https://img.shields.io/npm/v/mellos-mapping?logo=npm&logoColor=white&label=npm&color=cb3837)](https://www.npmjs.com/package/mellos-mapping)
[![downloads](https://img.shields.io/npm/dm/mellos-mapping?label=downloads&color=cb3837)](https://www.npmjs.com/package/mellos-mapping)
[![MCP registry](https://img.shields.io/badge/MCP_registry-listed-2f6feb)](https://registry.modelcontextprotocol.io/v0/servers?search=io.github.GuangminJu/mellos-mapping)
[![CI](https://img.shields.io/github/actions/workflow/status/GuangminJu/mellos-mapping/ci.yml?branch=main&label=CI)](https://github.com/GuangminJu/mellos-mapping/actions/workflows/ci.yml)
[![license](https://img.shields.io/badge/license-MIT-444)](LICENSE)
[![LINUX DO](https://img.shields.io/badge/LINUX%20DO-community-ffb400?logo=discourse&logoColor=white)](https://linux.do)

English | [简体中文](README.zh-CN.md)

A live, terminal-native map of bottom-up development for
[Claude Code](https://claude.com/claude-code), ChatGPT desktop Codex mode and Codex CLI.

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: light)" srcset="docs/demo-light.svg">
    <img alt="One declare puts the whole ghost design on screen, then the nodes light up from the bottom — a foundation cracks, the damage spreads upward, and green is earned back" src="docs/demo.svg" width="620">
  </picture>
</p>

While Claude builds your system, a split pane beside the conversation shows
the system's **layered dependency map**: primitive layers at the bottom,
dependency edges that may only point downward, ghost nodes for what is
designed, a spinner on what is being built right now, and solid green for
what is built *and verified*.

<p align="center">
  <img alt="A Claude Code session with the Mellos map pane beside it: a six-layer game engine design, L0 and L1 nodes solid green, the upper layers still dashed ghost nodes" src="docs/session-claude-code.png">
</p>

*A real session: Claude Code on the left, the map pane on the right. L0 is
verified, L1 just lit up, everything above is still a ghost.*

## Why

Most progress reporting is a task list — a top-down worldview. A Mellos map
grows the other way: an upper node can only stand on nodes below it, and the
picture makes the discipline visible:

- **The ghost design appears before any code.** Claude declares the whole
  intended structure as dashed ghost nodes first; you can veto a bad design
  while it is still only a picture.
- **The spinner is where Claude's attention is.** One glance answers "what is
  it doing right now, and on top of what?"
- **Done means verified.** A node turns solid green only with evidence (a
  passing test run). If later work cracks a foundation, the node turns red —
  a cracked foundation under a spinning upper floor is the most honest status
  report there is.
- **The map is a ledger, not a judge.** The tools refuse only structural
  corruption (an edge pointing upward, a duplicate rank). Workflow is
  Claude's discipline, defined in the bundled skill; violations are made
  *visible*, never silently blocked.

## Install

Clone the branch for your host, then run one command. No build is required.

| Branch | Audience | Command from the clone |
| --- | --- | --- |
| `main` | Shared source / either host | `node install.mjs chatgpt-app` or `node install.mjs claude` |
| `claude` | Claude Code | `node install.mjs` |
| `chatgpt-app` | ChatGPT desktop, Codex mode | `node install.mjs` |

Requires Node.js 18.17+ on the 18.x line, or 20.3+, and the corresponding host CLI
on PATH. Supported platforms are Windows 10+, macOS 13.0+ and Linux with glibc
2.28+, each on x64 or arm64. The OS must also meet the selected Node.js version's
requirements. Native lock bindings ship with the release; no local compilation
is needed. The installer checks
release integrity and all eight MCP tools, retains the runtime outside the clone,
and configures the host. Start a new conversation after installation.
See [release and branch instructions](docs/releasing.md).

Claude Code's marketplace installation is also available:

Two lines inside any Claude Code conversation:

```
/plugin marketplace add GuangminJu/mellos-mapping
/plugin install mellos-mapping@mellos-mapping
```

Or one line in a terminal:

```
claude plugin marketplace add GuangminJu/mellos-mapping && claude plugin install mellos-mapping@mellos-mapping
```

Requires the same [Node.js and platform versions](#install) on PATH (install
Node separately when using native Claude Code). No build step: `dist/` is
committed, so a clone runs as-is —
`dist/server.mjs` (the MCP server), `dist/watch.mjs` (the pane),
`dist/mmap.mjs` (the `mmap` toggle), `dist/hook-session-start.mjs` (the
`SessionStart` hook that `hooks/hooks.json` registers) and
`dist/store-paths.mjs` (the store's path vocabulary, which the plain-node pane
launcher imports instead of restating filenames).

**omp (Oh My Pi)** reads the same plugin layout the Claude Code edition ships —
the same marketplace catalog, `.mcp.json`, skill and slash command — so one
release serves both hosts:

```
omp plugin marketplace add GuangminJu/mellos-mapping
omp plugin install mellos-mapping@mellos-mapping
```

That install follows `main`, where a release is a version bump. To freeze a
version instead, add a checkout of one — the Claude Code edition ZIP from the
GitHub Release, or a clone at the release tag:

```
omp plugin marketplace add "<checkout or extracted release directory>"
omp plugin install mellos-mapping@mellos-mapping
```

omp never reads `hooks/hooks.json`, so the session paragraph arrives through the
plugin's omp host adapter (`dist/omp-extension.mjs`, declared in
`package.json#omp.extensions`): the same store, the same policy text, and the
same `mmap` shim install as the Claude hook. The pane is the same terminal
split beside the session. Install it *as a plugin*: neither `omp plugin link`
nor the npm package is a plugin, and both would leave the tools behind. See
[omp installation and limitations](docs/distributions/omp.md).

**pi** reads none of that layout: it loads an extension and has no MCP of its
own, by design. So the plugin speaks MCP to itself — the extension declared in
`package.json#pi.extensions` (`dist/pi-extension.mjs`) starts the same
`dist/server.mjs` in the session's directory, which makes the eight `mmap_*`
tools the same eight, under the same names, and builds the session paragraph
from the same store and the same policy text the Claude hook prints:

```
pi install npm:mellos-mapping
```

Add `-l` to install it for the current project instead of every one, then start
a new session: pi discovers packages at session start. The package's `skills/`
and `commands/` travel with it — the same map discipline, and `/mmap` for the
same slash command. See [pi installation and behaviour](docs/distributions/pi.md).

The first session after installing asks you **one** question — how eager
mapping should be — and records the answer for every project you will ever
open. From then on the hook carries it into each new session by itself; there
is no per-project setup step. See
[Setup: choose when maps open](#setup-choose-when-maps-open).

The `mmap` terminal command (the pane's open/close toggle) installs itself on
Windows: the same hook notices on session start when the shim is missing or
points at an older install, writes `mmap.cmd` (cmd, PowerShell) and `mmap`
(git-bash) into `%LOCALAPPDATA%\mellos-mapping\bin`, appends that one
directory to your **user** PATH, and tells you so through the assistant. The
PATH change reaches only new processes — and a new tab of a running Windows
Terminal inherits the old environment, so close the terminal app entirely and
reopen it before the first `mmap`. The PATH edit keeps the
installer's guarantees: nothing happens when the entry is already there, and
a PATH that `setx` would damage (flattened `%VARIABLE%` references, truncation
past its limit) is refused outright.

A refused PATH is not a dead end: the same two shims are then written into
`%LOCALAPPDATA%\Microsoft\WindowsApps` (or `~/.local/bin`) — a directory your
PATH already names — so `mmap` is runnable in a new terminal with no PATH
change at all. Machines with a long PATH are exactly where that happens, and
where it matters most. The command takes the page to open as an argument
(`mmap omp-host-support`), and closes the pane when it is already open.

The step behind it is still a command of its own, for the cases the hook does
not cover — `--uninstall`, or re-adding a PATH entry you removed while the
shims stayed put:

```
node "<plugin dir>/scripts/install-mmap-command.mjs" [--uninstall]
```

(`--json` prints the install outcome as one JSON line instead of prose — the
mode the hook itself calls it in.) `npm i -g mellos-mapping` provides the
same `mmap` via `bin`, no shims involved.

## Update

```
claude plugin marketplace update mellos-mapping && claude plugin update mellos-mapping@mellos-mapping
```

Two steps because `plugin update` compares against the locally cached
marketplace clone — the first command is what actually pulls this repo.
Restart Claude Code to apply. Releases are version bumps on `main`.
(In-app, `/plugin` opens the same management UI.)

omp updates with its own two steps:

```
omp plugin marketplace update mellos-mapping && omp plugin upgrade mellos-mapping@mellos-mapping
```

Close the pane first (`q` in it): the watcher runs from the plugin copy, and on
Windows an open file cannot be renamed away, so an upgrade attempted while a
pane is open can fail with `EPERM` and leave the plugin cache empty — repaired
by `omp plugin install mellos-mapping@mellos-mapping --force`. Then: the first
command refreshes the catalog, the second reinstalls from it. omp also
refreshes a catalog entry it has not updated for a day at startup, unless
`marketplace.autoUpdate` is `off` — in the default `notify` mode that check
writes its finding to the debug log only, so `marketplace update` is the step
that makes an update visible. `upgrade` does not compare versions: it
force-reinstalls whatever the catalog names, which is why a release is
identified by its version bump rather than gated by it. Restart omp afterwards,
and reopen a pane still showing the old bundle (`q` in it, then `mmap_open`
again).

pi updates the package it installed, with no marketplace in between:

```
pi update npm:mellos-mapping
```

That is the npm source; a git source is reconciled to the ref your settings name
by pi's own update, and a versioned spec (`npm:mellos-mapping@0.27.0`) is pinned
and skipped. Restart the session afterwards: a session owns the map server
process it started, so one still running keeps the copy it began with until it
ends.

### Upgrading from 0.19

0.20 moved the store out of `.claude/` — the map belongs to this tool, not to
one client — into `.mellos/`. The server and the watcher each perform the move
once at startup, and print exactly one line on stderr when they do
(`mellos-mapping: moved the legacy .claude map store to .mellos/ — commit the
move.`):

| 0.19 and earlier | 0.20 and later |
| --- | --- |
| `.claude/mellos-mapping.json` | `.mellos/map.json` |
| `.claude/mellos-mapping.pages/` | `.mellos/pages/` |
| `.claude/mellos-mapping.config.json` | `.mellos/config.json` |

Nothing is merged and nothing is overwritten: a project that already has a
`.mellos/` store (map, pages or config) is left untouched, whatever the legacy
directory still holds. If you keep your maps in git, commit the move —
`git add -A .claude .mellos` records it as renames rather than as a pile of
deletions plus untracked files.

That one move is the only time either process touches `.claude/`. Afterwards
the tools write nowhere but `.mellos/`, and never outside the project
directory they resolved at startup.

## ChatGPT App · Codex mode

This is Codex mode in the ChatGPT desktop app (also called Codex App).
From the source branch run the following command; on `chatgpt-app`, omit the
host argument. It configures the desktop skill, marketplace and eight MCP tools.

```
node install.mjs chatgpt-app
```

Start a new conversation. The desktop skill defaults to **web-terminal**:
`mmap_open {surface: "web-terminal", page: "<slug>"}` starts a local service,
then the AI opens its URL in the current conversation's right browser panel.
The mmap terminal starts automatically, with its own font-size selector;
no manual paste or Computer Use is needed. Graphical SVG, native terminal and
Markdown remain available. With an older MCP schema, run
`node "<plugin root>/dist/web.mjs" "<project>" --terminal --page <slug>`.
A queued panel request does not prove that the user can see it.
See [desktop installation and limitations](docs/codex.md).

For Codex CLI inside Windows Terminal (not the desktop integrated terminal), run
`node <plugin root>/scripts/open-pane.mjs <project dir>` — it splits the
terminal window hosting the session, keeping keyboard focus on the conversation.
If that window cannot be identified or focused, it reports failure without
opening elsewhere. `--window` explicitly chooses a separate window. Add
`--page <slug>` to open on a particular page — and with a pane already open,
rerunning with `--page` retargets the pane belonging to this console. A map
open in another session or a separate window does not count as this split. The pane
auto-follows the page being written — the map the agent is operating on
right now; press `f` to toggle that (a manual page switch also turns it
off), or start with `--no-follow`. Elsewhere run
`node <plugin root>/dist/watch.mjs` from the project directory in a second
terminal (or any terminal split). Both take the same flags — see
[Pane flags](#pane-flags).

### Desktop Markdown map

When the user chooses the document surface, the skill uses
`mmap_open {surface: "markdown", page: "<slug>"}` and opens the returned file
beside the current conversation. The document embeds a colored SVG dependency
map plus module details, evidence, and child-page links. It needs no web server,
browser rendering process, Mermaid support, or additional runtime dependencies.

Successful MCP map writes regenerate enabled previews in `.mellos/previews/`;
JSON remains the source of truth. Image nodes are static. File visibility and
automatic viewer refresh belong to the desktop host. See the
[desktop map guide](docs/codex.md#desktop-right-side-map) for regeneration,
cache behavior, and limitations.

### Optional interactive web viewer

An additional interactive web viewer is available with
`mmap_open {surface: "web", page: "<slug>"}` or
`node "<plugin root>/dist/web.mjs" "<project directory>" --page <slug>`.
Open its returned URL in the host's right browser panel. It provides live
updates, pan/zoom, hover and pinned details, search, dependency highlighting,
filters, group overview, submaps and themes. Stop it with
`node "<plugin root>/dist/web.mjs" "<project directory>" --stop`.
Markdown/SVG and terminal workflows remain available; all surfaces share the
same map data. See [web viewer details](docs/codex.md#optional-interactive-web-viewer).

## Any MCP client

The server ships on npm, so any MCP client (Cursor, Windsurf, Zed,
Gemini CLI, …) can run it with a standard stdio entry:

```
npx -y mellos-mapping
```

The map file lands in the client session's working directory
(`.mellos/map.json`). Open the live pane from the same project:

```
npx -y -p mellos-mapping mellos-mapping-watch
```

The server picks its project directory in this order: `MELLOS_MAPPING_CWD`
(an explicit override, for clients that spawn servers from a fixed
directory), then `CLAUDE_PROJECT_DIR` (what Claude Code sets for plugin MCP
servers), then the server process's own working directory. Set
`MELLOS_MAPPING_CWD` when your client would otherwise start the server
somewhere other than the project you are working in.

The skill/discipline layer is Claude Code + Codex specific; other clients
get the eight `mmap_*` tools and the pane, and bring their own prompting.

## Use

1. Ask Claude to build something non-trivial. The bundled skill has Claude
   declare the ghost design and keep the map current as it works.
2. The pane opens itself. Every write tells Claude whether anybody is
   actually looking (see [Who is watching](#who-is-watching)), and Claude
   opens or retargets the pane with `mmap_open` when nobody is — you never
   have to remember to. Open or close it yourself with `mmap` in any
   terminal, or `/mellos-mapping:mmap` in the conversation (Windows
   Terminal split on Windows, tmux split inside tmux, a Pebrel split inside
   Pebrel, or a printed command to run in any second terminal). On Windows the pane opens in the terminal
   window hosting YOUR session, even with several windows open; pass
   `--window` to put the map in its own dedicated window instead. Prefer
   `--ascii` if your font lacks box-drawing glyphs.
3. Watch nodes light up from the bottom. Interrupt when the picture worries
   you — that is what it is for.

On Linux and macOS, `mmap_open` and `mmap` automatically use tmux. The launcher
targets the inherited `TMUX`/`TMUX_PANE`; if the tool process lost those variables,
it discovers the single attached session on the default tmux server. The default
is a right split that preserves input focus; `--window` creates a new tmux window.
Repeated opens reuse the watcher bound to that source pane or session window.

For multiple attached sessions or a custom tmux socket, set these variables in
the environment of the MCP server (restart it after changing them):

| Variable | Meaning |
| --- | --- |
| `MELLOS_MAPPING_TMUX_TARGET` | Explicit tmux session or pane, such as `work:2.1` or `%7` |
| `MELLOS_MAPPING_TMUX_SOCKET` | Absolute socket path, for example `/tmp/my-tmux/socket` |

Inside [Pebrel](https://github.com/Kuddev/pebrel), `mmap_open` and `mmap` split the pane that hosts
the conversation through Pebrel's runtime CLI (`PEBREL_CLI`). The launcher
confirms which window that pane is in from its process tree, returns input focus
to the conversation and types the watcher into the new pane's shell (PowerShell,
cmd or a POSIX shell), which exits with it so the pane closes with the map.
`--window` opens a new Pebrel window. Inside tmux, tmux owns the split.

Ambiguous sessions, detached sessions and missing tmux produce a concrete error
and a fully quoted watcher command, including the project and requested page,
to paste into any visible terminal. Map writes remain available; after an open
failure the assistant retries only when the environment changes or you ask.

The pane is mouse-aware (xterm SGR any-event tracking — the same protocol
htop and tmux speak):

| Input | Action |
| --- | --- |
| hover a node | spotlight its wires; preview its details below the map |
| click a node | pin it — details stay resident after the mouse leaves |
| click empty space / `Esc` | unpin; with nothing pinned, `Esc` climbs out of a dive |
| wheel / `+` `-` | zoom, anchored on the focused node (see the ladder below) |
| left-drag | grab and pan when the map outgrows the pane |
| shift+wheel | scroll vertically |
| wheel tilt (horizontal) | pan sideways |
| `hjkl` / arrows | nudge the view |
| `Tab` / `Shift+Tab` / `1-9` / click a tab | switch pages (parallel maps) |
| wheel on the tab row / click `‹` `›` | browse an overflowing tab strip without switching pages |
| `f` | toggle auto-follow (see [Pages](#pages)) |
| `x`, or click the `×` on the active tab | ask to delete the page on screen; press again inside the window and its file is removed (see [Pages](#pages)) |
| double-click a `⊞` node | dive into its sub-map (a child page) |
| `Backspace` / `Esc` | climb back out of the last dive |
| drag the `⋯` divider | resize the detail panel — pull it up to read long design notes in full |
| `0` | reset pan and zoom |
| `q` / `Ctrl+C` | quit the pane |

Every other key is inert, on purpose: an escape sequence the pane does not
know (F-keys, Home/End, PgUp/PgDn, Insert/Delete