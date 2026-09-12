---
name: update-app
description: A skill to update an existing app in the Umbrel community store (version bumps, image tags, metadata, compose changes) while keeping it compliant with store standards.
---

# Skill: Update App

## Description
This skill guides the process of updating an existing app in the Umbrel community store
repository. It covers version bumps, image tag changes, metadata edits, and
docker-compose changes, and ensures the app still adheres to the store standards.

See `.github/instructions/umbrel-apps.instructions.md` for the field reference and
Umbrel conventions.

## Implementation

1. **Identify the app**: Locate the app directory `{store-id}-{app}` and read both
   `umbrel-app.yml` and `docker-compose.yml`.
2. **Determine the change**: Confirm what is being updated (app version, image tag,
   description, gallery, ports, volumes, permissions, etc.).
3. **Apply the change**:
   - Bump `version` in `umbrel-app.yml` to the new semantic version.
   - Update the pinned `image` tag in `docker-compose.yml` to match the new version.
   - Update `releaseNotes` to describe the new version.
   - Update any other affected fields (description, gallery, dependencies, permissions).
4. **Validate**: Run the checklist below and check for errors in the edited files.

## Update Checklist

Verify against `.github/instructions/umbrel-apps.instructions.md`; the items below are
checks, not a second copy of the rules.

### Consistency
- [ ] `version` in `umbrel-app.yml` matches the pinned `image` tag in `docker-compose.yml`
- [ ] `releaseNotes` reflects the new version
- [ ] Directory name still matches the app `id` (all lowercase)
- [ ] `id` still matches the store ID prefix (`{store-id}-...`)

### umbrel-app.yml Validation
- [ ] All "umbrel-app.yml Fields" still present, including defaults/blank values
- [ ] `manifestVersion: 1` unchanged
- [ ] `developer`, `website`, `repo`, `support` URLs valid and HTTPS
- [ ] Valid YAML: keys camelCase, `description` uses `>-`, arrays formatted, no trailing whitespace
- [ ] Icon and gallery assets resolve over HTTPS (repo-hosted or external)

### docker-compose.yml Validation
- [ ] `version: "3.7"` and the app service are present
- [ ] Networking follows "Ports & Proxy" (`app_proxy` with a matching `APP_HOST`, or a host-bound port matching `umbrel-app.yml`'s `port`)
- [ ] `APP_HOST` matches `{directory-name}_{service-name}_1`
- [ ] Umbrel additions applied: `restart: unless-stopped`, `${APP_DATA_DIR}` subdirectory mounts, `PUID`/`PGID` where the image supports them
- [ ] `image` pinned to a version tag (not `latest`)
- [ ] Valid YAML

### Quality Checks
- [ ] Description includes setup instructions or important warnings
- [ ] Port does not conflict with common services
- [ ] All external URLs are HTTPS
- [ ] No unintended changes to unrelated apps or files