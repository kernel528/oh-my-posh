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

| Path                          | Purpose                                             |
| ----------------------------- | --------------------------------------------------- |
| `src/segments/`               | One `.go` + one `_test.go` per segment              |
| `src/config/segment_types.go` | Segment type registry (gob + string constants)      |
| `src/cli/`                    | CLI commands (Cobra); `root.go` is the entry point  |
| `src/prompt/engine.go`        | Segment rendering loop                              |
| `src/cache/`                  | Existing TTL/file/command-path cache infrastructure |
| `src/runtime/`                | `Environment` abstraction + mock                    |

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

## Themes

Themes are JSON files in `themes/`. Must validate against `website/static/schema.json`.
