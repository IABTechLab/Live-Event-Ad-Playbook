![IAB Tech Lab](https://drive.google.com/uc?id=10yoBoG5uRETSXRrnJPUDuONujvADrSG1)

# Implementation Guidance for the Live Event Ad Playbook (LEAP)

This directory contains implementation guidance for the IAB Tech Lab Live Event Ad Playbook (LEAP). The guidance helps ecosystem participants implement the Forecast API and Concurrent Streams API consistently, whether using either API independently or both APIs together.

The documentation is organized around an **interactive end-to-end walkthrough**, a shared API conventions reference, and API-specific implementation guides. Additional LEAP capabilities may be accompanied by related guidance as the program evolves.

## Start Here: Multiple API Guidance — Interactive Walkthrough

**[Launch the LEAP Interactive Walkthrough](https://iabtechlab.github.io/Live-Event-Ad-Playbook/implementation-guidance/Multiple_API_Guidance_Walkthrough.html)**

**Recommended starting point for all readers.** Before exploring individual API fields and conventions, follow a complete illustrative live event from advance planning through live operation and post-event review.

The walkthrough demonstrates how the two LEAP APIs work together:

- **Before the event — Forecast API:** Discover upcoming live events, evaluate expected stream counts and advertising inventory configuration, and prepare campaigns, deals, and infrastructure.
- **While the stream is open — Concurrent Streams API:** Monitor near-real-time stream counts, understand how streams change over the event, and inform pacing, bidding, and operational scaling decisions.
- **After the event — Review:** Compare saved forecast and live stream snapshots to understand where actual conditions differed from the plan.

Use **First call** and **Next call** to advance through example API requests and responses. For each call, explore **When it's called**, **What comes back**, and **What changed**, while watching the forecast and live streams chart and cumulative snapshot tables evolve.

The walkthrough illustrates the **LEAP API v1.0 designs** using fictional event data and numerical assumptions. Stream counts are not a direct measurement of biddable impressions, programmatic availability, or bid-request volume.

**Other ways to access it:** [Multiple API Guidance Markdown entry point](Multiple_API_Guidance.md) | [HTML source](Multiple_API_Guidance_Walkthrough.html). GitHub's source preview does not execute the HTML's interactive JavaScript; use the GitHub Pages link above for the full experience.

## Recommended Reading and Implementation Workflow

After completing the walkthrough, use the following documents as needed:

| Step | Document | Why read it |
|---|---|---|
| 1 | **[Interactive Multiple API Guidance](https://iabtechlab.github.io/Live-Event-Ad-Playbook/implementation-guidance/Multiple_API_Guidance_Walkthrough.html)** | Understand how forecasting, live stream monitoring, and post-event comparison fit together. |
| 2 | [Common API Conventions Reference](Common_API_Guidance.md) | Learn the conventions shared across LEAP APIs, including authentication, formats, errors, and cross-API identity correlation. |
| 3 | [Forecast API specification](../Forecast_API.md) and/or [Concurrent Streams API specification](../Concurrent_Streams_API.md) | Review the authoritative API objects, fields, and requirements for the API or APIs you will implement. |
| 4 | [Forecast API Guidance](Forecast_API_Guidance.md) and/or [Concurrent Streams API Guidance](Concurrent_Streams_API_Guidance.md) | Apply the specifications with API-specific operational and implementation guidance. |

Implementers working with only one API can focus on that specification and its companion guide after reviewing the walkthrough and common conventions. Implementers using both APIs should review both sets of documents and the walkthrough's cross-API workflow.

## Implementation Guidance Documents

### Multiple API Guidance — Interactive Walkthrough

**Files:** [Interactive walkthrough (GitHub Pages)](https://iabtechlab.github.io/Live-Event-Ad-Playbook/implementation-guidance/Multiple_API_Guidance_Walkthrough.html) · [`Multiple_API_Guidance.md`](Multiple_API_Guidance.md) · [`Multiple_API_Guidance_Walkthrough.html`](Multiple_API_Guidance_Walkthrough.html)

This is the recommended introduction to LEAP implementation. It demonstrates how Forecast API results support pre-event planning and how Concurrent Streams API snapshots support in-event adjustment. Readers advance through an illustrative event one API call at a time, inspect requests, responses, and field changes, and follow the evolving forecast and live streams chart.

### Common API Conventions Reference

**File:** [`Common_API_Guidance.md`](Common_API_Guidance.md)

This document defines common implementation conventions that apply across LEAP APIs unless an API-specific specification or guide provides more specific direction. It covers:

- cross-API identity correlation
- naming conventions
- HTTP methods
- authentication
- request and response formats
- status codes and error handling
- pagination and sorting
- versioning
- rate limiting

Use this document when implementing any LEAP API. API-specific guides should not repeat shared conventions unless they need to clarify how a particular API applies or narrows them.

### Forecast API Guidance

**File:** [`Forecast_API_Guidance.md`](Forecast_API_Guidance.md)  
**Specification:** [`Forecast_API.md`](../Forecast_API.md)

This document provides guidance for future live-event discovery and pre-event planning workflows, including:

- future event discovery
- expected stream-count forecasting
- expected ad inventory forecasting
- infrastructure preparation
- deal planning
- campaign planning before a live event

The Forecast API supports planning ahead of an event. It should not be treated as the source for current live stream counts once live data is available; implementers should use the Concurrent Streams API for that purpose.

### Concurrent Streams API Guidance

**File:** [`Concurrent_Streams_API_Guidance.md`](Concurrent_Streams_API_Guidance.md)  
**Specification:** [`Concurrent_Streams_API.md`](../Concurrent_Streams_API.md)

This document provides guidance for retrieving near-real-time live-event stream-count signals, including:

- near-real-time stream monitoring
- traffic scaling
- live operational optimization
- campaign pacing adjustments
- bid strategy adjustments during live playback
- delivery risk management during concurrency spikes

The Concurrent Streams API supports in-event adjustment. It does not replace the Forecast API for pre-event planning.

## How the Documents Work Together

The documentation is modular. The **API specifications** define the applicable objects, fields, values, and requirements. The **Common API Conventions Reference** describes shared implementation patterns. The **API-specific guides** explain how to apply those patterns to the Forecast and Concurrent Streams APIs. The **interactive walkthrough** demonstrates the combined operational workflow.

If implementation guidance conflicts with an API specification, **the API specification takes precedence**. Where a common convention and API-specific guidance differ, use the API-specific guidance for that API, subject to its specification.

For workflows using both APIs, preserve a consistent, agreed identifier to correlate the same event across forecasting, live stream counts, OpenRTB transactions, and reporting. Either `content.id` or a source-qualified Extended Content ID (`content.data[].cids[]` together with `content.data[].name`) may support this correlation; neither is inherently preferred. See [Cross-API Identity Correlation](Common_API_Guidance.md#cross-api-identity-correlation) in the common guide and the Shared Key section of the interactive walkthrough.

## Learn More and Get Involved

### About the Advanced TV Pillar at IAB Tech Lab

https://iabtechlab.com/standards/advanced-tv/

### About the Live Event Ad Playbook (LEAP)

https://iabtechlab.com/leap

### About OpenRTB

https://iabtechlab.com/openrtb

### Contact

For more information, or to get involved, please email support@iabtechlab.com.

### About IAB Tech Lab

The IAB Technology Laboratory is a nonprofit research and development consortium charged with producing and helping companies implement global industry technical standards and solutions. The goal of the Tech Lab is to reduce friction associated with the digital advertising and marketing supply chain while contributing to the safe growth of an industry.

The IAB Tech Lab spearheads the development of technical standards, creates and maintains a code library to assist in rapid, cost-effective implementation of IAB standards, and establishes a test platform for companies to evaluate the compatibility of their technology solutions with IAB standards, which have been the foundation for interoperability and profitable growth in the digital advertising supply chain.

Learn more about IAB Tech Lab: https://www.iabtechlab.com/
