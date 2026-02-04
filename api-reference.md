# Ditto API Reference

Complete endpoint reference for the Ditto synthetic research API.

**Base URL:** `https://app.askditto.io`
**Auth:** `Authorization: Bearer YOUR_API_KEY`
**Content-Type:** `application/json`

---

## Authentication

### Free Tier Setup

```bash
curl -sL https://app.askditto.io/scripts/free-tier-auth.sh | bash
```

Or manually: visit https://app.askditto.io/docs/free-tier-oauth

- Free keys start with `rk_free_` — ~12 shared personas, no custom filters
- Paid keys start with `rk_live_` — custom groups, demographic filtering, unlimited studies

### Checking Your Key

```bash
curl -s "https://app.askditto.io/v1/research-groups" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

A 200 response confirms your key works.

---

## Research Groups

### POST /v1/research-groups/recruit

Create a new research group with demographic filters.

**Request:**
```json
{
  "name": "Canadian Coffee Drinkers 25-45",
  "description": "Regular specialty coffee consumers in urban Canada",
  "group_size": 10,
  "filters": {
    "country": "Canada",
    "age_min": 25,
    "age_max": 45
  }
}
```

**Response:**
```json
{
  "group": {
    "id": 184,
    "uuid": "60a157f6620b43f6b0b9dbbe98eb2420",
    "name": "Canadian Coffee Drinkers 25-45",
    "agent_count": 10
  }
}
```

**IMPORTANT:**
- Use `group_size`, NOT `size`
- Save the `uuid` string — use it (not `id`) when creating studies
- Country is required. All other filters are optional.
- State MUST be a 2-letter code: "MI", "TX", "CA" (never "Michigan", "Texas")
- Income filter is NOT supported — do not include it

### GET /v1/research-groups/{group_id}

Retrieve group details including all agent profiles.

```bash
curl -s "https://app.askditto.io/v1/research-groups/184" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Returns agent profiles with: name, age, city, state, country, occupation,
summary, and image URL.

### POST /v1/research-groups/{group_uuid}/append

Add more personas with different demographics to an existing group.

```json
{
  "group_size": 5,
  "filters": {
    "country": "USA",
    "state": "TX",
    "age_min": 55,
    "age_max": 70
  }
}
```

Useful for over-recruiting, then curating the panel by removing
unsuitable profiles.

### POST /v1/research-groups/{group_id}/agents/remove

Remove specific agents from a group before running a study.

```json
{
  "agent_ids": [12345, 67890]
}
```

Use this after reviewing agent profiles to remove those who don't match
your research criteria.

---

## Research Studies

### POST /v1/research-studies

Create a new study linked to a research group.

**Request:**
```json
{
  "title": "Specialty Coffee Purchase Psychology",
  "objective": "Understand what drives trial, repeat purchase, and switching in specialty coffee",
  "shareable": true,
  "research_group_uuid": "60a157f6620b43f6b0b9dbbe98eb2420"
}
```

**Response:**
```json
{
  "study": {
    "id": 122,
    "uuid": "4dc9fd3a4a984ad28a9ccc61dc54ea58",
    "title": "Specialty Coffee Purchase Psychology",
    "stage": "active"
  }
}
```

**IMPORTANT:**
- Always set `shareable: true` so you can generate share links later
- Use the group `uuid` string, not the numeric `id`
- Save the study `id` (numeric) for subsequent API calls

### GET /v1/research-studies/{study_id}

Get study metadata including stage, participant count, and share URL.

```bash
curl -s "https://app.askditto.io/v1/research-studies/122" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### GET /v1/research-studies

List all studies (paginated).

```bash
curl -s "https://app.askditto.io/v1/research-studies?limit=50" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

---

## Questions & Responses

### POST /v1/research-studies/{study_id}/questions

Submit a question to the study panel. Returns job IDs for async processing.

