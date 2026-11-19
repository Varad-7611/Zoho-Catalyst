# Live Cricket Score

A cricket scores website with a static HTML, CSS, and JavaScript frontend and a Python Flask API. The API reads match and scorecard data from Cricbuzz through RapidAPI.

## Live deployment

- **Frontend (Catalyst Slate):** https://cricket-frontend-lmzgzuds.onslate.in
- **Backend (Catalyst AppSail):** https://cricket-api-50047246450.development.catalystappsail.in
- **Health check:** https://cricket-api-50047246450.development.catalystappsail.in/health
- **Catalyst project:** `cricket`, Development environment
- **AppSail service:** `cricket-api` (Python 3.12)

The `/health` endpoint returns a small JSON response when the API is running. The API also exposes routes for live, recent, and upcoming matches, commentary, match details, and scorecards. See `cricket-api/app.py` for the route list.

## Project layout

```text
cricket-api/
  app.py                 Flask API
  requirements.txt       Python packages
  static/                Static assets packaged with the backend
  templates/             Flask templates packaged with the backend
  OPERATIONS.md          Deployment and operations notes
.github/workflows/
  deploy-catalyst.yml    GitHub Actions deployment workflow
```

The frontend is hosted separately on Slate. The deployed AppSail service needs the RapidAPI credentials as runtime environment variables named `RAPIDAPI_KEY` and `RAPIDAPI_HOST`.

## Automatic deployment from GitHub

The workflow at `.github/workflows/deploy-catalyst.yml` is configured to deploy the AppSail backend when changes under `cricket-api/` (or the workflow itself) are pushed to `main`. It can also be started manually from the repository's **Actions** tab using **Run workflow**. After deployment, it requests `/health` and fails the job if the endpoint does not return success.

To connect the repository to Catalyst:

1. Add a Catalyst CLI authentication token to GitHub as the Actions secret `CATALYST_TOKEN` (**Settings → Secrets and variables → Actions**). Never put the token in this README or in source code.
2. Keep `RAPIDAPI_KEY` and `RAPIDAPI_HOST` configured securely in the AppSail runtime environment in Catalyst. The deployment package must not contain `.env` or a secret-filled `app-config.json`.
3. Push the backend change to `main`, or run the workflow manually. Check the GitHub Actions job output and then open `/health` above.

The GitHub workflow deploys to Catalyst **Development**. To deploy to Production, first create a separate reviewed workflow/environment with the correct Catalyst project and secret configuration; do not simply change the existing Development IDs without checking them.

## Catalyst MCP connection

Catalyst MCP is the assistant's authenticated connection to Catalyst management tools. It lets Codex inspect Catalyst projects and, where supported, read service information and logs. It is separate from the GitHub Actions deployment: **Actions deploys through the Catalyst CLI and `CATALYST_TOKEN`; it does not deploy through MCP.**

Authorize the Catalyst by Zoho MCP server in Codex using the correct Zoho data center for the Catalyst account. Once authorized, the assistant can use the available `CatalystbyZoho_*` tools for supported read or management operations. MCP authorization does not replace GitHub repository secrets and should not be copied into the repository.

## Monitoring

For this Python AppSail backend, use **Catalyst Console → project `cricket` → AppSail → `cricket-api` → Instances**. The AppSail instance view provides request and instance analytics, execution history, and links to logs. The `/health` endpoint is suitable for a basic availability check and is also used by the GitHub deployment workflow after a deploy.

Catalyst's APM feature is for supported Catalyst Functions (Java and Node.js); it does not provide function APM traces for this Python AppSail service. Use AppSail analytics and Catalyst Logs for this backend. The Slate frontend is hosted separately, so check its deployment status in the Slate section of the Catalyst Console.

## Local development

Use Python 3.12. From the repository root in PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r cricket-api/requirements.txt
python cricket-api/app.py
```

Before starting the app, create a local `.env` containing `RAPIDAPI_KEY=your_key` and `RAPIDAPI_HOST=cricbuzz-cricket.p.rapidapi.com`. Replace `your_key` with your own RapidAPI key. Do not commit `.env` or put a real key in an example file.

## Files to commit

Commit the source, dependency list, frontend assets, deployment workflow, and documentation. Do not commit `.env`, `cricket-api/app-config.json` when it contains credentials, API tokens, virtual environments, Python caches, or generated dependency bundles. Review `git status` before committing so intended source files are not accidentally omitted or deleted.
