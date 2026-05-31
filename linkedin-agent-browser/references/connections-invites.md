# Connections And Invites

Use for connection requests, invitation accept/list/withdraw, and quota signals.

## Connect: Desktop Invitation

```http
POST /voyager/api/voyagerRelationshipsDashMemberRelationships?action=verifyQuotaAndCreateV2&decorationId=com.linkedin.voyager.dash.deco.relationships.InvitationCreationResultWithInvitee-2
```

Inputs:

- `memberId`: fsd profile ID
- `note`: optional custom message

Body:

```json
{
  "invitee": {
    "inviteeUnion": {
      "memberProfile": "urn:li:fsd_profile:<memberId>"
    }
  },
  "customMessage": "<optional note>"
}
```

Header:

```http
x-restli-protocol-version: 2.0.0
```

Note rules:

- omit `customMessage` when no note
- before sending notes in bulk, check note allowance if available
- note unavailable when max note length is effectively 200 or remaining note credits are 0
- final interpolated note should stay within 300 chars

Success:

- HTTP 2xx
- no returned data required
- first target DOM shows pending/invited or no Connect button

Failure mapping:

- `400 MAX_INVITATION_SENT`: desktop restriction
- `400 PRIMARY_HANDLE_NOT_CONFIRMED`: desktop restriction
- `400 CANT_RESEND_YET`: 3-week reinvite delay
- `400 CANT_INVITE_CONNECTION_LIMIT_REACHED`: weekly connection limit
- other `400`: likely account blocked or unsupported invitation state
- `403`: profile inaccessible
- `406`: invalid invitation state
- `429`: desktop quota/rate restriction

Set short cooldown only for desktop/weekly-limit restriction classes. Do not treat every 403/406 as same quota cooldown.

## Connect: Mobile Fallback

Historical fallback. Prefer desktop connect + DOM verification.

```http
POST /mwlite/invite
```

Body:

```json
{"inviteeVanityName":"<publicIdentifier>","trackingId":"<random tracking id>"}
```

Headers:

```http
x-restli-protocol-version: 2.0.0
content-type: application/json;charset=UTF-8
x-simulate-mobile: true
```

Failure mapping:

- HTTP `429`: mobile restriction
- body `dropReason: FUSE_LIMIT`: mobile restriction
- other `dropReason`: return reason as-is
- `inviteSuccess: false`: failed

## Batch Email/Profile Connect

Specialized connect path. Verify current behavior before use.

```http
POST /voyager/api/growth/normInvitations?action=batchCreate
```

Body:

```json
{
  "invitations": [{
    "emberEntityName": "growth/invitation/norm-invitation",
    "invitee": {
      "com.linkedin.voyager.growth.invitation.InviteeProfile": {
        "profileId": "<memberId>"
      }
    },
    "trackingId": "<random tracking id>",
    "message": "<optional message>"
  }],
  "uploadTransactionId": "<random tracking id>",
  "defaultCountryCode": "<optional country code>"
}
```

Header:

```http
x-li-page-instance: urn:li:page:d_flagship3_abi_m2m;<random tracking id>
```

Failure: `429` or `503` = restriction/rate state.

## Invitation Quota Summary

Use to read sent-invitation quota summary.

```http
GET /voyager/api/voyagerRelationshipsDashGenericInvitationFacets?q=sent
```

Return compact field: `numTotalSentInvitations`.

## Accept Invitation

```http
POST /voyager/api/voyagerRelationshipsDashInvitations/<encoded urn:li:fsd_invitation:<id>>?action=accept
```

Inputs:

- `invitationId`
- `sharedSecret`

Body:

```json
{"invitationType":"CONNECTION","sharedSecret":"<sharedSecret>"}
```

Header:

```http
x-restli-protocol-version: 2.0.0
```

Failure: `400` containing `CANT_ACCEPT_CONNECTION_LIMIT_REACHED` means stop accepting.

## Accept Invitation V2: RSC

Newer RSC action path.

```http
POST /flagship-web/rsc-action/actions/server-request?sduiid=com.linkedin.sdui.requests.mynetwork.addaInvitationAction
```

Headers:

```http
x-restli-protocol-version: 2.0.0
accept: */*
content-type: application/json
```

Body shape:

- top-level request wrapper contains `requestId`
- `serverRequest` contains nested `requestedArguments`
- action type: `InviteeActionType_ACCEPT`
- invitation type: `GenericInvitationType_CONNECTION`
- required values: `invitationId`, `validationToken`
- first/last names included when present
- origin: `InvitationOrigin_INVITATION_PREVIEW`

RSC response containing `responses.error.ServerError` is failure; extract embedded message.

## Received Invitations V1

```http
GET /voyager/api/relationships/invitationViews?invitationTypes=List(CONNECTION)&q=pendingInvitationsBasedOnRelevance&includeInsights=true&start=<start>&count=<count>&paginationToken=<token>
```

Optional params: `start`, `count`, `paginationToken`.

Return:

- row fields: profile, sent time, shared secret, invitation ID
- collection field: pagination token

## Received Invitations V2 / RSC

Initial:

```http
POST /flagship-web/mynetwork/invitation-manager/received/CONNECTION/
```

Pagination:

```http
POST /flagship-web/rsc-action/actions/pagination?sduiid=com.linkedin.sdui.pagers.mynetwork.invitationsList
```

Headers:

```http
accept: */*
content-type: application/json
x-li-rsc-stream: true
```

Initial body: navigation-to-screen request for received connection invitations.

Pagination body:

- `pagerId`
- `startIndex`
- invitation types: `GenericInvitationType_CONNECTION`, `GenericInvitationType_MEMBER_FOLLOW`
- direction: `PendingInvitationDirection_RECEIVED`

Valid parsed row requires:

- invitation ID
- validation token
- first name
- last name
- member ID from encoded `fsd_profile`

If non-empty RSC cannot produce those fields, treat as parser drift.

## Sent Invitations

```http
GET /voyager/api/relationships/sentInvitationViewsV2?count=<count>&invitationType=CONNECTION&q=invitationType&start=<start>
```

Inputs: `count` default around 100, `start` offset.

Header:

```http
x-restli-protocol-version: 2.0.0
```

Return rows only include:

- `invitationId`
- `sentTime`
- `toMemberId`

No status field unless live response proves one.

## Withdraw Pending Invitation

```http
POST /voyager/api/voyagerRelationshipsDashInvitations/urn%3Ali%3Afsd_invitation%3A<invitationId>?action=withdraw
```

Header:

```http
x-restli-protocol-version: 2.0.0
```

Body:

```json
{"invitationType":"CONNECTION"}
```

Success: Rest.li `ActionResponse`; sent-invitation check no longer shows pending invite.

Cleanup pattern: page sent invites in chunks of 100, withdraw older than threshold, sleep roughly 500-1000 ms between removals.