**Request:**
```json
{
  "question": "Walk me through the last time you tried a new coffee brand. What made you pick it up? What made you keep buying it — or stop?"
}
```

**Response:**
```json
{
  "job_ids": [
    "3d298218-2c39-48cc-80a8-bf4276d8574e",
    "a1b2c3d4-5678-90ab-cdef-1234567890ab"
  ],
  "question_id": 1195,
  "status": "queued"
}
```

**CRITICAL: Questions MUST be asked one at a time.** Poll ALL job_ids
from a question until every one shows `finished` before submitting the
next question. The API does not support concurrent questions.

### GET /v1/research-studies/{study_id}/questions

Retrieve all questions and their responses.

```bash
curl -s "https://app.askditto.io/v1/research-studies/122/questions" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Response structure:**
```json
{
  "study_id": 122,
  "count": 7,
  "questions": [
    {
      "id": 1195,
      "question": "Walk me through the last time...",
      "optimized_question": "...",
      "answer_count": 10,
      "answers": [
        {
          "response_text": "I picked up <b>Dark Horse Coffee</b> because...",
          "agent_name": "Evelyn Cheng",
          "agent_age": 50,
          "agent_city": "Toronto",
          "agent_state": "ON",
          "agent_country": "Canada",
          "agent_occupation": "Project Coordinator",
          "agent_summary": "Toronto-based project coordinator, 50, married..."
        }
      ]
    }
  ]
}
```

Note: `response_text` may contain HTML tags (`<b>`, `<i>`, `<ul>`, `<li>`, `<br>`).
Strip or convert these as needed for your output format.

---

## Job Polling

### GET /v1/jobs/{job_id}

Check the status of an async response job.

```bash
curl -s "https://app.askditto.io/v1/jobs/3d298218-2c39-48cc-80a8-bf4276d8574e" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Status progression:** `queued` → `started` → `finished`

**Response when finished:**
```json
{
  "status": "finished",
  "result": {
    "agent_id": 652419,
    "response_text": "[Full persona response]",
    "study_id": 122
  }
}
```

**Polling guidance:**
- Poll every 10-15 seconds
- Typical response time: 30-90 seconds per question batch
- Must poll ALL job_ids from a question before asking the next

---

## Study Completion & Sharing

### POST /v1/research-studies/{study_id}/complete

Trigger AI-generated analysis after all questions are answered.

```json
{
  "force": false
}
```

Generates:
- Executive summary
- Key insights and themes
- Segment analysis (by demographics, attitudes)
- Points of divergence (where personas disagreed)
- Shared mindsets (universal findings)
- Recommended follow-up questions

Processing time: 1-2 minutes.

### POST /v1/research-studies/{study_id}/share

Enable or disable public sharing.

```json
{
  "enabled": true
}
```

**Response:**
```json
{
  "url": "https://app.askditto.io/organization/studies/shared/TOKEN"
}
```

This URL is publicly accessible — anyone can view the study without
authentication. Use it for stakeholder reviews, blog posts, or investor
presentations.

### GET /v1/research-studies/{study_id}/share

Check current share status and retrieve the URL.

---

## Removing Agents from Studies

### POST /v1/research-studies/{study_id}/agents/remove

Remove specific agents from a study before running questions. Useful for
the over-recruit-and-curate strategy.

```json
{
  "agent_ids": [12345, 67890]
}
```

---

## Rate Limits & Constraints

- Questions must be asked sequentially (not in parallel)
- Maximum group size: 20 personas per recruitment call
- Groups can be expanded with `/append` for larger panels
- Free tier: shared personas, no custom demographic filters
- Paid tier: full demographic filtering, unlimited groups and studies
- No income filter available on any tier

---

## Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 400 | Bad request (check field names and types) |
| 401 | Invalid or missing API key |
| 404 | Resource not found |
| 422 | Validation error (e.g., unsupported filter) |
| 429 | Rate limited |
| 500 | Server error (retry after 30 seconds) |
