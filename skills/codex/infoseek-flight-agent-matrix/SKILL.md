---
name: infoseek-flight-agent-matrix
description: Search flights across major OTAs, rank the best itineraries, and optionally email an HTML summary to the authenticated Gmail account.
---

# Flight Agent Matrix

Use this skill when the user wants to search or compare flights across multiple providers, rank the best options, or email themselves a summary.

## Required MCPs

- Required: `infoseek_flight_agent_matrix`
- Optional: `infoseek_gmail`

## MCP Availability

If `infoseek_flight_agent_matrix` is missing, unavailable, or not authenticated, stop and tell the user that this skill requires an Infoseek-hosted MCP. To obtain access, they should contact `hello@infoseek.ai` and Infoseek will reply with their own custom instance and setup details.

If `infoseek_gmail` is missing, unavailable, or not authenticated, continue with the on-screen flight comparison and skip the email offer.

## Inputs To Gather

Collect these values before running the flight comparison tool:

- `Leaving`: city or airport code
- `Going to`: city or airport code
- `Depart Date`: accept `MM/DD/YYYY` from the user, then convert to `YYYY-MM-DD`
- `Return Date`: optional, convert to `YYYY-MM-DD` when present
- `Flight Class`: optional, default `economy`
- `Max Flight Duration`: optional, default `30` hours
- `Max Stops`: optional, default `1`

If the user gives cities instead of airport codes, ask a short follow-up only if the intended airport is ambiguous. Otherwise use the most obvious airport code or the code the user already supplied.

## Tool Call

Call `compare_flights_all_providers` from `infoseek_flight_agent_matrix` with:

- `from_airports`: array of airport codes
- `to_airports`: array of airport codes
- `departure_date`
- `return_date` when present
- `flight_class`: one of `economy`, `premium-economy`, `business`, `first`
- `max_total_duration_hours`
- `max_stops_allowed`

Leave provider selection unset unless the user explicitly asks for a subset.

## Ranking

Pick the top 10 itineraries using price as the primary factor, then duration, then stops. A practical ranking formula is:

- score = price + (tripHours * 12) + (stops * 35)

with:
tripHours = totalTripMinutes / 60 if available, otherwise leg duration fallback
missing maxStops defaults to 2
Lower score ranks higher. Tie-breakers are:

lower total trip minutes
lower price
fewer stops

Before ranking the final display rows, group together duplicate itineraries offered by multiple providers. Treat itineraries as duplicates when the flight legs materially match, including route, carrier, departure time, arrival time, duration, and stops, even if the provider differs.

Within each duplicate group:

- sort provider offers by lowest price first
- treat the cheapest offer as the primary result
- keep the other provider offers attached to that same result instead of listing them as separate top-level itineraries

If provider payloads are incomplete, make the best comparison you can from available price, duration, and stop data, and note missing values clearly.

## Output Format

Render an easy-to-read Markdown table with these columns:

- `Price`
- `Airline`
- `From/To`
- `Depart Time`
- `Arrival Time`
- `Duration`
- `Stops`
- `Booking Link`
- `Notes`

Formatting rules:

- Show up to 10 ranked itineraries.
- Each ranked itinerary may contain multiple provider offers if duplicates were grouped.
- For Markdown output, use one table row per flight leg. Do not place multiple legs in a single cell.
- Do not use HTML such as `<br>` inside Markdown table cells.
- For round trips, the outbound leg should be followed by a second row for the return leg.
- In the return-leg row, leave `Price` and `Booking Link` empty rather than repeating them.
- For grouped duplicate-provider offers, show the cheapest provider first as the main row for that itinerary. 
- Immediately below the main itinerary row, add one additional comparison row per duplicate provider offer. For these secondary rows leave *all columns empty* except Booking Link and Notes, so that the row is not repetitive.
- For duplicate-provider comparison rows, make it visually clear in `Notes` that the row is an alternate booking option for the itinerary above.
- Label booking links with the destination domain, for example `google.com`, `skyscanner.com`, or `booking.com`.
- Do not hide duplicate-provider offers only in `Notes`; they should appear as their own comparison rows directly under the main itinerary.
- If a value is unavailable, use `—`.

## Email Step

After showing the table, ask whether the user wants the summary emailed to themselves.

Only if the user explicitly says yes:

1. Call `who_am_i` from `infoseek_gmail` to get the authorized Gmail address.
2. Build a Gmail-compatible HTML table with the same ranked results.
5. Send the email with `send_email` from `infoseek_gmail`.

If `infoseek_gmail` is not available at that point, tell the user email sending is unavailable in the current setup and continue without sending.

Subject format:

- `<origin> to <destination> <depart-date> to <return-date> <flight-class> flight Search Summary`

If no return date exists, omit the return-date segment rather than inventing one.

## Guardrails

- Do not send email unless the user explicitly confirms.
- Keep the final answer focused on the ranked options, not raw provider dumps.
- Preserve exact dates in the final response after conversion so the user can verify the search window.
- Always print out the full matrix markdown verbatim with no changes so the reader has it for posterity
