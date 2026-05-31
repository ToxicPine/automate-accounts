# X Task Notes

Load only the file needed for the current task.

- `request-basics.md`: same-origin X fetch wrapper, headers, `ct0`, auth/session probes, response/UI signals.
- `dms-audiences.md`: DM send/permission/conversation surfaces and audience extraction from users, tweets, search, followers, following, favoriters, retweeters.
- `dom-social-actions.md`: DOM action recipes for follow, unfollow, reply, like, repost, bookmark, scraping visible users, search/keyword workflows.
- `state-results.md`: queue/state patterns, pacing, compact outputs, and loop control.

Runtime rule: treat notes as starting maps. For state-changing actions, verify the first operation in the live X session with compact eval output plus a narrow DOM/snapshot check when the task calls for confidence.
