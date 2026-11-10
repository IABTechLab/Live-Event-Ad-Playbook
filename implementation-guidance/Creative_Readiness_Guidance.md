![IAB Tech Lab](https://drive.google.com/uc?id=10yoBoG5uRETSXRrnJPUDuONujvADrSG1)

# Live Event Ad Playbook (LEAP)
# Creative Readiness Implementation Guide (Ad Management API)

> **Status: proposal for public comment.** This guide is contributed by The Trade Desk for review by LEAP Work Stream 2. It introduces no new API. It describes how the existing IAB Tech Lab [Ad Management API v1.1](https://github.com/InteractiveAdvertisingBureau/AdManagementAPI) fits into the live-event timeline that the Forecast API and Concurrent Streams API already describe, so that creatives are approved and ingested before the first ad break of a live event. Where this guide proposes something that the Ad Management API does not define today, it says so explicitly.

## Table of Contents

- [Status of this Guide](#status-of-this-guide)
- [Purpose](#purpose)
- [Why Live Events Need This](#why-live-events-need-this)
- [Relationship to the Forecast and Concurrent Streams APIs](#relationship-to-the-forecast-and-concurrent-streams-apis)
- [Supported Interaction Model](#supported-interaction-model)
- [Timeline: When to Submit, Check and Re-check](#timeline-when-to-submit-check-and-re-check)
- [The Creative Brief Comes from the Forecast](#the-creative-brief-comes-from-the-forecast)
- [Ad Object Recommendations for Live Video](#ad-object-recommendations-for-live-video)
- [Audit Status Interpretation](#audit-status-interpretation)
- [Approval Is Not Ingestion](#approval-is-not-ingestion)
- [Bidding Policy: Restrictive vs. Permissive for Live](#bidding-policy-restrictive-vs-permissive-for-live)
- [Exchange-hosted Markup and Pre-caching](#exchange-hosted-markup-and-pre-caching)
- [Webhooks and Polling Cadence](#webhooks-and-polling-cadence)
- [Expiration and Re-activation Before an Event](#expiration-and-re-activation-before-an-event)
- [Intermediaries](#intermediaries)
- [Associating Creatives with Events (Proposed Extension)](#associating-creatives-with-events-proposed-extension)
- [Cross-API Identity Guidance](#cross-api-identity-guidance)
- [Example Exchange](#example-exchange)
- [Notes on Common API Guide Application](#notes-on-common-api-guide-application)
- [Open Questions for Work Stream 2](#open-questions-for-work-stream-2)

## Status of this Guide

This document is an API-specific implementation guide for using the Ad Management API in the context of LEAP. It is designed to be used together with the LEAP Common API Guidance and the Ad Management API v1.1 specification.

**Files:** `Common_API_Guidance.md`; [Ad Management API v1.1](https://github.com/InteractiveAdvertisingBureau/AdManagementAPI/blob/master/Ad%20Management%20API%20v1.md)

The Ad Management API specification is the source of truth for its objects, endpoints and behavior. The common guide applies to transport conventions where the Ad Management API specification is silent. This guide only explains how and when to use the API for live events.

## Purpose

The Forecast API tells a buyer that an event is coming and what its inventory looks like. The Concurrent Streams API tells it how many people are watching once the stream opens. Neither says whether the ads the buyer intends to run will be eligible to serve when the first break starts.

For live events that question has to be answered before kickoff. This guide describes how to use the Ad Management API to:

- submit the creatives intended for a live event far enough ahead of `scheduledstart` to allow for audit, correction and ingestion
- learn each creative's audit status, and any corrections, before the event
- keep that status current through the event, since it can change
- avoid bidding into live breaks with creatives that will be discarded

## Why Live Events Need This

In on-demand inventory a creative that is quarantined, rejected or not yet transcoded costs a few impressions and is fixed the same day. In a live break there is no later. A tentpole event's halftime pod is the highest-value inventory of the year, it arrives once, and a bid carrying an ineligible creative is a wasted slot in the bid response that an eligible creative could have filled. At the publisher's side the outcome is worse: an unfilled or late-filled slot in a server-side inserted stream is slate, or a house ad, in front of the largest audience it will see all season.

Three things make live different from the steady state the Ad Management API was originally written for:

1. **Lead time is fixed by the schedule, not by the buyer.** Audit queues, correction loops and transcoding each take time measured in hours or days. The forecast gives that time; the creative process has to use it.
2. **Server-side ad insertion adds a step after approval.** Most live streams are stitched server-side (SSAI or SGAI). The stitcher must have transcoded the creative into the stream's renditions before it can be inserted. An approved creative that has not been ingested still cannot run.
3. **Submission via bidding fails at exactly the wrong moment.** The Ad Management API notes that an exchange using restrictive bidding may let new ads enter its audit queue simply by seeing them in bid responses. For live, a creative first seen at kickoff is quarantined through the breaks that matter.

The Ad Management API's own rationale names programmatic TV as a format whose "stringent ad approval requirements necessitate an API." LEAP is the context where that requirement is sharpest.

## Relationship to the Forecast and Concurrent Streams APIs

```text
Forecast API              -> discover the event; read its inventory configuration (the creative brief)
Ad Management API         -> submit creatives; receive audit status and corrections; confirm readiness
Concurrent Streams API    -> scale and pace while the stream is open, bidding only with eligible creatives
```

The Forecast API's `inventoryconfig` is the trigger and the brief: `supportedmtype` says which formats the event accepts, `totaladdurationsec` and `expectedpodcount` imply the pod structure, and any duration constraints the provider communicates (see below) bound the creatives that can run. The moment an event's `eventstatus` moves to `1` (scheduled) is the latest sensible moment to begin submission; many buyers will begin while the event is still tentative.

The Concurrent Streams API has no creative dimension. Its role here is indirect: the live counts that drive scaling and pacing should only be converted into bids that carry creatives with an approved status at that moment.

## Supported Interaction Model

The Ad Management API is a demand-partner-to-supply-partner API. The supply partner (publisher, ad server, SSAI platform or exchange) implements the endpoints; the demand partner (DSP or other bidder) calls them.

Typical flow for a live event:

1. Demand partner reads the event's `inventoryconfig` from the Forecast API.
2. Demand partner `POST`s each creative intended for the event as an AdCOM `Ad` object to `{baseUrl}/bidder/{bidderId}/ads`.
3. Supply partner audits the creative and, if its insertion stack requires it, transcodes it. It updates the ad's `audit` object (`status`, `feedback`, `corr`, `lastmod`).
4. Supply partner notifies the demand partner by webhook (if supported) and the demand partner reconciles by polling `GET {baseUrl}/bidder/{bidderId}/ads?auditStart={timestamp}`.
5. Demand partner fixes or replaces anything denied, and confirms every creative it will use is approved before `scheduledstart`.
6. During the event the demand partner bids only with creatives whose last known status permits it, and keeps polling at a low rate because status can change (`5` changed, `6` expired).

The Ad Management API's "exchange" and "bidder" should be read generically, as its specification says: the supply partner may be the publisher's own ad server or SSAI platform, and the demand partner may be an SSP acting on behalf of its DSPs.

## Timeline: When to Submit, Check and Re-check

The following aligns creative work with the Forecast API's stability table and the Concurrent Streams API's polling guidance. Times are illustrative starting points for agreement between partners; the supply partner's published lead times take precedence.

| Time before event | Forecast state | Creative action |
|---|---|---|
| 30+ days | Event discovered, possibly tentative | Read `inventoryconfig`. Confirm the supply partner's Ad Management API endpoint, bidding policy and creative lead times (Integration Checklist, Ad Management API Appendix A). Begin building creatives to the brief. |
| 14 days | Still tentative or now scheduled | Submit creatives that are ready. Tentative events are a reason to submit early, not late: approval carries across to the scheduled event. |
| 7 days | Scheduled; inventory firming | Submit every remaining creative intended for the event. Treat this as the deadline for first submission so that denials can be fixed and resubmitted. |
| 72 hours | | Reconcile status for every submitted creative. Resubmit anything denied after addressing `audit.feedback`. Ask the supply partner to confirm ingestion/transcoding where applicable. |
| 24 hours | Final forecast baseline | Confirm the eligible creative set. Remove or replace any creative not approved from line items targeting the event. Store any `corr` corrections. |
| 1 hour | Stream about to open | Final status poll. Verify nothing has moved to `5` (changed) or `6` (expired). |
| Live | Concurrent Streams authoritative | Bid only with eligible creatives. Poll status at the normal low cadence; treat webhook denials as immediate stop-bid signals. |
| After | Review | Record which creatives served, which were denied and why. Feed lead times back into the next event's plan. |

## The Creative Brief Comes from the Forecast

Consumers should read the following from the Forecast API event record and use it to select or build creatives:

- `inventoryconfig.supportedmtype`: accepted creative formats (`2` = video for most live events).
- `inventoryconfig.totaladdurationsec` and `expectedpodcount`: the implied average pod length, which bounds practical creative durations.
- Duration constraints where the provider communicates them. The Forecast API specification's JSON sample carries `maxadduration`; the current object definition does not. Until the Forecast API defines it, providers should communicate accepted durations, VAST versions, mime types and creative lead time either in `inventoryconfig.ext` by agreement, or in the Ad Management API Integration Checklist they publish to demand partners.

Providers are encouraged to make the live-event creative specification machine-readable in a future Forecast API revision. This guide does not define those fields.

### Looking ahead: a structured creative-requirements object

What broadcasters' insertion stacks actually require of a creative goes well beyond `supportedmtype`: container and codec, resolution and aspect ratio, frame rate, video and audio bitrates, audio channels and loudness, maximum file size, accepted durations, VAST version, and whatever else a given stitcher enforces. Today those requirements travel in PDFs and onboarding emails, and a mismatch is discovered when the creative fails audit or, worse, fails to render in the break.

A later iteration of the Forecast API (per event, in `inventoryconfig`) and of the Deal Sync API (per deal) should carry a structured creative-requirements object expressing these constraints. It should be **one object, defined once in AdCOM and referenced by both APIs**, not two parallel definitions: both APIs already embed AdCOM objects, AdCOM is where creative attributes are defined today (the `Ad`, `Video` and `Audio` objects, and the Placement group's mechanical specifications), and a seller's requirements for a creative should read the same whether they arrive per event or per deal. A DSP could then compare a buyer's assets against the seller's requirements at planning time and raise the mismatch to the buyer before submission, which is earlier and cheaper than a denial at audit and far cheaper than a rendering failure during the event. This guide is written with that destination in mind: today, read the supply partner's published creative specification; later, read it from the API. Defining the object is out of scope here and is raised in the Open Questions.

## Ad Object Recommendations for Live Video

The Ad Management API `Ad` is an AdCOM 1.x `Ad` object. For live video the following attributes matter most. Which are required is the supply partner's business policy (Ad Management API: "Attributes Required").

| Attribute | Recommendation for live |
|---|---|
| `id` | The demand partner's creative ID when the supply partner uses bidder IDs (the common case). The same value should appear as `crid` in OpenRTB bid responses so that approvals, bids and delivery reconcile. |
| `adomain`, `cat`, `attr` | Provide accurately. Auditors frequently correct these via `audit.corr`; the demand partner should store corrections and bid with the corrected values. |
| `video.dur` | Provide. Duration is the first thing a live auditor checks against the event's pod structure, and the first reason for denial. |
| `video.mime`, `video.ctype` | State the actual media type(s) and VAST version. SSAI stitchers have narrow accepted lists. |
| `video.curl` vs `video.adm` | For audit, a stable `curl` that returns representative VAST is usually preferable to inline `adm`, because the supply partner can fetch the mezzanine from the VAST `MediaFile`s for transcoding. Do not send both. |
| `secure` | Set `1`. |
| `ext` | See [Associating Creatives with Events](#associating-creatives-with-events-proposed-extension). |

The markup submitted should be a faithful example of what will be returned in bid responses. Supply partners should state which runtime variations (impression URLs, cache busters, macros) are not material changes, per the Ad Management API's guidance on material change.

## Audit Status Interpretation

Audit status values are defined in AdCOM (List: Audit Status Codes). For live events, interpret them as follows:

| `audit.status` | Meaning | Live-event handling |
|---:|---|---|
| 1 | Pending audit | Not eligible under restrictive bidding. If still pending at T-72h, escalate with the supply partner. |
| 2 | Pre-approved | Eligible subject to policy, but may still change. Keep polling. |
| 3 | Approved | Eligible. Apply any `corr` corrections. Confirm ingestion separately where SSAI applies. |
| 4 | Denied | Not eligible. Read `feedback`, fix, and resubmit by touching the ad (`PUT`/`PATCH`). Do not bid with it. |
| 5 | Changed; resubmission requested | The supply partner detected a material change in use. Stop bidding with it in live breaks and resubmit. |
| 6 | Expired | Re-activate by touching the ad; whether re-audit is required is supply-partner policy. Re-activate well before the event. |
| 500+ | Vendor-specific | Follow the supply partner's documentation. A vendor code is a reasonable place to signal "approved and ingested" today (see next section). |

`audit.feedback` is free text intended for humans. Demand partners should surface it to the trader or creative team rather than parse it.

## Approval Is Not Ingestion

For server-side inserted streams, an approved creative is not necessarily a servable one. The stitcher must have fetched the mezzanine and transcoded it into the stream's renditions, and some platforms also require a successful test insertion. The Ad Management API has no status for this.

Recommendations:

- **Supply partners** that require ingestion should (a) publish the expected time from approval to ingestion in their Integration Checklist, and (b) signal ingestion completion in a way the demand partner can read programmatically, using a vendor-specific audit status (`500+`) or an `audit.ext` field, with the meaning documented a priori. Until such a signal exists, `status: 3` should be issued only when the creative is actually servable, and `feedback` should say so.
- **Demand partners** should treat a creative as eligible for a live break only when both the audit status and, where applicable, the ingestion signal permit it.
- **Both** should treat ingestion lead time as part of the event plan, like QPS capacity.

This is the single most useful thing a future Ad Management API revision could add for live: a standard way to say "approved and ready to insert." See Open Questions.

## Bidding Policy: Restrictive vs. Permissive for Live

The Ad Management API describes two policies. Both are compatible with live events, with different obligations:

- **Restrictive bidding** (new creatives cannot win until approved). The supply partner should discard bids carrying unapproved creatives rather than let them time out, and the demand partner must complete submission and approval before the event. This is the safer policy for live and the one this guide assumes in its examples.
- **Permissive bidding** (new creatives can win until disapproved). Live breaks are where a disapproval arriving mid-event does the most damage. Supply partners using permissive bidding should still encourage pre-submission so that policy or technical blocks are known before kickoff, and should deliver denials by webhook so the demand partner can stop bidding immediately.

In either case, when a bid response can carry several bids, demand partners should prefer approved creatives for live-event impressions and not rely on the supply partner to pick a fallback.

## Exchange-hosted Markup and Pre-caching

Where the supply partner hosts the ad markup, the demand partner bids with a reference to the approved ad (`mid` in OpenRTB 3.x, or the equivalent arranged for OpenRTB 2.x) and supplies macro values at bid time. For live this has two advantages: the bid response is smaller and faster at peak, and the stitcher already holds the asset. Supply partners that support exchange-hosted markup should say so in their Integration Checklist; demand partners should prefer it for live events where offered.

Pre-caching of decisions and assets ahead of a break is the subject of LEAP Work Stream 1 (PreFetch & Cache) and is out of scope for this guide, but it depends on the same precondition: the creative must already be approved and ingested.

## Webhooks and Polling Cadence

The Ad Management API recommends webhooks to reduce polling and periodic polling to catch missed webhooks. For live events:

- Enable webhooks where the supply partner offers them. A denial or `5` (changed) delivered by webhook during an event should stop bidding with that creative immediately.
- Poll `GET .../ads?auditStart={lastobserved}` at the normal low cadence day to day, and at the T-72h, T-24h and T-1h checkpoints in the timeline above regardless of webhook delivery.
- Respect the supply partner's rate limit (Integration Checklist). Reconciliation polls are cheap because `auditStart` returns only changes.
- Keep every response. Audit history is part of the post-event review.

## Expiration and Re-activation Before an Event

Supply partners may expire ads after a period of inactivity (the Ad Management API's example policy is 14 to 30 days). A creative approved for a season may be expired by the time of a late-season event. Demand partners should check for `6` (expired) in the T-7 reconciliation and re-activate by touching the ad; whether re-activation triggers a fresh audit is supply-partner policy, so allow for it in the timeline.

## Intermediaries

Where an SSP or other intermediary sits between the demand partner and the publisher's insertion stack, the Ad Management API's generic roles apply twice: the intermediary is the supply partner to the DSP and the demand partner to the publisher. Intermediaries should pass submissions through and return the publisher's audit status for publisher-audited creatives rather than substituting their own, and should preserve the demand partner's creative `id` so that it survives to the bid response and to delivery reporting. Publishers who audit at their own level should say so in the Integration Checklist so demand partners know whose approval governs a live break.

## Associating Creatives with Events (Proposed Extension)

The Ad Management API has no concept of an event: an `Ad` is approved for a supply partner, not for a broadcast. For live events it is useful for the demand partner to say which event(s) a submission is intended for, so that the supply partner can prioritize its audit queue and apply event-specific creative rules, and for the supply partner to say which event(s) an approval covers.

**This guide proposes, for Work Stream 2 discussion, an extension and does not define it.** An illustrative shape, reusing the LEAP shared identifiers:

```json
"ext": {
  "leap": {
    "contentids": ["event-content-12345"],
    "cids": [{ "name": "id-provider.example", "cids": ["shared-content-id-12345"] }]
  }
}
```

Placed on the `Ad` by the demand partner it means "intended for these events"; echoed by the supply partner in `audit.ext` it could mean "approved for these events." The identifiers are the same `content.id` / source-qualified Extended Content IDs that the Forecast API, Concurrent Streams API and OpenRTB already share, so no new identifier namespace is introduced. Whether this belongs in `ext`, in the Ad Management API itself, or in the Creative Readiness proposal is for the working group.

## Cross-API Identity Guidance

Two joins matter here, and they are different from the event join described in the Common API Guidance:

1. **Creative to bidstream.** The Ad Management API `Ad.id` (when bidder IDs are used) should equal the `crid` the demand partner returns in bid responses, so that approvals, bids and delivery reports reconcile on one creative identifier. Where the supply partner assigns its own ad IDs, the demand partner should store the mapping in the `Ad`'s `ext` as the Ad Management API suggests.
2. **Creative to event.** There is no standard join today. Operationally, the event is identified by `content.id` or a source-qualified Extended Content ID in the bid request; the creative is identified by `crid` in the bid response. The proposed extension above would let the association be stated ahead of the event rather than inferred from delivery.

## Example Exchange

Illustrative only. Values are made up and follow the walkthrough's championship event (`content.id` `event-content-12345`, `scheduledstart` Nov 8, 01:00 UTC). Supply partner policy in this example: restrictive bidding, markup in bid response, bidder IDs of record, 30-second maximum creative duration for this event, SSAI with transcoding on approval.

**T-7 days — submission** (one of three creatives submitted the same way)

```http
POST /v1/bidder/492/ads
Authorization: Bearer {access_token}
Content-Type: application/json
```

```json
{
  "id": "crt-30s-00417",
  "adomain": ["advertiser.example"],
  "cat": ["IAB2"],
  "secure": 1,
  "video": {
    "mime": ["video/mp4"],
    "ctype": 13,
    "dur": 30,
    "curl": "https://cdn.dsp.example/vast/crt-30s-00417.xml"
  },
  "ext": { "leap": { "contentids": ["event-content-12345"] } }
}
```

Response (`200 OK`):

```json
{
  "count": 1,
  "ads": [
    { "id": "crt-30s-00417", "init": 1793496600000, "lastmod": 1793496600000,
      "audit": { "status": 1, "lastmod": 1793496600000 } }
  ]
}
```

**T-24 hours — status** (webhook received earlier; confirming poll)

```http
GET /v1/bidder/492/ads?auditStart=1793496600000
Authorization: Bearer {access_token}
Accept: application/json
```

```json
{
  "count": 3,
  "more": 0,
  "ads": [
    { "id": "crt-30s-00417", "lastmod": 1793496600000,
      "audit": { "status": 3, "feedback": ["Category corrected to Automotive. Transcoded for SSAI; ready to insert."],
                 "corr": { "cat": ["IAB2-3"] }, "lastmod": 1793720400000 } },
    { "id": "crt-15s-00418", "lastmod": 1793496700000,
      "audit": { "status": 3, "feedback": ["Transcoded for SSAI; ready to insert."], "lastmod": 1793720460000 } },
    { "id": "crt-60s-00419", "lastmod": 1793496800000,
      "audit": { "status": 4, "feedback": ["Duration 60s exceeds the 30s maximum for this event's ad pods."], "lastmod": 1793585000000 } }
  ]
}
```

The demand partner stores the corrected `cat` for `crt-30s-00417`, removes `crt-60s-00419` from line items targeting the event (or replaces it with a 30-second cut and resubmits), and enters the live window with two eligible creatives.

## Notes on Common API Guide Application

- The Ad Management API defines its own conventions (status codes, sparse fieldsets, `auditStart`/`paginationId` pagination, Unix-millisecond timestamps, major-version-only base URL). Where they differ from the LEAP common guide, the Ad Management API specification governs for this API.
- Authentication and webhook registration are left to the supply partner by the Ad Management API, as the LEAP common guide also does for its APIs. OAuth 2.0 is a reasonable default for both.
- Field names in the Ad Management API follow AdCOM (camel-case `nextPage`, `paginationId`, `auditStart`) rather than LEAP's all-lowercase convention. Do not normalize them.

## Open Questions for Work Stream 2

1. Should "approved and ingested" become a standard audit status or an `audit.ext` field in the Ad Management API revision under discussion, rather than a vendor-specific `500+` code?
2. Should the Forecast API define creative constraints (`maxadduration`, accepted durations, VAST versions, lead time) in `inventoryconfig`, given that its own sample already carries `maxadduration`?
3. Is an event association on the `Ad` (the `ext.leap` sketch above) useful enough to standardize, and does it belong in the Ad Management API or in the Creative Readiness proposal?
4. For publishers that audit at their own level while an SSP operates the Ad Management API endpoint, how should the two audit outcomes be represented to the demand partner?
5. For a later release: the Forecast API (per event) and the Deal Sync API (per deal) should carry a structured creative-requirements object (codec/container, resolution, aspect ratio, frame rate, video/audio bitrates, audio channels/loudness, file size, durations, VAST version), defined once in AdCOM and referenced by both, so that DSPs can detect buyer-asset mismatches at planning time rather than at audit or render. Which working group owns the AdCOM definition (Programmatic Supply Chain, with LEAP as the first consumer), and what is the minimum attribute set for a v1?
