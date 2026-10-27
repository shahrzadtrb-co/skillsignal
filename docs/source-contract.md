# Source contract: Arbeitnow Job Board API

Checked on: 2026-10-27

## Endpoint

- URL: https://www.arbeitnow.com/api/job-board-api
- Method: GET
- Authentication: none, no API key needed
- Docs: https://documenter.getpostman.com/view/18545278/UVJbJdKh

## Pagination

- Request pages with ?page=N, starting at 1. 300 jobs per page (meta.per_page).
- No total count: links.last is null. The last page is reached when links.next is null.
- Sorted by created_at according to meta.info. Data refreshes every hour.

## Fields

| Field | Type | Example | Notes |
| --- | --- | --- | --- |
| `slug` | str | senior-sap-hcm-payroll-consultant-py-de- | Unique ID, used as the dedup key |
| `company_name` | str | Sidekick Network GmbH & Co. KG | Plain text |
| `title` | str | Senior SAP HCM Payroll Consultant - PY-D | German and English mixed |
| `description` | str | (long HTML text) | HTML, sometimes double-escaped, ends with a promo link |
| `remote` | bool | False | true or false |
| `url` | str | https://www.arbeitnow.com/jobs/companies | Link to the Arbeitnow posting, use as the back link |
| `tags` | list | ['IT'] | List of strings |
| `job_types` | list | ['Freiberufler / selbstständiger', 'free | List, German and English mixed, can be empty |
| `location` | str | Hanover | City name as text |
| `created_at` | int | 1793122266 | Unix timestamp in seconds, many ties |

## Observed quirks

- Pagination overlap: page 1 and page 2 (300 jobs each, fetched 27 Oct 2026)
  shared 110 slugs. Page 1 fetched twice returned the same 300 slugs.
  All 110 overlapping jobs share their created_at timestamp with other jobs
  (522 of 600 jobs have a tied timestamp). Likely cause: unstable tie order
  across pages (not confirmed). Consequence: deduplicate by slug across pages;
  a single run may miss jobs.
- Unique jobs across pages 1 and 2: 490 of 600 fetched (110 duplicates removed by slug).
- Double-escaped HTML in description: 50 of 490. Unescape until stable before parsing.
- Arbeitnow promo link at the end of description: 490 of 490. Strip it before skill or language matching.
- Empty job_types: 60 of 490. Treat as unknown, not as a category.
- Same company and title posted in several cities: 12 ads spread over 57 postings. Counting rule decided in ADR-001.
- Tech roles: roughly TODO of 20 sampled titles are tech jobs.

## Terms I must follow

- Link back to Arbeitnow on anything public (terms section 11 and meta.terms).
- Raw descriptions stay private. Publish only aggregate counts and links to postings.
- Permission to use the API can be revoked at any time.

## How politely I fetch

- At most one request every 3 seconds.
- Cache fetched pages locally in data/raw/ (never committed).
- A full run must finish well within one hour, because data refreshes hourly.

## Open questions

- Rate limit is not documented.
- Page 1 is not strictly descending by created_at, although meta.info says
  jobs are ordered by it. Direction or secondary sort key unknown.
