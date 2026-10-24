# oh-my-posh AGENTS.md
# Agent Instructions

Oh My Posh is a cross-shell prompt theme engine written in Go, with the module root in `src/`. Rendering runs on every
shell prompt, so it is a hot path: no extra process spawns, network calls, or file reads per render without
`src/cache`.

## Non-negotiables

Corrections in past sessions keep landing on these. Check each one before you report a change as done.

Project knowledge lives at:

`~/Projects/Agentic_Engineering/projects/shell-tools/oh-my-posh`

Read before work:
- `project-profile.md`
- `current-work.md`
- `ai-handoff.md`
- `roadmap.md`
- `decisions.md`

## Tech Stack
1. **Reuse before building.** Find how the codebase already solves the problem before designing anything: a similar
   segment, an `Environment` method, `src/cache`, or a CLI call behind an option like other segments use. Never add a
   dependency without asking. Remove deprecated logic instead of maintaining it.
2. **Go through `runtime.Environment`** (`s.env` in segments) for every OS, shell, file, and command call, so tests can
   mock it. Check its methods before reaching for `os`, `os/exec`, or `path/filepath`.
3. **Tests prove behavior, not the patch.** A test must fail when the behavior breaks and survive a correct refactor.
   Write them table-driven; the `golang` skill has the rules.
4. **No `else`.** Use guard clauses, early returns, or a `switch`. The Stop hook blocks new `else` branches.
5. **Comments explain a why the code cannot.** Never restate a name or signature, exported symbols included, in any
   language. Default to no comment.
6. **Log with `src/log`:** `log.Error(err)` where the error occurs, and `defer log.Trace(time.Now(), args...)` in
   functions worth tracing. Do not format errors yourself.
7. **Commit only when asked, push only when asked.** Fold every fix to a commit that is not on `main` into that commit
   with `git commit --fixup <sha>` and `git rebase --autosquash`, lint and CI fallout included. Never rewrite `main`.

## Repository map

| Path                          | Purpose                                                                      |
| ----------------------------- | ---------------------------------------------------------------------------- |
| `src/segments/`               | One `.go` and one `_test.go` per segment                                     |
| `src/config/`                 | Segment registry; an unregistered segment fails silently at runtime          |
| `src/prompt/`                 | Rendering engine                                                             |
| `src/runtime/`                | `Environment` abstraction and its mock                                       |
| `src/cache/`                  | TTL, file, and command-path caching; do not build another cache              |
| `src/cli/`                    | CLI commands on `src/cmdtree`, registered in `root.go`                       |
| `src/shell/`                  | `oh-my-posh init <shell>`; `scripts/` are embedded, so rebuild after editing |
| `themes/`                     | Bundled themes, validated against `themes/schema.json`                       |
| `website/`                    | Docusaurus docs; segment pages live in `website/docs/segments/<category>/`   |

A shell integration change usually spans `src/shell/<shell>.go` and its script in `src/shell/scripts/`.

## Commands

```bash
# from src/
go test ./...
golangci-lint run

# from website/
npm run build    # before a docs pull request
```

The Stop hook (`.agents/hooks/main.go`) formats the changed files, runs `modernize`, `fieldalignment`, `golangci-lint`,
`go test`, `markdownlint`, and the `else` check, and builds, lints, and analyzes platform-specific code for the other
operating systems. Fix what it reports; never run `fieldalignment -fix`. `.claude/settings.json` and
`.github/hooks/quality.json` both wire it, so change them together.

## Docs

Docs pages have no H1, because the front-matter `title` renders as the heading; use only H2 and H3. Wrap lines at 120
characters. Use the `segment-create` skill for a new segment and `segment-docs` for its documentation.

- `src/` - Go module: `segments/`, `prompt/`, `runtime/`
- `themes/` - Bundled JSON theme files
- `website/` - Docusaurus docs site
- `packages/` - Installer/package manifests
- `build/` - CI build helpers
## Project knowledge

Before touching shell scripts, terminal or pty behavior, WSL-based testing, engine internals, or release and CI work,
read the matching file in the `project-knowledge` skill. When a session uncovers a verified, actionable gotcha, append
it to that file and commit it with the related change.

## Pull request reviews

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
=======
Validate every comment against the code. Fold each fix into the commit that introduced the code with a fixup and
`git rebase --autosquash`, then force-push the pull request branch with `--force-with-lease`. Reply to every thread
with the conclusion and the commit, then resolve it.
