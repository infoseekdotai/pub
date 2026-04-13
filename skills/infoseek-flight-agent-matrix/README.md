# Infoseek Flight Agent Matrix

`infoseek-flight-agent-matrix` is a Codex skill for comparing flight options across major OTAs, ranking the best itineraries, and optionally emailing a summary to the authenticated Gmail account.

This skill is designed to work with an Infoseek MCP server. Without it, the skill has limited value on its own.

Learn more and request access at [infoseek.ai Flight Agent Matrix](https://infoseek.ai/mcp/c1GoJtQ82i/flight-agent-matrix/).

## Requirements

- Codex with local skills enabled
- Access to the `infoseek_flight_agent_matrix` MCP
- Optional: access to the `infoseek_gmail` MCP for email delivery

## Get Access

To obtain an Infoseek MCP instance, visit [the Flight Agent Matrix page](https://infoseek.ai/mcp/c1GoJtQ82i/flight-agent-matrix/) or contact [hello@infoseek.ai](mailto:hello@infoseek.ai).

Infoseek will reply with setup details for your own custom instance.

## Install

Copy this folder into your local Codex skills directory as:

```text
$CODEX_HOME/skills/infoseek-flight-agent-matrix
```

Then restart Codex so the skill is loaded.

## Configure MCPs

This skill expects the following MCP names in your Codex setup:

- Required: `infoseek_flight_agent_matrix`
- Optional: `infoseek_gmail`

If the required flight MCP is missing or unauthenticated, the skill should stop and direct the user to contact Infoseek for access.

If Gmail is missing or unauthenticated, the skill should still produce the flight comparison but skip email sending.

## What It Does

- Searches flights across multiple major providers
- Ranks itineraries using price, duration, and stops
- Groups duplicate itineraries sold by different providers
- Produces a concise Markdown comparison table
- Optionally emails an HTML summary to the authenticated Gmail user

## Example Prompts

- `Find the best flights from HKG to Seattle on 5/14/26, one way, premium economy, 1 adult`
- `Compare SEA to Barcelona one way business class flights under 18 hours on 11/11/26.`
- `Search SEA to New York, economy, 1 stop max on 11/11/2026 and email me the top options.`

## Files

- `SKILL.md`: skill instructions and workflow
- `agents/openai.yaml`: Codex UI metadata
- `assets/flight-agents.svg`: icon asset

## License

This skill is distributed under the `MIT` license
