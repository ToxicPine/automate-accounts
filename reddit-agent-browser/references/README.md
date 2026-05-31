# Reddit References

Use the smallest file that matches the task:

- `session-state.md`: session/auth checks, page classification, duplicate and target-state signals.
- `replies-comments.md`: public replies/comments, composer activation, insertion, submit, posted verification.
- `direct-messages-chat.md`: profile/chat navigation, DM insertion, send, delivery verification.
- `queues-targeting-state.md`: target extraction, batching, delays, duplicate/contact state, compact results.

Operational rule: load one reference, run one small `agent-browser eval`, verify the first state-changing operation with DOM/snapshot, then continue script-first with compact results.
