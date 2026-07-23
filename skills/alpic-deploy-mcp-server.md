---
name: Deploy and manage an MCP server on Alpic
description: Create a project, configure an environment, deploy it, and watch logs on the Alpic MCP-native cloud, using the Alpic REST API.
api: openapi/alpic-openapi-original.json
method: generated
generated: '2026-07-17'
operations:
  - teams.list.v1
  - projects.create.v1
  - environments.create.v1
  - environmentVariables.create.v1
  - environments.deploy.v1
  - environments.getLatestLogs.v1
  - beacon.create.v1
  - distribution.publish.v1
---

# Deploy and manage an MCP server on Alpic

Operating instructions for an agent using the Alpic REST API (`https://api.alpic.ai`, all paths under `/v1/`).

## Authentication
Send `Authorization: Bearer <token>` on every request. The token is either an API key
(created in team settings) or an OAuth access token. If you are an agent without a human,
self-register for an API key first — see `skills/alpic-agent-auth.md`
(POST `https://api.alpic.ai/agent/auth` with an identity assertion / ID-JAG).

## Steps
1. **Find your team.** `GET /v1/teams` (`teams.list.v1`) and pick a `teamId`.
2. **Create a project.** `POST /v1/projects` (`projects.create.v1`) with the `teamId` and a name. A project is one MCP server.
3. **Create an environment.** `POST /v1/environments` (`environments.create.v1`) with `projectId`, `name`, `sourceBranch`, and optional `environmentVariables[]` (each `key` must match `^[a-zA-Z]([a-zA-Z0-9_])+$`).
4. **Add secrets (optional).** `POST /v1/environments/{environmentId}/environment-variables` (`environmentVariables.create.v1`), marking sensitive values secret.
5. **Deploy.** `POST /v1/environments/{environmentId}/deploy` (`environments.deploy.v1`). The response/environment carries the live MCP server `urls`.
6. **Watch logs.** Poll `GET /v1/environments/{environmentId}/latest-logs` (`environments.getLatestLogs.v1`) — pagination is cursor-based (`limit`, `nextToken`).
7. **Audit compliance (optional).** `POST /v1/beacon/audits` (`beacon.create.v1`) then poll `GET /v1/beacon/audits/{auditId}` (`beacon.get.v1`) to check MCP spec compliance and AI-client compatibility.
8. **Distribute (optional).** `POST /v1/distribution/publish` (`distribution.publish.v1`) to publish the server to the MCP registry.

## Conventions & error handling
- **Pagination:** list/log endpoints take `limit` + an opaque `nextToken`; follow `nextToken` to page.
- **Idempotency:** none advertised — do not blindly retry non-idempotent writes (create/deploy).
- **Errors:** JSON envelope `{ defined, code, status, message, data }`. Handle `BAD_REQUEST` (400, fix the body), `FORBIDDEN` (403, permission/plan — note tunnel tickets need user auth, not API keys), `NOT_FOUND` (404, bad id), `DNS_RESOLUTION_FAILED` (400, custom-domain DNS). See `errors/alpic-problem-types.yml`.
