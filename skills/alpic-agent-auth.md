# Alpic auth.md

Alpic supports agentic registration. If you are an AI agent acting on behalf of a user, you can obtain an Alpic API key without a human filling in a sign-up form, then use it to deploy and manage MCP servers (e.g. `alpic deploy`).

- Resource server (API): https://api.alpic.ai
- Authorization server (agent auth): https://api.alpic.ai

## 1. Discover

### Protected Resource Metadata

Fetch `GET https://api.alpic.ai/.well-known/oauth-protected-resource`

### Authorization Server Metadata

Alpic uses multiple valid authorization servers, all returned on the previous endpoint response.
You should use one that advertize the `agent_auth` block.

Fetch `GET https://api.alpic.ai/.well-known/oauth-authorization-server`

The `agent_auth` block tells you:
- `register_uri` — where to POST registration requests
- `identity_types_supported` — `["identity_assertion"]`
- `credential_types_supported` — `["api_key"]`

## 2. Pick a method

Alpic currently supports one registration shape: **identity assertion with an ID-JAG** (`identity_assertion` + `urn:ietf:params:oauth:token-type:id-jag`), issuing an `api_key`. Anonymous and email-verification flows are not yet available. Cross-check against the `agent_auth.identity_assertion` block before sending.

## 3. Register

Obtain an audience-specific ID-JAG from your provider with `aud = https://api.alpic.ai`, then:

```http
POST /agent/auth HTTP/1.1
Host: api.alpic.ai
Content-Type: application/json
```

```json
{
  "type": "identity_assertion",
  "assertion_type": "urn:ietf:params:oauth:token-type:id-jag",
  "assertion": "eyJhbGc...",
  "requested_credential_type": "api_key"
}
```

Success returns a non-expiring API key:

``json
{
  "registration_id": "reg_...",
  "registration_type": "agent-provider",
  "credential_type": "api_key",
  "credential": "...",
  "credential_expires": null,
  "scopes": ["api.read", "api.write"]
}
```

If the asserted email does not yet match an Alpic user, a new team is created and a claim invitation is emailed to the user so a human can take ownership later. The API key is usable immediately.

An email is sent to the verified email asserted in the ID-JAG token in order for a human to claim the newly created team.

## 4. Use the credential

Send the API key as `Authorization: Bearer ...` to the API, or export it as `ALPIC_API_KEY` for the CLI (`alpic deploy --non-interactive`). On a 401 from a previously working key, drop it and restart at Step 1.

## 5. Errors

| Code | Meaning |
| --- | --- |
| `invalid_issuer` | `iss` is not a trusted provider. |
| `invalid_signature` | Signature did not verify against the provider JWKS. |
| `expired` | `exp` is in the past. |
| `invalid_audience` | `aud` is not ${audience}. |
| `invalid_client_id` | `client_id` does not resolve to the trusted provider. |
| `missing_verified_email` | No verified email on the assertion. |
| `unsupported_credential_type` | Requested credential type is not offered. |

Register endpoint: https://api.alpic.ai/agent/auth

Contact: contact@alpic.ai