---
name: ditto-product-research
description: >
  Conduct customer research, product validation, pricing tests, competitive
  analysis, or market research using Ditto's synthetic research platform
  (300K+ AI personas, 92% overlap with real focus groups). Covers the full
  workflow: recruitment, study design, question asking, insight extraction,
  and share link generation. Also handles quick questions, free-tier usage,
  Zeitgeist surveys, media attachments, and natural-language study requests.
  Use when the user mentions Ditto, synthetic research, persona studies,
  or customer validation.
allowed-tools: Bash(curl *), Bash(python3 *), Read, Grep, WebFetch
---

# Ditto Product Research

Run customer research, pricing tests, and product validation studies using
Ditto's 300,000+ synthetic personas — directly from the terminal.

**Full documentation:** https://askditto.io/claude-code-guide

## What Ditto Does

Ditto maintains 300,000+ AI-powered synthetic personas calibrated to census
data across USA, UK, Germany, and Canada. You ask them open-ended questions
and get qualitative responses with the specificity of real interviews.

- **92% overlap** with traditional focus groups (EY Americas validation)
- **95% correlation** with traditional research
- **Harvard/Cambridge/Stanford/Oxford** peer-reviewed methodology
- A 10-persona, 7-question study completes in **10-12 minutes**
- Traditional equivalent: 4-8 weeks, $10,000-50,000

## Quick Start (Free Tier)

Get a free API key — no credit card, no sales call:

```bash
curl -sL https://app.askditto.io/scripts/free-tier-auth.sh | bash
```

Or visit: https://app.askditto.io/docs/free-tier-oauth

Ask a question immediately:

```bash
curl -s -X POST "https://app.askditto.io/v1/free/questions" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"question": "What frustrates you most about grocery shopping?"}'
```

Free keys (`rk_free_`): ~12 shared personas, no demographic filtering.
Paid keys (`rk_live_`): custom groups, demographic filtering, unlimited studies.

## API Essentials

**Base URL:** `https://app.askditto.io`
**Auth header:** `Authorization: Bearer YOUR_API_KEY`
**Content-Type:** `application/json`

## The Standard Workflow (6 Steps)

IMPORTANT: Follow these steps in order. Questions MUST be asked
sequentially — wait for all responses before asking the next.

### Step 1: Recruit Your Panel

```bash
curl -s -X POST "https://app.askditto.io/v1/research-groups/recruit" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "US Adults 30-55",
    "group_size": 10,
    "filters": {"country": "USA", "age_min": 30, "age_max": 55}
  }'
```

Save the `uuid` from the response (5-15 seconds).

**CRITICAL:** Use `group_size` not `size`. Use group `uuid` not `id`.
State filter uses 2-letter codes ("MI" not "Michigan").

### Step 2: Create Study

```bash
curl -s -X POST "https://app.askditto.io/v1/research-studies" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Product Concept Validation",
    "objective": "Understand customer pain points and validate pricing",
    "research_group_uuid": "UUID_FROM_STEP_1"
  }'
```

Save the study `id`. Response nests under `data.study` — access via
`response["study"]["id"]`, NOT `response["id"]`.

### Step 3: Ask Questions (One at a Time)

```bash
curl -s -X POST "https://app.askditto.io/v1/research-studies/STUDY_ID/questions" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"question": "Your open-ended question here"}'
```

Returns `job_ids` (one per persona). Poll until complete before asking next.

### Step 4: Poll Until Complete

```bash
curl -s "https://app.askditto.io/v1/jobs/JOB_ID" \
  -H "Authorization: Bearer $DITTO_API_KEY"
```

**Polling strategy for 10-persona study:**
- Wait **45-50 seconds** before first poll
- Then poll every **20 seconds**
- Poll **ONE** job_id as proxy — all jobs from the same question finish together
- Status: `queued` → `started` → `finished` (or `failed`)

ALL jobs must show `finished` before asking the next question.

### Step 5: Complete the Study

```bash
curl -s -X POST "https://app.askditto.io/v1/research-studies/STUDY_ID/complete" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"force": false}'
```

Triggers AI analysis: summary, segments, divergences, recommendations (20-40s).
Use `"force": true` to re-run analysis on an already-completed study (avoids 409).

### Step 6: Get Share Link

```bash
curl -s -X POST "https://app.askditto.io/v1/research-studies/STUDY_ID/share" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"enabled": true}'
```

Returns a public URL. Use `share_link` field (preferred over `share_url`).
To check existing share state without changing it:

