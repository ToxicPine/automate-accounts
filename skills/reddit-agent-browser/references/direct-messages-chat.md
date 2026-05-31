# Direct Messages And Chat

Use for Reddit DM/chat workflows in the user's authenticated browser session.

## Compact Flow

1. Navigate to `https://www.reddit.com/user/{username}/`.
2. Run `session-state.md` checks: auth, target profile exists, not suspended/deleted.
3. Click Start Chat or follow its chat link.
4. If an existing conversation timeline appears, return that state.
5. Insert message into chat composer.
6. Send once.
7. Verify narrowly.
8. Return compact, task-appropriate result details.

## Profile And Start Chat

Profile URL forms:

- `https://www.reddit.com/user/{username}/`
- `/u/{username}` and `/user/{username}` links when extracting from pages.

Useful profile loaded selectors:

- `.profile-page`
- `[data-testid="user-profile"]`
- `[data-testid="profile-page"]`
- `.ProfileHeader`

Start Chat selectors:

- `a[data-testid="private-chat-button"]`
- `reddit-chat-anchor a`
- `[data-testid="user-hover-card"] a[data-testid="private-chat-button"]`
- `faceplate-hovercard div[slot="content"] a[data-testid="private-chat-button"]`
- `a[href*="chat.reddit.com"]`
- fallback deep search over `button,a` and shadow roots for text `Start Chat`, `Start chat`, or `Chat`

## Chat Frame Handling

Chat may run on `chat.reddit.com` or in a frame reached from the profile. If direct page eval cannot reach the composer:

1. Try sending the action to the active tab/frame first.
2. Enumerate frames and target the one whose URL contains `chat.reddit.com`.
3. Retry composer detection in that frame.

Frame retry window: up to 20 attempts with about 2s between attempts.

## Existing Conversation

After Start Chat, wait about `5s` and check for `rs-timeline`. If present, return `existing_conversation` so the calling task can decide the next action.

## Composer And Insert

Composer selectors:

- `rs-message-composer` and its open shadow root
- `textarea[name="message"]`
- `textarea[placeholder="Message"]`
- generic `textarea`

Insertion sequence:

1. Click and focus textarea.
2. Select existing content.
3. Try `document.execCommand("insertText", false, text)`.
4. Fallback to native `HTMLTextAreaElement.prototype.value` setter.
5. Dispatch `InputEvent("input", {inputType:"insertText", data:text})`.
6. Dispatch `change`.

A simpler shadow-root path: find `rs-message-composer.shadowRoot`, set `textarea[name="message"]`, dispatch `input`, then optionally wait before send.

## Send

Send selectors:

- inside `rs-message-composer.shadowRoot`: `button[type="submit"]`
- document fallback `button[type="submit"]`
- `[data-testid="send-button"]`
- `input[type="submit"]`
- `button[aria-label*="Send"]`, case-insensitive
- fallback button text `send`

If the send button is disabled, return `send_disabled` with any visible validation state.

For auto-send, click once after insertion. If timing knobs are useful for the task, keep them explicit in helper inputs.

## Verify Delivery

Use the narrowest available delivery evidence:

- `rs-timeline` exists after send
- new own message appears or composer clears
- visible success/sent/delivered marker appears

Weak selectors/text seen:

- `.success-message`
- `[data-testid="message-sent"]`
- `.confirmation`
- `.sent-confirmation`
- page text `sent`, `delivered`, or `message sent`

If only weak proof exists, report that uncertainty in the compact result.

## Failure Handling

- Profile load wait: up to about `10s`.
- Tab load timeout: about `30s`.
- Orchestration message dispatch retry: up to `5` attempts, `1s` apart.
- DM paste/send itself: return the failure reason if composer, send, or proof does not resolve.
- Waiting states timeout after about `90s`.
- Return explicit statuses for missing Start Chat, existing conversation, missing composer, missing send button, suspended/deleted target, account state, or throttle text.
