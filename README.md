<!-- Generated from the SocQ API catalog and docs. Do not edit by hand. -->
# Douyin API

[![SocQ Douyin API](https://img.shields.io/badge/API-SocQ%20Douyin%20API-F64C31)](https://socq.ai/apis/douyin?utm_source=github&utm_medium=readme&utm_campaign=socq-douyin-api)
[![API documentation](https://img.shields.io/badge/Docs-docs.socq.ai-111827)](https://docs.socq.ai/api-manual/douyin/overview)
[![License: MIT](https://img.shields.io/badge/License-MIT-2563EB)](LICENSE)
[![Check examples](https://github.com/SocQAPI/douyin-api/actions/workflows/check.yml/badge.svg)](https://github.com/SocQAPI/douyin-api/actions/workflows/check.yml)

6 endpoints for public Douyin data through one SocQ API key, with runnable cURL, Node.js and Python examples and a valid request body for every endpoint.

[Browse Douyin APIs](https://socq.ai/apis/douyin?utm_source=github&utm_medium=readme&utm_campaign=socq-douyin-api) · [Get an API key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-douyin-api) · [Documentation](https://docs.socq.ai/api-manual/douyin/overview) · [All SocQ examples](https://github.com/SocQAPI/socq-examples)

## Endpoints

Prices are in SocQ credits; 1 credit = $0.005.

| Endpoint | publicId | What it returns | Price | Launched |
| --- | --- | --- | --- | --- |
| [Douyin Live Room Detail API](#douyin-live-room-detail-api) | `douyin/live-room-detail` | Retrieve one public Douyin live room from a known numeric room ID, returning normalized room context, host identity, media, visible viewer and like counts, and collection time. | 0.1 credits ($0.0005) per call | 2026-08-17 |
| [Douyin User Profile API](#douyin-user-profile-api) | `douyin/user-profile` | Retrieve one public Douyin profile from a supported profile URL or user identifier, returning normalized identity, biography, imagery, account flags, visible counts, and collection context. | 0.1 credits ($0.0005) per call | 2026-08-17 |
| [Douyin User Videos API](#douyin-user-videos-api) | `douyin/user-videos` | Collect up to 20 public videos from one Douyin profile URL or known user ID, returning normalized content, author, media, duration, timestamps, and visible engagement fields. | 0.1 credits ($0.0005) per call | 2026-08-17 |
| [Douyin Video Comments API](#douyin-video-comments-api) | `douyin/video-comments` | Collect one page of public comments from a direct Douyin video URL, returning normalized text, author, relationship, time, and visible engagement fields. | 0.1 credits ($0.0005) per call | 2026-08-18 |
| [Douyin Video Detail API](#douyin-video-detail-api) | `douyin/video-detail` | Retrieve one public Douyin video from a known douyin.com or iesdouyin.com video URL, returning normalized video identity, caption, creator, media, duration, timestamps, and visible engagement. | 0.1 credits ($0.0005) per call | 2026-08-17 |
| [Douyin Video Search API](#douyin-video-search-api) | `douyin/video-search` | Search one fixed first page of public Douyin videos by text query, returning normalized captions, author context, media, duration, timestamps, and visible engagement when available. | 0.1 credits ($0.0005) per call | 2026-08-18 |

## Quick start

1. Create an API key at [socq.ai/dashboard/api-key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-douyin-api).
2. Export it. The examples read only `SOCQ_API_KEY`.

```bash
git clone https://github.com/SocQAPI/douyin-api.git
cd douyin-api
export SOCQ_API_KEY="your-api-key"
```

3. Run any endpoint by its publicId. Without a second argument the script sends `payloads/<slug>.json`; pass a JSON file or an inline JSON object to change the input.

```bash
# Node.js 20+
node examples/node/socq.mjs douyin/live-room-detail

# Python 3.8+ (standard library only)
python3 examples/python/socq.py douyin/live-room-detail payloads/live-room-detail.json

# cURL + jq
bash examples/curl/socq.sh douyin/live-room-detail '{"room_id":"7462723839303093032"}'
```

Each script submits the task, polls it until it succeeds or fails, follows every result page, and writes the records to `output/<slug>-<task_id>.json`. Run `npm run check` to validate the payloads and scripts offline.

Never expose `SOCQ_API_KEY` in browser code, mobile apps, public repositories, screenshots, fixtures or logs.

## How the API works

Base URL `https://api.socq.ai`. Every request sends the API key as a bearer token:

```http
Authorization: Bearer <SOCQ_API_KEY>
Content-Type: application/json
```

All endpoints are asynchronous:

```text
POST /v1/<publicId>  ->  data.task_id  ->  GET /v1/tasks/{task_id}  ->  queued | running | succeeded | failed
```

1. **Submit** the JSON body to `POST https://api.socq.ai/v1/<publicId>`. The response returns `data.task_id` immediately; it means the task was created, not that data is ready.
2. **Poll** `GET https://api.socq.ai/v1/tasks/{task_id}` until `data.status` is `succeeded` or `failed` (a failed task carries `data.error_message`).
3. **Read results.** A succeeded task returns normalized records in `data.results.items`. Use `limit` (default 50, max 500) and pass `data.results.next_cursor` as `cursor` while `data.results.has_more` is true. `fields` selects up to 50 comma-separated fields or dot paths.
4. **Raw files (optional).** `GET https://api.socq.ai/v1/tasks/{task_id}/files` lists raw JSONL exports.

```bash
curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100" \
  -H "Authorization: Bearer $SOCQ_API_KEY"

curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100&cursor=$NEXT_CURSOR" \
  -H "Authorization: Bearer $SOCQ_API_KEY"
```

`callback_url` (where an endpoint accepts it) makes SocQ POST to your URL when the task reaches a terminal state; keep polling as a fallback.

**Billing.** A submit may reserve credits from `results_limit` or the number of submitted inputs; when the task finishes SocQ settles the charge from the actual result count and returns unused reserved credits. `402` means insufficient credits.

**Errors.** `400` invalid body or unsupported field, `401` missing or invalid key, `402` insufficient credits, `403` IP allowlist or task owned by another account, `404` endpoint or task not found, `422` input validation failed, `429` rate or credit-window limit (respect `Retry-After`), `500` submission failed. A client-side timeout does not cancel a task: resume polling the same `task_id` instead of submitting again.

Reference: [Authentication](https://docs.socq.ai/api-manual/agent-api/authentication) · [Task status and results](https://docs.socq.ai/api-manual/agent-api/status) · [Task files](https://docs.socq.ai/api-manual/agent-api/files) · [Callbacks](https://docs.socq.ai/api-manual/agent-api/callbacks) · [Errors](https://docs.socq.ai/api-manual/agent-api/errors)

## Endpoint reference

### Douyin Live Room Detail API

Retrieve one public Douyin live room from a known numeric room ID, returning normalized room context, host identity, media, visible viewer and like counts, and collection time.

- **Request:** `POST https://api.socq.ai/v1/douyin/live-room-detail`
- **publicId:** `douyin/live-room-detail`
- **Price:** 0.1 credits ($0.0005) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/live-room-detail.json`](payloads/live-room-detail.json)

Fetch normalized live room info data from Douyin.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `room_id` | string | Yes | A public live room identifier. |

```bash
curl -X POST "https://api.socq.ai/v1/douyin/live-room-detail" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "room_id": "7462723839303093032"
}'
```

Full docs: [https://docs.socq.ai/api-manual/douyin/live-room-detail](https://docs.socq.ai/api-manual/douyin/live-room-detail) · Landing page: [socq.ai/apis/douyin/live-room-detail](https://socq.ai/apis/douyin/live-room-detail?utm_source=github&utm_medium=readme&utm_campaign=socq-douyin-api)

### Douyin User Profile API

Retrieve one public Douyin profile from a supported profile URL or user identifier, returning normalized identity, biography, imagery, account flags, visible counts, and collection context.

- **Request:** `POST https://api.socq.ai/v1/douyin/user-profile`
- **publicId:** `douyin/user-profile`
- **Price:** 0.1 credits ($0.0005) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/user-profile.json`](payloads/user-profile.json)

Fetch normalized profiles data from Douyin.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `url` | string | One of* | A public content or product URL. |
| `user_id` | string | One of* | A public account identifier. |

*Required rule: send at least one of `url`, `user_id`.

```bash
curl -X POST "https://api.socq.ai/v1/douyin/user-profile" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "user_id": "MS4wLjABAAAAW9FWcqS7RdQAWPd2AA5fL_ilmqsIFUCQ_Iym6Yh9_cUa6ZRqVLjVQSUjlHrfXY1Y"
}'
```

Full docs: [https://docs.socq.ai/api-manual/douyin/user-profile](https://docs.socq.ai/api-manual/douyin/user-profile) · Landing page: [socq.ai/apis/douyin/user-profile](https://socq.ai/apis/douyin/user-profile?utm_source=github&utm_medium=readme&utm_campaign=socq-douyin-api)

### Douyin User Videos API

Collect up to 20 public videos from one Douyin profile URL or known user ID, returning normalized content, author, media, duration, timestamps, and visible engagement fields.

- **Request:** `POST https://api.socq.ai/v1/douyin/user-videos`
- **publicId:** `douyin/user-videos`
- **Price:** 0.1 credits ($0.0005) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/user-videos.json`](payloads/user-videos.json)

Fetch normalized user videos data from Douyin.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `results_limit` | integer | No | (value 1 to 20; default `20`) |
| `url` | string | One of* | A public content or product URL. |
| `user_id` | string | One of* | A public account identifier. |

*Required rule: send at least one of `url`, `user_id`.

```bash
curl -X POST "https://api.socq.ai/v1/douyin/user-videos" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "user_id": "MS4wLjABAAAANXSltcLCzDGmdNFI2Q_QixVTr67NiYzjKOIP5s03CAE",
  "results_limit": 3
}'
```

Full docs: [https://docs.socq.ai/api-manual/douyin/user-videos](https://docs.socq.ai/api-manual/douyin/user-videos) · Landing page: [socq.ai/apis/douyin/user-videos](https://socq.ai/apis/douyin/user-videos?utm_source=github&utm_medium=readme&utm_campaign=socq-douyin-api)

### Douyin Video Comments API

Collect one page of public comments from a direct Douyin video URL, returning normalized text, author, relationship, time, and visible engagement fields.

- **Request:** `POST https://api.socq.ai/v1/douyin/video-comments`
- **publicId:** `douyin/video-comments`
- **Price:** 0.1 credits ($0.0005) per call
- **Launched:** 2026-08-18
- **Example body:** [`payloads/video-comments.json`](payloads/video-comments.json)

Fetch normalized comments data from Douyin.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `results_limit` | integer | No | (value 1 to 20; default `20`) |
| `url` | string | Yes | A public content or product URL. |

```bash
curl -X POST "https://api.socq.ai/v1/douyin/video-comments" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://www.douyin.com/video/7448118827402972455",
  "results_limit": 3
}'
```

Full docs: [https://docs.socq.ai/api-manual/douyin/video-comments](https://docs.socq.ai/api-manual/douyin/video-comments) · Landing page: [socq.ai/apis/douyin/video-comments](https://socq.ai/apis/douyin/video-comments?utm_source=github&utm_medium=readme&utm_campaign=socq-douyin-api)

### Douyin Video Detail API

Retrieve one public Douyin video from a known douyin.com or iesdouyin.com video URL, returning normalized video identity, caption, creator, media, duration, timestamps, and visible engagement.

- **Request:** `POST https://api.socq.ai/v1/douyin/video-detail`
- **publicId:** `douyin/video-detail`
- **Price:** 0.1 credits ($0.0005) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/video-detail.json`](payloads/video-detail.json)

Fetch normalized videos data from Douyin.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `url` | string | Yes | A public content or product URL. |

```bash
curl -X POST "https://api.socq.ai/v1/douyin/video-detail" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://www.douyin.com/video/7448118827402972455"
}'
```

Full docs: [https://docs.socq.ai/api-manual/douyin/video-detail](https://docs.socq.ai/api-manual/douyin/video-detail) · Landing page: [socq.ai/apis/douyin/video-detail](https://socq.ai/apis/douyin/video-detail?utm_source=github&utm_medium=readme&utm_campaign=socq-douyin-api)

### Douyin Video Search API

Search one fixed first page of public Douyin videos by text query, returning normalized captions, author context, media, duration, timestamps, and visible engagement when available.

- **Request:** `POST https://api.socq.ai/v1/douyin/video-search`
- **publicId:** `douyin/video-search`
- **Price:** 0.1 credits ($0.0005) per call
- **Launched:** 2026-08-18
- **Example body:** [`payloads/video-search.json`](payloads/video-search.json)

Fetch normalized search data from Douyin.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `query` | string | Yes | Search query. |

```bash
curl -X POST "https://api.socq.ai/v1/douyin/video-search" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "query": "人工智能"
}'
```

Full docs: [https://docs.socq.ai/api-manual/douyin/video-search](https://docs.socq.ai/api-manual/douyin/video-search) · Landing page: [socq.ai/apis/douyin/video-search](https://socq.ai/apis/douyin/video-search?utm_source=github&utm_medium=readme&utm_campaign=socq-douyin-api)

## Repository contents

| Path | Purpose |
| --- | --- |
| [`examples/curl/socq.sh`](examples/curl/socq.sh) | Submit, poll and paginate with cURL and jq |
| [`examples/node/socq.mjs`](examples/node/socq.mjs) | Same workflow in Node.js 20+ (no dependencies) |
| [`examples/python/socq.py`](examples/python/socq.py) | Same workflow in Python 3.8+ (standard library) |
| [`payloads/`](payloads) | One documented request body per endpoint |
| [`endpoints.json`](endpoints.json) | Endpoint list, prices and request schemas used by the scripts and checks |
| [`fixtures/`](fixtures) | Synthetic submit and task response shapes |
| [`docs/production-notes.md`](docs/production-notes.md) | Retries, timeouts, pagination and data handling |
| [`scripts/check.mjs`](scripts/check.mjs) | Offline check: payloads, syntax and secrets |

## Responsible use and platform scope

- Use only publicly accessible Douyin data and the fields supported by each endpoint.
- Do not use the examples to access restricted content, login-only surfaces, authentication controls or non-public personal information.
- SocQ is not an official API of the represented platform and is not affiliated with or endorsed by it. Platform names and trademarks belong to their respective owners.
- Before production use, assess the laws, platform terms, privacy obligations and retention requirements that apply to your organization and use case.
- Collect only the fields needed for a defined purpose, restrict access, set retention periods, and support correction or deletion workflows where required.

This section describes the public-data boundary; it is not legal advice.

## License

[MIT](LICENSE)
