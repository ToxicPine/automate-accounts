# DOM Social Actions

Use when X API mutation details are stale, unavailable, or when DOM confirmation is the clearest way to operate.

## Common DOM Primitives

Tweet cards:

```css
article[data-testid="tweet"]
```

Tweet text:

```css
[data-testid="tweetText"]
```

Tweet ID:

- preferred: first link matching `/status/<id>`
- fallback: short text hash for local dedupe only

Safe click pattern:

1. reject elements inside `a[href]` unless navigation is intended
2. `scrollIntoView({block:"center"})`
3. dispatch mouseover, mousemove, mousedown, mouseup, click with center coordinates
4. fallback to `.click()`

## Like

Selector:

```css
[data-testid="like"]
```

Loop:

1. scan visible `article[data-testid="tweet"]`
2. skip processed tweet IDs
3. apply filters: keywords, skip ads, skip retweets, language, min likes/followers when available
4. skip if button aria-label contains `unlike`
5. click, wait, record compact result
6. if no action, scroll and retry a bounded number of passes

Verify first like: article button changes away from `like` or visible count/state changes.

## Bookmark

Direct selector:

```css
[data-testid="bookmark"]
```

Skip if aria-label contains `remove` or `bookmarked`.

Fallback:

1. click `[data-testid="share"]`
2. find `[role="menuitem"][data-testid="bookmark"]`
3. fallback to menu item whose text contains `bookmark`

Verify first bookmark by button/menu state or saved toast.

## Repost/Retweet

Selectors:

```css
[data-testid="retweet"]
[data-testid="unretweet"]
[data-testid="retweetConfirm"]
```

Loop:

1. skip already reposted if `unretweet` exists
2. click `retweet`
3. find confirmation menu item whose text contains `repost`, `retweet`, or `podaj dalej`
4. click confirm
5. wait, record `tweet_id`

For bulk repost loops, keywords are a useful filter and ads/replies can be skipped with DOM filters.

## Reply

Open composer:

```css
[data-testid="reply"]
```

Composer:

```css
[data-testid="tweetTextarea_0"]
```

Send buttons:

```css
[data-testid="tweetButton"]
[data-testid="tweetButtonInline"]
```

Loop:

1. use `/home` or `/search` for feed-style reply loops; for tweet pages, calibrate the composer route first
2. pick unprocessed visible tweet
3. click reply
4. wait for composer
5. insert normalized text within 280 chars
6. if auto-send requested, click send; otherwise close dialog after draft/verification
7. verify first reply from composer close, sent toast, or visible reply state

Simple insertion can use `InputEvent("textInput", {data: message})`; image replies can paste a `File` through a `ClipboardEvent`. Prefer direct DOM insertion only after live composer calibration.

## Follow

Direct selectors:

```css
button[data-testid="follow"]
div[role="button"][data-testid="follow"]
button[data-testid$="-follow"]
div[role="button"][data-testid$="-follow"]
```

Button must be visible, enabled, outside links, and text must look like `follow` while not looking like `following` or `requested`.

Tweet-menu fallback:

1. open tweet caret: `[data-testid="caret"]`, `button[aria-label="More"]`, or `div[role="button"][aria-label="More"]`
2. wait for visible `div[role="menu"]`
3. click menu item whose text starts with follow-like words
4. close menu/dialog

Profile-list workflow:

1. scrape handles from tweet/user cells
2. store deduped `scrapedUsers`
3. navigate `https://x.com/<handle>`
4. wait up to 15s for follow button or already-following state
5. click, verify following state, and mark the target as handled

Verify first follow: profile button changes to following/requested or no follow button remains.

## Unfollow

Selectors:

```css
button[data-testid="unfollow"]
div[role="button"][data-testid="unfollow"]
button[data-testid="following"]
div[role="button"][data-testid="following"]
button[aria-label*="Following"]
div[role="button"][aria-label*="Following"]
```

Require visible user context: closest `UserCell` or tweet article.

Confirmation selectors:

```css
[data-testid="confirmationSheetConfirm"]
[data-testid="confirmationSheetConfirmButton"]
```

Fallback confirmation: dialog button text `unfollow`, `przestan obserwowac`, or `przestań obserwować`.

Tweet-menu fallback mirrors follow, but chooses unfollow-like menu text and then confirms.

## Scrape Visible Users

Selectors:

```css
article[data-testid="tweet"] a[href^="/"]
[data-testid="UserCell"] a[href^="/"]
```

Handle parser:

```text
^/([A-Za-z0-9_]{1,15})$
```

Reject route words: `home`, `explore`, `search`, `notifications`, `messages`, `settings`, `i`, `compose`, `bookmarks`, `lists`.

Return compact user results with only the useful identity/origin fields.

End the scrape loop after repeated no-new passes. Return compact results instead of full page DOM.

## Search/Keyword Workflows

For search-backed workflows:

1. navigate to `https://x.com/search?q=<encoded>&src=typed_query&f=live`
2. verify visible results
3. process visible tweets with the relevant action loop
4. scroll with bounded no-new passes

Keyword monitor actions: `like`, `bookmark`, `follow`, `reply`.

## Loop Exit Signals

Useful exit/result reasons:

- rate-limit dialog/toast/body text
- login/checkpoint/CAPTCHA/2FA/lock/restriction
- repeated missing selector after live route is verified
- confirmation dialog cannot be resolved
- first action result cannot be verified