```bash
curl -s "https://app.askditto.io/v1/research-studies/STUDY_ID/share" \
  -H "Authorization: Bearer $DITTO_API_KEY"
```

## Workflow Patterns

### Standard 7-Question Study
The default. Recruit 10 personas → create study → ask 7 questions → complete → share.
~10-12 minutes total.

### Over-Recruit & Curate
For niche audiences or when participant quality matters most (e.g., VC diligence):

1. Recruit 15-20 participants: `"group_size": 15`
2. Create study from that group
3. Ask Q1 as a screening question ("Tell me about your role and how often you...")
4. Review Q1 responses — score each agent: High / Medium / Low relevance
5. Remove low-relevance agents from the **study** (not the group):
   ```bash
   curl -s -X POST "https://app.askditto.io/v1/research-studies/STUDY_ID/agents/remove" \
     -H "Authorization: Bearer $DITTO_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{"agent_ids": [123, 456, 789]}'
   ```
   `agent_ids` must be `list[int]` — never strings or UUIDs.
6. Ask Q2-Q7 to curated panel only. Keep minimum 8 personas.

### Quick Question (No Study)
For 1-5 quick answers without creating a full study:

**To a specific agent:**
```bash
curl -s -X POST "https://app.askditto.io/v1/research-agents/AGENT_ID/questions" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"question": "What matters most when choosing a coffee brand?"}'
```

**To an entire group:**
```bash
curl -s -X POST "https://app.askditto.io/v1/research-groups/GROUP_ID/questions" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"question": "What matters most when choosing a coffee brand?"}'
```

### Free Tier
~12 shared personas, no demographic filtering. Good for quick tests:

```bash
curl -s -X POST "https://app.askditto.io/v1/free/questions" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"question": "What frustrates you most about meal planning?"}'
```

### Three-Phase Iterative
For deeper validation, run 3 sequential studies (can share the same group):

| Phase | Purpose | Questions |
|-------|---------|-----------|
| 1. Pain Discovery | Find the real problem | 7 open-ended |
| 2. Deep Dive | Understand requirements and trust | 7 targeted |
| 3. Concept Test | Validate positioning and pricing | 7 structured |

Each phase's questions should be informed by the previous phase's findings.
~30-45 minutes total.

### Resume Stalled Study
If a study was interrupted mid-way:

1. Get study details: `GET /v1/research-studies/STUDY_ID`
2. List existing questions: `GET /v1/research-studies/STUDY_ID/questions`
3. Resume from the next unanswered question
4. Complete when all questions are asked

### Zeitgeist Survey
Quick single-question survey with predefined answer options:

```bash
curl -s -X POST "https://app.askditto.io/v1/zeitgeist/surveys/create" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Brand Awareness Pulse",
    "question": "Which brand comes to mind first for sustainable coffee?",
    "research_group_uuid": "GROUP_UUID",
    "answer_options": ["Brand A", "Brand B", "Brand C", "Other"]
  }'
```

Get results: `GET /v1/zeitgeist/surveys/SURVEY_ID/results`
Delete: `DELETE /v1/zeitgeist/surveys/SURVEY_ID`

### Natural Language Requests
Let the system design your study or group from a plain-text brief:

**Study request:**
```bash
curl -s -X POST "https://app.askditto.io/v1/research-study-requests" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"request_text": "I want to understand why millennials are switching from coffee to matcha"}'
```

**Group request:**
```bash
curl -s -X POST "https://app.askditto.io/v1/research-group-requests" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"request_text": "US women aged 25-40 who drink matcha daily"}'
```

Check group request status: `GET /v1/research-group-requests/REQUEST_ID`

### AI-Assisted Recruitment
Provide an objective and let the system design the group:

```bash
curl -s -X POST "https://app.askditto.io/v1/research-groups/interview" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "objective": "Understand premium pet food purchasing decisions",
    "group_size": 10
  }'
```

## The 7-Question Framework

Design studies with this proven sequence (adapt to your domain):

1. **Establish relevance** — "Walk me through how you currently handle [task]. What does a typical week look like?"
2. **Identify pain** — "What's the most frustrating part of [task]? What makes you want to throw your laptop out the window?"
3. **Quantify impact** — "Roughly how much time/money do you lose to [problem] per week?"
4. **Current solutions** — "What tools or workarounds do you currently use? What works? What doesn't?"
5. **Past attempts** — "Have you tried switching to something new? What happened?"
6. **Magic wand** — "If you could fix ONE thing about [task], what would it be and why?"
7. **Adoption barriers** — "Imagine a tool that solves [problem]. What would make you hesitant to try it?"

