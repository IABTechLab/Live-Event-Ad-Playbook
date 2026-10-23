![IAB Tech Lab](https://drive.google.com/uc?id=10yoBoG5uRETSXRrnJPUDuONujvADrSG1)

# **Implementation Guidance for the Live Event Ad Playbook**

#### About the Advanced TV Pillar at IAB Tech Lab
https://iabtechlab.com/standards/advanced-tv/

#### About the Live Event Ads Playbook (LEAP)
https://iabtechlab.com/leap

#### About OpenRTB
https://iabtechlab.com/openrtb

#### Contact
For more information, or to get involved, please email support@iabtechlab.com.

#### About IAB Tech Lab  
The IAB Technology Laboratory is a nonprofit research and development consortium charged with producing and helping companies implement global industry technical standards and solutions. The goal of the Tech Lab is to reduce friction associated with the digital advertising and marketing supply chain while contributing to the safe growth of an industry. The IAB Tech Lab spearheads the development of technical standards, creates and maintains a code library to assist in rapid, cost-effective implementation of IAB standards, and establishes a test platform for companies to evaluate the compatibility of their technology solutions with IAB standards, which for 18 years have been the foundation for interoperability and profitable growth in the digital advertising supply chain.

Learn more about IAB Tech Lab here: [https://www.iabtechlab.com/](https://www.iabtechlab.com/)

# LEAP Implementation Guidance

This directory contains implementation guidance for the IAB Tech Lab Live Event Ad Playbook (LEAP).

The guidance is organized into a shared API conventions document, API-specific implementation guides, and a multi-API workflow guide that explains how the Forecast API and Concurrent Streams API can be used together.  Any additional products from the LEAP project will also exist in this directory and related guides.

## Documents

### Common API Conventions Reference

**File:** [`Common_API_Guidance.md`](Common_API_Guidance.md)

This document defines common implementation conventions that apply across LEAP APIs unless an API-specific guide states otherwise.

Use this document first when implementing any LEAP API.

It covers:

- naming conventions
- HTTP methods
- authentication
- request and response formats
- status codes
- error handling
- pagination
- versioning
- rate limiting
- sorting

API-specific guides should not repeat this common guidance unless they need to clarify how a specific API applies, narrows, or overrides the shared convention.

---

### Forecast API Guidance

**File:** [`Forecast_API_Guidance.md`](Forecast_API_Guidance.md)

This document provides implementation guidance specific to the Forecast API.

Use this guide when implementing support for future live event forecast discovery and planning workflows.

The Forecast API is intended for pre-event use cases, including:

- future event discovery
- expected audience forecasting
- expected ad inventory forecasting
- infrastructure preparation
- deal planning
- campaign planning before a live event

The Forecast API should not be treated as the authoritative source for real-time audience levels once an event is live. During the live event, implementers should use the Concurrent Streams API for real-time adjustment signals.

---

### Concurrent Streams API Guidance

**File:** [`Concurrent_Streams_Guidance.md`](Concurrent_Streams_Guidance.md)

This document provides implementation guidance specific to the Concurrent Streams API.

Use this guide when implementing support for near-real-time live event stream count signals.

The Concurrent Streams API is intended for in-event use cases, including:

- near-real-time stream monitoring
- traffic scaling
- live operational optimization
- campaign pacing adjustments
- bid strategy adjustments during live playback
- delivery risk management during concurrency spikes

The Concurrent Streams API should be used while an event is live and should not replace the Forecast API for pre-event planning.

---

### Multiple LEAP API Guidance — An Interactive Walkthrough

**Files:** [`Multiple_API_Guidance.md`](Multiple_API_Guidance.md) launches the interactive walkthrough (GitHub Pages) [`Multiple_API_Guidance_Walkthrough.html`](https://iabtechlab.github.io/Live-Event-Ad-Playbook/implementation-guidance/Multiple_API_Guidance_Walkthrough.html)

The interactive walkthrough demonstrates how Forecast API results support pre-event planning and how Concurrent Streams API snapshots support in-event adjustment. Readers can advance through an example event scenario one API call at a time, inspect requests, responses, and field changes, and follow the evolving forecast and live audience chart.

GitHub's Markdown and HTML source views do not run JavaScript. Use the [`GitHub Pages Link`](https://iabtechlab.github.io/Live-Event-Ad-Playbook/implementation-guidance/Multiple_API_Guidance_Walkthrough.html) to interact with the full HTML walkthrough.
