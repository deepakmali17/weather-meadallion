# Deployment setup

## Azure DevOps objects

Create:

- Service connection: `SC-Azure-Weather`
- Variable group: `weather-dev-databricks`
- Variable group: `weather-prod-databricks`
- Environment: `weather-dev`
- Environment: `weather-prod`

Variable groups:

```text
DATABRICKS_CLIENT_ID
DATABRICKS_CLIENT_SECRET   # secret
```

`weather-prod` should have an approval check.

## Branch policy

Recommended:

```text
feature/* -> dev -> main
```

Require PR approval for:

- `dev`
- `main`

Require the pipeline to pass before completing the PR.

## Databricks CI/CD identity

Use a dedicated Microsoft Entra service principal for the pipeline.

Add it to both Databricks workspaces and give it the minimum workspace permissions required to import/update notebooks.

OAuth M2M is preferred over PAT authentication.

## ADF identity permissions

Each ADF system-assigned managed identity needs:

- `Storage Blob Data Contributor` on its environment's ADLS Gen2 storage
- the required Azure Databricks workspace access for the Databricks linked service

## Local test

Validate:

```bash
python scripts/validate_notebooks.py
python scripts/render_adf.py --env dev --output .generated/adf
```

Deploy notebooks manually:

```bash
export DATABRICKS_HOST="<dev workspace URL>"
export DATABRICKS_CLIENT_ID="<client id>"
export DATABRICKS_CLIENT_SECRET="<client secret>"
bash scripts/deploy_databricks_notebooks.sh dev
```

Deploy ADF:

```bash
az login
bash scripts/deploy_adf.sh dev
```

Do not commit `.generated/`.

## Production

Before merging to `main`, make sure `deployment/config/prod.json` contains real production resource values.

Production deployment is gated by the `weather-prod` Azure DevOps environment approval.
