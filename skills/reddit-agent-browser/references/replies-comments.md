# Replies And Comments

Use for public Reddit replies/comments on post or comment pages.

## Compact Flow

1. Navigate to the target Reddit thread/comment URL.
2. Run `session-state.md` checks: auth, URL match, removed/deleted/suspended state, duplicate reply.
3. Activate the composer.
4. Insert text through Reddit's editor path.
5. Submit once.
6. Verify with DOM first; if needed, use same-origin Reddit JSON reads.
7. Return compact, task-appropriate result details.

## Composer Activation

New Reddit selectors:

- `comment-composer-host[slot="ready"]`
- fallback `comment-composer-host`
- hidden fallback: `comment-composer-host.nd\\:hidden`
- trigger inside component: `faceplate-textarea-input[data-testid="trigger-button"]`
- shadow trigger label: `div.label-container.interior-label.without-label`

Activation sequence:

1. Find trigger, including open shadow roots.
2. Dispatch `pointerdown`, `mousedown`, `focus`, `mouseup`, `click`.
3. Wait about `500ms`.
4. Poll up to `5s` for Lexical/editor readiness.

Old Reddit fallback:

- textarea `.usertext-edit textarea[name="text"]`
- submit `.usertext-edit button[type="submit"], .save-button button`

## Editor Selectors

Try these in order and search open shadow roots:

- `shreddit-composer div[slot="rte"][data-lexical-editor="true"][contenteditable="true"]`
- `div[data-lexical-editor="true"][contenteditable="true"]`
- `shreddit-composer[name="content"][mode="richText"]`
- inside `shreddit-composer`: `div[slot="rte"]`, `div[slot="editor"]`
- shadow fallback: `reddit-rte`

## Insert Text

Robust insertion sequence:

1. Focus editor.
2. Clear existing content with `beforeinput` using `inputType:"deleteContentBackward"`.
3. Insert through paste-style event with `DataTransfer` containing `text/plain`.
4. Verify visible text.
5. If visible length is much larger than expected, clear and retry to avoid double-paste.
6. Fallback to main-world Lexical insertion or `document.execCommand("insertText", false, text)`.
7. Final fallback: `beforeinput` with `inputType:"insertText"` and `data`.

For old Reddit, set textarea value, dispatch `input`, then submit.

## Submit

Submit selectors:

- `shreddit-composer button[slot="submit-button"]`
- `button[slot="submit-button"]`
- `button[type="submit"]`
- fallback button text exactly `comment`, `reply`, or `post`

For auto-send, click once after insertion. If timing knobs are useful for the task, keep them explicit in helper inputs.

## Verify Posted

Cheap DOM proof:

- editor disappears, or
- editor text is empty after submit, or
- new comment by logged-in user appears in thread.

Stronger JSON proof from same-origin reads:

- Post JSON: `https://www.reddit.com{postPath}.json?sort=new&limit=500`
- Profile comments JSON: `https://www.reddit.com/user/{username}/comments.json?sort=new&limit=25`

Match:

- `author` equals logged-in username
- normalized body equals sent text
- `link_id` equals `t3_{postId}` when available
- `created_utc` is recent, about the last 600 seconds

Return the permalink when found. If DOM says editor cleared but JSON cannot find the comment, report weak verification in the compact result.

## Failure Handling

- Tab/page load timeout: about `30s`.
- Paste attempts: up to `5`, about `1s` apart.
- Missing editor after old Reddit fallback: return `editor_not_found`.
- Missing submit button: return `submit_not_found`.
- If Reddit may have blocked/network-failed the post, return `not_confirmed` with the visible reason and any retry count the task chose to use.
- For batches, return consecutive failure counts so the calling task can decide whether to continue.
