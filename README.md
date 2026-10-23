![IAB Tech Lab](https://drive.google.com/uc?id=10yoBoG5uRETSXRrnJPUDuONujvADrSG1)

# Live Event Ad Playbook

This repository contains specifications and implementation guidance for the IAB Tech Lab Live Event Ad Playbook (LEAP).

LEAP provides technical protocols intended to help advertising systems prepare for, transact against, and optimize monetization of live event advertising supply. The current repository includes specifications for pre-event forecasting and near-real-time concurrent stream signals, along with implementation guidance for using those APIs independently and together.

## Repository Contents

### API Specifications

| File | Description |
|---|---|
| [`Forecast_API.md`](Forecast_API.md) | Defines the Forecast API, which enables authorized partners to retrieve information about future live event advertising supply, including event schedules, expected streams scale, and expected ad inventory. |
| [`Concurrent_Streams_API.md`](Concurrent_Streams_API.md) | Defines the Concurrent Streams API, which enables authorized subscribers to retrieve near-real-time stream-count information while live events are occurring. |

### Implementation Guidance

Implementation guidance is maintained in the [`implementation-guidance`](implementation-guidance/) directory.

| File | Description |
|---|---|
| [`Common_API_Guidance.md`](implementation-guidance/Common_API_Guidance.md) | Defines common API conventions that apply across LEAP APIs unless an API-specific guide states otherwise. |
| [`Forecast_API_Guidance.md`](implementation-guidance/Forecast_API_Guidance.md) | Provides implementation guidance specific to the Forecast API. |
| [`Concurrent_Streams_API_Guidance.md`](implementation-guidance/Concurrent_Streams_API_Guidance.md) | Provides implementation guidance specific to the Concurrent Streams API. |
| [`Multiple_API_Guidance.md`](implementation-guidance/Multiple_API_Guidance.md) | Explains how the Forecast API and Concurrent Streams API can be used together across the live event monetization lifecycle. |

## How the APIs Work Together

The Forecast API and Concurrent Streams API address different phases of live event advertising.

```text
Forecast API              -> pre-event planning
Concurrent Streams API    -> live-event adjustment
```

### Forecast API

The Forecast API is intended for use before a live event begins.

It helps advertising systems:

- discover future live events
- estimate streams scale before the event
- estimate advertising inventory availability
- support infrastructure planning
- inform deal planning and packaging
- support campaign planning before the event begins

Once an event begins, the Forecast API should not be treated as the authoritative signal for current live concurrency.

### Concurrent Streams API

The Concurrent Streams API is intended for use while a live event is occurring.

It helps advertising systems:

- understand current live streams scale
- prepare for and respond to QPS spikes
- scale regional infrastructure
- adjust bidding and pacing behavior
- reduce latency, slate, and lost monetization opportunities

During a live event, Concurrent Streams API data should be treated as the more relevant operational signal for current stream counts.

## Recommended Reading Order

### Implementing the Forecast API

1. [`Common_API_Guidance.md`](implementation-guidance/Common_API_Guidance.md)
2. [`Forecast_API.md`](Forecast_API.md)
3. [`Forecast_API_Guidance.md`](implementation-guidance/Forecast_API_Guidance.md)

### Implementing the Concurrent Streams API

1. [`Common_API_Guidance.md`](implementation-guidance/Common_API_Guidance.md)
2. [`Concurrent_Streams_API.md`](Concurrent_Streams_API.md)
3. [`Concurrent_Streams_API_Guidance.md`](implementation-guidance/Concurrent_Streams_API_Guidance.md)

### Implementing Both APIs Together

1. [`Common_API_Guidance.md`](implementation-guidance/Common_API_Guidance.md)
2. [`Forecast_API.md`](Forecast_API.md)
3. [`Concurrent_Streams_API.md`](Concurrent_Streams_API.md)
4. [`Forecast_API_Guidance.md`](implementation-guidance/Forecast_API_Guidance.md)
5. [`Concurrent_Streams_API_Guidance.md`](implementation-guidance/Concurrent_Streams_API_Guidance.md)
6. [`Multiple_API_Guidance.md`](implementation-guidance/Multiple_API_Guidance.md)

## Guidance vs. Specification

The API specifications define the objects, fields, and values for each API.

The implementation guidance documents explain recommended implementation patterns, operational considerations, and API usage workflows.

If implementation guidance conflicts with an API specification, the API specification is the source of truth.

If common implementation guidance conflicts with API-specific guidance, the API-specific guidance applies for that API.

## Cross-API Identity Correlation

Implementers using more than one LEAP API should preserve stable identifiers that allow the same live event to be correlated across planning, live adjustment, bidding, and reporting workflows.

Recommended identifiers include:

- `content.id`
- `content.data.cids`
- Forecast API event-level `id`
- event timing and stable content metadata as a fallback

Where possible, identifiers used in LEAP APIs should also align with identifiers used in downstream OpenRTB bid requests and reporting systems.

#### About the Advanced TV Pillar at IAB Tech Lab
https://iabtechlab.com/standards/advanced-tv/

#### About the Live Event Ads Playbook (LEAP)
https://iabtechlab.com/leap

#### About OpenRTB
https://iabtechlab.com/openrtb

## Contact

For more information, or to get involved, please email support@iabtechlab.com.

## About IAB Tech Lab

The IAB Technology Laboratory is a nonprofit research and development consortium charged with producing and helping companies implement global industry technical standards and solutions. The goal of the Tech Lab is to reduce friction associated with the digital advertising and marketing supply chain while contributing to the safe growth of an industry.

The IAB Tech Lab spearheads the development of technical standards, creates and maintains a code library to assist in rapid, cost-effective implementation of IAB standards, and establishes a test platform for companies to evaluate the compatibility of their technology solutions with IAB standards, which have been the foundation for interoperability and profitable growth in the digital advertising supply chain.

Learn more about IAB Tech Lab:

https://www.iabtechlab.com/

## Contributors and Technical Governance

The Live Event Ad Program is part of the IAB Tech Lab working group process. Participants in the Live Event Ad Program and related IAB Tech Lab working groups must be members of IAB Tech Lab.

Technical governance and code commits for this project are provided by the appropriate IAB Tech Lab Commit Group.

Learn more about how to submit changes in an IAB Tech Lab working group:

https://iabtechlab.com/blog/so-youd-like-to-propose-a-change-to-openrtb-adcom/

## License

Specifications pertaining to Live Event Ad Protocols from IAB Tech Lab are licensed under a Creative Commons Attribution 3.0 License. To view a copy of this license, visit creativecommons.org/licenses/by/3.0/ or write to Creative Commons, 171 Second Street, Suite 300, San Francisco, CA 94105, USA.

By submitting an idea, specification, software code, document, file, or other material (each, a “Submission”) to the Live Event Ad Protocol repository, to any member of the applicable IAB Tech Lab working group, or to IAB Tech Lab in relation to the Live Event Ad Protocols, you agree to and hereby license such Submission to IAB Tech Lab under the Creative Commons Attribution 3.0 License and agree that such Submission may be used and made available to the public under the terms of such license.

If you are a member of IAB Tech Lab, the terms and conditions of the IAB Tech Lab IPR Policy may also be applicable to your Submission. If the IPR Policy is applicable to your Submission, then the IPR Policy will control in the event of a conflict between the Creative Commons Attribution 3.0 License and the IPR Policy.

## Disclaimer

THE STANDARDS, THE SPECIFICATIONS, THE MEASUREMENT GUIDELINES, AND ANY OTHER MATERIALS OR SERVICES PROVIDED TO OR USED BY YOU HEREUNDER (THE “PRODUCTS AND SERVICES”) ARE PROVIDED “AS IS” AND “AS AVAILABLE,” AND IAB TECHNOLOGY LABORATORY, INC. (“TECH LAB”) MAKES NO WARRANTY WITH RESPECT TO THE SAME AND HEREBY DISCLAIMS ANY AND ALL EXPRESS, IMPLIED, OR STATUTORY WARRANTIES, INCLUDING, WITHOUT LIMITATION, ANY WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, AVAILABILITY, ERROR-FREE OR UNINTERRUPTED OPERATION, AND ANY WARRANTIES ARISING FROM A COURSE OF DEALING, COURSE OF PERFORMANCE, OR USAGE OF TRADE.

TO THE EXTENT THAT TECH LAB MAY NOT AS A MATTER OF APPLICABLE LAW DISCLAIM ANY IMPLIED WARRANTY, THE SCOPE AND DURATION OF SUCH WARRANTY WILL BE THE MINIMUM PERMITTED UNDER SUCH LAW. THE PRODUCTS AND SERVICES DO NOT CONSTITUTE BUSINESS OR LEGAL ADVICE.

TECH LAB DOES NOT WARRANT THAT THE PRODUCTS AND SERVICES PROVIDED TO OR USED BY YOU HEREUNDER SHALL CAUSE YOU AND/OR YOUR PRODUCTS OR SERVICES TO BE IN COMPLIANCE WITH ANY APPLICABLE LAWS, REGULATIONS, OR SELF-REGULATORY FRAMEWORKS, AND YOU ARE SOLELY RESPONSIBLE FOR COMPLIANCE WITH THE SAME, INCLUDING, BUT NOT LIMITED TO, DATA PROTECTION LAWS.