Introduce brand/product at Q3 earliest — not Q1 (avoid anchoring bias).
The magic wand question (Q6) is consistently the most revealing.

## Complete API Reference

### Research Groups

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/v1/research-groups/recruit` | Recruit group with demographic filters |
| `POST` | `/v1/research-groups/create` | Create group from explicit agent IDs |
| `POST` | `/v1/research-groups/interview` | AI-assisted recruitment from objective |
| `GET` | `/v1/research-groups` | List all groups (`?limit=N&offset=N`) |
| `GET` | `/v1/research-groups/{id}` | Get group details + agent profiles |
| `POST` | `/v1/research-groups/{id}/update` | Update name/description/dedupe |
| `DELETE` | `/v1/research-groups/{id}` | Archive group |
| `POST` | `/v1/research-groups/{id}/agents/add` | Add agents by ID |
| `POST` | `/v1/research-groups/{id}/agents/remove` | Remove agents permanently |
| `POST` | `/v1/research-groups/{uuid}/append` | Recruit more agents into existing group |

**⚠️ Note:** `append` uses `group_uuid` (not `group_id`) — inconsistent with other endpoints.

### Research Studies

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/v1/research-studies` | Create study (`research_group_uuid` required) |
| `GET` | `/v1/research-studies` | List studies (`?limit=N&offset=N`) |
| `GET` | `/v1/research-studies/{id}` | Get study details (stage, share_url, counts) |
| `POST` | `/v1/research-studies/{id}/complete` | Trigger AI analysis |
| `POST` | `/v1/research-studies/{id}/agents/remove` | Remove agents from study (curation) |

### Questions

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/v1/research-studies/{id}/questions` | Ask question in study (returns job_ids) |
| `GET` | `/v1/research-studies/{id}/questions` | Get all Q&A data |
| `POST` | `/v1/research-agents/{id}/questions` | Quick question to one agent |
| `POST` | `/v1/research-groups/{id}/questions` | Quick question to entire group |

### Jobs

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/v1/jobs/{job_id}` | Poll async job status |

### Sharing

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/v1/research-studies/{id}/share` | Enable/disable sharing |
| `GET` | `/v1/research-studies/{id}/share` | Check current share state |

### Media Attachments

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/v1/media-assets` | Upload image/PDF for question attachments |

### Agents

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/v1/agents/find` | Find one matching persona |
| `GET` | `/v1/agents/search` | Search personas by demographics |

### Natural Language Requests

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/v1/research-study-requests` | Create study from plain-text brief |
| `POST` | `/v1/research-group-requests` | Create group from description |
| `GET` | `/v1/research-group-requests/{id}` | Get group request status |

### Zeitgeist Surveys

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/v1/zeitgeist/surveys/create` | Create quick survey with answer options |
| `GET` | `/v1/zeitgeist/surveys/{id}/results` | Get survey results |
| `DELETE` | `/v1/zeitgeist/surveys/{id}` | Delete survey |

### Free Tier

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `POST` | `/v1/free/questions` | Ask question to shared free-tier group |

## Demographic Filters

Filters go inside the `filters` dict when recruiting:

| Filter | Type | Examples | Notes |
|--------|------|----------|-------|
| `country` | string | `"USA"`, `"UK"`, `"Canada"`, `"Germany"` | Required. Only these 4 supported |
| `state` | string | `"TX"`, `"MI"`, `"CA"` | **2-letter codes ONLY** |
| `city` | string | `"Austin"`, `"Detroit"` | Supported but narrows pool |
| `age_min` | integer | 25, 30, 45 | Recommended |
| `age_max` | integer | 45, 55, 65 | Recommended |
| `gender` | string | `"male"`, `"female"`, `"non_binary"` | Optional |
| `is_parent` | boolean | `true`, `false` | Good for family/consumer |
| `education` | string | `"high_school"`, `"bachelors"`, `"masters"`, `"phd"` | Optional |
| `industry` | array | `["Healthcare", "Technology"]` | Optional |

**NOT supported:** `income`, `employment`, `ethnicity`, `political_affiliation`.

**If 0 agents returned:** Broaden filters — remove state, remove industry,
widen age by +/- 10 years. Try up to 10 relaxation attempts.

**Industry proxies:** Auto mechanics → `Automotive Manufacturing`,
restaurant owners → `Food & Beverages`, pet owners → use age filter (not `Veterinary`),
security professionals → `Cybersecurity`.

## Media Attachments

Upload screenshots, ad creative, or PDFs before asking questions:

```bash
# Upload image
curl -s -X POST "https://app.askditto.io/v1/media-assets" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "data_url": "https://example.com/screenshot.png",
    "filename": "landing-page.png",
    "mime": "image/png"
  }'
