# Request Basics

Use before writing LinkedIn eval helpers.

## Same-Origin Fetch

Eval helper should run from `https://www.linkedin.com` and use current browser session.

Common rules:

- `credentials: "include"`
- CSRF = `JSESSIONID` cookie, URL-decoded, wrapping quotes stripped
- `csrf-token: <csrf>` on authenticated calls
- `x-restli-protocol-version: 2.0.0` on most Voyager/Rest.li calls, but not every endpoint
- default read accept: `application/vnd.linkedin.normalized+json+2.1`
- messaging GraphQL accept: `application/graphql`
- message creation accept: `application/json`
- RSC action/list calls often use `accept: */*`, JSON body, and `x-li-rsc-stream: true` for streaming responses
- `x-li-track` usually present on generic LinkedIn request wrappers; refresh client version from live page when possible
- never print cookies, CSRF, full raw payloads, or secrets

`x-cki: cmoa` may appear in captured patterns as an internal recorder marker. Do not send it unless specifically recreating recorder behavior.

## Compact Helper Output

Return smallest useful proof:

```json
{"ok":true,"target":"id","action":"connect","status":201,"ui":"pending"}
{"ok":false,"target":"id","action":"message","status":403,"reason":"not_allowed"}
```

Good fields:

- `ok`
- `target`
- `action`
- `status`
- `reason`
- one result field: `ui`, `memberId`, `conversationUrn`, `messageId`, `invitationId`

Avoid raw response bodies except short error snippets while debugging.

## Health Check

Use this before risky actions.

```http
GET /voyager/api/relationships/connectionsSummary
```

Header:

```http
x-restli-protocol-version: 2.0.0
```

Success:

- HTTP 2xx
- response includes connection summary/count
- DOM has no login/checkpoint UI

Failure:

- missing `JSESSIONID`: wrong origin or unauthenticated
- 401/403/redirect/checkpoint UI: stop authenticated automation

## Dynamic GraphQL IDs

GraphQL query IDs drift.

Use order:

1. Prefer live-observed query IDs from current browser session.
2. Use bundled fallback seeds only to bootstrap.
3. If parser/query shape fails, navigate once to relevant LinkedIn UI and observe current request.
4. Do not retry stale constants repeatedly.

For helpers, store dynamic IDs in one local config object so a failed query can be swapped without rewriting every call.
