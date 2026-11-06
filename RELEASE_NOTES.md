![IAB Tech Lab](https://drive.google.com/uc?id=10yoBoG5uRETSXRrnJPUDuONujvADrSG1)
# Live Event Ad Playbook (LEAP) — Release Notes

This document tracks significant public-comment milestones, specification releases, and implementation guidance for the IAB Tech Lab Live Event Ad Playbook (LEAP).

> **Release status:** Dates below distinguish public-comment openings and closings, specification finalization, and GitHub release tags. A public-comment release is not a final specification.

## October 28, 2026 — LEAP Implementation Guidance v1.0 

**Status:** Public-comment release  
**Scope:** 
- Implementation guidance for the existing LEAP Forecast API and Concurrent Streams API designs.
- Existing specifications renamed for consistency, no changes to the normative API specifications are introduced by this documentation release.
  - "/Concurrent-Streams.md" renamed to "/Concurrent_Streams_AIP.md"
  - "/ForecastAPI.md" renamed to "/Forecast_API.md"
- Introduced this Release Notes document

### Included documentation

| Document | Description |
| --- | --- |
| [Common API Guidance](implementation-guidance/Common_API_Guidance.md) | Shared API conventions covering naming, transport and HTTP behavior, authentication, request and response formats, errors, pagination, versioning, rate limiting, and sorting. |
| [Forecast API Guidance](implementation-guidance/Forecast_API_Guidance.md) | Pre-event forecast retrieval, event and inventory interpretation, tentative events, forecast uncertainty, and planning practices. |
| [Concurrent Streams API Guidance](implementation-guidance/Concurrent_Streams_API_Guidance.md) | Live stream-count retrieval, SSAI/CSAI interpretation, regional data, snapshot freshness, and operational use during a live event. |
| [Multiple API Guidance](implementation-guidance/Multiple_API_Guidance.md) | Entry point for the [`Interactive Walkthrough`](https://iabtechlab.github.io/Live-Event-Ad-Playbook/implementation-guidance/Multiple_API_Guidance_Walkthrough.html) illustrating step-by-step how Forecast and Concurrent Streams APIs work together. Intended for viewing through GitHub Pages, it includes example requests, responses, audience visualizations, and cross-API identity correlation|

### What this guidance adds

- A common implementation reference so ecosystem partners do not need to interpret basic API conventions independently for each LEAP API.
- API-specific explanations that complement, rather than replace, the Forecast API and Concurrent Streams API specifications.
- A single-event walkthrough following the transition from **pre-event forecasts** to **live concurrency signals** and **post-event review**.
- Guidance on correlating an event across planning, live measurement, and downstream OpenRTB using shared content identifiers where available.
- Illustrative considerations for polling, capacity planning, bidding, pacing, and interpreting stream-count signals without equating concurrent streams with guaranteed biddable inventory.

---

## October 22, 2026 - Repository Renamed: Live Event Ad Playbook (LEAP)
**Status:** GitHub Repo Name Change 
- Renamed the GitHub repository from `Live-Event-Ad-Protocols` to `Live-Event-Ad-Playbook`.
- Aligned the repository name with the established Live Event Ad Playbook (LEAP) initiative, encompassing its API specifications and implementation guidance.
- This administrative change does not introduce changes to the technical specifications or API behavior.
- Existing repository links should continue to redirect through GitHub's repository rename functionality, subject to GitHub's normal redirect behavior.
- 
## July 28, 2026 — Concurrent Streams v1 (GitHub Release Tag)

**Status:** GitHub release record  
**Tag:** [`cs-202607`](https://github.com/IABTechLab/Live-Event-Ad-Playbook/releases/tag/cs-202607)

- GitHub published a release titled **Concurrent Streams v1**, referencing the Concurrent Streams API contribution.
- This dated GitHub tag is recorded separately from the earlier Concurrent Streams API v1.0 public-comment and finalization milestones; it should not be interpreted as the original public-comment opening date.

---

## March 20, 2026 — Forecasting API Public Comment Closed

**Status:** Public-comment period concluded

- Closed the public-comment window for the proposed LEAP Forecasting API.
- The API supports advance discovery of upcoming programming and live events, including event scheduling, expected concurrent stream counts, and anticipated advertising inventory configuration.
- Public-comment closure by itself does not establish a separately verified final v1.0 publication date.

[Forecasting API public-comment announcement](https://iabtechlab.com/press-releases/iab-tech-lab-opens-public-comment-on-leap-forecasting-api/)

---

## February 19, 2026 — Forecasting API Public Comment Opened

**Status:** Public-comment release

- IAB Tech Lab announced the proposed Forecasting API as a LEAP deliverable focused on pre-event inventory discovery and planning.
- The announced public-comment window ran through **March 20, 2026**.

[Official announcement](https://iabtechlab.com/press-releases/iab-tech-lab-opens-public-comment-on-leap-forecasting-api/)

---

## 2025 — Concurrent Streams API v1.0 and LEAP Launch

### Q3 2025 — Concurrent Streams API v1.0 Final

**Status:** Final specification; exact final publication day not confirmed in the sources reviewed

- Following its public-comment process, Concurrent Streams API v1.0 became LEAP's first final API specification.
- The API defines a mechanism for authorized participants to request near-real-time counts of streams associated with live content.

[LEAP standards page](https://iabtechlab.com/standards/leap/)

### August 5, 2025 — LEAP Program Announcement

**Status:** Public announcement

- IAB Tech Lab announced LEAP and highlighted Concurrent Streams API work developed with industry contributors.
- The program aims to improve live-event advertising interoperability, operational scalability, and reliability.

[Official announcement](https://prod-backup.iabtechlab.com/press-releases/iab-tech-lab-releases-live-event-ad-playbook-in-collaboration-with-amazon-nbcuniversal-freewheel-and-index-exchange/)

### July 28, 2025 — Concurrent Streams API Public Comment Closed

**Status:** Public-comment period concluded

- The Concurrent Streams API v1.0 public-comment window closed on **July 28, 2025**, preceding its finalization.

[LEAP standards page](https://iabtechlab.com/standards/leap/)

---

## Specification and Guidance Relationship

- **API specifications** define the supported objects, fields, values, and normative behavior.
- **Common implementation guidance** provides shared conventions across LEAP APIs.
- **API-specific guidance** explains implementation patterns and operational considerations for each API.
- **Multiple API Guidance** demonstrates an illustrative end-to-end workflow; its examples do not establish new normative API requirements.

## References

- [LEAP standards and current specification status](https://iabtechlab.com/standards/leap/)
- [LEAP GitHub repository](https://github.com/IABTechLab/Live-Event-Ad-Playbook)
- [GitHub releases](https://github.com/IABTechLab/Live-Event-Ad-Playbook/releases)
- [Forecast API specification](https://github.com/IABTechLab/Live-Event-Ad-Playbook/blob/develop/Forecast_API.md)
- [Concurrent Streams API specification](https://github.com/IABTechLab/Live-Event-Ad-Playbook/blob/develop/Concurrent_Streams_API.md)

For LEAP questions or participation, contact [support@iabtechlab.com](mailto:support@iabtechlab.com).
