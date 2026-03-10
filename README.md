# Holden Actions

GitHub Actions for [Holden](https://github.com/benjick/holden), a self-hosted container orchestrator.

## Actions

### validate

Validates `holden.yml` configuration using the Holden CLI.

```yaml
- uses: holden-run/actions/validate@main
```

For monorepos with multiple apps:

```yaml
- uses: holden-run/actions/validate@main
  with:
    paths: |
      apps/api
      apps/frontend
```

### deploy

Triggers a Holden deployment via webhook with HMAC-SHA256 signing.

```yaml
- uses: holden-run/actions/deploy@main
  with:
    webhook-url: ${{ secrets.HOLDEN_WEBHOOK_URL }}
    app-id: my-app
    webhook-secret: ${{ secrets.HOLDEN_WEBHOOK_SECRET }}
```

| Input | Required | Description |
|-------|----------|-------------|
| `webhook-url` | Yes | Webhook base URL (e.g. `https://holden.example.com/webhook`) |
| `app-id` | Yes | App ID as registered in Holden |
| `webhook-secret` | Yes | Secret for HMAC signature (same as `HOLDEN_WEBHOOK_SECRET`) |

## Full Example

```yaml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Validate Holden config
        uses: holden-run/actions/validate@main

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v6
        with:
          push: true
          tags: ghcr.io/myorg/myapp:latest

      - name: Deploy to Holden
        uses: holden-run/actions/deploy@main
        with:
          webhook-url: ${{ secrets.HOLDEN_WEBHOOK_URL }}
          app-id: my-app
          webhook-secret: ${{ secrets.HOLDEN_WEBHOOK_SECRET }}
```

## License

AGPL-3.0
