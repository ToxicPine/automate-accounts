# Request Basics

Use for authenticated X API reads/mutations from `agent-browser eval`.

## Same-Origin Fetch

Run helpers on `https://x.com`. Fetch with current browser credentials:

```js
const getCookie = name => document.cookie.split("; ").find(x => x.startsWith(name + "="))?.slice(name.length + 1);
const ct0 = decodeURIComponent(getCookie("ct0") || "");
```

Read X cookies directly in `agent-browser eval`. Use `ct0` for CSRF and `twid` when the current account numeric ID is needed, such as DM conversation IDs.

Core headers for authenticated X web requests:

- `authorization: Bearer AAAAAAAAAAAAAAAAAAAAANRILgAAAAAAnNwIzUejRCOuH5E6I8xnZz4puTs%3D1Zv7ttfk8LF81IUq16cHjhLTvJu4FA33AGWWjCpTnA`
- `content-type: application/json`
- `x-csrf-token: <ct0>`
- `x-twitter-active-user: yes`
- `x-twitter-auth-type: OAuth2Session`
- `x-twitter-client-language: en`
- optional `X-Client-UUID: localStorage.device_id`
- `x-client-transaction-id` generated per path/method when required by the current X request shape

Fetch options:

```js
{
  method,
  mode: "cors",
  credentials: "include",
  referrer: "https://x.com/",
  referrerPolicy: "strict-origin-when-cross-origin",
  headers
}
```

For GET, encode params with `URLSearchParams`. For POST, JSON stringify the body. Return only the fields needed for the task.

Some X web requests require a generated client transaction ID. In a fresh `agent-browser eval`, prefer observing or reusing X's live request shape when transaction IDs are required; otherwise a direct fetch may fail even with valid cookies.

## GraphQL Wrapper

X GraphQL web reads use:

```js
GET https://api.x.com/graphql/<queryId>/<operationName>
  ?variables=<json>&features=<json>
```

Known operation URLs:

- `UserByScreenName`: `https://api.x.com/graphql/hVhfo_TquFTmgL7gYwf91Q/UserByScreenName`
- `TweetDetail`: `https://api.x.com/graphql/NNiD2K-nEYUfXlMwGCocMQ/TweetDetail`
- alternate `TweetDetail`: `https://api.x.com/graphql/NmCeCgkVlsRGS1cAwqtgmw/TweetDetail`
- `Favoriters`: `https://api.x.com/graphql/rUyh8HWk8IXv_fvVKj3QjA/Favoriters`
- `Retweeters`: `https://api.x.com/graphql/0BoJlKAxoNPQUHRftlwZ2w/Retweeters`
- `SearchTimeline`: `https://api.x.com/graphql/WQd073gQrdNHqkTY1qjNuQ/SearchTimeline`

Query IDs drift. If a known query fails, load the target UI once and inspect live requests or page bundles before retrying.

## Auth/Session Probe

Minimal probe:

1. Confirm `location.origin` is `https://x.com`.
2. Confirm cookies include `ct0`; compact outputs usually only need presence.
3. Fetch a cheap read endpoint through the wrapper, such as `UserByScreenName` for the visible account or requested handle.
4. Check DOM text/dialogs for login, migrate, rate-limit, restriction, CAPTCHA, 2FA, checkpoint.

X can redirect through `https://x.com/x/migrate`. Return the current route/state in the auth probe result.

## Response Signals

Encode these compactly when they explain the result:

- HTTP 401/403 from authenticated API probe
- login/checkpoint/CAPTCHA/2FA/locked/restricted account UI
- page/dialog/toast text containing rate-limit or retry-later language
- `errors` array on GraphQL where the requested object is absent
- missing `ct0` or wrong origin

Keep the probe result short and situation-specific.
