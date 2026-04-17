---
name: release
description: Cut a release — commit any loose changes, tag, generate changelog, create the GitHub release. Use when the user says "release", "cut a release", "tag X", "release notes".
argument-hint: "[--major|--minor|--patch] [--draft]"
user_invocable: true
---

# Release

Tag and publish a GitHub release. Default: patch bump.

## Arguments

- `--major` / `--minor` / `--patch` (default) — which part of the version to bump
- `--draft` — create the GitHub release as a draft for review before publishing
- `<explicit-tag>` — skip auto-bump, use this exact tag (e.g. `v1.2.3`, `2026-04-17`)

## Steps

### 1. Clean working tree

```bash
git status --short
```

If there are uncommitted changes:
- **Feature branch**: run `/ship` first, then `/release` from the merged state.
- **Main branch with admin push rights** (solo repo, release-train convention): run `/precommit`, then commit directly. Use a `chore: release vX.Y.Z` or similar message.

Never release from a dirty tree.

### 2. Determine next tag

```bash
git tag --sort=-v:refname | head -5
```

- If tags are `vX.Y.Z`: bump per `--major` / `--minor` / `--patch` argument.
- If tags are date-based (`YYYY-MM-DD`): use today's date, add `-N` suffix if already released today.
- If no tags exist: start at `v0.1.0`.

Ask for confirmation before applying a major bump — those usually have breaking changes worth flagging.

### 3. Generate changelog

```bash
git log <prev-tag>..HEAD --oneline --no-merges
```

Synthesize into bullet points grouped by type (Features / Fixes / Chore), pulling from conventional-commit prefixes. Keep it scannable — skip noise like "bump version", "fix typo", "address review feedback".

### 4. Tag and push

```bash
git tag <new-tag> -m "<new-tag>"
git push origin <new-tag>
```

Use an annotated tag (`-m`), not a lightweight tag. Annotated tags carry author + message and are the convention for releases.

### 5. Create the GitHub release

```bash
gh release create <new-tag> \
  --title "<new-tag>" \
  --notes "$(cat <<'EOF'
## Highlights

<1-2 sentences on the most important change>

## Changes

### Features
- <bullet>

### Fixes
- <bullet>

### Chore
- <bullet>

**Full changelog**: https://github.com/<owner>/<repo>/compare/<prev-tag>...<new-tag>
EOF
)"
```

Add `--draft` if requested, or `--prerelease` if the tag has an `-alpha`/`-beta`/`-rc` suffix.

### 6. Report

Output the release URL (`gh release view <tag> --json url -q .url`). Mention what to do next:
- Deploy? → `/deploy` if the project has one.
- Publish to npm / PyPI? → trigger the publish workflow or run the publish command.
- Announce? → the release URL is the shareable artifact.

## Notes

- **Never delete or force-update a published tag.** If a release has an error, cut a new patch tag that supersedes it.
- **Pre-releases**: use suffixes like `v1.2.0-rc.1`. GitHub treats these as "pre-release" automatically if `--prerelease` is set.
- **Monorepo releases**: if packages have independent versions, this skill is per-package — run from the package dir, use tags like `@scope/pkg@1.2.3`. Changesets / release-please automate this better for monorepos at scale.
