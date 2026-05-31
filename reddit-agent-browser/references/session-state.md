# Session And Page State

Use before any Reddit state-changing helper.

## State Probe

Run on `https://www.reddit.com` for posts/profiles/subreddits and on `https://chat.reddit.com` for chat. Return only compact state relevant to the task.

## Auth/User Selectors

Useful logged-in user probes:

- New Reddit: `#user-drawer-content a[href*="/user/"]`, `[data-testid="user-drawer-username"]`, `faceplate-tracker[noun="profile"] a`, `header a[href*="/user/"]`, `#USER_DROPDOWN_ID span`, `meta[name="reddit.username"]`.
- Web component user hints: `rs-current-user[display-name]`, `community-author-flair[username]`, `after-login-toast-dispatcher[username]`, `achievements-entrypoint[username]`.
- Old Reddit: `#header-bottom-right .user a`.
- Logged-out hint: visible button text matching `log in` or `sign in`.

Treat missing user plus login UI as unauthenticated and return that state.

## Page Classification

Useful URL/DOM checks:

- Post/thread: path contains `/comments/{postId}`.
- Comment permalink: `/r/.../comments/.../.../{commentId}`.
- Subreddit: `/r/{subreddit}` plus hot/new/rising/top/posts/comments variants.
- Profile: `/user/{name}` or `/u/{name}`.
- Chat: hostname `chat.reddit.com`.

Useful post title selectors: `h1[id*="post-title"]`, `h1[slot="title"]`, `[data-testid="post-content"] h1`, `shreddit-title h1`.

## Target-State Signals

Return target state compactly:

- `author_deleted`: `shreddit-post[author]` is `[deleted]` or `[removed]`; deleted/removed author link text.
- `account_suspended`: page text includes `this account has been suspended`, `account suspended`, `user suspended`, `this account has been banned`, `banned`, or `sorry, nobody on reddit goes by that name`; selectors include `[data-testid="banned-profile"]`, `[data-testid="suspended-profile"]`, `[data-testid="suspended-user"]`, `.suspended-user`, `.banned-user`, `[data-user-status="suspended"]`.
- `post_removed`: selectors/text include `[slot="post-removed-banner"]`, `[data-testid*="removed"]`, `[data-testid*="deleted"]`, `[data-testid="post-removed-banner"]`.
- `existing_conversation`: chat timeline `rs-timeline` appears after Start Chat.
- `duplicate_reply`: current logged-in username is already present among comment author selectors.
- `already_contacted`: local task state says the username has status other than `found`.

Useful comment-author selectors for duplicate checks:

- `shreddit-comment[author]`
- `shreddit-comment a[href*="/user/"]`
- `a[data-testid="comment_author_link"]`
- `[data-testid="comment_author_icon"] + a`
- `.comment a.author`
- `faceplate-tracker[noun="comment_author"] a`
- `[id^="t1_"] a[href*="/user/"]`

## Page-State Signals

Return visible state for the calling task to interpret:

- challenge/login wall
- account, quota, rate, or throttle language
- missing send/submit composer after one retry
- worker tab closed or page navigation mismatch

Use visible page text and DOM snapshot when the page state is not covered by selectors above.
