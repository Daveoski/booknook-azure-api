# BookNook API – Azure Cloud Capstone

A reading-list REST API built with **Express** and deployed on **Microsoft Azure**. It shows a typical cloud-native setup: secrets in Key Vault, data in Azure SQL, files in Blob Storage, passwordless access through managed identity, and CI/CD with GitHub Actions.

## Architecture

```text
GitHub Actions ──deploy──▶ Azure App Service (Node 22, Express)
                                 │  DefaultAzureCredential (managed identity)
                 ┌───────────────┼──────────────────┐
                 ▼               ▼                  ▼
          Azure Key Vault    Azure SQL DB     Azure Blob Storage
       (SQL connection     (Books table)     ("covers" container)
           string)
```

The app holds no connection strings or keys. At startup it uses its managed identity to read the SQL connection string from Key Vault, then connects to Azure SQL and Blob Storage.

## Endpoints

| Method | Path | Description |
| ------ | ---- | ----------- |
| GET | `/api/health` | Checks the database connection |
| GET | `/api/books` | List books (optional `?status=` filter) |
| GET | `/api/books/:id` | Get one book |
| POST | `/api/books` | Add a book (`title` and `author` required) |
| PUT | `/api/books/:id` | Update a book |
| DELETE | `/api/books/:id` | Delete a book |
| POST | `/api/books/:id/cover` | Upload a cover image to Blob Storage (multipart field `file`) |

All SQL queries use parameterised inputs.

## Running locally

```bash
npm install
cp .env.example .env   # then fill in the values below
npm run dev
```

| Variable | Description |
| -------- | ----------- |
| `KEY_VAULT_URL` | e.g. `https://<vault-name>.vault.azure.net/` (must contain a `sql-connection-string` secret) |
| `STORAGE_URL` | e.g. `https://<account>.blob.core.windows.net` |
| `PORT` | Optional, defaults to `3000` |

Locally, `DefaultAzureCredential` picks up your `az login` session.

Database schema: [`schema.sql`](schema.sql).

## Deployment

Every push to `main` runs [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml), which installs dependencies and deploys to Azure App Service using the `AZURE_WEBAPP_PUBLISH_PROFILE` repository secret.

## Related

[booknook-api-prototype](https://github.com/Daveoski/booknook-api-prototype) is the in-memory version of the same API, used before the Azure services were wired in.

## License

[MIT](LICENSE)
