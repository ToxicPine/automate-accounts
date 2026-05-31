# Profiles, Search, Network Reads

Use for profile/contact/search/network read operations.

## Profile: Top Card

```http
GET /voyager/api/graphql?includeWebMetadata=true&variables=(vanityName:<publicIdentifier>)&queryId=<dynamicId>
```

Input: `publicIdentifier` from `/in/<publicIdentifier>/`.

Fallback seed:

```text
voyagerIdentityDashProfiles.aeba67850e106299f25de2eb3828c641
```

Success fields:

- `memberId`
- `publicIdentifier`
- `company`
- `status`

DOM can compare visible profile/URL when needed.

## Profile: Regular Profile

```http
GET /voyager/api/graphql?variables=(vanityName:<identifier>)&queryId=<dynamicId>
```

Fallback seed:

```text
voyagerIdentityDashProfiles.4d9e161cdf3cf64b1c9a7a7c1fc94cff
```

Use for broader profile enrichment. Return only fields needed by task.

## Profile Sections

```http
GET /voyager/api/graphql?variables=(profileUrn:urn%3Ali%3Afsd_profile%3A<memberId>,sectionType:education)&queryId=<dynamicId>
GET /voyager/api/graphql?variables=(profileUrn:urn%3Ali%3Afsd_profile%3A<memberId>,sectionType:experience,locale:en_US)&queryId=<dynamicId>
GET /voyager/api/graphql?variables=(profileUrn:urn%3Ali%3Afsd_profile%3A<memberId>,sectionType:languages,locale:en_US)&queryId=<dynamicId>
GET /voyager/api/graphql?variables=(profileUrn:urn%3Ali%3Afsd_profile%3A<memberId>,sectionType:skills,locale:en_US)&queryId=<dynamicId>
```

Fallback seed:

```text
voyagerIdentityDashProfileComponents.f9cecb2a2e7cfce62f53c2b7abbf42e0
```

## Contact Info

```http
GET /voyager/api/graphql?variables=(memberIdentity:<publicIdentifier>)&queryId=<dynamicId>
```

Fallback seed:

```text
voyagerIdentityDashProfiles.e9b0809465a07db1f02e70a82d455e10
```

Respect user scope. Contact info can include private-ish data; return only requested fields.

## Connections List

```http
GET /voyager/api/relationships/dash/connections?decorationId=<dynamicId>&count=<count>&q=search&sortType=RECENTLY_ADDED...
```

Fallback seed:

```text
com.linkedin.voyager.dash.deco.web.mynetwork.ConnectionListWithProfile-16
```

Headers:

```http
x-restli-protocol-version: 2.0.0
accept: application/vnd.linkedin.normalized+json+2.1
```

Return compact rows: `memberId`, `publicIdentifier`, name when needed, connected date when present.

## Following State / Follow Mutation

```http
POST /voyager/api/feed/dash/followingStates/urn:li:fsd_followingState:urn:li:fsd_profile:<memberId>
```

Headers:

```http
x-restli-protocol-version: 2.0.0
accept: application/vnd.linkedin.normalized+json+2.1
```

Body: patch/update following state. Verify exact follow/unfollow patch from live request before state-changing use.

## Search Notes

Search drifts more than profile/message endpoints. Prefer live request capture or current page request blobs.

Known families:

- advanced regular search lazy-loaded actions
- basic search RSC POST
- basic V2/V3 raw URL parsers
- Sales Navigator search/list/profile/highlights
- Recruiter search
- Talent search / pipeline search

Fallback behavior:

1. Navigate once to relevant search UI.
2. Capture current request shape if agent-browser/network tooling supports it.
3. Otherwise inspect LinkedIn SPA/request blobs via eval.
4. Ignore helper-generated requests marked by internal recorder headers.
5. Do not use feed fetch interception as search evidence; feed interception is separate.
6. Return compact rows: `memberId`, `publicIdentifier`, distance/status, maybe headline/company.

## Premium Search Surfaces

Sales Navigator / Recruiter / Talent search families have separate URLs and result schemas. Do not fake them from regular search.

Use live UI/request capture first, then wrap only observed fields. Keep output minimal:

```json
{"memberId":"...","publicIdentifier":"...","source":"sales","status":"ok"}
```
