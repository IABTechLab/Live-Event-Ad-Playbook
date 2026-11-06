![IAB Tech Lab](https://drive.google.com/uc?id=10yoBoG5uRETSXRrnJPUDuONujvADrSG1)
# Live Event Ad Playbook (LEAP)
# Forecast API Specification

# About the IAB Tech Lab <a name="techlab"></a>
The IAB Technology Laboratory is a nonprofit research and development consortium charged with producing and helping companies implement global industry technical standards and solutions. The goal of the Tech Lab is to reduce friction associated with the digital advertising and marketing supply chain while contributing to the safe growth of an industry. The IAB Tech Lab spearheads the development of technical standards, creates and maintains a code library to assist in rapid, cost-effective implementation of IAB standards, and establishes a test platform for companies to evaluate the compatibility of their technology solutions with IAB standards, which for 18 years have been the foundation for interoperability and profitable growth in the digital advertising supply chain. Further details about the IAB Technology Lab can be found at www.iabtechlab.com

### IAB Tech Lab Lead:
Hillary Slattery, Sr. Director of Product, IAB Tech Lab

### License <a name="license"></a>

This specification is licensed under a Creative Commons Attribution 3.0 License. To view a copy of this license, visit creativecommons.org/licenses/by/3.0/ http://creativecommons.org/licenses/by/3.0/ or write to Creative Commons, 171 Second Street, Suite 300, San Francisco, CA 94105, USA.

### Disclaimer <a name="disclaimer"></a>

THE STANDARDS, THE SPECIFICATIONS, THE MEASUREMENT GUIDELINES, AND ANY OTHER MATERIALS OR SERVICES PROVIDED TO OR USED BY YOU HEREUNDER (THE “PRODUCTS AND SERVICES”) ARE PROVIDED “AS IS” AND “AS AVAILABLE,” AND IAB TECHNOLOGY LABORATORY, INC. (“TECH LAB”) MAKES NO WARRANTY WITH RESPECT TO THE SAME AND HEREBY DISCLAIMS ANY AND ALL EXPRESS, IMPLIED, OR STATUTORY WARRANTIES, INCLUDING, WITHOUT LIMITATION, ANY WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, AVAILABILITY, ERROR-FREE OR UNINTERRUPTED OPERATION, AND ANY WARRANTIES ARISING FROM A COURSE OF DEALING, COURSE OF PERFORMANCE, OR USAGE OF TRADE. TO THE EXTENT THAT TECH LAB MAY NOT AS A MATTER OF APPLICABLE LAW DISCLAIM ANY IMPLIED WARRANTY, THE SCOPE AND DURATION OF SUCH WARRANTY WILL BE THE MINIMUM PERMITTED UNDER SUCH LAW. THE PRODUCTS AND SERVICES DO NOT CONSTITUTE BUSINESS OR LEGAL ADVICE. TECH LAB DOES NOT WARRANT THAT THE PRODUCTS AND SERVICES PROVIDED TO OR USED BY YOU HEREUNDER SHALL CAUSE YOU AND/OR YOUR PRODUCTS OR SERVICES TO BE IN COMPLIANCE WITH ANY APPLICABLE LAWS, REGULATIONS, OR SELF-REGULATORY FRAMEWORKS, AND YOU ARE SOLELY RESPONSIBLE FOR COMPLIANCE WITH THE SAME, INCLUDING, BUT NOT LIMITED TO, DATA PROTECTION LAWS, SUCH AS THE PERSONAL INFORMATION PROTECTION AND ELECTRONIC DOCUMENTS ACT (CANADA), THE DATA PROTECTION DIRECTIVE (EU), THE E-PRIVACY DIRECTIVE (EU), THE GENERAL DATA PROTECTION REGULATION (EU), AND THE E-PRIVACY REGULATION (EU) AS AND WHEN THEY BECOME EFFECTIVE.
# Forecast API

## Executive Summary

Today’s programmatic systems lack standardized mechanisms for surfacing future live event supply, limiting buyers’ ability to plan and reserve inventory in advance. To address this, we propose an API that enables broadcasters and content owners to publish structured metadata—such as event schedules, expected ad breaks, and audience forecasts—to exchanges and DSPs. Using a shared identifier and a schema aligned with OpenRTB and AdCOM standards, this API will make upcoming inventory discoverable and actionable, enabling a scalable advance market for live streaming advertising.

## Table of Contents

- [Introduction and Background](#intro)
- [Goals](#goals)
- [Scope](#scope)
- [Functional Requirements](#functional-requirements)
  - [Inventory Owner Endpoint](#inventory-owner-endpoint)
  - [Object: ForecastRequest](#forecastrequest)
  - [Object: UpcomingEvent](#upcomingevent)
  - [Object: Content](#object-content)
  - [Object: StreamsData](#object-streamsdata)
  - [Object: Programmatic](#object-programmatic)
  - [Object: AdInventoryConfiguration](#object-adinventoryconfiguration)
  - [Partner Endpoint](#partner-endpoint)
  - [Object: Response](#object-response)
- [Implementation Guidance](#implementation-guidance)
- [Uncertainty](#uncertainty)
  - [Using Tentative vs Scheduled](#using-tentative-vs-scheduled)
  - [Static vs Variable Ad Inventory](#static-vs-variable-ad-inventory)
  - [Direct-Sold vs Programmatic Supply](#direct-sold-vs-programmatic-supply)
- [JSON Examples](#json-examples)
  - [Sample: Sports Game](#sample-sports-game)
  - [Sample: Sports Event Series with Conditional Schedule](#sample-sports-event-series-with-conditional-schedule)
  - [Sample: Political Event (Debate)](#sample-political-event-debate)
  - [Sample: Political Event (Election Night News Coverage)](#sample-political-event-election-night-news-coverage)

## Introduction and Background <a name="intro"></a>

We want to enable both a “spot” market for streaming live events, as well as an “advance” market. In order to enable an advance market, buyers need to become aware of upcoming supply including but not limited to live events that will be available at some future point. To enable that, sellers must be able to notify buyers that a given program will be streamed, with anticipated parameters including start/end time, expected streams, and expected inventory configuration.

The proposed approach is to standardize a messaging format for syncing a Live Events Manifest, with a common identifier for any given event, which buyers and sellers can use to communicate about it. Broadcasters/streamers might push their manifest into the exchanges that are authorized to sell this supply via this API. Exchanges might subsequently push these manifests into their integrated DSPs (or perhaps broadcasters/streamers would do it) using the same standardized API.

### Goals <a name="goals"></a>

- Forecast is shared to facilitate pre-planning of both “spot” market and “advance”.

### Non Goals

- Realtime scaling use cases that require Real Time Active user volume estimates are covered under “Concurrent Streams API” scope.
- While meant to help inform what content goes into a given deal negotiation, the specifics of those deals should be communicated through the Deal API.

## In Scope <a name="scope"></a>

- This API addresses only future events. Once the start time happens, this API should not be used for that event.
- Ensure advertising systems and partners in the monetization supply chain have concurrency estimates that reflect expected traffic volume of upcoming events so they can plan infrastructure scaling to manage incoming QPS spikes for upcoming events.
- Provide information to help inform deal negotiations.

## Out of Scope

It is assumed that Inventory Owners standing up this endpoint will create a security model to ensure information is only available to parties where there is an existing agreement. The authentication protocol is left to the discretion of the event creator, and should be discussed with API users a priori.

## Functional Requirements

### Inventory Owner Endpoint

#### Object: ForecastRequest <a name="forecastrequest"></a>

| Attribute | Type | Description |
|---|---|---|
| version | string, required) | E.g., `1.0.0` |
| requestor | string, recommended) | Authorized party requesting information from the endpoint. Not required if the endpoint allows anonymous requests. |
| fdp | string | Forecast Data Provider. Typically the content owner or distributor for which live event schedule & forecast data is being requested. <br><br> Recommended only when the endpoint provides data from multiple providers, and the requestor needs to filter to specific sources. |

#### Object: UpcomingEvent <a name="upcomingevent"></a>

| Attribute | Type | Description |
|---|---|---|
| id | string, recommended | Publisher-provided ID uniquely identifying the event record. This should be the same value as `content.id`, if possible. If no ID has been assigned, implementers should leave blank. |
| scheduledstart | int, required | Date/time for when the event is scheduled to start (Unix timestamp in milliseconds). |
| scheduledend | int, required | Date/time for when the event is scheduled to end (Unix timestamp in milliseconds). |
| flexibleend | int, default 0 | Delineates if the `scheduledend` of the event is fixed or flexible where <br><br>`0 = fixed`<br>`1 = flexible` |
| content | object, recommended) | Description of content using the [AdCOM 1.0 Content Object](https://github.com/InteractiveAdvertisingBureau/AdCOM/blob/main/AdCOM%20v1.0%20FINAL.md#object--content-) Object. |
| eventstatus | int, recommended, default: 1) | Status of the event where <br><br> `1 = scheduled`<br>`2 = tentative`<br>`3 = cancelled` |
| streamsdata | array(object), recommended) | Streams (or viewership) data for the event. <br><br>See <a href="#streamsdata">Object: StreamsData</a> for additional information. |
| inventoryconfig | object | Information about the ad inventory in the event. <br><br>See <a href="#adinventoryconfig"> Object: AdInventoryConfiguration</a>. |
| lastmodifieddate | int | Most recent date/time when this resource is updated (Unix timestamp in milliseconds). |
| ext | object | Optional extensions. |

#### Object: Content <a name="content"></a>

Refer to the [AdCOM 1.0 Content Object](https://github.com/InteractiveAdvertisingBureau/AdCOM/blob/main/AdCOM%20v1.0%20FINAL.md#object--content-) for specific values.

#### Object: StreamsData <a name="streamsdata"></a>

| Attribute | Type | Description |
|---|---|---|
| country | string | Country code where expected viewership would happen using ISO-3166-1-alpha-3. |
| expectedpeak | int | Expected peak streams for this event in the country listed in the `country` attribute. <br><br>Required if passing this object. |
| lowerbound | int | Estimated lowest number of streams for this event in the country listed in the `country` attribute. |
| programmatic | object | Forecast of the portion of ad supply for this event, in the country listed in the `country` attribute, that is expected to be available for programmatic fulfillment (i.e., not committed to direct-sold or other non-programmatic channels). <br><br>See <a href="#programmatic">Object: Programmatic</a>. |
| ext | object | Optional extensions. |

#### Object: Programmatic <a name="programmatic"></a>

Viewership alone does not indicate how much inventory a programmatic buyer can actually transact against, since some or all of an event's ad load may be sold direct. This object lets the Forecast Data Provider express the expected programmatically available supply. Sellers should populate whichever measures they are able to forecast; at least one of `share`, `expectedimps`, or `addurationsec` should be present when this object is included.

| Attribute | Type | Description |
|---|---|---|
| scope | int, required | Perspective from which the values in this object are expressed, where <br><br>`1 = market` – all supply for this event/country not committed to direct-sold or other non-programmatic channels, regardless of which programmatic partners may access it<br>`2 = requestor` – only the supply accessible to the authenticated `requestor` (e.g., via existing agreements or deals) |
| share | float | Fraction of the event's total ad load in this country expected to be available programmatically, expressed as a value from `0.0` to `1.0`. E.g., `0.35` indicates 35% of ad load is expected to be programmatic and 65% is committed to other channels. |
| expectedimps | int | Expected number of programmatically available ad impressions for this event in this country. |
| lowerbound | int | Estimated lowest number of programmatically available ad impressions. |
| upperbound | int | Estimated highest number of programmatically available ad impressions. |
| addurationsec | int | Expected programmatically available ad duration per stream (in seconds). This is a subset of `totaladdurationsec` in the <a href="#adinventoryconfig">AdInventoryConfiguration</a> object. |
| ext | object | Optional extensions. |

#### Object: AdInventoryConfiguration <a name="adinvnetoryconfig"></a>

| Attribute | Type | Description |
|---|---|---|
| supportedmtype | array(int) | Accepted ad creative formats where <br><br>`1 = Banner`<br>`2 = Video`<br>`3 = Audio`<br>`4 = Native` |
| totaladdurationsec | int | Expected total length of all ad pods (in seconds). |
| expectedpodcount | int | Estimated number of ad pods throughout the event. |
| unplanned | int | Event may contain unplanned ad breaks where <br><br>`0 = No`<br>`1 = Yes`. <br><br>See <a href="#uncertanty">Implementation Guidance</a> for additional detail. |
| ext | object | Optional extensions. |

### Partner Endpoint

#### Object: Response <a name="response"></a>

| Attribute | Type | Description |
|---|---|---|
| version | string, required) | E.g., `1.0.0` |
| timestamp | int, required) | Date/time of the snapshot (Unix timestamp in milliseconds) |
| events | array(object), required) | Events data requested by the caller |

## Implementation Guidance <a name="impguidance"></a>

### Getting Started

1. Forecast Data Provider (typically a Content Owner, Streamer, MVPD, or Distributor) will stand up a Forecasting API endpoint with information about upcoming events contained in this API.
2. Forecast Data Provider informs partners that the API endpoint is available.
3. Forecast Data Provider provisions partner(s) to call the endpoint with some cadence based on the permissions that have been given the partner.
4. Partner (typically a monetization partner, such as an ad exchange or DSP) stands up a mechanism to call API and consume information about upcoming events.

### General guidance

- Values are likely to change over time as the event gets closer. Farther-out events will have more approximate viewership estimations, but it is still helpful to send an expected viewership range with the understanding that forecasts will become more accurate over time.
- Some event metadata may be omitted based on real-time conditions. For example, a post-season game will not be determined until very close to the event because the teams cannot be known.

## Uncertainty <a name="uncertainty"></a>

Because this API deals with things happening in the future, a change in values should be expected. They fall into two main categories: if an event will happen (see Using tentative vs. scheduled) and if the timing of the ad breaks are known prior to the event (see Planned vs. Unplanned Ad Inventory).

### Using Tentative vs Scheduled

It is possible that the event itself may not happen and should be flagged as tentative. For example, a game series of 7 games that was won in 5—games 6 and 7 will not be played.

### Static vs Variable Ad Inventory

The `unplanned` attribute in the Ad Inventory Configuration object signals whether an event may contain commercial breaks that are not fully known or scheduled in advance at the time the forecast is published.

This field is intended to help downstream systems distinguish between:

- **Planned:** inventory where the ad breaks are known beforehand and fixed
- **Unplanned:** inventory whose timing, frequency, or existence depends on real-time event conditions

#### When to Set `unplanned = 1` (Yes)

When any material portion of the event’s ad inventory is expected to be triggered dynamically or conditionally, including but not limited to:

- Game-state-driven breaks (live sports where breaks are inserted based on gameplay rather than a fixed rundown)
- Series-based or conditional events (events that may not occur depending on prior outcomes)
- Live events with elastic ad load (total ad load expands or contracts based on pacing/duration/retention)
- Dynamic pod construction (variable pod durations, variable slot counts, breaks skipped or consolidated)

#### When to Set `unplanned = 0` (No)

When all commercial breaks are pre-defined and expected to occur as scheduled such as:

- Award shows with scripted act breaks
- Season premieres with fixed pod patterns
- Pre-recorded content
- Live events where ad breaks are contractually fixed (e.g., halftime-only sponsorships)

#### Mixed Inventory Scenarios (Important)

Many live events contain both planned and unplanned inventory. Recommended practices include:

- Set `unplanned = 1` if any unplanned breaks exist
- Use optional extensions (e.g., `podpattern`) to describe known, fixed pods
- Treat remaining inventory as probabilistic or opportunistic

#### Seller Interpretation Guidance

- Avoid selling guaranteed pod positions within unplanned inventory unless risk is explicitly managed
- Use conservative assumptions for advance deals
- Reserve unplanned inventory for impression-based buying, programmatic auctions, and outcome-based guarantees

#### Buyer Interpretation Guidance

- Treat `unplanned = 1` as a signal of forecast uncertainty
- Expect inventory delivery to be non-uniform and potentially front- or back-loaded
- Plan for dynamic pacing and potential makegoods

### Direct-Sold vs Programmatic Supply

The `streamsdata` and `adinventoryconfig` objects describe the total audience and total ad load of an event. In practice, a significant portion of that ad load is often committed to direct-sold sponsorships, upfronts, or other non-programmatic channels before the event airs. Without a signal for this, a buyer computing `expectedpeak × expectedpodcount × slots` will systematically overestimate what can be bought programmatically.

The `programmatic` object in <a href="#streamsdata">StreamsData</a> closes this gap by forecasting the programmatically available portion of supply, per country.

#### Choosing a Measure

Sellers may express programmatic availability as a share of ad load (`share`), as absolute impressions (`expectedimps` with optional `lowerbound`/`upperbound`), as programmatic ad seconds per stream (`addurationsec`), or any combination. Guidance:

- `share` is the least sensitive measure and is recommended as a baseline, since it does not disclose absolute direct-sold volume.
- `expectedimps` is the most directly actionable for buyers and is recommended when the seller has a mature forecasting model.
- `addurationsec` is useful when direct sales are structured around fixed pod positions (e.g., halftime sold direct, in-game available programmatically).
- When multiple measures are provided, they should be mutually consistent (e.g., `addurationsec ≈ share × totaladdurationsec`).

#### Deriving Impressions

When `expectedimps` is absent, buyers can approximate programmatic impressions as: `expectedpeak × (addurationsec ÷ average ad length)` or `expectedpeak × share × (totaladdurationsec ÷ average ad length)`. These are rough estimates; peak concurrency is not sustained across all pods, so actual impressions will typically be lower. Sellers who can provide `expectedimps` directly should do so.

#### Market vs Requestor Scope

The `scope` attribute is required so that buyers do not misinterpret the figures.

- `scope = 1` (market) reflects the total programmatic opportunity for the event, and will be the same value for every partner querying the endpoint. This is appropriate for planning and for deal negotiation.
- `scope = 2` (requestor) reflects only the supply the authenticated `requestor` can access, given existing agreements. Sellers should use this when they allocate supply across exchanges or DSPs and want each partner to see a realistic figure.
- The specific commercial terms governing access remain out of scope and should be communicated through the Deal API.

#### Interaction with `unplanned`

When `unplanned = 1`, the programmatic forecast is subject to the same uncertainty as the total ad load. Sellers should use `lowerbound`/`upperbound` to convey that uncertainty, and buyers should treat `expectedimps` as an expected value rather than a commitment.

#### Absence of the Object

If `programmatic` is omitted, buyers should not assume all supply is programmatically available. It simply indicates the seller has not provided this forecast.

## JSON Examples

> All examples below are **linted and valid JSON** (fixed invalid quotes, typos, duplicate keys, and missing/trailing commas from the source examples).

### Sample: Sports Game

```json
{
  "scheduledstart": 1750006800000,
  "scheduledend": 1750024800000,
  "lastmodifieddate": 1749950000000,
  "eventstatus": 1,
  "content": {
    "id": "game_12345",
    "title": "TeamA vs TeamB - Game 5",
    "language": "en",
    "gtax": "9",
    "genres": "483",
    "series": "Sportsball Playoffs 2026",
    "season": "2025",
    "rating": "TV-G",
    "network": {
      "name": "Network Name"
    },
    "url": "https://www.network.com",
    "cattax": 7,
    "cat": [
      "640",
      "325",
      "36",
      "1"
    ],
    "prodq": 1,
    "contentrating": "PG",
    "len": 7200
  },
  "streamsdata": [
    {
      "country": "USA",
      "expectedpeak": 1200000,
      "lowerbound": 950000,
      "programmatic": {
        "scope": 1,
        "share": 0.4,
        "expectedimps": 9600000,
        "lowerbound": 7200000,
        "upperbound": 11500000,
        "addurationsec": 384
      }
    },
    {
      "country": "GBR",
      "expectedpeak": 300000,
      "lowerbound": 250000,
      "programmatic": {
        "scope": 1,
        "share": 0.75
      }
    }
  ],
  "adinventoryconfig": {
    "totaladdurationsec": 960,
    "expectedpodcount": 8,
    "maxadduration": 120,
    "media_types_supported": [
      "video",
      "display"
    ]
  }
}
```

### Sample: Sports Event Series with Conditional Schedule

```json
{
  "version": "1.0.0",
  "timestamp": 1751205600000,
  "upcomingevent": [
    {
      "id": "content_management_system_id_game1",
      "scheduledstart": 1751044800000,
      "scheduledend": 1751059200000,
      "flexibleend": 1,
      "eventstatus": 1,
      "content": {
        "id": "content_management_system_id_game1",
        "title": "TeamA vs TeamB - Series Name - Game 1",
        "series": "Series Name 2026",
        "season": "2026",
        "gtax": "9",
        "genres": "483",
        "language": "en",
        "producer": {
          "name": "Content Name"
        },
        "network": {
          "name": "Network Name"
        },
        "len": 10800
      },
      "streamsdata": [
        {
          "country": "USA",
          "expectedpeak": 2000000
        }
      ],
      "adinventoryconfig": {
        "supportedmtype": [
          2
        ],
        "totaladdurationsec": 1400,
        "expectedpodcount": 12,
        "unplanned": 1
      },
      "lastmodifieddate": 1750958400000
    },
    {
      "id": "content_management_system_id_game2",
      "scheduledstart": 1751131200000,
      "scheduledend": 1751145600000,
      "flexibleend": 1,
      "eventstatus": 1,
      "content": {
        "id": "content_management_system_id_game2",
        "title": "TeamA vs TeamB - Series Name - Game 2",
        "series": "Series Name 2026",
        "season": "2026",
        "gtax": "9",
        "genres": "483",
        "language": "en",
        "producer": {
          "name": "Content Name"
        },
        "network": {
          "name": "Network Name"
        },
        "len": 10800
      },
      "streamsdata": [
        {
          "country": "USA",
          "expectedpeak": 1900000
        }
      ],
      "adinventoryconfig": {
        "supportedmtype": [
          2
        ],
        "totaladdurationsec": 1360,
        "expectedpodcount": 11,
        "unplanned": 1
      },
      "lastmodifieddate": 1750958400000
    },
    {
      "id": "content_management_system_id_game3",
      "scheduledstart": 1751217600000,
      "scheduledend": 1751232000000,
      "flexibleend": 1,
      "eventstatus": 1,
      "content": {
        "id": "content_management_system_id_game3",
        "title": "TeamA vs TeamB - Series Name - Game 3",
        "series": "Series Name 2026",
        "season": "2026",
        "gtax": "9",
        "genres": "483",
        "language": "en",
        "producer": {
          "name": "Content Name"
        },
        "network": {
          "name": "Network Name"
        },
        "len": 10800
      },
      "streamsdata": [
        {
          "country": "USA",
          "expectedpeak": 1850000
        }
      ],
      "adinventoryconfig": {
        "supportedmtype": [
          2
        ],
        "totaladdurationsec": 1340,
        "expectedpodcount": 11,
        "unplanned": 1
      },
      "lastmodifieddate": 1750958400000
    },
    {
      "id": "content_management_system_id_game4",
      "scheduledstart": 1751304000000,
      "scheduledend": 1751318400000,
      "flexibleend": 1,
      "eventstatus": 1,
      "content": {
        "id": "content_management_system_id_game4",
        "title": "TeamA vs TeamB - Series Name - Game 4",
        "series": "Series Name 2026",
        "season": "2026",
        "gtax": "9",
        "genres": "483",
        "language": "en",
        "producer": {
          "name": "Content Name"
        },
        "network": {
          "name": "Network Name"
        },
        "len": 10800
      },
      "streamsdata": [
        {
          "country": "USA",
          "expectedpeak": 1800000
        }
      ],
      "adinventoryconfig": {
        "supportedmtype": [
          2
        ],
        "totaladdurationsec": 1320,
        "expectedpodcount": 11,
        "unplanned": 1
      },
      "lastmodifieddate": 1751202000000
    },
    {
      "id": "content_management_system_id_game5",
      "scheduledstart": 1751390400000,
      "scheduledend": 1751404800000,
      "flexibleend": 1,
      "eventstatus": 2,
      "content": {
        "id": "content_management_system_id_game5",
        "title": "TeamA vs TeamB - Series Name - Game 5",
        "series": "Series Name 2026",
        "season": "2026",
        "gtax": "9",
        "genres": "483",
        "language": "en",
        "producer": {
          "name": "Content Name"
        },
        "network": {
          "name": "Network Name"
        },
        "len": 10800
      },
      "streamsdata": [
        {
          "country": "USA",
          "expectedpeak": 1650000
        }
      ],
      "adinventoryconfig": {
        "supportedmtype": [
          2
        ],
        "totaladdurationsec": 1280,
        "expectedpodcount": 10,
        "unplanned": 1
      },
      "lastmodifieddate": 1751202000000
    },
    {
      "id": "content_management_system_id_game6",
      "scheduledstart": 1751476800000,
      "scheduledend": 1751491200000,
      "flexibleend": 1,
      "eventstatus": 2,
      "content": {
        "id": "content_management_system_id_game6",
        "title": "TeamA vs TeamB - Series Name - Game 6 (If Necessary)",
        "series": "Series Name 2026",
        "season": "2026",
        "gtax": "9",
        "genres": "483",
        "language": "en",
        "producer": {
          "name": "Content Name"
        },
        "network": {
          "name": "Network Name"
        },
        "len": 10800
      },
      "streamsdata": [
        {
          "country": "USA",
          "expectedpeak": 1500000
        }
      ],
      "adinventoryconfig": {
        "supportedmtype": [
          2
        ],
        "totaladdurationsec": 1240,
        "expectedpodcount": 9,
        "unplanned": 1
      },
      "lastmodifieddate": 1751202000000
    },
    {
      "id": "content_management_system_id_game7",
      "scheduledstart": 1751563200000,
      "scheduledend": 1751577600000,
      "flexibleend": 1,
      "eventstatus": 2,
      "content": {
        "id": "content_management_system_id_game7",
        "title": "TeamA vs TeamB - Series Name - Game 7 (If Necessary)",
        "series": "Series Name 2026",
        "season": "2026",
        "gtax": "9",
        "genres": "483",
        "language": "en",
        "producer": {
          "name": "Content Name"
        },
        "network": {
          "name": "Network Name"
        },
        "len": 10800
      },
      "streamsdata": [
        {
          "country": "USA",
          "expectedpeak": 2100000
        }
      ],
      "adinventoryconfig": {
        "supportedmtype": [
          2
        ],
        "totaladdurationsec": 1400,
        "expectedpodcount": 12,
        "unplanned": 1
      },
      "lastmodifieddate": 1751202000000
    }
  ]
}
```

### Sample: Political Event (Debate)

```json
{
  "id": 198291,
  "scheduledstart": 1726016400000,
  "scheduledend": 1726022400000,
  "lastmodifieddate": 1726016200000,
  "eventstatus": 1,
  "flexibleend": 0,
  "content": {
    "id": "exampleID-abc-1",
    "title": "Presidential Debate 2",
    "language": "en",
    "gtax": "9",
    "genres": "386",
    "series": "Election 2024",
    "season": "2024",
    "network": {
      "name": "Network Name"
    },
    "cattax": 7,
    "cat": [
      "387"
    ],
    "prodq": 1,
    "contentrating": "PG",
    "len": 6000
  },
  "streamsdata": [
    {
      "country": "USA",
      "expectedpeak": 35000000,
      "programmatic": {
        "scope": 2,
        "share": 0.1,
        "expectedimps": 14000000
      }
    }
  ],
  "adinventoryconfig": {
    "totaladdurationsec": 240,
    "expectedpodcount": 2,
    "unplanned": 0,
    "supportedmtype": [
      2
    ]
  }
}
```

### Sample: Political Event (Election Night News Coverage)

```json
{
  "id": 198311,
  "scheduledstart": 1730847600000,
  "scheduledend": 1730862000000,
  "lastmodifieddate": 1730845600000,
  "eventstatus": 1,
  "flexibleend": 1,
  "content": {
    "id": "exampleID-abc-2",
    "title": "Election Night Coverage",
    "language": "en",
    "gtax": "9",
    "genres": "386",
    "series": "Election 2024",
    "season": "2024",
    "network": {
      "name": "Network Name"
    },
    "cattax": 7,
    "cat": [
      "387"
    ],
    "prodq": 1,
    "contentrating": "PG",
    "len": 14400
  },
  "streamsdata": [
    {
      "country": "USA",
      "expectedpeak": 19000000
    }
  ],
  "adinventoryconfig": {
    "totaladdurationsec": 1440,
    "expectedpodcount": 12,
    "unplanned": 1,
    "supportedmtype": [
      2
    ]
  }
}
```
