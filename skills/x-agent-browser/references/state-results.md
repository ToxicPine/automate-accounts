# State And Results

Use for bulk workflows, pacing, local queues, compact outputs, and loop control.

## State Model

Keep long-running state out of the prompt. Store it in an eval helper global or local file, and return compact snapshots.

Useful state concepts:

- running/stopped state
- processed/skipped counts
- target cursor or current handle
- started/updated timestamps
- retry or failure context

Status entries should stay short and should live in memory or a helper file when they help long-running work.

## Compact Outputs

Shape helper outputs for the current task. Include the smallest useful identifiers, action/result status, and proof or failure context needed to decide what happened. Keep result payloads narrow, and include short error snippets only when they explain failures.

## Pacing

Useful timing ranges for state-changing loops:

- follow/unfollow: 4-10s between successful actions
- follow from list: 6-15s
- reply: 12-25s
- repost: 8-20s
- like/bookmark: 3-8s

Use the user's task to choose limits, delays, retries, and exit behavior. Keep those choices visible in the helper config so a future eval call can adjust them without rewriting the workflow.

## Bulk Flow

1. Calibrate session and route.
2. Install helper and dry-run target resolution.
3. Process one target.
4. Verify DOM result when useful.
5. Continue batch with compact results.
6. Periodically sample route/UI/API signals.
7. Return a compact summary with the counts and exit context useful for the task.

## Signals

Useful route/UI/API signals to encode when relevant:

- login screen
- checkpoint
- CAPTCHA
- 2FA prompt
- locked/restricted/suspended warning
- "Sorry, you are rate limited" or "Rate limit exceeded"
- "try again later" action dialogs
- missing `ct0`
- auth probe failure
- HTTP 401/403 on session probe

## Duplicate And History Rules

For DMs and repeated engagement:

- dedupe by `rest_id` when available
- dedupe by lowercase handle when IDs are not available
- dedupe tweets by `/status/<id>` link
- keep a sent/action history for bulk jobs
- mark target results as completed, skipped, or retryable when useful

Duplicate/history messages and DM validation failures should be explained compactly when useful.

## Verification

First state-changing action proof options:

- follow: button is following/requested, or follow button no longer visible
- unfollow: confirmation closed and following button gone
- like: like state changed or unlike available
- repost: unretweet available or confirmation closed with count/state changed
- bookmark: remove/bookmarked state or saved toast
- reply: composer closed and tweet/send proof visible
- DM: mutation success plus conversation read or visible latest message snippet if needed

After first proof, continue script-first while route/API/DOM signals stay consistent.
