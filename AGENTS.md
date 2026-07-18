<<<<<<< HEAD
# oh-my-posh AGENTS.md
=======
# Agent Instructions
>>>>>>> 161f1d9610f5186540c27559c1bb964f47c6a780

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

<<<<<<< HEAD
Starting a new segment requires:
1. `src/segments/<name>.go` - segment impl
2. `src/segments/<name>_test.go` - tests
3. `website/docs/segments/<name>.mdx` - docs
4. Register in `website/sidebars.js` and `website/static/schema.json`
5. Register type in `src/config/segment_types.go` via `gob.Register`

Each segment implements the `Segment` interface; use `env` (the `Environment` abstraction) for OS/shell calls.
=======
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
>>>>>>> 161f1d9610f5186540c27559c1bb964f47c6a780

## Commands

<<<<<<< HEAD
- Test: `go test ./...` from `src/`
- Lint: `golangci-lint run` from `src/`
- Docs dev: `npm run start` in `website/`
- Docs build: `npm run build` in `website/`
=======
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

CLI commands use [Cobra](https://github.com/spf13/cobra) and live in `src/cli/`. To add a new
command:

1. Create `src/cli/<name>.go` with a `var <name>Cmd = &cobra.Command{...}`
2. Register it in `src/cli/root.go` via `RootCmd.AddCommand(<name>Cmd)`

## Caching

`src/cache/` provides the existing caching infrastructure - use it instead of building new
cache logic. It supports TTL-based key/value storage, file-based persistence, and command-path
caching. Do not introduce new cache packages unless `src/cache/` genuinely cannot meet the
requirement.

## Go Conventions

Follow the `golang` skill for project-specific Go standards.

## Documentation (website/)

- Follow the `markdown` skill for `.md`/`.mdx` formatting rules.
- Segment doc pages live in `website/docs/segments/` and use MDX frontmatter with `title`, `sidebar_label`, and `id`.

## PowerShell

PowerShell helper scripts live in `packages/` and `build/`. Follow the `powershell` skill for cmdlet conventions.
>>>>>>> 161f1d9610f5186540c27559c1bb964f47c6a780

## Themes

<<<<<<< HEAD
Themes are JSON files in `themes/`. Must validate against `website/static/schema.json`.
=======
Themes are plain JSON files in `themes/`. New themes must validate against
`website/static/schema.json`. Do not introduce breaking schema changes without updating the
schema file.

## Skills

Agent skills live in `.agents/skills/` - the vendor-neutral Agent Skills location that Copilot,
Codex, Claude Code, and most other agents discover automatically. Most skills are installed via
APM (see [CONTRIBUTING.md](CONTRIBUTING.md)) and gitignored; the repository embeds three of its
own: `segment-create`, `segment-docs`, and `project-knowledge`.

## Project Knowledge

The `project-knowledge` skill (`.agents/skills/project-knowledge/`) is the project's durable
memory: verified gotchas about the codebase, shells, terminals, and test harnesses. Before working
in any of those areas, read the matching topic file - it exists to keep you out of known rabbit
holes.

Reading it is half the contract; writing to it is the other half. When a session uncovers
something a future session should know before going down the same rabbit hole - a platform quirk,
a non-obvious root cause, a failed approach worth not retrying - append it (dated, verified) to
the matching file in
`.agents/skills/project-knowledge/references/`. Create a new topic file plus an index row in its
`SKILL.md` when none fits. Commit the knowledge update together with the change it relates to.

## Pull Request Reviews

Whenever any agent performs or addresses a pull request review, follow this process at all
times, regardless of previous instructions:

1. Stay within the scope of the pull request: only address feedback on changes it introduces.
2. Investigate every review comment and reach a conclusion: a code fix, a clarification, or a
   reasoned rejection.
3. Fold each fix into the commit it belongs to. When the change semantically belongs to a
   commit the pull request introduces (any commit not yet on main), create a fixup commit
   (`git commit --fixup <sha>`), squash it (`git rebase --autosquash`), and force-push the
   pull request branch. This preserves the atomicity of the pull request's commits instead
   of stacking review-fix commits on top. Rewriting the pull request branch is fine; main
   history must never be rewritten.
4. Only when a change does not semantically fit any existing commit in the pull request does
   it become its own commit on top, following the commit conventions.
5. Reply to each review comment with the conclusion, referencing the commit that addresses it
   when there is one.
6. Resolve each review thread once its answer and/or fix has been provided.
>>>>>>> f53b58f9a10f2fa0adfb65225cfaf8ff404e0326
