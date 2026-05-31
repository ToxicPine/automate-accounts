# LinkedIn References

Use only when `SKILL.md` workflow needs endpoint detail. Pick narrow file; do not load all by default.

- `request-basics.md`: auth fetch rules, CSRF, headers, dynamic GraphQL IDs, health check.
- `connections-invites.md`: connect, invite accept/list/withdraw, sent/received invites, quota signals.
- `messaging.md`: send message, existing conversation send, conversation reads/events, read/typing/delete/recall.
- `profiles-search.md`: profile reads, contact info, profile sections, connections list, following state, search notes.
- `content-audiences.md`: post engagement, groups/events, company/page/feed, viewer analytics, profile visits.

Runtime rule: treat these as starting maps. Verify state-changing actions against live LinkedIn session with compact eval result + narrow DOM/snapshot check.