```

Or use base64: `"file_data": "base64_encoded_content"`, `"encoding": "base64"`.
Allowed types: PNG, JPEG, GIF, WEBP, PDF.

Save the `media_asset.id` and pass as `attachments` when asking questions:

```bash
curl -s -X POST "https://app.askditto.io/v1/research-studies/STUDY_ID/questions" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "Look at this landing page. What would make you trust this product?",
    "attachments": [MEDIA_ASSET_ID]
  }'
```

## Polling Strategy

| Study size | First poll delay | Subsequent interval | Strategy |
|-----------|-----------------|---------------------|----------|
| 10 personas | 45-50 seconds | 20 seconds | Poll ONE job_id as proxy |
| 15-20 personas | 60 seconds | 20 seconds | Poll ONE job_id as proxy |
| Free tier (~12) | 45 seconds | 20 seconds | Poll ONE job_id |

All jobs from the same question finish together — no need to poll each one.
If a job returns `failed`, report partial failure. Do NOT auto-retry.

## Response Structure

**Study creation:** `response["study"]["id"]` (nested under `study` key)

**Question responses** (from `GET /v1/research-studies/{id}/questions`):
- `response_text` — the persona's answer (may contain HTML: `<b>`, `<ul>`, `<li>`)
- `agent_name`, `agent_age`, `agent_city`, `agent_state`, `agent_country`
- `agent_occupation`, `agent_summary`

**Completion results:** `overall_summary`, `key_segments`, `divergences`,
`shared_mindsets`, `next_questions`, `stats`.

**Share link:** Prefer `share_link` field. If absent, use `share_url`.

## Reading Responses

After polling is complete, retrieve all Q&A data:

```bash
curl -s "https://app.askditto.io/v1/research-studies/STUDY_ID/questions" \
  -H "Authorization: Bearer $DITTO_API_KEY"
```

Use agent demographics to segment findings by age, location, occupation.

## What You Can Test

- **Pain points** — Do customers actually have this problem?
- **Pricing** — What price is a bargain? A stretch? A dealbreaker?
- **Positioning** — Which tagline/value prop resonates most?
- **Features** — What do customers prioritise vs. ignore?
- **Landing pages** — Upload screenshots and get qualitative reactions
- **Ad creative** — Test headlines, images, and messaging variants
- **Competitive switching** — What would make someone leave their current solution?
- **Deal breakers** — What kills the sale even if the product is good?

## Common Mistakes

- Using `size` instead of `group_size` in recruitment
- Using numeric `id` instead of string `uuid` for `research_group_uuid`
- Using `group_id` with the `append` endpoint (it requires `group_uuid`)
- Using `response["id"]` instead of `response["study"]["id"]` for study creation
- Passing `agent_ids` as strings/UUIDs instead of `list[int]`
- Polling every 10-15s (too aggressive — use 45-50s first, then 20s)
- Batching questions (ask one, poll to completion, then ask next)
- Using full state names ("Michigan") instead of 2-letter codes ("MI")
- Including `income` or `employment` filters (not supported)
- Skipping the `complete` step (you miss AI-generated analysis)
- Asking closed-ended yes/no questions (use open-ended instead)
- Asking leading questions ("Don't you think X is great?")
- Designing questions before researching the product/market
- Introducing the brand in Q1 (introduce at Q3 earliest to avoid bias)

## Limitations

Ditto personas have NOT used your specific product. For:
- Actual UX feedback from real users → use real user testing
- Legal/compliance decisions → use human research
- Safety-critical decisions → use human validation
- Exact quantitative metrics (NPS, conversion) → use real data

**Recommended hybrid:** Ditto for the fast first pass (80% of insight),
then human research for the remaining 20% requiring real customer nuance.

## Error Handling

| Error | Cause | Fix |
|-------|-------|-----|
| 409 Conflict | Study already completed | Retry with `"force": true` |
| 429 Too Many Requests | Rate limited | Wait 30-60s, serialize requests |
| 0 agents returned | Filters too narrow | Broaden: remove state/industry, widen age |
| Job status `failed` | Persona generation error | Report partial failure, don't auto-retry |
| 500/502/504 | Server error | Wait 15-30s, retry once |

## Further Reading

- Full API guide: https://askditto.io/claude-code-guide
- Question design: @question-playbook.md
- Worked examples: @examples/carequarter.md
- API endpoint reference: @api-reference.md
