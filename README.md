# Hugging Face Talent Radar — Claude Edition

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A Claude-native fork of [Huggingface-Talent-Radar-with-Claude-n8n-Airtable](https://github.com/search?q=Huggingface-Talent-Radar-with-Claude-n8n-Airtable+user%3Acatiasousa&type=repositories), same sourcing pipeline, rebuilt to run entirely inside Claude instead of n8n + Airtable.

## What this project does

This project builds a Hugging Face sourcing system you run through Claude. It searches the Hugging Face Hub using multiple discovery methods, evaluates each profile itself (no separate AI API call, Claude does the scoring in-context), ranks candidate fit for your target role and keeps high-signal candidates in a live tracker page with a personalized outreach line for each one.

Ask Claude any time you want a fresh batch for a role. With a computer linked and a scheduled task set up, it can also run unattended on a cadence you choose, but that's opt-in, not required.

## Why Hugging Face

Hugging Face is where ML practitioners publish and share actual working models, not just resumes or profiles. Someone who has published models people actually download and use is showing direct, verifiable technical output, often before that work shows up anywhere else.

## Workflow logic explained

This pipeline applies a consistent sourcing process to Hugging Face profiles, run whenever you ask.

It begins with multiple discovery paths across the Hub (by task, by trending activity, by organization and by library) then merges and de-duplicates profiles so each candidate is evaluated once.

Each profile is enriched with practical context from their published models, then scored by Claude directly (structured fit score, strengths, gaps and a priority action, with no separate scoring call or JSON round-trip needed). 

Candidates are saved to a live tracker page with standardized fields and outreach hooks grounded in their actual published work, so your review and contact process stays fast and repeatable.

## Who this is for (and not for)

**This is for you if**

- You recruit ML/AI engineers or researchers and want sourcing signals from real published model work.
- You want to identify candidates through practical indicators such as model downloads, trending activity and organizational affiliation.
- You want an on-demand (or scheduled) workflow that scores profiles, prioritizes outreach and writes structured candidate records to a live tracker.

**This is not for you if**

- Your role has no meaningful overlap with applied ML/model publishing.
- You cannot use external APIs or automated profile analysis due to policy constraints.

## The search methods this workflow runs

This workflow runs up to 4 searches and merges results into one ranked candidate pipeline. You choose which ones to use for each run.

**Search 1 — By Task.** Find practitioners active in a specific ML task (e.g. text-generation, image-classification).  
What it does: pulls top models for a given `pipeline_tag`, ranked by downloads.  
Why it matters: surfaces people actively publishing in your exact domain.

**Search 2 — Trending.** Find practitioners with recent momentum.  
What it does: pulls models ranked by likes in the last 7 days.  
Why it matters: captures current activity, not just historical output.

**Search 3 — By Organization.** Mine a specific company or lab's published models.  
What it does: pulls models published under a given org account.  
Why it matters: supports competitor and peer-organization sourcing.

**Search 4 — By Library.** Find practitioners working with a specific framework or tool.  
What it does: pulls models tagged with a given library.  
Why it matters: surfaces people with hands-on depth in a specific stack.

Note: you give Claude your search criteria in plain language each time you ask it to source (see Customizing below). 

## Accounts you need to create

- **Hugging Face** (huggingface.co), your source of candidates. No account needed to read public data.
- **Claude** (claude.ai), with Cowork/Artifacts enabled. This is the automation engine, the scoring engine and the candidate database, all in one. It replaces n8n, the separate Anthropic API account and Airtable.
- **GitHub** (optional), used only for best-effort cross-referencing of a candidate's GitHub profile alongside their Hugging Face activity.

## How to get access set up

Hugging Face's public model/user endpoints don't require a token for this workflow's read-only usage. No credentials to set up for Hugging Face itself.

If Claude cross-references GitHub and asks for an optional personal access token to raise API rate limits, give it directly in the chat when prompted (it should never be written into a committed file). 

## Quickstart — two ways to use it

**Option A — Install it as a skill (recommended)**

1. Open a conversation with Claude.
2. Paste the contents of `skill/SKILL.md` or attach the file and ask Claude to save it as a skill.
3. Review and save the skill when Claude shows you the confirmation card.
4. From then on, just tell Claude the role and your search criteria whenever you want a batch sourced.

**Option B — One-off, without installing anything**

Paste `skill/SKILL.md` into a conversation and ask Claude to follow it for a single run. Nothing is saved for next time, but it's a fast way to try it once.

## How the workflow operates

1. You tell Claude the role (a job posting link or description) and your search criteria.
2. Claude runs the Hugging Face discovery searches you specified.
3. Candidate data is normalized, deduped and filtered.
4. Claude enriches each profile and evaluates role fit directly.
5. Qualified candidates are saved to a live tracker page Claude publishes.
6. You review candidates on that page and send outreach, checking off progress as you go.

## Tracker fields required by this workflow

Fields the workflow writes:

`hf_username`, `full_name`, `hf_url`, `github_url`, `source`, `source_role`, `date_sourced`, `fit_score`, `priority_action`, `key_strengths`, `key_gaps`, `outreach_hook`, `profile_summary`, `top_model_id`, `top_model_downloads`, `top_model_task`, `num_models_published`, `hf_followers`, `hf_organisations`, `estimated_seniority`, `impact_signal`

Manual tracking fields (yours to fill in on the tracker page as you work outreach):

`contacted`, `contact_date`, `replied`, `reply_sentiment`, `do_not_contact`, `notes`

## Setup details

**1) Install the skill**

Give Claude `skill/SKILL.md` and ask it to save the skill (see Quickstart, Option A).

**2) Connect access**

No credentials required for Hugging Face itself. Optionally link your computer to Claude for more reliable GitHub cross-referencing and give Claude a GitHub personal access token when it asks and never store it in a file.

**3) Tell Claude the role you're hiring for**

Give it a job posting link or a pasted description and the role title. No config file to edit.

**4) Choose your search criteria**

Tell Claude which of the 4 search methods to use and their values (a task, trending, an org, a library) and you can change this on every run.

**5) Run a test**

Ask Claude to source a small batch and review the tracker it publishes before relying on it further.

**6) Automate it (optional)**

Ask Claude to set up a scheduled task if you want it running on a cadence without you asking each time (this needs a linked computer for reliable GitHub cross-referencing). 

**7) Schedule**

If you set up scheduling, pick your own cadence when asking Claude to create it.

## Customizing

- Change your search criteria on any run to match new hiring goals (nothing to edit in a file).
- Ask Claude to adjust scoring strictness (default qualification bar is `fit_score > 55`) for different seniority or role profiles.
- Ask Claude to add fields to the tracker if your process needs more than what's listed above.

## Repository structure

```text
.
├── .github/workflows/gitleaks.yml
├── .gitleaks.toml
├── .gitignore
├── LICENSE
├── README.md
└── skill/SKILL.md
```

## Security notes

This repo is a public skill template. It should contain logic only, never live secrets or personal data.

**What is safe to publish:** the skill's instructions, API endpoint patterns, field mappings and example queries.

**What must not be committed:** GitHub tokens, Claude API keys, `.env` files, private keys, or real candidate data from any run.

Gitleaks runs via `.github/workflows/gitleaks.yml`. Run locally before pushing:

```bash
gitleaks detect --source . --verbose
```

## License

MIT License. See [LICENSE](LICENSE).
