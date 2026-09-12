---
applyTo: "**"
---

# Umbrel App Store Conventions

Reference for defining apps in the Umbrel community store. These conventions apply to
every app in this repo.

## Configuration

### umbrel-app.yml Fields
All properties below must be present in `umbrel-app.yml`, even when using a default or
blank value.

- `manifestVersion`: Always set to `1`
- `id`: Unique identifier for the app (lowercase, hyphens allowed). It must match the app directory name and be prefixed with the store ID, e.g., `{store-id}-example-app`
- `name`: Display name shown in UI (e.g., "Example App")
- `tagline`: Short one-line description (max ~80 chars, e.g., "A short description of the app")
- `icon`: HTTPS URL to the app icon. Assets may be committed to the app directory and
  referenced via a raw GitHub URL
  (e.g. `https://raw.githubusercontent.com/{owner}/{repo}/{branch}/{app-dir}/icon.svg`),
  but hosting in the repo is **not required** — an external HTTPS URL is also allowed
- `category`: App category (e.g., "ai", "productivity", "utilities")
- `version`: Semantic version of the app (e.g., "1.0.0")
- `port`: External port on the host where the app is reachable (e.g., 8080). See "Ports & Proxy" below.
- `description`: Detailed markdown description with usage instructions, warnings, setup steps
- `developer`: Organization/developer name (e.g., "Example Org")
- `website`: Official website URL (e.g., "https://example.com/")
- `repo`: GitHub repository URL
- `support`: Issue tracker or support URL
- `gallery`: Array of image URLs for app showcase. Assets may
  be committed to the app directory and referenced via raw GitHub URLs, but this is
  **not required** — external HTTPS URLs are also allowed.
- `releaseNotes`: Release notes text
- `dependencies`: Array (can be empty: `[]`)
- `path`: Empty string: `""`
- `defaultUsername`: Empty string unless app requires default credentials
- `defaultPassword`: Empty string unless app requires default credentials
- `submitter`: Your username or handle
- `submission`: Link to your app store repository
- `permissions`: Array of required Umbrel permissions (e.g. `[]`, or
  `- STORAGE_DOWNLOADS`, or `["GPU"]` for GPU-dependent apps)

### Umbrel-Specific Docker Compose Additions
These are Umbrel conventions layered on top of standard Docker Compose. See the Docker
Compose docs for general syntax; only the Umbrel-specific parts are covered here.

- **Ports**: Normally the app sits behind Umbrel's platform-provided `app_proxy` service,
  which handles host routing, so the app service does **not** publish a port itself. If
  you omit `app_proxy`, the app service must publish the port in the compose file
  instead. See "Ports & Proxy" below for the rules.
- **Volume path variables**: Umbrel injects environment variables you can use in
  `volumes`:
  - `${APP_DATA_DIR}` — the app's persistent data directory on the host. Always mount
    under its `./data` subdirectory (never at the variable root), e.g.
    `${APP_DATA_DIR}/data:/app/data`
  - `${UMBREL_ROOT}` — the Umbrel install root (use for host paths the user manages,
    e.g. `${UMBREL_ROOT}/home/Downloads:/downloads`)
- **User permissions via environment variables**: When the image supports it, prefer
  setting the user/group through environment variables so files are owned correctly:
  ```yaml
  environment:
    - PUID=99
    - PGID=100
  ```
  (Values shown are examples; check the image's docs for the expected variables.)
- **Restart policy**: Always set `restart: unless-stopped` so the app comes back up.
- **Images**: Pin to a specific version tag (never `latest`).

### Inter-Service Communication
To reach another service, use its Docker container alias
(`{directory-name}_{service-name}_1`), not `localhost` or the host port. This applies
whether the service is in the same `docker-compose.yml`, another app in this repo, or an
app from another store. Include the target's internal container port, e.g.:

```yaml
environment:
  - EXAMPLE_SERVICE_URL=http://{store-id}-{other-app-dir}_{other-service-name}_1:{internal-port}
```

### Ports & Proxy
- `port` in `umbrel-app.yml` **always** refers to the host port where the app is
  reachable.
- Providing `app_proxy` in `docker-compose.yml` makes the host route traffic to the
  proxy automatically. The proxy's `APP_PORT` refers to the app's internal container
  port and does **not** need to match the `port` in `umbrel-app.yml`.
- When `app_proxy` is used, its `APP_HOST` must match the app container's Docker alias:
  `{directory-name}_{service-name}_1`.
- `app_proxy` is **optional**. When it is omitted, the `docker-compose.yml` must bind
  the port to the host itself, and the bound host port **must** match the `port` in
  `umbrel-app.yml`:
  ```yaml
  example-app:
    image: example/example-app:1.0.0
    ports:
      - "8080:8080"   # host:container; host port must match `port` in umbrel-app.yml
  ```

## Examples

### Example 1: Creating a basic app

**umbrel-app.yml configuration:**
```yaml
manifestVersion: 1
id: example-store-example-app
name: Example App
tagline: A short description of the app
icon: https://raw.githubusercontent.com/{owner}/{repo}/{branch}/example-store-example-app/icon.svg
category: utilities
version: "1.0.0"
port: 8080
description: >-
  A detailed description of the app and how to use it...
developer: Example Org
website: https://example.com/
repo: https://github.com/example/example-app
support: https://github.com/example/example-app/issues
gallery:
  - https://raw.githubusercontent.com/{owner}/{repo}/{branch}/example-store-example-app/1.jpg
  - https://raw.githubusercontent.com/{owner}/{repo}/{branch}/example-store-example-app/2.jpg
releaseNotes: >-
  v1.0.0: Initial release.
dependencies: []
path: ""
defaultUsername: ""
defaultPassword: ""
submitter: your-username
submission: https://github.com/{owner}/{repo}
permissions: []
```

**docker-compose.yml service configuration:**
```yaml
version: "3.7"

services:

  app_proxy:
    environment:
      # The format here is: {app-id}_{docker-service-name}_1
      APP_HOST: example-store-example-app_example-app_1
      APP_PORT: 8080

  example-app:
    image: example/example-app:1.0.0
    volumes:
      - ${APP_DATA_DIR}/data:/app/data
    restart: unless-stopped
```

### Example 2: Creating an app with a different internal/external port
**Key difference:** Internal port (e.g., 3000) differs from external port (e.g., 8080)

**docker-compose.yml service configuration:**
```yaml
version: "3.7"

services:

  app_proxy:
    environment:
      APP_HOST: example-store-example-app_example-app_1
      APP_PORT: 3000

  example-app:
    image: example/example-app:1.0.0
    volumes:
      - ${APP_DATA_DIR}/data:/app/data
    environment:
      - CUSTOM_VAR=value
    restart: unless-stopped
```
