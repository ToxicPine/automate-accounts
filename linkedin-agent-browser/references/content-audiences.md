# Content, Audiences, Company, Analytics

Use for post engagement extraction, group/event audiences, company/page reads, feed posts, viewer analytics, profile visit actions.

These surfaces drift. Prefer live UI/request capture before bulk use.

## Post Engagement Leads

Supported capability families:

- post comments
- post reactions
- post reposts

Use when user asks for people who engaged with a post.

Pattern:

1. Resolve post/activity URN from URL or current page.
2. Use comments/reactions/reposts endpoint family for that URN.
3. Return compact people rows: `memberId`, `publicIdentifier`, `action`, optional name/headline.
4. Verify first page against visible post engagement UI if possible.

Avoid returning full comment bodies unless user asked.

## Group Audience

Supported capability families:

- group detail
- group members
- group membership pages

Use when user asks to import/extract members from a LinkedIn group.

Pattern:

1. Resolve group ID/URN.
2. Page members with count/start or dynamic cursor.
3. Return compact people rows.
4. Stop on parser drift; group pages change often.

## Event Audience

Supported capability families:

- event detail
- event attendees / participants
- professional event pages

Use when user asks for attendees or people linked to an event.

Pattern:

1. Resolve event ID.
2. Fetch event detail first.
3. Page attendees/participants.
4. Return compact people rows.

## Company And Page Reads

Supported capability families:

- company info
- company feed posts
- profile feed posts
- organizational page updates

Use when user asks for company/page data or posts.

Return only needed fields: `companyUrn`, name, URL, staff count, follower count, post URNs, author/profile IDs.

## Profile Viewers / Analytics

Supported capability families:

- profile viewer counts
- premium/full viewer lists
- viewer analytics cards

Use only if user asks for viewer analytics and session has access. These are permission-sensitive; stop on paywall/restriction.

Compact row: `memberId`, `publicIdentifier`, viewer context/date if available.

## Visit Profile Action

Supported capability: visit a profile URL or profile route to trigger normal LinkedIn profile-view behavior.

Use cautiously:

- user must request it
- pace slowly
- verify page loaded
- stop on restriction/checkpoint

Prefer normal navigation for visit action; eval endpoint calls are not necessary unless paired with profile read.
