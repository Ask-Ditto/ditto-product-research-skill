# Ditto Product Research — Claude Code Skill

Run customer research, pricing tests, and product validation studies
using [Ditto](https://askditto.io)'s synthetic persona platform — directly
from your terminal via Claude Code.

## What This Skill Does

When installed, Claude Code automatically loads this skill whenever you
ask it to do customer research, validate a product idea, test pricing,
or run any kind of market analysis. You can also invoke it manually
with `/ditto-product-research`.

Claude will:
- Design research studies with proven question frameworks
- Recruit demographically filtered synthetic panels
- Run studies via the Ditto API (asking questions, polling for responses)
- Extract insights and generate reports
- Produce shareable links for stakeholders

## Installation

Copy the `ditto-product-research` folder into your Claude Code skills
directory:

```bash
# For a specific project
cp -r ditto-product-research /path/to/your/project/.claude/skills/

# For all your projects (personal skills)
cp -r ditto-product-research ~/.claude/skills/
```

## Setup

Get a free Ditto API key (no credit card required):

```bash
curl -sL https://app.askditto.io/scripts/free-tier-auth.sh | bash
```

Or visit: https://app.askditto.io/docs/free-tier-oauth

Set it as an environment variable:

```bash
export DITTO_API_KEY="rk_free_YOUR_KEY_HERE"
```

## Usage

Just ask Claude Code to do research:

```
"Validate whether there's a market for an AI-powered recipe app for
people with dietary restrictions. Test with 10 US adults aged 25-45."

"Run a pricing study for my SaaS product. Test price sensitivity
between $9/month and $49/month with 10 personas."

"Test these three positioning options for my landing page with
10 Canadian millennials and tell me which one wins."
```

Or invoke the skill directly:

```
/ditto-product-research "Test pricing for an AI writing assistant targeting freelance writers"
```

## What's Included

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill file — workflow, question framework, gotchas |
| `api-reference.md` | Complete API endpoint reference |
| `question-playbook.md` | Question design patterns for different study types |
| `examples/quick-start.md` | 5-command quick start guide |
| `examples/carequarter.md` | Full 3-phase worked example (32 personas, public study links) |

## Requirements

- Claude Code (any version with skills support)
- A Ditto API key (free tier works, paid unlocks demographic filtering)
- `curl` available in your terminal

## Links

- **Ditto:** https://askditto.io
- **Full API Guide:** https://askditto.io/claude-code-guide
- **Question Design Guide:** https://askditto.io/claude-code-guide/question-design
- **Case Studies:** https://askditto.io/case-studies

## About Ditto

Ditto maintains 300,000+ AI-powered synthetic personas calibrated to
census data across 15+ countries. EY validated the methodology at 92%
statistical overlap with traditional focus groups. Studies complete in
15-30 minutes — the traditional equivalent takes 4-8 weeks and costs
$10,000-50,000.
