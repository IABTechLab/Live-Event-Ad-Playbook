![IAB Tech Lab](https://drive.google.com/uc?id=10yoBoG5uRETSXRrnJPUDuONujvADrSG1)

# Live Event Ad Playbook (LEAP)
# Concurrent Streams API Implementation Guide

## Table of Contents

- [Status of this Guide](#status-of-this-guide)
- [Purpose](#purpose)
- [Relationship to Forecast API](#relationship-to-forecast-api)
- [Supported Interaction Model](#supported-interaction-model)
- [Supported HTTP Methods](#supported-http-methods)
- [Request Object](#request-object)
- [Response Object](#response-object)
- [StreamsData Object](#streamsdata-object)
- [MediaStreams Object](#mediastreams-object)
- [StreamCount Object](#streamcount-object)
- [Content Identification](#content-identification)
- [Regional Interpretation](#regional-interpretation)
- [SSAI, SGAI, and CSAI Interpretation](#ssai-sgai-and-csai-interpretation)
- [Snapshot Freshness](#snapshot-freshness)
- [Caching Guidance](#caching-guidance)
- [Operational Guidance for Consumers](#operational-guidance-for-consumers)
- [Example Response](#example-response)
- [Notes on Common API Guide Application](#notes-on-common-api-guide-application)

## Status of this Guide

This document is an API-specific implementation guide for the LEAP Concurrent Streams API. It is designed to be used together with the LEAP Common API Conventions Reference.

The common guide applies unless this document provides API-specific guidance or an API-specific override.

## Purpose

Concurrent Streams API provides a standardized way for authorized subscribers to retrieve near-real-time viewership information for live streams while live events are occurring.

The API is intended to help advertising systems:

- understand current live event scale
- prepare for and respond to QPS spikes
- scale regional infrastructure
- adjust bidding and pacing behavior
- reduce latency, slate, and lost monetization opportunities


## Relationship to Forecast API

Forecast API is used before the event for planning.

Concurrent Streams API is used during the event for live adjustment.

Once an event is live, Concurrent Streams API should be treated as the more relevant signal for current concurrency.


## Supported Interaction Model

Concurrent Streams API is a retrieval API. A subscriber calls the Streams Data Provider endpoint and receives a snapshot of live stream counts for events the subscriber is authorized to access.

Typical request flow:

1. Streams Data Provider provisions an authorized subscriber.
2. Subscriber calls the Concurrent Streams API endpoint during a live event.
3. Provider returns a timestamped snapshot of live event stream counts.
4. Subscriber uses the snapshot for operational scaling, pacing, and bidding decisions.


## Supported HTTP Methods

The primary method is:

```http
GET /concurrentstreams
```

The common guide describes general HTTP method semantics. This API-specific guide does not define consumer-facing create, update, patch, or delete workflows for stream-count resources.


## Request Object

A Concurrent Streams request may include:

| Field | Type | Guidance |
|---|---|---|
| `version` | string | API version, such as `1.0.0`. |
| `requestor` | string | Authorized party requesting information. Recommended unless anonymous access is allowed. |
| `sdp` | string | Streams Data Provider filter. Recommended when one endpoint provides data from multiple providers. |


## Response Object

A Concurrent Streams response includes:

| Field | Type | Guidance |
|---|---|---|
| `version` | string | API version, such as `1.0.0`. |
| `timestamp` | integer | Snapshot generation time. The API specification describes this as a Unix timestamp in milliseconds. |
| `streamsdata` | array of objects | Stream data requested by the caller. |


## StreamsData Object

| Field | Type | Guidance |
|---|---|---|
| `sdp` | string | Streams Data Provider. |
| `mediastreams` | array of objects | Snapshot data for live media streams. |


## MediaStreams Object

| Field | Type | Guidance |
|---|---|---|
| `content` | object | AdCOM Content object used to identify the live event. |
| `eventstart` | integer | Event start time. The API specification describes this as Unix timestamp in milliseconds. |
| `eventend` | integer | Event end time. The API specification describes this as Unix timestamp in milliseconds. |
| `streamcount` | array of objects | Regional SSAI and/or CSAI stream counts. |


## StreamCount Object

| Field | Type | Guidance |
|---|---|---|
| `region` | integer | Region code representing traffic distribution geography. |
| `sstreams` | integer | Concurrent SSAI streams. At least one of `sstreams` or `cstreams` should be present. |
| `cstreams` | integer | Concurrent CSAI streams. At least one of `sstreams` or `cstreams` should be present. |


## Content Identification

Concurrent Streams API consumers need to associate current stream counts with the same event and content referenced in planning, bidding, and reporting systems.

Providers should pass at least one stable content identifier whenever possible.

Recommended identity hierarchy:

1. `content.id`
2. `content.data.cids`
3. event timing plus stable content metadata as a fallback

When possible, the same content identifier should be present in OpenRTB bid requests and other downstream transaction or reporting systems.


## Regional Interpretation

The `region` field should represent coarse traffic distribution areas useful for infrastructure and demand-system scaling.

Providers and subscribers should agree on the region-code mapping before production use.

Examples of useful regional grouping include:

- broad datacenter region
- broad geographic audience region
- provider-defined traffic distribution region

## SSAI, SGAI, and CSAI Interpretation

The `sstreams` field represents concurrent server-side ad insertion streams. Server-guided ad insertion workflows should be treated as SSAI workflows for the purpose of this API.

The `cstreams` field represents concurrent client-side ad insertion streams.

At least one of `sstreams` or `cstreams` should be present for each stream-count entry.


## Snapshot Freshness

The response `timestamp` indicates when the stream-count snapshot was generated.

Consumers should compare the response `timestamp` to local receipt time and treat old snapshots as stale.

Suggested starting guidance for working group review:

| Event State | Possible Polling Frequency | Suggested Staleness Threshold |
|---|---|---|
| Pre-event warmup | Every 5 minutes | 10 minutes |
| Live event | Every 30-60 seconds | 2 minutes |
| Peak moments | Every 15-30 seconds | 60 seconds |
| Post-event wind-down | Every 5 minutes | 10 minutes |

Providers and consumers should tune these values based on provider capacity, event scale, and operational need.

## Caching Guidance

Concurrent Streams data is operationally time-sensitive.

Consumers should not rely on aggressively cached responses during the live event. Short-lived caching may be appropriate for provider protection, but cache TTLs should be aligned with the expected polling interval and staleness threshold.

## Operational Guidance for Consumers

Consumers should use Concurrent Streams signals to:

- scale QPS capacity
- adjust bid throttles
- modify pacing assumptions
- avoid rejecting live-event bid opportunities due to capacity limits
- detect spikes or declines in number of streams
- protect user experience by reducing latency-sensitive failures

## Example Response

```json
{
  "version": "1.0.0",
  "timestamp": 1752007200000,
  "streamsdata": [
    {
      "sdp": "provider-1",
      "mediastreams": [
        {
          "content": {
            "id": "event-content-12345",
            "title": "Championship Event",
            "series": "Generic Sports Series",
            "data": {
              "name": "id-provider.example",
              "cids": ["shared-content-id-12345"]
            }
          },
          "eventstart": 1752000000000,
          "eventend": 1752014400000,
          "streamcount": [
            {
              "region": 1,
              "sstreams": 900000,
              "cstreams": 300000
            },
            {
              "region": 2,
              "sstreams": 700000,
              "cstreams": 250000
            }
          ]
        }
      ]
    }
  ]
}
```

## Notes on Common API Guide Application

- Use the common guide for authentication, error handling, status codes, rate limiting, and versioning unless overridden by the Concurrent Streams API specification.
- The response root for this API is `streamsdata`, not a generic `data` array.
- Timestamp fields follow the Concurrent Streams API specification.
- Pagination is not required for the basic response model where the provider returns all live events the subscriber can access at the time of request. If pagination is added by an implementation, follow the common guide.
