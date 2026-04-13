---
name: infoseek-google-analytics
description: Use when the user wants GA4 weekly reporting through the `infoseek_google_analytics` MCP, including key metrics trends, sessions by source/medium, readable report formatting, or emailing the report to themselves.
---

# Infoseek Google Analytics

Use this skill as a helpful assistant that is expert in the `infoseek_google_analytics` MCP for Google Analytics reporting.

The goal is to help the user retrieve one of the weekly GA reports from the Google MCP, present the results cleanly in markdown-friendly output, and optionally email the report to the user's own Gmail account using the Gmail tools in the same MCP.

## User Configuration

Users should hardcode their GA4 property IDs in this placeholder before relying on the skill:

`PROPERTY_IDS = ["YOUR_GA4_PROPERTY_ID"]`

If the user has configured that placeholder with one or more real property IDs, use those values as the default `property_ids` for report tool calls.

If the placeholder is still unset or clearly contains only example text, instruct the user to replace it with their real GA4 property ID list before running reports.

## Required MCP

- `infoseek_google_analytics`

If the MCP is missing or unavailable, stop and tell the user Google Analytics access is unavailable in the current setup.

If the MCP requires OAuth, do not guess or handcraft auth URLs for the user. Use the MCP client's native OAuth flow.

## Supported Report Tools

Treat the analytics MCP as exposing these analytics tools:

- `google_analytics_fetch_key_metrics_weekly`
- `google_analytics_fetch_sessions_by_source_weekly`

Also use these Gmail tools from the same MCP when email follow-up is needed:

- `who_am_i`
- `draft_email`
- `send_email`

## Core Rules

- The skill should rely on the hardcoded `PROPERTY_IDS` placeholder when it has been configured with real values.
- If the placeholder is not configured and the user has not explicitly provided `property_ids` in chat, instruct them to set the placeholder or provide one or more GA4 property IDs. Do not guess.
- Pass `property_ids` exactly as user-supplied strings. Do not invent or rewrite IDs.
- Use `end_date` only if the user gives one or asks for a specific cutoff date.
- Present report output as markdown tables that include all returned columns and data for the report.
- After generating a report, always ask whether the user wants it emailed to themselves.
- Do not send email unless the user explicitly says to send it now.
- If the user wants the email but does not explicitly say send, default to creating a draft.
- Use `who_am_i` to get the authenticated Gmail address before drafting or sending to self unless it is already known in the current thread.
- For email delivery, convert the report into Gmail-compatible HTML with inline-styled `<table>` markup so the message renders cleanly.

## OAuth Behavior

- Treat Google Analytics auth here as standard MCP OAuth, not a custom browser-link workflow.
- If a report tool returns `oauth_required` or equivalent auth-needed metadata, the agent should initiate the MCP client's native OAuth flow automatically.
- For Codex or other MCP clients, prefer truthful `authorization_server` and protected-resource metadata. Do not replace that with a guessed `oauth_start_url` or a manually assembled auth link.
- Do not ask the user to copy, paste, or inspect OAuth URLs unless the client has already failed to launch its own auth flow.
- If a browser consent step is required, instruct the user only to complete the consent flow that the MCP client opened.
- After the user completes consent, retry the original analytics tool call automatically.
- Only report auth as unavailable if the MCP is missing, broken, or the native OAuth flow fails.

## Codex OAuth Contract

- In Codex, a tool response with `oauth_required` means the MCP client or harness should launch native MCP OAuth automatically.
- The agent must not fabricate auth URLs, convert the flow into a manual browser-link workflow, or ask the user to inspect OAuth metadata unless the native Codex OAuth flow has already failed to start.
- If the native OAuth flow does not launch automatically after an `oauth_required` response, treat that as a Codex or harness bug rather than a reason to fall back to hand-built auth links.
- Expected Codex behavior:
  1. the analytics tool call returns `oauth_required`
  2. Codex launches native MCP OAuth using `authorization_server` and protected-resource metadata
  3. the user completes consent in the browser
  4. Codex stores credentials locally
  5. the agent retries the original analytics tool call
- Do not tell the user Google Analytics access is unavailable merely because OAuth is required. Only report unavailability if the MCP is missing, broken, or the native Codex OAuth flow fails.

## Task Mapping

- "Show my weekly key metrics":
  `google_analytics_fetch_key_metrics_weekly`
- "Show weekly sessions by source":
  `google_analytics_fetch_sessions_by_source_weekly`
- "Email me this report":
  `who_am_i` then `draft_email` or `send_email`

## Required Input Handling

- If the user asks for a report without a property ID, respond with a short instruction such as:
  "Please set `PROPERTY_IDS = [\"YOUR_GA4_PROPERTY_ID\"]` in the skill or provide `property_ids: [\"YOUR_GA4_PROPERTY_ID\"]`, and I can run the report."
