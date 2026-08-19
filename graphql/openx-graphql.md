---
generated: '2026-08-13'
method: searched
source: https://docs.openx.com/openxselect/oxs-api-reference/
endpoint: https://api.openx.com/oa/graphql
introspection: gated
status: BETA
---

# OpenXSelect API (GraphQL)

OpenXSelect is OpenX's curation platform, and its API is GraphQL-only. This is
the machine-readable contract for audiences, segments, deals, packages and
bidder objects — there is no REST equivalent and no OpenAPI.

## Endpoint

| | |
|---|---|
| Production endpoint | `https://api.openx.com/oa/graphql` |
| Auth header | `x-apikey: <api key>` |
| Key issuance | OpenXSelect UI → username menu → Settings → API Keys → Create New API Key |
| Key scope | organization-wide |
| Key expiry | 90 days (default and only documented value) |
| Explorer | <https://docs.openx.com/openxselect/oxs-api-graphql-explorer/> (Schema / SDL / Explorer tabs) |
| Reference | <https://docs.openx.com/openxselect/oxs-api-reference/> |

## Introspection is gated

An unauthenticated introspection query returns **HTTP 401** from the Apigee
gateway in front of the service:

```
POST https://api.openx.com/oa/graphql
{"query":"{__schema{queryType{name}}}"}

HTTP/2 401
{"fault":{"faultstring":"Failed to resolve API Key variable request.header.x-apikey",
          "detail":{"errorcode":"steps.oauth.v2.FailedToResolveAPIKey"}}}
```

Probed 2026-08-13. **No SDL is stored in this repo** — recovering the real
schema requires an OpenXSelect API key, and the SDL is downloadable from the
Explorer's SDL tab once authenticated. Everything below is transcribed from
OpenX's own published reference, not from introspection.

## Published schema shape

The reference publishes seven sections: **Query**, **Mutation**, **Objects**
(~120 types), **Inputs** (~40 types), **Enums**, **Scalars** and **Interfaces**.

### Custom scalars

`uuid`, `timestamptz`, `numeric`, `Decimal`, `Decimal2`, `Decimal4`,
`Decimal6`, `JSON`, `StringOrInt`, `UBigInt64`, `GraphQLUBigInt64`,
`IntersectionOperation`.

### Query surface (selected)

| Field | Returns | Notes |
|---|---|---|
| `ping` | `String` | health check; returns `pong` |
| `providers(limit, offset)` | `[Provider!]!` | offset pagination, both args required |
| `providerById(id)` | `Provider` | |
| `segments(limit, offset)` | `[Segment!]!` | |
| `segmentById(id: uuid!)` | `Segment` | |
| `segmentsByCategory(category, limit, offset)` | `[Segment!]!` | |
| `segmentsByCountry(country, limit, offset)` | `[Segment!]!` | country must match `optionsByPath(path: "segment.country")` |
| `segmentsByRegion(...)` | `[Segment!]!` | |
| `audienceById(id: uuid!)` | `Audience` | |
| `dealForecastingJob(job_id)` | `JSON` | poll for async forecasting results |
| `dealPromptAudienceSegmentSearch(...)` | `DealPromptAudienceSegmentSearchResult` | |
| `optionsByPath(path, filter)` | `OptionsByPathResult` | runtime enumeration source for most constrained fields |

### Mutation surface (selected)

| Field | Returns | Notes |
|---|---|---|
| `audienceCreate(input: AudienceCreateParams!)` | `Audience!` | |
| `audienceUpdate(id: uuid!, input: AudienceUpdateParams!)` | `Audience!` | |
| `audienceArchive(id: uuid!)` | `uuid!` | archive = delete |
| `audienceActivate(id: uuid!)` | `uuid!` | required before an audience can be used in a deal |
| `routedSegmentCreate/Update/Archive` | `RoutedSegment!` / `String!` | retargeting segments only |
| `createDealForecastingJob(deal, package)` | `JSON` | async; returns a `job_id` to poll |
| `dealCreate` / `dealUpdate` / deal pause / archive | `Deal` | full deal lifecycle |
| `bidderOrder*` / `bidderLineItem*` / `bidderCreative*` | | create/update for the bidder object family |

### Core objects

`Account` → `Advertiser`, `Provider`, `AccountProvider`, `Segment`, `Audience`,
`AudienceSegment`, `Deal`, `DealParticipants`, `Package`, `Targeting`,
`URLTargeting`, `BidderOrder` → `BidderLineItem` → `BidderCreative`,
`RetargetingSegment` / `PredefinedSegment` (both implement the `RoutedSegment`
interface), `Export`, `FrequencyCapping`, `CuratedDealBuyerDiscount`,
`ThirdPartyFeesConfig`, `ThirdPartyRevenue`, plus ~70 `Targeting*` types
covering geo, content, device, video, viewability and inventory-quality
criteria.

See `data-model/openx-data-model.yml` for the entity graph and the documented
relationships between these types.

## Error semantics

Application errors come back as **HTTP 200** with a top-level `errors` array
alongside `data`; the caller must test for `errors` rather than the status code.
Connection- and auth-level failures surface as HTTP status codes. Example
published by OpenX:

```json
{
  "errors": [{
    "message": "Validation error",
    "path": ["audienceById"],
    "extensions": {
      "code": "GRAPHQL_VALIDATION_FAILED",
      "errors": [{"code": "data-exception",
                  "message": "invalid input syntax for type uuid: \"bad-uuid\""}]
    }
  }],
  "data": {"audienceById": null}
}
```

Source: <https://docs.openx.com/openxselect/oxs-api-errors-and-exceptions/>

## Deprecation practice

OpenX manages GraphQL change with the `@deprecated` directive plus a dated
release-note entry naming the replacement and a removal date. The one captured
example:

```graphql
"""Target by the ad unit's maximum duration."""
adunit_max_duration: TargetingAdunitMaxDuration
  @deprecated(reason: "Use adunit_max_duration_range instead")

"""Target by the ad unit's maximum duration using a range."""
adunit_max_duration_range: TargetingAdunitMaxDurationRange
```

Announced June 2025, removal date 2025-07-31.
Source: <https://docs.openx.com/resources/release-notes/>

## Caveat on the documentation host

Every `docs.openx.com` content page currently returns a 1,132-byte Angular
shell whose script bundles 404 at the site root, so the rendered reference does
not display. The content itself is still published and machine-readable at
`https://docs.openx.com/assets/js/search-data.json` (964 sections, 1.5 MB),
which is where this capture came from.
