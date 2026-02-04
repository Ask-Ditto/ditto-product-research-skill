# Quick Start: Your First Ditto Study in 5 Commands

Copy-paste these commands to run a complete study. Replace the
placeholders with your details.

---

## Prerequisites

```bash
# Get your free API key (opens browser for Google sign-in)
curl -sL https://app.askditto.io/scripts/free-tier-auth.sh | bash

# Set it as an environment variable
export DITTO_API_KEY="rk_free_YOUR_KEY_HERE"
```

---

## Command 1: Recruit Your Panel

```bash
curl -s -X POST "https://app.askditto.io/v1/research-groups/recruit" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "US Adults 25-50",
    "group_size": 10,
    "filters": {"country": "USA", "age_min": 25, "age_max": 50}
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Group UUID: {d[\"group\"][\"uuid\"]}')"
```

Copy the UUID.

## Command 2: Create Your Study

```bash
curl -s -X POST "https://app.askditto.io/v1/research-studies" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "YOUR STUDY TITLE",
    "objective": "YOUR OBJECTIVE",
    "shareable": true,
    "research_group_uuid": "PASTE_UUID_HERE"
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Study ID: {d[\"study\"][\"id\"]}')"
```

Copy the study ID.

## Command 3: Ask a Question

```bash
curl -s -X POST "https://app.askditto.io/v1/research-studies/STUDY_ID/questions" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"question": "YOUR QUESTION HERE"}' \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Jobs: {d[\"job_ids\"]}')"
```

Wait 60-90 seconds for responses. Poll a job to check:

```bash
curl -s "https://app.askditto.io/v1/jobs/JOB_ID" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Status: {d[\"status\"]}')"
```

Repeat Command 3 for each additional question (wait for all jobs to
show `finished` between questions).

## Command 4: Complete the Study

```bash
curl -s -X POST "https://app.askditto.io/v1/research-studies/STUDY_ID/complete" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"force": false}'
```

Wait 1-2 minutes for AI analysis.

## Command 5: Get Your Share Link

```bash
curl -s -X POST "https://app.askditto.io/v1/research-studies/STUDY_ID/share" \
  -H "Authorization: Bearer $DITTO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"enabled": true}' \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Share URL: {d[\"url\"]}')"
```

Open the URL to see your complete study with AI-generated insights.

---

## What to Ask (Starter Questions)

### Product Validation
1. "Walk me through how you currently handle [task]. What does your typical process look like?"
2. "What's the most frustrating part of [task]?"
3. "What tools or workarounds do you use today? What works? What doesn't?"
4. "If you could fix ONE thing about [task], what would it be?"
5. "Imagine a product that [your value prop]. What's your gut reaction? What excites you? What concerns you?"

### Pricing Test
1. "At what monthly price would [product] feel like a no-brainer?"
2. "At what price would you start to hesitate?"
3. "At what price is it too expensive to consider, no matter how good it is?"
4. "What do you currently pay for the closest alternative?"
5. "What would justify paying more than you're paying now?"

### Landing Page Test
1. "Look at this page. What's the first thing you notice?"
2. "What do you think this product does?"
3. "Would you scroll down or leave? Why?"
4. "What's missing that you'd want to see before making a decision?"
5. "If a friend sent you this link, what would you tell them about it?"

---

## Next Steps

- Read the full question playbook: @question-playbook.md
- See a complete 3-phase example: @carequarter.md
- Browse the API reference: @api-reference.md
- Full documentation: https://askditto.io/claude-code-guide
