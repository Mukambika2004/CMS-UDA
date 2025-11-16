# Article CMS (Flask Web Project)

## Overview

Article CMS is a Flask-based content management system. Authenticated users can create and edit posts that include a title, author, body text, and an optional image. Posts are stored in Azure SQL Database and images are uploaded to Azure Blob Storage. Microsoft identity (MSAL) lets users authenticate with Azure Active Directory, while Flask-Login supports local credentials.

## Key Features

- Flask 2.2 web UI with server-side sessions
- SQLAlchemy models backed by Azure SQL Database
- Image uploads to Azure Blob Storage using the SDK v12 client
- Microsoft identity sign-in with MSAL
- Structured logging for authentication events

## Architecture at a Glance

| Component | Purpose |
| --- | --- |
| Flask application (`FlaskWebProject/`) | Routes, forms, templates, logging |
| Azure SQL Database | Persists users and posts |
| Azure Blob Storage | Stores uploaded images |
| Azure App Service | Recommended hosting platform (see below) |

## Prerequisites

- Python 3.10+
- Azure subscription with permissions to create SQL, Storage, and App Service resources
- Azure CLI or access to the Azure Portal
- GitHub account (for optional CI/CD via GitHub Actions)

## Local Setup

1. Clone the repository: `git clone <repo-url>`
2. Create and activate a virtual environment:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Windows: .venv\Scripts\activate
   ```
3. Install dependencies: `pip install -r requirements.txt`
4. Export the configuration values (use `.env` or shell variables). Required keys mirror `Config` in `config.py`:

   | Variable | Description |
   | --- | --- |
   | `SECRET_KEY` | Flask session secret |
   | `BLOB_ACCOUNT` / `BLOB_STORAGE_KEY` / `BLOB_CONTAINER` | Azure Storage credentials |
   | `SQL_SERVER`, `SQL_DATABASE`, `SQL_USER_NAME`, `SQL_PASSWORD` | Azure SQL connection info |
   | `CLIENT_ID`, `CLIENT_SECRET` | Azure AD app registration for MSAL |

5. Initialize the database tables by running the scripts in `sql_scripts/` against the target Azure SQL Database.

## Running the Application Locally

```bash
python application.py
```

The development server listens on `https://localhost:5555/` (self-signed certificate).

### Default Credentials

- Username: `admin`
- Password: `pass`

After MSAL sign-in is configured, the Microsoft login button signs the user in as `admin` automatically.

## Tests

Run the placeholder tests (extend as needed):

```bash
pytest
```

## Deployment to Azure App Service (Recommended)

1. Provision resources:
   - Azure SQL Database + server with firewall rules for App Service
   - Azure Storage account with `images` container
   - Azure App Service (Linux, Python 3.10 runtime)
2. Configure App Service **Settings → Configuration → Application settings** for the variables listed in the Local Setup section.
3. Ensure ODBC Driver 17 for SQL Server is available (built into App Service for Linux) and enable managed identity or connection security as required.
4. Deploy via GitHub Actions using the provided `azure/webapps-deploy@v3` workflow or `az webapp up`. Allow a brief delay between any site restart and the deployment to avoid SCM restarts.
5. After deployment, browse to the App Service URL, sign in, create a new article, and confirm images load from Blob Storage.

## Logging & Monitoring

- Application logs write to standard output and can be viewed with **App Service → Log Stream**.
- Add Azure Monitor alerts for HTTP 5xx or CPU thresholds when scaling beyond a single instance.

## Troubleshooting Tips

- Install Microsoft ODBC Driver 17 locally for development (macOS example):
  ```bash
  brew install unixodbc msodbcsql17
  ```
- If OneDeploy fails due to SCM restarts, wait 30–60 seconds after management operations (restart, configuration changes) before redeploying.
- Confirm Blob Storage credentials by uploading a test file using Azure Storage Explorer when diagnosing image issues.

## License

© 2025 Mukambika. All rights reserved. This project is licensed for Mukambika’s internal use; redistribution or modification outside the organization requires written permission.
