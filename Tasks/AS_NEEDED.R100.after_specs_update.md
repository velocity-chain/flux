# R100 – Update files after architecture.yaml changes

**Status**: Shipped 
**Task Type**: Updates
**Run Mode**: Run as needed

## Goal

Update the Developer Edition docker-compose.yaml, and the welcome page index.html with changes after the architecture.yaml file has been updated with new services.

## Context / Input files

These files must be treated as **inputs** and read before implementation:

- `Specifications/architecture.yaml`

**Target files** (to be updated):

- `DeveloperEdition/docker-compose.yaml`
- `index.html` (welcome page)

## Requirements

Update `DeveloperEdition/docker-compose.yaml` and the welcome page `index.html` to match the services defined in `Specifications/architecture.yaml`.

### docker-compose.yaml

- Add or update service definitions for each new domain in the architecture.
- Do not change the existing welcome, runbook, or schema/mongodb services. 
- Do not create services or links for the common_code domain.
- **Remove** the sample profile and any sample_api/sample_spa services (they are legacy placeholders).
- **Welcome service** must be included in ALL profiles (every profile in the file). When adding new domains, add their profiles (e.g. `{domain}`, `{domain}-api`) to the welcome service profiles list so the welcome page always starts with any profile.
- **Use ports from architecture.yaml exactly** – The template merge process configures APIs to listen on the port specified in the architecture. Docker-compose ports and API_PORT env must match (e.g. profile_api: 9096, profile_spa: 9097).
- **For each new microservice domain, define two profiles:**
  - `{domain}-api` – API service only (e.g. `profile-api` → profile_api)
  - `{domain}` – API + SPA (e.g. `profile` → profile_api + profile_spa)
- **Add API_PORT** to each API service environment so the app binds correctly.
- **Add IDP_LOGIN_URI** to each SPA with its own port (e.g. `http://localhost:9097/login` for profile_spa) for dev-login flow.
- Ensure backing services (e.g. mongodb) are included in the profiles of any new services.
- Ensure all new services are included in the all profile.

### index.html

- Add links for each service SPA (with correct ports from the architecture).
- Add new domains to the top of the list
- Add an API Explorer link for each backing API at `/docs/explorer.html`.
- Do not create services or links for the common_code domain.
- There is no need to adjust the schema or runbook links

## Testing expectations

- **None**

## Packaging / build checks

Before marking this task as completed:
- Run ``make container`` and ensure that the container builds cleanly.

## Dependencies / Ordering

- Should run **after**:
  - **None**
- Should run **before**:
  - **None**

## Implementation notes (to be updated by the agent)

**Summary of changes**

- **DeveloperEdition/docker-compose.yaml**
  - Removed legacy `sample`, `sample_api`, and `sample_spa` services and all references to the sample profile.
  - Added services for domains from `Specifications/architecture.yaml`: operations (8389/8390), planning (8391/8392), suppliers (8393/8394), inventory (8395/8396), analytics (8397/8398), integrations (8399/8400). Each domain has `{domain}_api` and `{domain}_spa` with ports and profiles `{domain}-api` and `{domain}`.
  - Welcome service profiles updated to include all new profiles (operations-api, operations, planning-api, planning, suppliers-api, suppliers, inventory-api, inventory, analytics-api, analytics, integrations-api, integrations) so the welcome page starts with any profile.
  - MongoDB, mongodb_api, and mongodb_spa profiles updated to include all new domain profiles; sample/sample_api removed.
  - Each new API service has `API_PORT` and Mongo env; each new SPA has `IDP_LOGIN_URI` (e.g. `http://localhost:8390/login` for operations_spa). All new services are in the `all` profile.
  - common_code domain not added (per task).
- **index.html**
  - Replaced placeholder “Add new Services Here” with links for each new SPA (Integrations 8400, Analytics 8398, Inventory 8396, Suppliers 8394, Planning 8392, Operations 8390), with new domains at the top of the list.
  - Added an API Explorer link for each backing API at `/docs/explorer.html` (8399, 8397, 8395, 8393, 8391, 8389). Normalized Schema Configuration API Explorer to `/docs/explorer.html`.
  - Script updated to set dynamic hostname for all new link IDs. Schema and runbook links unchanged except database explorer path.
- **Packaging**: `make container` completed successfully.