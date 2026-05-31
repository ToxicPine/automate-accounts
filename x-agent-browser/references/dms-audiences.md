# DMs And Audiences

Use for direct messages, conversation checks, and extracting compact people/tweet results from user or tweet audiences.

## Resolve User

Use `UserByScreenName` before DM/follow/user-audience work.

```http
GET https://api.x.com/graphql/hVhfo_TquFTmgL7gYwf91Q/UserByScreenName
```

Variables:

```json
{"screen_name":"<handle>","withSafetyModeUserFields":true,"withSuperFollowsUserFields":true}
```

Features:

```json
{
  "responsive_web_twitter_blue_verified_badge_is_enabled": true,
  "verified_phone_label_enabled": true,
  "responsive_web_graphql_timeline_navigation_enabled": true
}
```

Success: `data.user.result.__typename === "User"`. Extract only the identity, count, and permission fields useful for the task.

If user missing or typename is not `User`, treat the lookup as not found, suspended, or otherwise unavailable.

## Tweet Detail

Use to resolve tweet metadata and conversation roots.

```http
GET https://api.x.com/graphql/NNiD2K-nEYUfXlMwGCocMQ/TweetDetail
```

Variables include tweet ID and cursor/context fields. Parse `threaded_conversation_with_injections_v2.instructions`, find `TimelineAddEntries`, then entries whose `entryId` starts with `tweet-` or `conversationthread-`.

Return only the tweet metadata useful for the task.

## Tweet Audiences

Favoriters:

```http
GET https://api.x.com/graphql/rUyh8HWk8IXv_fvVKj3QjA/Favoriters
```

Retweeters:

```http
GET https://api.x.com/graphql/0BoJlKAxoNPQUHRftlwZ2w/Retweeters
```

Replies and quotes can be extracted through tweet-detail/search style timelines. For quotes, use a search query pattern like `quoted_tweet_id:<tweetId>`.

Keep audience results compact and include enough context to tell which audience type produced each person.

## Search Timeline

```http
GET https://api.x.com/graphql/WQd073gQrdNHqkTY1qjNuQ/SearchTimeline
```

Use X search query syntax and compact timeline parsing. Live search routes are shaped like:

```text
https://x.com/search?q=<encoded query>&src=typed_query&f=live
```

For live automation, prefer navigating to the search route once, verify visible results, then batch DOM/API extraction.

## Followers And Following

Follower/following reads use fixed operation IDs plus the generic GraphQL URL shape:

```http
GET https://api.x.com/graphql/nfEPWtWXVmCLHEtR366o6g/Followers
GET https://api.x.com/graphql/qvw4wDWT9-EgYnnCZWJAqg/Following
```

Variables:

```json
{"userId":"<rest_id>","count":20,"includePromotedContent":false,"cursor":"<optional>"}
```

Return compact people results only. Verify current query IDs from live requests if these fail.

## DM Permission

Before sending, check:

```http
GET https://x.com/i/api/1.1/dm/permissions.json
```

Params:

```json
{"recipient_ids":"<rest_id>","dm_users":true}
```

Success when `permissions.id_keys[rest_id].can_dm` is boolean true.

Useful compact categories:

- permission check returned false
- verified-account requirement, including `SenderIsNotVerifiedForMessageRequests`
- account already replied in prior history
- sent-message threshold or history rule matched
- duplicate or history exclusion

## Send DM

Send through:

```http
POST https://api.x.com/graphql/MaxK2PKX1F9Z-9SwqwavTw/useSendMessageMutation
```

Body:

```json
{
  "variables": {
    "message": {"text": {"text": "<message>"}},
    "requestId": "<uuid-like request id>",
    "target": {"conversation_id": "<sender_twid>-<recipient_rest_id>"}
  },
  "queryId": "MaxK2PKX1F9Z-9SwqwavTw"
}
```

For media DMs, `message` becomes `{"media":{"id":"<media_id>","text":"<message>"}}`.

Result typenames:

- `CreateDmSuccess`
- `CreateDmFailed`

On `CreateDmFailed`, include the `dm_validation_failure_type` in the compact explanation. The send sequence starts with `<sender_twid>-<recipient_rest_id>`, then retries once with reversed order if the result is neither success nor explicit failure. Additional retry behavior depends on the calling task.

Sequence:

1. Resolve recipient via `UserByScreenName`.
2. Check `dm/permissions.json`.
3. Check local duplicate/history rules if doing bulk sends.
4. Send DM mutation.
5. Optionally verify with conversation read or visible Messages UI.

## Conversation Read

Read conversations with:

```http
GET https://x.com/i/api/1.1/dm/conversation/<conversation_id>.json
```

Params include:

```json
{
  "context": "FETCH_DM_CONVERSATION",
  "include_profile_interstitial_type": 1,
  "include_blocking": 1,
  "include_blocked_by": 1,
  "include_followed_by": 1,
  "include_want_retweets": 1,
  "include_can_dm": 1,
  "include_conversation_info": true,
  "dm_users": false,
  "include_inbox_timelines": true
}
```

Conversation read path converts both IDs to strings and orders them by string comparison as `<lower_string>-<higher_string>`. Keep send and read ID construction separate: send starts sender-first with one reversed retry; read uses the sorted string pair.

## Media Upload

Upload media through:

```http
POST/GET https://upload.x.com/i/media/upload.json
```

Commands: `INIT`, `APPEND`, `FINALIZE`, `STATUS`. Media categories are shaped like `dm_image`, `dm_gif`, or tweet equivalents. Use only when the user requested media; verify upload completion before sending.

## Bulk DM Rules

For bulk work:

- keep per-target output compact and appropriate to the task
- dedupe by `rest_id` and/or handle before send
- persist a sent/history set outside the prompt when running long jobs
- surface verified-required, restriction, rate-limit, and mutation-drift reasons when they explain a result
- include raw DM text in output only when it is useful for the requested task
