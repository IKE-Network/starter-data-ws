# starter-data-ws

## First Steps

Run `mvn ws:scaffold-init` to clone components, then `mvn validate` to unpack
full build standards into `.claude/standards/`.

## Build

```bash
mvn clean verify -DskipTests -T 1C   # compile + javadoc
mvn clean verify -T 1C               # full build with tests
```

## Key Conventions

- Maven 4 with POM modelVersion 4.1.0
- `<subprojects>` (not `<modules>`) for aggregation
- All projects use `--enable-preview` (Java 25)
- Parent: `network.ike.platform:ike-parent` (from ike-platform)

## Prohibited Patterns

These are the most critical rules. Full standards are in `.claude/standards/MAVEN.md`
after building.

- **Never use `maven-antrun-plugin`** — use a proper Maven goal or `exec-maven-plugin`
  with an external script
- **Never use `build-helper-maven-plugin` for multi-execution property chaining** —
  write a proper Maven goal in `ike-maven-plugin` instead
- **Never embed shell commands inline in POM** — extract to a named script
- **Never use `git add -A` or `git add .`** — stage specific files
- **Never use raw git for workspace ops** (commit, push, checkout, merge, branch,
  stash) — use the `ws:` goals (`ws:commit-publish`, `ws:push`, `ws:switch-publish`, …).
  `workspace.yaml` `sha:` pins are checkpoint-managed — never hand-edit them.
  `depends-on` `build` edges are machine-derived from POMs the same way; to declare
  an edge derivation must not overwrite, use `relationship: bundle`/`content`
  (hand-declared, preserved). See `.claude/standards/IKE-WORKSPACE.md`

## Project-Specific Notes

See `WS-REFERENCE.md` for complete workspace goal documentation.
See `CLAUDE-starter-data-ws.md` for workspace-specific information.
See `.claude/standards/` (after `mvn validate`) for full build standards.
