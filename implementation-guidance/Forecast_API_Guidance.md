![IAB Tech Lab](https://drive.google.com/uc?id=10yoBoG5uRETSXRrnJPUDuONujvADrSG1)
# Live Event Ad Playbook (LEAP)
# Forecast API Implementation Guide

## Table of Contents

- [Status of this Guide](#status-of-this-guide)
- [Purpose](#purpose)
- [Relationship to Concurrent Streams API](#relationship-to-concurrent-streams-api)
- [Supported Interaction Model](#supported-interaction-model)
- [Supported HTTP Methods](#supported-http-methods)
- [Request Object](#request-object)
- [Recommended Query Parameters](#recommended-query-parameters)
- [Response Object](#response-object)
- [UpcomingEvent Object](#upcomingevent-object)
- [StreamsData Object](#streamsdata-object)
- [InventoryConfig Object](#inventoryconfig-object)
- [Supported Media Type Values](#supported-media-type-values)
- [Forecast Uncertainty](#forecast-uncertainty)
- [Tentative Events](#tentative-events)
- [Planned vs. Unplanned Inventory](#planned-vs-unplanned-inventory)
- [Cross-API Identity Guidance](#cross-api-identity-guidance)
- [Example Response](#example-response)
- [Notes on Common API Guide Application](#notes-on-common-api-guide-application)

## Status of this Guide

This document is an API-specific implementation guide for the LEAP Forecast API. It is designed to be used together with the LEAP Common API Guidance.

**File:** `Common_API_Guidance.md`

The common guide applies unless this document provides API-specific guidance or an API-specific override.

## Purpose

Forecast API provides a standardized way for authorized partners to retrieve information about future live event advertising supply.

The API is intended to help advertising systems:

- discover future live events
- estimate viewership before the event
- estimate advertising inventory availability
- support infrastructure planning
- inform deal planning and packaging
- support campaign planning before the event begins

## Relationship to Concurrent Streams API

Forecast API is used before the event.

Concurrent Streams API is used during the event.

Once the event begins, Forecast API should no longer be treated as the authoritative signal for current live concurrency. Concurrent Streams API should be used for current stream-count signals during live playback.

Forecast API and Concurrent Streams API should use stable identifiers that allow consumers to correlate the forecasted event with live stream-count records and downstream OpenRTB transactions.

## Supported Interaction Model

Forecast API is a provider-to-authorized-partner retrieval API.

Typical flow:

1. Forecast Data Provider publishes an endpoint containing upcoming event forecast data.
2. Forecast Data Provider provisions authorized partners.
3. Authorized partners call the endpoint on an agreed cadence.
4. Forecast Data Provider returns forecast data for upcoming events the partner is authorized to access.

Forecast API is not intended to define consumer-facing forecast creation, update, patch, or delete workflows.

## Supported HTTP Methods

The primary method is:

```http
GET /eventforecasts
```

Implementations may also support retrieval of a specific forecast resource:

```http
GET /eventforecasts/{id}
```

The common guide describes general HTTP method semantics. This API-specific guide does not define consumer-facing create, update, patch, or delete workflows for forecast resources.


## Request Object

A Forecast API request may include:

| Field | Type | Required | Guidance |
|---|---|---:|---|
| `version` | string | Yes | API version, such as `1.0.0`. |
| `requestor` | string | Recommended | Authorized party requesting information. Recommended unless anonymous access is allowed. |
| `fdp` | string | Optional | Forecast Data Provider filter. Recommended when one endpoint provides data from multiple providers. |

## Recommended Query Parameters

Implementations may support query parameters to reduce payload size and improve retrieval efficiency.

Consumers should be familiar with the Forecast API specification before relying on query parameters. This guide explains common retrieval patterns, but the API specification remains the source of truth for supported fields, required fields, and response objects.

| Parameter | Required | Guidance |
|---|---:|---|
| `scheduledstart` | Yes | Return events scheduled on or after the specified timestamp. |
| `scheduledend` | Yes | Return events scheduled on or before the specified timestamp. |
| `lastmodifieddate` | Optional | Return events modified on or after the specified timestamp. |
| `eventstatus` | Optional | Return events matching a specific event status value. |
| `id` | Optional | Filter to a specific Forecast API event record identifier, if supported. This is the event-level `id` field and is distinct from `content.id` nested in the `content` object. |
| `fdp` | Optional | Filter to a specific Forecast Data Provider where applicable. |
| `limit` | Optional | Pagination limit, if supported. |
| `offset` | Optional | Pagination offset, if supported. |
| `sort` | Optional | Sort field, if supported. |
| `order` | Optional | Sort direction, if supported. |

### Discovery and Filtering Sequence

A typical consumer workflow is:

1. Call `GET /eventforecasts` with `scheduledstart` and `scheduledend` to discover events within a time window.
2. Store the event-level `id`, `content.id`, and any identifiers in `content.data.cids` returned in the response.
3. Use `GET /eventforecasts/{id}` or the `id` query parameter, where supported, to retrieve a specific forecast record later.
4. Use `lastmodifieddate` to retrieve only records that have materially changed since the previous request.

Consumers should not assume that the event-level `id` and `content.id` are always identical. The event-level `id` identifies the forecast event record. The nested `content.id` identifies the content or event identity used for cross-system correlation where available.

## Response Object

A Forecast API response includes:

| Field | Type | Required | Guidance |
|---|---|---:|---|
| `version` | string | Yes | API version, such as `1.0.0`. |
| `timestamp` | integer | Yes | Snapshot generation time. The API specification describes this as a Unix timestamp in milliseconds. |
| `events` | array of objects | Yes | Event forecast records requested by the caller. |

## UpcomingEvent Object

| Field | Type | Guidance |
|---|---|---|
| `id` | string | Publisher-provided event record identifier. Should match `content.id` where possible, but consumers should not assume these values are always identical. |
| `scheduledstart` | integer | Scheduled event start time as Unix timestamp in milliseconds. |
| `scheduledend` | integer | Scheduled event end time as Unix timestamp in milliseconds. |
| `flexibleend` | integer | `0` = fixed, `1` = flexible. |
| `content` | object | AdCOM Content object. |
| `eventstatus` | integer | `1` = scheduled, `2` = tentative, `3` = cancelled. |
| `streamsdata` | array of objects | Forecast stream-count data. |
| `inventoryconfig` | object | Expected advertising inventory configuration. |
| `lastmodifieddate` | integer | Most recent material update time as Unix timestamp in milliseconds. |
| `ext` | object | Optional extensions. |

### Event Timing and Status Interpretation

The Forecast API `eventstatus` values indicate whether a future event is scheduled, tentative, or cancelled. The Forecast API does not define live or completed event status values.

Consumers should use `scheduledstart`, `scheduledend`, and the current time to infer whether a forecasted event may already be live or expired. Once an event is live, consumers should transition to Concurrent Streams API for current stream-count signals. If an event has passed its scheduled end time, consumers should not assume the Forecast API remains the authoritative source for current event state.

For events with `flexibleend = 1`, consumers should expect the event end time to be approximate and should allow additional operational tolerance before treating the event as expired.

## StreamsData Object

| Field | Type | Guidance |
|---|---|---|
| `country` | string | ISO-3166-1-alpha-3 country code. |
| `expectedpeak` | integer | Expected peak streams for the event in the listed country. |
| `lowerbound` | integer | Estimated lower bound of expected streams for the event in the listed country. |
| `ext` | object | Optional extensions. |

`streamsdata` represents forecast stream-count data for the event. The same terminology should be used consistently when describing forecasted stream counts in Forecast API guidance and current stream counts in Concurrent Streams API guidance.

## InventoryConfig Object

| Field | Type | Guidance |
|---|---|---|
| `supportedmtype` | array of integers | Accepted ad creative formats. |
| `totaladdurationsec` | integer | Expected total duration of all ad pods, in seconds. |
| `expectedpodcount` | integer | Estimated number of ad pods during the event. |
| `unplanned` | integer | `0` = no, `1` = yes. Indicates whether event inventory may contain unplanned ad breaks. |
| `ext` | object | Optional extensions. |

## Supported Media Type Values

| Value | Meaning |
|---:|---|
| `1` | Banner |
| `2` | Video |
| `3` | Audio |
| `4` | Native |

## Forecast Uncertainty

Forecast values are expected to change as the event approaches.

Consumers should treat forecast data as probabilistic, not guaranteed. Forecasts that are farther away from the event are expected to be less precise than forecasts closer to event time.

Suggested planning interpretation:

| Time Before Event | Forecast Stability | Consumer Interpretation |
|---|---|---|
| 30+ days | Low | Use for early planning and budget allocation. |
| 14 days | Medium-low | Use for preliminary deal and infrastructure planning. |
| 7 days | Medium | Refine pacing and QPS assumptions. |
| 24 hours | Higher | Finalize pre-event operational plans. |
| Event start | Superseded for live concurrency | Use Concurrent Streams API for current stream counts. |

### Ad Break and In-Event Variation

Forecast API fields such as `expectedpeak` and `lowerbound` describe expected stream counts at the event and market level represented by the forecast record. They should not be interpreted as guaranteed stream counts for every ad break within the event.

Live events may have different concurrency levels at different moments, including pre-event, intermission, final segment, overtime, or post-event coverage. If an implementation needs to communicate forecasted differences by ad break, pod, or event phase, that detail should be represented only where supported by the Forecast API specification or through an agreed extension in `ext`.

Once the event is live, consumers should use Concurrent Streams API for current stream-count signals rather than relying on Forecast API values to model ad-break-level concurrency.

## Tentative Events

Use `eventstatus = 2` when the event may not occur.

Consumers should avoid treating tentative events as guaranteed inventory. Tentative events may still be useful for planning conditional budgets, inventory packages, and infrastructure capacity.

## Planned vs. Unplanned Inventory

The `unplanned` field helps consumers understand whether ad inventory is expected to follow a fixed break schedule or depend on live-event conditions.

Use `unplanned = 1` when any material portion of inventory may be triggered dynamically or conditionally.

Consumers should interpret `unplanned = 1` as a signal that:

- pod timing may not be fixed
- pod count may vary
- total ad duration may vary
- delivery may be uneven across the event
- pacing assumptions should include flexibility

## Cross-API Identity Guidance

Forecast records should include identifiers that allow consumers to match forecasted events to live stream-count records and OpenRTB bid requests.

Recommended identity hierarchy:

1. Use `content.id` as the preferred stable content/event linkage identifier where possible.
2. Use `content.data.cids` for shared or third-party identifiers where available.
3. Align the Forecast API event `id` with `content.id` where possible.
4. Avoid relying only on title and scheduled time unless no stable identifier exists.

Consumers should retain the Forecast API event `id`, `content.id`, and `content.data.cids` values so they can correlate pre-event forecast records with live Concurrent Streams API records, OpenRTB bid requests, deal planning systems, and reporting systems.

## Example Response

```json
{
  "version": "1.0.0",
  "timestamp": 1751205600000,
  "events": [
    {
      "id": "event-content-12345",
      "scheduledstart": 1752000000000,
      "scheduledend": 1752014400000,
      "flexibleend": 1,
      "eventstatus": 1,
      "content": {
        "id": "event-content-12345",
        "title": "Championship Event",
        "series": "Generic Sports Series",
        "data": {
          "name": "id-provider.example",
          "cids": ["shared-content-id-12345"]
        }
      },
      "streamsdata": [
        {
          "country": "USA",
          "expectedpeak": 1800000,
          "lowerbound": 1200000
        }
      ],
      "inventoryconfig": {
        "supportedmtype": [2],
        "totaladdurationsec": 1200,
        "expectedpodcount": 10,
        "unplanned": 1
      },
      "lastmodifieddate": 1751202000000
    }
  ]
}
```

## Notes on Common API Guide Application

- Use the common guide for authentication, error handling, status codes, rate limiting, versioning, and sorting unless overridden by the Forecast API specification.
- The response root for this API is `events`, not a generic `data` array.
- Timestamp fields follow the Forecast API specification.
- Pagination may be useful for large forecast collections. If used, follow the common guide unless the Forecast API specification defines different behavior.
- The common guide describes general HTTP method semantics. Forecast API-specific consumer-facing guidance is limited to retrieval workflows unless the Forecast API specification defines additional methods.
- API-specific field names in the Forecast API specification take precedence over generic naming examples in the common guide.
