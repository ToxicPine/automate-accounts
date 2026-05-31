---
name: reddit-agent-browser
description: Use for Reddit via agent-browser... operate Reddit, reply, comment, DM, inspect users/posts/subreddits, discover targets, check login/session, bulk workflows. Trigger on "Reddit" + action. Goal is lowest token burn via eval-first helpers, compact outputs, batching, narrow DOM verification.
---

# Reddit via agent-browser

Thesis: lowest-token Reddit automation = use `agent-browser eval` to install/call small JS helper functions, so repeated work avoids large DOM snapshots, element-ref chatter, and navigation loops. Use DOM/navigation once to know session state and verify the first helper result.

## Method

- `references/README.md` routes to narrow platform notes.
- `agent-browser eval` executes those notes inside an authenticated Reddit tab.
- Tiny wrappers/aliases make repeated actions cheap to call.
- Helper inputs define targets/actions; outputs report compact page state + result proof.
- DOM/snapshot calibrates first use and debugs mismatches; eval drives repeated work.

Use `agent-browser` as auth runtime + live verifier. After first proof, stay wrapper-first unless helper output and page state disagree.

## Loop

1. Reuse/open Reddit session.
2. Run one cheap state check: URL + visible page state + account/username probe.
3. Run/install a tiny eval wrapper on `https://www.reddit.com` or `https://chat.reddit.com`.
4. If detail is needed, load `references/README.md`, then one narrow reference.
5. Run the first requested operation through the wrapper.
6. Verify first state-changing or selector-brittle operation by narrow DOM/snapshot.
7. Continue wrapper-first unless helper result and page state disagree.

## Token Law

- Alias repeated `agent-browser` cmds.
- Save helper files; avoid pasting large scripts repeatedly.
- Do not read large volumes of data. Ask helpers for narrow fields, small limits, and task-shaped slices.
- Return terse rows: `target`, `action`, `status`, `id`, `reason`, maybe one `ui` proof.
- Batch targets in one eval when it reduces round trips.
- Prefer IDs over prose: `username`, `postId`, `commentId`, `subreddit`, `threadUrl`.
- Omit cookies, local tokens, full responses, raw DOM, and full private message bodies from routine output.

## Calibrate

Determine first:

- signed in vs login screen.
- challenge, warning, empty/broken page, duplicate/contact state.
- correct origin: `www.reddit.com` for pages, `chat.reddit.com` for chat.
- current route: post, comment, subreddit, profile, chat.
- target state: author deleted/suspended, post removed, conversation already present.

## Eval First

Eval helpers should:

- run on Reddit origin
- use existing browser session
- use DOM and same-origin JSON reads only for the requested action
- traverse open shadow roots when selectors are inside Reddit web components
- emit compact task rows
- keep any batching behavior explicit in the helper inputs

Need selector, sequencing, state, or failure detail? Load `references/README.md`, then one narrow reference.

## DOM Feedback

DOM = verifier, not driver. Check narrow state:

- start session and wrapper install.
- after first wrapper-driven public reply/comment or DM/chat send.
- when duplicate state, target state, composer state, or button state is ambiguous.
- when helper result fails or Reddit UI/request shape changes.

After first successful wrapper use, continue eval-first. Inspect only route, visible identity, target state, composer state, button state, last proof snippet, and warning/banner markers.

## References

- `references/session-state.md`: auth/page detection, duplicate and state signals
- `references/replies-comments.md`: public reply/comment composers, insertion, submit, verification
- `references/direct-messages-chat.md`: profile/chat navigation, DM compose/send, verification
- `references/queues-targeting-state.md`: batching, delays, target extraction, duplicate/contact state

If wrapper fails, inspect live UI/request shape narrowly, patch the task helper, retry once, and return the mismatch reason.
