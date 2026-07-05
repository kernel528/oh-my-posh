# oh-my-posh AGENTS.md

| Layer                     | Technology                    |
| ------------------------- | ----------------------------- |
| Core engine               | Go (module root: `src/`)      |
| Documentation site        | Docusaurus (MDX) - `website/` |
| Themes                    | JSON - `themes/`              |
| Config format             | TOML / JSON / YAML            |
| Package/installer scripts | `packages/`                   |
| Build scripts             | `build/`                      |

## Repository Layout

- `src/` - Go module: `segments/`, `prompt/`, `runtime/`
- `themes/` - Bundled JSON theme files
- `website/` - Docusaurus docs site
- `packages/` - Installer/package manifests
- `build/` - CI build helpers

## Segment Development

Starting a new segment requires:
1. `src/segments/<name>.go` - segment impl
2. `src/segments/<name>_test.go` - tests
3. `website/docs/segments/<name>.mdx` - docs
4. Register in `website/sidebars.js` and `website/static/schema.json`
5. Register type in `src/config/segment_types.go` via `gob.Register`

Each segment implements the `Segment` interface; use `env` (the `Environment` abstraction) for OS/shell calls.

## Commands

- Test: `go test ./...` from `src/`
- Lint: `golangci-lint run` from `src/`
- Docs dev: `npm run start` in `website/`
- Docs build: `npm run build` in `website/`

## Themes

Themes are JSON files in `themes/`. Must validate against `website/static/schema.json`.
