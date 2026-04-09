# Infoseek Tesla Skill

This skill turns Codex into a helpful assistant that is expert in the `infoseek_tesla` MCP.

It can handle direct Tesla commands, status checks, navigation, and higher-level assistant workflows around the car.

## Get Access

If you do not already have a Tesla MCP instance, get one here:

- [https://infoseek.ai/mcp/KpSW0Kw3zX/Tesla/](https://infoseek.ai/mcp/KpSW0Kw3zX/Tesla/)

## What It Supports

- Climate on and off
- Lock and unlock
- Flash lights, honk, and sound ping
- Battery, range, and status checks
- Window and sunroof controls
- Navigation to coordinates, Place IDs, or waypoint routes
- Helpful assistant workflows such as reading the next calendar meeting location and sending it to Tesla navigation when `infoseek_gmail` is available

## Optional Integration

- `infoseek_gmail`

When available, the skill can use Gmail and Calendar context for more advanced workflows. Example:

1. read the next calendar event
2. extract the meeting location
3. resolve it precisely
4. send it to the Tesla

## Files

- `SKILL.md`: agent instructions and tool-use guardrails
- `agents/openai.yaml`: UI metadata for the skill surface
- `assets/tesla.svg`: icon asset

## Notes

- The skill requires the `infoseek_tesla` MCP.
- If the user does not already have the MCP, they can get it at [https://infoseek.ai/mcp/KpSW0Kw3zX/Tesla/](https://infoseek.ai/mcp/KpSW0Kw3zX/Tesla/).
- Vehicle selection must be resolved safely. The skill does not guess vehicle IDs or destinations.
