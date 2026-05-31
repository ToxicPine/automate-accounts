---
name: linkedin-agent-browser
description: Use for LinkedIn via agent-browser... operate LinkedIn, message, connect, accept/withdraw invites, inspect profiles, search, check login/session, bulk workflows. Trigger on "LinkedIn" + action. Goal is lowest token burn via eval-first helpers, compact outputs, batching, narrow DOM verification.
---

# LinkedIn via agent-browser

Thesis: lowest-token LinkedIn automation = use `agent-browser eval` to install/call small JS helper functions, so repeated work avoids large DOM snapshots, element-ref chatter, and navigation loops. Use DOM/navigation once to know session state and verify the first helper result.

## Method

- `references/README.md` routes to narrow platform notes.
- `agent-browser eval` executes those notes inside an authenticated LinkedIn tab.
- Tiny wrappers/aliases make repeated actions cheap to call.
- Helper inputs define targets/actions; outputs report compact page state + result proof.
- DOM/snapshot calibrates first use and debugs mismatches; eval drives repeated work.

Use `agent-browser` as auth runtime + live verifier. After first proof, stay wrapper-first unless helper output and page state disagree.

## Loop

1. Reuse/open LinkedIn session.
2. Run one cheap state check: URL + visible login/checkpoint + auth health probe.
3. Run/install a tiny eval wrapper on `https://www.linkedin.com`.
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
- Prefer IDs over prose: `memberId`, `publicIdentifier`, `profileUrn`, `conversationUrn`.
- Omit cookies, CSRF, auth headers, full responses, raw DOM, and full private message bodies from routine output.

## Calibrate

Determine first:

- signed in vs login screen.
- checkpoint, CAPTCHA, 2FA, restriction, quota banner.
- correct origin: `www.linkedin.com`.
- current route: profile, search, messaging, invitations, feed, company.

## Eval First

Eval helpers should:

- run on `https://www.linkedin.com`
- use existing browser session
- derive CSRF from `JSESSIONID`
- fetch same-origin with credentials
- emit compact task rows
- keep batching behavior explicit in helper inputs

Need endpoint, body, header, queryId, state, or failure detail? Load `references/README.md`, then one narrow reference.

## DOM Feedback

DOM = verifier, not driver. Check narrow state:

- start session and wrapper install.
- after first wrapper-driven connect/message/invite accept/withdraw.
- when button state, invite card state, visible message proof, or checkpoint marker is ambiguous.
- when helper result fails or LinkedIn UI/request shape changes.

After first successful wrapper use, continue eval-first. Inspect only route, visible identity, target state, button state, last proof snippet, and warning/checkpoint markers.

## References

- `references/request-basics.md`: CSRF, headers, health, dynamic IDs
- `references/connections-invites.md`: connect/invites/quota
- `references/messaging.md`: messages/conversations
- `references/profiles-search.md`: profiles/contact/search/network reads
- `references/content-audiences.md`: posts/groups/events/company/viewers

If wrapper fails, inspect live UI/request shape narrowly, patch the task helper, retry once, and return the mismatch reason.
