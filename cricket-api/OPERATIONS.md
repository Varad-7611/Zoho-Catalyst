# Catalyst operations

## Current deployment

- Catalyst project: `cricket` (Development)
- AppSail service: `cricket-api`
- Health check: `/health`
- Deployment URL: `https://cricket-api-50047246450.development.catalystappsail.in`

AppSail manages traffic distribution among its active instances. It creates
instances as utilization grows and removes excess instances after demand falls.
Catalyst documents a maximum of five instances and 100 concurrent requests per
instance, with a new instance spawned at 80% utilization. Check the AppSail
instance dashboard and request/instance graphs to see scaling and service
health. The existing `/health` endpoint is suitable for the AppSail health
check setting.

## GitHub deployment

The workflow in `.github/workflows/deploy-catalyst.yml` deploys this AppSail
service after changes to `cricket-api/` reach `main`. Add a Catalyst CLI token
as the repository secret `CATALYST_TOKEN` in GitHub Settings → Secrets and
variables → Actions. Do not put the token or API provider credentials in the
workflow file. The workflow performs a `/health` smoke check after deployment.

## Capacity test

Install Locust in a disposable environment, then run a controlled test against
the Development service:

```powershell
python -m pip install locust
locust -f loadtest/locustfile.py --headless --host https://cricket-api-50047246450.development.catalystappsail.in -u 10000 -r 10000 -t 30s
```

This creates 10,000 virtual users and ramps them up at 10,000 users per second,
requesting only `/health`. The output groups responses by the
`X-AppSail-Instance` header so you can see whether traffic reached more than
one instance. This indicates distribution; it does not establish that
Catalyst uses strict round-robin scheduling. The load is intentionally not part of CI. The requested
concurrency is 20 times Catalyst's documented maximum of 500 concurrent requests
across five instances and will likely produce throttling, timeouts, or errors;
coordinate a maintenance window before running it. Start with a smaller user
count and increase gradually while watching AppSail's instance and request
graphs.

## Rate limiting and gateway

API Gateway is disabled in the current Catalyst project. Its throttling cannot
be configured until the Gateway is enabled, and the available Catalyst MCP
tools do not expose an enable operation. The Gateway route schema also does not
offer AppSail as a backend target. A correct shared rate limit for this
multi-instance API therefore needs an explicit shared limiter in the app or a
fronting gateway that can target AppSail; a per-process memory counter would be
multiplied across scaled instances and is not a reliable global limit.
