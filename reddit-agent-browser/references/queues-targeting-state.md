# Queues, Targeting, And State

Use for multi-target Reddit work and discovery. Keep batching parameters and state explicit.

## Compact Output

Return one compact, task-appropriate item per target, plus a tiny summary when useful.

Avoid raw DOM, raw API bodies, local tokens, full message text, or full queue snapshots.

## Target Extraction

Useful selectors for usernames on Reddit pages:

- links matching `/user/` or `/u/`
- `[data-testid*="author"]`
- `.author`
- `shreddit-comment-author-display-name`
- `shreddit-post-author-display-name`
- author links `[data-testid="post-author-link"]`, `[data-click-id="user"]`, `a.author-name[aria-label^="Author:"]`

Username extraction:

- href regex `/\/u(?:ser)?\/([^\/\?#]+)/`
- text fallback should validate `^[a-zA-Z0-9_-]{3,20}$`

Return state markers for moderators, promoted/advertiser nodes, `[deleted]`, `[removed]`, and usernames on an explicit ignore list.

Thread/contact proof:

- post id from `/comments/([a-z0-9]+)/`
- comment id from comment permalink path when present
- contacted-thread state can be keyed by post id

## Duplicate And Contact State

Before state-changing work, check:

- duplicate reply: logged-in username already appears among comment authors.
- existing DM: `rs-timeline` appears after Start Chat.
- already contacted: current task/local state has username with status other than `found`.
- already processed thread: post id appears in contacted-post state.

Useful local scratch-state model:

- indexes: `redditUsername`, `campaign`, `status`, `firstContactDate`
- action fields: `status`, `discoveryMethod`, `firstContactDate`, `lastUserActionDate`, `lastLeadResponseDate`, `hasResponded`, `hasUnreadResponse`, `introMessage`, `redditPostId`, `redditPost`, `subreddit`, `sentByRedditAccount`
- status examples: `found`, `dm_sent`, `unreachable`, `interested`, `meeting_booked`, `meeting_completed`, `won`, `not_interested`, `lost`

Use this as a practical scratch-state shape when the task needs dedupe.

## Batching

Timing knobs, if the calling task wants them:

- 20-32s between leads.
- 10-15s between reply and DM for the same lead.
- random 2-10s before sending chat messages.
- track consecutive non-confirmed failures.

State machine ideas worth copying:

- states: `idle`, `fetchingLeads`, `processingLead`, `leadCompleted`, `leadError`, `leadSkipped`, `waitingBetweenLeads`, `paused`, `stopped`, `completed`
- queues: `_leadsQueue`, `_successLeadsQueue`, `_errorLeadsQueue`, `_skippedLeadsQueue`
- counters: `_currentLeadIndex`, `_consecutiveErrors`
- wait state: `_waitingUntil`, `_currentDelayMs`

For an agent-browser helper, keep internal state local and return only compact results plus a tiny summary when useful.

## Failure And Resume

Useful failure/recovery signals:

- waiting state over 90s -> mark current target failed and report timeout
- worker/browser tab closed -> report `worker_tab_closed`
- page URL mismatch after navigation -> `navigation_mismatch`
- daily/campaign limit reached -> `limit_reached`
- quota/throttle text -> `quota_or_throttle`
- missing composer/send/submit after one retry -> return the visible reason

Useful result reasons:

- `CAMPAIGN_LIMIT_REACHED`
- `DAILY_LIMIT_REACHED`
- `NO_MORE_USERS`
- `Send already in progress`
- `Paused after 3 consecutive errors`

## Same-Origin JSON Reads

For verification only, Reddit public JSON can help confirm comments:

- `https://www.reddit.com{postPath}.json?sort=new&limit=500`
- `https://www.reddit.com/user/{username}/comments.json?sort=new&limit=25`

Use credentials from the current browser session, return only compact confirmation details, and avoid dumping response bodies.
