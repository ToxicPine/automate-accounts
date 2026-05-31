---
name: x-agent-browser
description: Use for X/Twitter via agent-browser... operate X, check session, reply, like, repost, bookmark, follow/unfollow, scrape visible users, send DMs, extract audiences, monitor/search, or run bulk workflows. Goal is lowest token burn via eval-first helpers, compact outputs, batching, narrow DOM verification.
---

# X/Twitter via agent-browser

Thesis: lowest-token X automation = use `agent-browser eval` to install/call small JS helper functions, so repeated work avoids large DOM snapshots, element-ref chatter, and navigation loops. Use DOM/navigation once to know session state and verify the first helper result.

## Method

- `references/README.md` routes to narrow platform notes.
- `agent-browser eval` executes those notes inside an authenticated X tab.
- Tiny wrappers/aliases make repeated actions cheap to call.
- Helper inputs define targets/actions; outputs report compact page state + result proof.
- DOM/snapshot calibrates first use and debugs mismatches; eval drives repeated work.

Use `agent-browser` as auth runtime + live verifier. After first proof, stay wrapper-first unless helper output and page state disagree.

## Loop

1. Reuse/open X session.
2. Run one cheap state check: URL + visible account/login/checkpoint state + auth probe.
3. Run/install a tiny eval wrapper on `https://x.com`.
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
- Prefer IDs over prose: `rest_id`, `screen_name`, `tweet_id`, `conversation_id`.
- Omit cookies, `ct0`, auth headers, full responses, raw DOM, and full private message bodies from routine output.

## Calibrate

Determine first:

- signed in vs login screen.
- checkpoint, CAPTCHA, 2FA, locked/restricted account, quota/rate-limit banner.
- correct origin: `x.com` preferred; `twitter.com` may redirect/migrate.
- current route: home, search, profile, tweet, messages.

## Eval First

Eval helpers should:

- run on `https://x.com`
- use existing browser credentials
- derive CSRF from `ct0`
- fetch with `credentials: "include"`
- emit compact task rows
- keep batching behavior explicit in helper inputs

Need endpoint, selector, state, or failure detail? Load `references/README.md`, then one narrow reference.

## DOM Feedback

DOM = verifier, not driver. Check narrow state:

- start session and wrapper install.
- after first wrapper-driven follow/unfollow/like/repost/bookmark/reply/DM.
- when button state, visible reply/message proof, or dialog/toast state is ambiguous.
- when helper result fails or X UI/request shape changes.

After first successful wrapper use, continue eval-first. Inspect only route, visible identity, target state, button state, last proof snippet, and warning/dialog markers.

## References

- `references/request-basics.md`: auth fetch wrapper, headers, current-user/auth probes, failure signals.
- `references/dms-audiences.md`: DM permissions/send, conversation reads, user/tweet/audience extraction.
- `references/dom-social-actions.md`: follow, unfollow, reply, like, repost, bookmark, scraping, keyword/search workflows.
- `references/state-results.md`: queues, pacing, local state, compact outputs, loop control.

If wrapper fails, inspect live UI/request shape narrowly, patch the task helper, retry once, and return the mismatch reason.
