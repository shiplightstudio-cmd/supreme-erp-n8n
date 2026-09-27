# Supreme ERP n8n

This directory contains the Docker image definition, local configuration
template, and workflow exports for Supreme ERP automations. The n8n server is
run by the repository-root Docker Compose stack.

## Start n8n

From the repository root, create the local n8n configuration if it does not
already exist:

```bash
cp supreme-erp-n8n/.env.example supreme-erp-n8n/.env
```

Set the required values in `supreme-erp-n8n/.env`, then build and start the
service:

```bash
docker compose up -d --build n8n
```

Open [http://localhost:5678](http://localhost:5678) to complete the owner
account setup or sign in.

The service builds the `supreme-erp/n8n:local` image from this directory's
`Dockerfile`. Workflow data, credentials, and the n8n settings file persist in
the `n8n_data` Docker volume.

## Configuration

Do not commit `.env`. In particular, keep `N8N_ENCRYPTION_KEY` stable after
the first startup: changing it prevents n8n from decrypting existing
credentials. Generate a key for a new instance with:

```bash
openssl rand -hex 32
```

For local development, n8n is available at `http://localhost:5678` and uses
the `Asia/Jakarta` timezone. The root Compose file provides
`host.docker.internal`, which can be used by n8n nodes to reach services on
the Docker host.

For a public deployment, place n8n behind HTTPS and set all of these values to
the public URL before starting it:

```dotenv
N8N_HOST=n8n.example.com
N8N_PROTOCOL=https
N8N_EDITOR_BASE_URL=https://n8n.example.com/
N8N_WEBHOOK_URL=https://n8n.example.com/
N8N_SECURE_COOKIE=true
```

## Finance workflows

- `daily-finance-digest.json` sends the scheduled finance digest.
- `monthly-finance-digest.json` sends the completed prior-month finance digest.
- `daily-finance-error-alert.json` sends an alert when the digest fails.

Import them from the n8n editor using **Workflows → Import from File**. Then
assign the error-alert workflow as the digest workflow's **Error Workflow**,
configure SMTP credentials for the email nodes, and activate the workflows.

The automation requires these environment variables:

| Variable | Purpose |
| --- | --- |
| `ERP_REPORTING_URL` | Private URL for the ERP reporting API. |
| `AUTOMATION_REPORT_SECRET` | Shared secret sent to the ERP reporting endpoint. |
| `FINANCE_REPORT_FROM` | Sender address used by the finance email workflow. |
| `AUTOMATION_ALERT_RECIPIENTS` | Comma-separated recipients for failure alerts. |

## Operations

```bash
# View server logs
docker compose logs -f n8n

# Restart after changing the local environment file
docker compose up -d --no-deps n8n

# Stop n8n without deleting workflows or credentials
docker compose stop n8n
```

Avoid `docker compose down --volumes` unless you intentionally want to delete
