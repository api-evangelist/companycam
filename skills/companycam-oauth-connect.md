---
name: Connect a CompanyCam account via OAuth 2.0
description: Run the authorization-code (PKCE) flow to obtain a partner access token, then call the API.
api: openapi/companycam-openapi-original.yml
operations: [getCurrentUser, getCurrentCompany, listProjects]
---

# Connect a CompanyCam account via OAuth 2.0

Use this for partner integrations that act on behalf of a CompanyCam customer.

## Endpoints (RFC 8414 metadata: https://app.companycam.com/.well-known/oauth-authorization-server)
- Authorize: `https://app.companycam.com/oauth/authorize`
- Token / refresh: `https://app.companycam.com/oauth/token`
- Revoke: `https://app.companycam.com/oauth/revoke`

## Steps
1. Register your app (client_id/client_secret) via the CompanyCam partner form linked from docs/oauth (or dynamic registration at `/oauth/register`).
2. Redirect the user to `/oauth/authorize` with `response_type=code`, `client_id`, `redirect_uri`, `code_challenge` (S256), and a space-delimited `scope` (e.g. `read write` or granular `projects:read photos:write`). See scopes/companycam-scopes.yml.
3. On callback, exchange the `code` at `/oauth/token` (with `code_verifier`) for an access token + refresh token.
4. Call the API with `Authorization: Bearer <access_token>`. Verify the connection with `getCurrentUser` and `getCurrentCompany`.
5. `listProjects` to begin reading account data. Refresh with the refresh token before expiry; revoke at `/oauth/revoke` on disconnect.

## Rules
- Request only the scopes you need (least privilege); `destroy`-class scopes grant deletes.
- Rate limits and error envelope are the same as token auth (see conventions/companycam-conventions.yml).
