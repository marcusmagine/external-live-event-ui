# External Live Event UI

A small static web UI for managing **external live events** (streams hosted on a
third-party CDN such as CDN77) through the
[`external-live-event-api`](https://github.com/tvoli/infra-pylambdas/tree/master/media/external-live-event-api)
Lambda HTTP API.

It lets an operator create, list, edit and archive external live events without
touching the AWS console, the CLI scripts, or the Magine CMS. The API in turn
orchestrates `metadata-core` (viewable + live event playable) and `asset-info`
(playlists + CDN77 URL signing).

## What it does

- **Create** — title, start time, stream URL, optional CDN77 URL signing
  (`signingToken` / `expirySeconds`).
- **List** — shows all external live events for the configured partner; lazily
  enriches each row with the merged title + stream URL from the detail endpoint.
- **Edit** — update stream URL, signing token/expiry, start time.
- **Archive** — soft-delete (marks the live event `archived` in metadata-core;
  maps to "deleted" in the CMS console).

## Run it

It's a single `index.html` — no build step. Serve it over HTTP (a `file://`
origin is treated as a unique origin by browsers and breaks `fetch`):

```bash
cd external-live-event-ui
python3 -m http.server 8000
```

Then open <http://localhost:8000/>.

## Configure

On first load, open **API Configuration** and enter:

| Field | Value |
|-------|-------|
| Function URL | the Lambda Function URL, e.g. `https://xxxx.lambda-url.eu-west-1.on.aws/` |
| API Key | the `X-Api-Key` value (SSM `/config/external-live-event-api/{env}/ApiKey`) |
| Partner ID | e.g. `magine` |

Values are stored in `localStorage` on your machine only. They are not committed
to this repo and are sent only to the configured API.

## Environments

The same page works against devel / test / prod by changing the Function URL.
The `environment` field in create/update requests is derived from the URL
(`devel` / `test` / `prod`).

## Where the secrets live

- **API key** — in SSM, created by the
  [`external-live-event-api`](https://github.com/tvoli/infra-cloudformation/blob/master/live-event/external-live-event-api.yaml)
  CloudFormation stack. Paste it into the config box; it is not stored in this
  repo.
- **CDN77 signing token** — entered per event and stored in `asset-info` via the
  API. Visible in the edit modal (Show/Hide toggle).

## Related

- Lambda API: `tvoli/infra-pylambdas` → `media/external-live-event-api`
- CloudFormation stack: `tvoli/infra-cloudformation` → `live-event/external-live-event-api.yaml`
- Backend services: `metadata-core` (viewables + live events), `asset-info`
  (playlists + URL signing)
