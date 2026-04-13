# Infoseek Google Analytics

`infoseek-google-analytics` is a local AI assistant skill for running GA4 weekly reports through the `infoseek_google_analytics` MCP and optionally drafting or sending the report to the authenticated Gmail account.

This skill is designed to work with an Infoseek MCP server. Without that MCP, the skill has limited value on its own.

## Requirements

- Codex or Claude with local skills enabled
- Access to the `infoseek_google_analytics` MCP
- Google OAuth completed for the analytics account you want to query

## Install

Copy this folder into your local skills directory:

```text
Codex:  $CODEX_HOME/skills/infoseek-google-analytics
Claude: ~/.claude/skills/infoseek-google-analytics
```

Then restart your client so the skill is loaded.

## Configure MCP

This skill expects the following MCP name in your client setup:

- Required: `infoseek_google_analytics`

The skill also relies on Gmail tools exposed by the same MCP for draft and send workflows.

If the MCP is missing or unauthenticated, the skill should stop and report that Google Analytics access is unavailable in the current setup.

## What It Does

- Runs weekly GA4 key metrics reports
- Runs weekly sessions-by-source reports
- Formats time-series output into readable fixed-width matrices
- Uses native MCP OAuth behavior instead of manual auth links
- Optionally drafts or sends an HTML email version of the report to the authenticated Gmail user

## Configuration

Before relying on the skill, set your GA4 property IDs in `SKILL.md`:

```text
PROPERTY_IDS = ["YOUR_GA4_PROPERTY_ID"]
```

You can also override the property IDs in chat by providing `property_ids` explicitly.

## Example Prompts

- `Use $infoseek-google-analytics to show weekly key metrics for property 123456789.`
- `Use $infoseek-google-analytics to show weekly sessions by source for property_ids ["123456789"].`
- `Use $infoseek-google-analytics to run the key metrics weekly report for 123456789 ending 2026-04-11.`
- `Use $infoseek-google-analytics to email me that report.`

## Files

- `SKILL.md`: skill instructions and workflow
- `agents/openai.yaml`: UI metadata for Codex
- `assets/google_analytics-icon.svg`: icon asset