- If the user gives one ID in prose, normalize it into a single-item `property_ids` array for the tool call.
- If the user gives several IDs, pass them all through in the order given.
- If the placeholder is configured and the user does not override it in chat, use the placeholder values.

## Output Formatting

Always summarize the report briefly.

For any report that contains over-time or time-series metrics, prefer a readable fixed-width text matrix inside a fenced `text` code block instead of markdown tables with array-valued cells.

Use this matrix style whenever values are organized across weeks or other time buckets:

- first column: row label such as `Metric` or `Source / Medium`
- one column per time bucket
- each time bucket column may use a two-line header when needed, such as:
  - first line: bucket start date
  - second line: bucket end date
- each row should show raw values, and include parenthetical deltas only when the tool also returns change values

General rules for over-time formatting:

- keep entities as rows and time buckets as columns
- include all returned buckets in order
- include all returned rows in order unless the tool returns a separate totals or other section
- use a fenced `text` code block so columns stay aligned
- do not dump raw arrays into markdown table cells when a matrix would be more readable

### Key Metrics Weekly

Render:

- one readable fixed-width text matrix inside a fenced code block

Format it exactly as a week-by-week matrix:

- first column: `Metric`
- one column per week
- each week column should use a two-line header:
  - first line: `start_date`
  - second line: `end_date`
- each metric row should show:
  - the raw weekly value for the first week
  - for later weeks, `value (pct_change%)`

Formatting rules:

- use a fenced `text` code block so columns stay aligned
- keep metrics as rows, weeks as columns
- include all returned weeks in order
- include all returned metrics in order
- render percent deltas with explicit signs like `(+21%)` or `(-4%)`
- if the percent change is `null`, show only the raw value with no parenthetical
- if a metric is all zeros, still include the row

Example shape:

```text
Metric             2026-02-15     2026-02-22
                   2026-02-21     2026-02-28
------------------------------------------------
sessions           56             104 (+86%)
users              26             56 (+115%)
```

### Sessions by Source Weekly

Render:

- one table for `weeks`
- one table for `rows`
- one table for `other` if present
- one table for `totals`

Use these columns:

- `weeks` table:
  - `start_date`
  - `end_date`
- `rows` table:
  - `source_medium`
  - `weekly_sessions`
  - `total_sessions`
  - `latest_week_sessions`
- `other` table:
  - `source_medium`
  - `weekly_sessions`
  - `total_sessions`
  - `latest_week_sessions`
- `totals` table:
  - `weekly_sessions`
  - `total_sessions`

## Email Follow-Up Workflow

After any analytics report succeeds, ask one short follow-up:

- "Do you want me to email this report to your Gmail account?"

If the user says yes:

1. call `who_am_i` unless their authenticated email is already known in the thread
2. build an HTML email body with:
   - a short heading naming the report
   - a short metadata section showing `property_ids`, `requested_end_date`, and `resolved_end_date` when present
   - one HTML table per markdown table shown in chat
3. choose an email subject line personalized to the specific report name
   - prefer descriptive subjects such as `Google Analytics Trailing Weekly Key Metrics` or `Google Analytics Weekly Sessions by Source`
   - if the user explicitly provides a subject, use it exactly
   - otherwise, derive the subject from the report type instead of using a generic fallback
4. use inline styles on the tables and cells for Gmail compatibility
5. if the user explicitly says send, use `send_email`
6. otherwise use `draft_email`

## HTML Table Requirements

Use plain HTML tables with inline styles only. Do not rely on CSS classes or `<style>` blocks.

Recommended structure:

- `<table style="border-collapse:collapse;width:100%;font-family:Arial,sans-serif;font-size:14px;">`
- header cells with dark text, light background, 1px borders, and padding
- body cells with 1px borders, top alignment, and padding
- wrap arrays and long values in `<code>` or escaped text within the cell when useful

Keep the email body clean and businesslike. Optimize for readability in Gmail web and mobile clients.

## Response Style

- Be concise and practical.
- If OAuth is needed, say that you are starting the Google consent flow through the MCP client. Do not paste or invent an auth URL unless the client cannot launch auth natively.
- After report generation, explain what report was run and for which property IDs.
- For any over-time report, use the fixed-width fenced text matrix format above rather than markdown tables with raw arrays.
- Use markdown tables only for non-time-series sections such as metadata, totals, or small auxiliary sections when that is clearer.
- After the tables, ask whether the user wants the report emailed to themselves.
- After email draft/send success, state whether a draft was created or the email was sent, and to which address.

## Example Requests

- "Use $infoseek-google-analytics to show weekly key metrics for property YOUR_GA4_PROPERTY_ID."
- "Use $infoseek-google-analytics to show weekly sessions by source for property_ids [\"YOUR_GA4_PROPERTY_ID\"]."
- "Use $infoseek-google-analytics to run the key metrics weekly report for YOUR_GA4_PROPERTY_ID ending 2026-04-11."
- "Use $infoseek-google-analytics to email me that report."
