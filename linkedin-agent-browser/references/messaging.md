# Messaging And Conversations

Use for message send, conversation lookup, read state, typing, delete, recall, rich media, inbox ops.

## Message Body Basics

Plain text body:

```json
{"text":"<message>","attributes":[]}
```

Common send fields:

- `mailboxUrn`: `urn:li:fsd_profile:<currentMemberId>`
- `trackingId`: random tracking token
- `dedupeByClientGeneratedToken`: `false`
- `message.body`
- `message.renderContentUnions`
- `message.originToken`

Rich content:

- GIF/files/videos use `renderContentUnions`
- GIF with `gifQuery` may require side-call: `POST /voyager/api/messaging/thirdPartyMedia?action=registerGifShare`
- audio uses protobuf content type and `hexFile` body conversion

Audio header:

```http
Content-Type: application/x-protobuf2; symbol-table=voyager-17844
```

## Send Message: New Conversation

```http
POST /voyager/api/voyagerMessagingDashMessengerMessages?action=createMessage
```

Inputs:

- `memberId`: current account profile ID
- `prospectMemberId`: recipient fsd profile ID
- `messageContent`: text/rich content
- `contextEntityUrn`: optional context

Headers:

```http
x-restli-protocol-version: 2.0.0
accept: application/json
```

Body shape:

```json
{
  "mailboxUrn": "urn:li:fsd_profile:<memberId>",
  "trackingId": "<random tracking id>",
  "dedupeByClientGeneratedToken": false,
  "message": {
    "body": "<message body object>",
    "renderContentUnions": [],
    "originToken": "<random origin token>"
  },
  "hostRecipientUrns": ["urn:li:fsd_profile:<prospectMemberId>"]
}
```

Optional context:

```json
"messageRequestContextByRecipient": [{
  "contextEntityUrn": "<contextEntityUrn>",
  "hostRecipientUrn": "urn:li:fsd_profile:<prospectMemberId>"
}]
```

Success return:

- `conversationUrn`
- `conversationId`
- `messageId`
- `sentAt`

First send DOM check: intended participant + last message snippet.

Failure: 400/403 may mean not connected, blocked, invalid recipient, or body drift. Stop after first unexplained failure; inspect live UI/request shape.

## Send Message: Existing Conversation

```http
POST /voyager/api/voyagerMessagingDashMessengerMessages?action=createMessage
```

Inputs:

- `conversationUrn` or raw conversation ID plus current member ID
- `messageContent` or text

Headers:

```http
x-restli-protocol-version: 2.0.0
accept: application/json
```

Body shape:

```json
{
  "mailboxUrn": "urn:li:fsd_profile:<memberId>",
  "trackingId": "<random tracking id>",
  "dedupeByClientGeneratedToken": false,
  "message": {
    "conversationUrn": "<normalized conversationUrn>",
    "body": "<message body object>",
    "renderContentUnions": [],
    "originToken": "<random origin token>"
  }
}
```

Higher-level flow:

1. mark conversation read
2. sort multiple message contents by descending `dispatchOrder`
3. send each message

## Conversation URNs

Profile URN:

```text
urn:li:fsd_profile:<memberId>
```

Conversation URN:

```text
urn:li:msg_conversation:(urn:li:fsd_profile:<memberId>,<conversationId>)
```

If value already starts with `urn:li:msg_conversation:`, use as-is.

## Conversation Data

```http
GET /voyager/api/voyagerMessagingGraphQL/graphql?queryId=<dynamicId>&variables=(messengerConversationsId:<encoded conversationUrn>,count:20)
```

Headers:

```http
x-restli-protocol-version: 2.0.0
accept: application/graphql
```

Fallback seed:

```text
messengerConversations.9dcca8beeb2ea48b1bff937044631888
```

Success fields include conversation identity, participants, read/unread state, notification status, backend URN, and latest event data when present.

## Conversation Events

```http
GET /voyager/api/voyagerMessagingGraphQL/graphql?queryId=<dynamicId>&variables=(deliveredAt:<timestamp>,conversationUrn:<encoded conversationUrn>,countBefore:<n>,countAfter:0)
```

Headers:

```http
x-restli-protocol-version: 2.0.0
accept: application/graphql
```

Fallback seed:

```text
messengerMessages.b52340f92136e74c2aab21dac7cf7ff2
```

Success fields per event: body, entity/message URN, delivered time, render content, sender.

## Conversation IDs By Member IDs

```http
GET /voyager/api/voyagerMessagingGraphQL/graphql?queryId=<dynamicId>&variables=(mailboxUrn:<encoded current profile urn>,recipients:List(<encoded recipient urns>))
```

Headers:

```http
x-restli-protocol-version: 2.0.0
accept: application/graphql
```

Fallback seed:

```text
messengerConversations.09995bad1c3b51bdf60c10a49ae874f0
```

Use to avoid creating duplicate conversations when thread already exists.

## Conversations Last Event / Search

```http
GET /voyager/api/voyagerMessagingGraphQL/graphql?queryId=<dynamicId>&variables=(<criteria>,count:20,mailboxUrn:urn%3Ali%3Afsd_profile%3A<memberId>,<cursor?>)
```

Headers:

```http
x-restli-protocol-version: 2.0.0
accept: application/graphql
```

Fallback seed:

```text
messengerConversations.755b19e066ebdfc4c70db8b2aaae855e
```

Criteria:

- default categories: `PRIMARY_INBOX`, `INBOX`, `SPAM`, `ARCHIVE`
- archived-only: `ARCHIVE`
- optional keywords
- optional next cursor

Success fields: conversations, next cursor, retryability when nullable/empty response suggests transient fetch problem.

## Read State

```http
POST /voyager/api/voyagerMessagingDashMessengerConversations?ids=List(<encoded conversationUrn>)
```

Body: `entities[conversationUrn].patch.$set.read = <boolean>`.

## Typing

```http
POST /voyager/api/voyagerMessagingDashMessengerConversations?action=typing
```

Headers:

```http
x-restli-protocol-version: 2.0.0
```

Body:

```json
{"conversationUrn":"<conversationUrn>"}
```

## Delete Conversation

```http
DELETE /voyager/api/voyagerMessagingDashMessengerConversations/<conversationUrn>
```

Headers:

```http
x-restli-protocol-version: 2.0.0
accept: application/graphql
```

## Recall/Delete Message

```http
POST /voyager/api/voyagerMessagingDashMessengerMessages?action=recall
```

Body:

```json
{"messageUrn":"<messageUrn>"}
```

## Inbox Ops

Additional supported operation families:

- edit message
- set conversation category / archive-like state
- search GIF
- register GIF share
- upload attachment metadata

Use live request capture before state-changing rich-media or inbox-category operations; these drift more than plain text send.
