# oh-my-posh AGENTS.md
# Agent Instructions

General coding guidelines, commit conventions, and agent workflows for this repository.

## Project Overview

Oh My Posh is a cross-shell prompt theme engine written in Go. It renders prompt segments by
querying an `Environment` abstraction that wraps all OS/shell interactions.

Project knowledge lives at:

`~/Projects/Agentic_Engineering/projects/shell-tools/ohmyzsh`

Read before work:
- `project-profile.md`
- `current-work.md`
- `ai-handoff.md`
- `roadmap.md`
- `decisions.md`

## Tech Stack

| Layer                     | Technology                    |
| ------------------------- | ----------------------------- |
| Core engine               | Go (module root: `src/`)      |
| Documentation site        | Docusaurus (MDX) - `website/` |
| Themes                    | JSON - `themes/`              |
| Config format             | TOML / JSON / YAML            |
| Package/installer scripts | `packages/`                   |
| Build scripts             | `build/`                      |

## Key Commands

```bash
# Go - run from src/
go test ./...
go test ./segments/... -run TestFoo  # single test
golangci-lint run

# Docs - run from website/
npm run start    # local dev server
npm run build    # validate before opening a docs PR
```

## Codebase Exploration

**Always explore the actual codebase before planning or writing code.** Do not rely on memory
or assumptions. Use the file system tools to read relevant files first - the codebase evolves
and the feature you're asked to add may already exist.

## Repository Layout

- `src/` - Go module: `segments/`, `prompt/`, `runtime/`
- `themes/` - Bundled JSON theme files
- `website/` - Docusaurus docs site
- `packages/` - Installer/package manifests
- `build/` - CI build helpers

Key paths inside `src/`:

| Path                           | Purpose                                               |
| ------------------------------ | ----------------------------------------------------- |
| `src/segments/`                | One `.go` + one `_test.go` per segment                |
| `src/config/segment_types.go`  | Segment type registry (gob + string constants)        |
| `src/cli/`                     | CLI commands (cmdtree); `root.go` is the entry point  |
| `src/prompt/engine.go`         | Segment rendering loop                                |
| `src/cache/`                   | Existing TTL/file/command-path cache infrastructure   |
| `src/runtime/`                 | `Environment` abstraction + mock                      |

## Segment Development

Starting a new segment requires:
1. `src/segments/<name>.go` - segment impl
2. `src/segments/<name>_test.go` - tests
3. `website/docs/segments/<name>.mdx` - docs
4. Register in `website/sidebars.js` and `website/static/schema.json`
5. Register type in `src/config/segment_types.go` via `gob.Register`

Each segment implements the `Segment` interface; use `env` (the `Environment` abstraction) for OS/shell calls.
Every segment lives in `src/segments/` and implements the `SegmentWriter` interface. Use the
`Environment` abstraction (`env`) for **all** OS/shell calls - never call OS APIs directly.

Adding a segment requires **five** artifacts - use the `segment-create` skill to scaffold all
of them automatically:

1. `src/segments/<name>.go` - segment source
2. `src/segments/<name>_test.go` - unit tests
3. `website/docs/segments/<name>.mdx` - user-facing docs
4. Update `website/sidebars.js` and `website/static/schema.json`
5. Register the type in `src/config/segment_types.go` via `gob.Register(&segments.MySegment{})`

Missing step 5 causes the segment to fail silently at runtime.

## Commands

- Test: `go test ./...` from `src/`
- Lint: `golangci-lint run` from `src/`
- Docs dev: `npm run start` in `website/`
- Docs build: `npm run build` in `website/`

## Shell Integration

`oh-my-posh init <shell>` is how users wire oh-my-posh into their shell. It:

1. Writes a shell-specific init script to the cache (source: `src/shell/scripts/omp.<ext>`)
2. Returns a one-liner for the shell to `eval` - this sources the cached script, which hooks
   into prompt rendering

The `src/shell/` package contains per-shell logic (`pwsh.go`, `bash.go`, `zsh.go`, etc.) that
generates the hook commands. The scripts in `src/shell/scripts/` are embedded and templated at
init time. When modifying shell behaviour, changes typically span both the `.go` file and the
corresponding script.

Supported shells: `bash`, `zsh`, `fish`, `powershell`/`pwsh`, `cmd`, `nu`, `elvish`, `xonsh`.

## CLI Commands

CLI commands use the internal `src/cmdtree` command tree and live in `src/cli/`. To add a new
command:

1. Create `src/cli/<name>.go` with a `var <name>Cmd = &cmdtree.Command{...}`
2. Register it in `src/cli/root.go` via `RootCmd.AddCommand(<name>Cmd)`

## Caching

`src/cache/` provides the existing caching infrastructure - use it instead of building new
cache logic. It supports TTL-based key/value storage, file-based persistence, and command-path
caching. Do not introduce new cache packages unless `src/cache/` genuinely cannot meet the
requirement.

## Comments

Applies to every language in this repository (Go, shell scripts, PowerShell, JavaScript/TypeScript,
Lua, etc.) - not just the primary language of whatever file you're touching.

- Default to no comment. Add one only when the code cannot say it on its own.
- Never restate what a function/type/variable already makes obvious from its name, signature,
  and body. A comment that just paraphrases the name is noise - delete it.
- Only comment the WHY: a hidden constraint, a non-obvious invariant, a workaround for a specific
  bug, an external requirement, or a caveat that would surprise a reader. If there's nothing like
  that to say, leave the declaration uncommented - even exported/public ones.
- When a comment is warranted, keep it to the minimum needed to convey that non-obvious point.
  Don't pad it with restating context the code already shows.
- Language-specific skills (e.g. `golang`) may add formatting conventions (complete sentences,
  doc-comment placement) on top of this rule as a stricter minimum, but must not relax it.

## Go Conventions

Follow the `golang` skill for project-specific Go standards.

## Documentation (website/)

- Follow the `markdown` skill for `.md`/`.mdx` formatting rules.
- Segment doc pages live in `website/docs/segments/` and use MDX frontmatter with `title`, `sidebar_label`, and `id`.

## PowerShell

PowerShell helper scripts live in `packages/` and `build/`. Follow the `powershell` skill for cmdlet conventions.

## Themes

Themes are JSON files in `themes/`. Must validate against `website/static/schema.json`.
