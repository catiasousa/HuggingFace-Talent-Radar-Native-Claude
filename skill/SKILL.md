---
name: huggingface-talent-radar
description: Source and score ML/AI candidates from the Hugging Face Hub and maintain a live, editable candidate tracker — the Claude-native replacement for an n8n + Claude + GitHub + Airtable Hugging Face sourcing workflow. Use when the user wants to source candidates from Hugging Face, find ML engineers/researchers by published models, refresh an existing role's Hugging Face tracker, or mentions "huggingface talent radar" or replacing that n8n/Airtable automation.
---

# Hugging Face Talent Radar

Replaces an n8n + Claude + GitHub + Airtable Hugging Face sourcing workflow with a Claude-native pipeline: search the Hub across multiple angles, enrich each profile, cross-reference GitHub as a best-effort lead, score, and keep a live editable tracker.

## 1. Gather inputs

Ask for (or infer from context already given):
- Role context: title, must-haves, nice-to-haves, seniority
- Search angles to run, each with a concrete value:
  - Task — a `pipeline_tag` (e.g. `text-generation`, `image-classification`)
  - Trending — no value needed, just opt in
  - Organization — a Hugging Face org/account name
  - Library — a library/framework tag (e.g. `transformers`, `diffusers`)
- New run vs. refresh: if this is a follow-up for a role already tracked, ask for that tracker's Artifact URL (or find it via `Artifact` `list`, matching title `"<Role> — Hugging Face Talent Radar"`) instead of creating a duplicate.

## 2. Run the search angles

Hugging Face's public Hub API doesn't require authentication for this read-only usage, and — unlike `api.github.com` — is not blocked by the cloud sandbox's egress proxy, so WebFetch works directly for all of these:

- **By task**: `GET https://huggingface.co/api/models?pipeline_tag=<task>&sort=downloads&direction=-1&limit=50`
- **Trending**: `GET https://huggingface.co/api/models?sort=likes7d&direction=-1&limit=50`
- **By organization**: `GET https://huggingface.co/api/models?author=<org>&sort=downloads&direction=-1&limit=50`
- **By library**: `GET https://huggingface.co/api/models?library=<lib>&sort=downloads&direction=-1&limit=50`

Each model result includes an `author` field — that's the candidate's Hugging Face username (or an org name; see step 3). Merge all results, dedupe by author, and record which angle(s) surfaced each person as their `source`. Keep track of each candidate's top-performing model (highest downloads) for enrichment.

## 3. Filter before enriching

Drop authors that are organizations, not individuals — you can usually tell from the name pattern, and confirm in step 4 (an org account 404s on the user overview endpoint; if it does, drop it rather than treating it as a dead profile). If the merged list is large, prioritize by download/like count on their top model rather than enriching everyone.

## 4. Enrich

For each remaining candidate: `GET https://huggingface.co/api/users/<username>/overview` — this returns their model/dataset counts, follower count, and organization memberships. If this 404s, the account is an org, not a person — drop it (per step 3) rather than erroring out.

Best-effort GitHub cross-reference: try to find a matching GitHub profile (username match, or a link surfaced on their Hugging Face profile page). Treat this as an inferred signal, not a verified fact, and label it as such in the tracker.

GitHub access note: direct calls to `api.github.com` from the cloud sandbox are blocked by a proxy restriction regardless of any token — use WebFetch for GitHub lookups, not Bash/curl. WebFetch hits GitHub's API unauthenticated on a shared IP and can 403 under load; retry once or twice. If the user has linked their computer, GitHub calls can run from there instead for higher, token-backed rate limits. (This restriction does not apply to the Hugging Face API calls above — those work fine from the cloud sandbox.)

## 5. Exclude already-tracked / do-not-contact

Before scoring, check the existing tracker (if one exists for this role) for this candidate by `hf_username`. Skip anyone already marked `do_not_contact` or already present with unchanged top-model data. This avoids duplicate entries across repeated runs.

## 6. Score

For each remaining candidate, produce:
- `fit_score` (0–100)
- `key_strengths` (from their published models and any GitHub signal — cite specific, real work, never invented)
- `key_gaps`
- `priority_action` (concrete next step)
- `profile_summary` (2–3 sentences)
- `outreach_hook` — a personalized line grounded in their actual published model work
- `estimated_seniority` and `impact_signal` — inferred from model count, download/like volume, and org affiliations; label as inferred, not verified

Default qualification bar (carried over from the original n8n template): `fit_score > 55`. Only candidates above the bar get written to the tracker — adjust per role at the user's request.

## 7. Create or update the live tracker

Use the Artifact tool with the `db` capability. One tracker per role, titled `<Role> — Hugging Face Talent Radar`. If a tracker for this role already exists, update it (add new candidates, don't duplicate existing ones by `hf_username`) rather than creating a new one.

Tracker doc ID: the candidate's Hugging Face username.

Workflow-populated fields: `hf_username`, `full_name`, `hf_url`, `github_url`, `source`, `source_role`, `date_sourced`, `fit_score`, `priority_action`, `key_strengths`, `key_gaps`, `outreach_hook`, `profile_summary`, `top_model_id`, `top_model_downloads`, `top_model_task`, `num_models_published`, `hf_followers`, `hf_organisations`, `estimated_seniority`, `impact_signal`.

Manual outreach fields (left for the user to fill in via the tracker UI, never overwritten by a re-run): `contacted`, `contact_date`, `replied`, `reply_sentiment`, `do_not_contact`, `notes`.

## 8. Report back

Summarize: how many candidates were found/enriched/scored, how many cleared the fit bar, the top 2-3 candidates and why, and a link to the tracker. Flag any GitHub cross-references that are low-confidence guesses so the user knows to verify before outreach.

## Notes

- This skill can run on demand or on a schedule.
- Never store a GitHub token in the tracker, in this skill file, or anywhere written to disk — if the user provides one for a session, use it only for that session's API calls.
- `estimated_seniority` and `impact_signal` are inferred from public activity, not verified employment/experience data — label them as such so outreach messages don't overstate certainty.
