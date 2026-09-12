---
name: define-new-app
description: A skill to define a new app in the Umbrel community store by creating the necessary directory and files.
---

# Skill: Define New App

## Description
This skill guides the process of defining a new app in the Umbrel community store repository. It involves creating the app directory, umbrel-app.yml metadata file, and docker-compose.yml configuration file.

See `.github/instructions/umbrel-apps.instructions.md` for the field reference and Umbrel conventions.

## Implementation

1. **Determine store ID**: Read `umbrel-app-store.yml` to get the store ID (e.g., `{store-id}`).
2. **Create app directory**: The directory should be named using the full `id` value from umbrel-app.yml (e.g., `{store-id}-example-app`).
3. **Create umbrel-app.yml**: 
   - Include all fields from the "umbrel-app.yml Fields" section in
     `.github/instructions/umbrel-apps.instructions.md` (use defaults/blank values
     where not applicable)
   - Use proper YAML formatting with multi-line strings using `>-` for descriptions
   - Set `manifestVersion: 1` always
4. **Create docker-compose.yml**: 
   - Set version to `"3.7"`
   - Either add an `app_proxy` service with the correct `APP_HOST` naming convention, or
     bind the host port directly in the app service (matching `umbrel-app.yml`'s `port`)
   - Create main app service with image, environment, volumes, devices as needed
   - Include `restart: unless-stopped` policy
5. **Validate**: Check for any errors in the generated files and ensure they follow the repository conventions.

## Validation Checklist

Verify against `.github/instructions/umbrel-apps.instructions.md`; the items below are
checks, not a second copy of the rules.

### Directory & Files
- [ ] Directory name matches the app `id` (all lowercase)
- [ ] `umbrel-app.yml` and `docker-compose.yml` present
- [ ] Icon and gallery assets resolve over HTTPS (repo-hosted or external)

### umbrel-app.yml Validation
- [ ] All "umbrel-app.yml Fields" present, including defaults/blank values
- [ ] `developer`, `website`, `repo` URLs valid
- [ ] Valid YAML: keys camelCase, `description` uses `>-`, arrays formatted, no trailing whitespace

### docker-compose.yml Validation
- [ ] `version: "3.7"` and the app service are present
- [ ] Networking follows "Ports & Proxy" (`app_proxy` with a matching `APP_HOST`, or a host-bound port matching `umbrel-app.yml`'s `port`)
- [ ] Umbrel additions applied: `restart: unless-stopped`, `${APP_DATA_DIR}` subdirectory mounts, `PUID`/`PGID` where the image supports them
- [ ] `image` pinned to a version tag (not `latest`)
- [ ] Valid YAML

### Quality Checks
- [ ] Description includes setup instructions or important warnings
- [ ] Port does not conflict with common services
- [ ] All external URLs are HTTPS