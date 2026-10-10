# DORA Critical ICT Third-Party Provider (CTPP) Monitor

For third-party-risk and ICT-risk teams at banks, insurers and investment firms who need
to know, without re-reading a PDF by hand, whether one of their own ICT vendors is on the
ESAs' official **DORA Article 31(9) Critical ICT Third-Party Provider (CTPP) list** -- and
to find out the moment that changes.

Tracks the European Supervisory Authorities' (EBA, EIOPA, ESMA) joint list of
**Critical ICT Third-Party Providers (CTPPs)** designated under Article 31(9) of the
Digital Operational Resilience Act (DORA) -- the EU-level "who is critical enough to be
directly overseen by the ESAs" list, currently 19 providers (first published 18 November
2025: Accenture, AWS, Microsoft, Google Cloud, Oracle, SAP, IBM, Deutsche Telekom, Orange,
and others).

## What you get

One record per currently-designated provider, plus one per provider that was on the list
at your last check but has since dropped off it. Every record carries an explicit
`matchState`: `FIRST_SEEN` (your own baseline run), `NEW` (added since your last check),
`UNCHANGED`, or `REMOVED`. If you pass an optional `watchlist` of your own ICT vendor
names, each record also gets a `matchesWatchlist` flag so you can filter to the vendors
you actually use.

## What it does not do

**This is not the confidential Register of Information.** DORA Article 28(3) requires
financial entities to maintain their own detailed register of ICT contractual
arrangements, reported annually to national competent authorities in a structured
XBRL-CSV format -- that data is supervisory-only and never made public. What the ESAs
*do* publish is the short, high-level outcome of their own criticality assessment: the
names of the providers designated as "critical" at Union level. This Actor tracks that
published outcome, not the private underlying registers.

**The source is a PDF, not an API.** The ESAs' Joint Committee publishes this list as a
short PDF (mirrored identically on eba.europa.eu, eiopa.europa.eu and esma.europa.eu),
not a CSV, JSON feed, or searchable database. No per-provider detail is published either
-- no lead overseer, no sector, no designation date, no services covered -- just the
legal-entity name, in alphabetical order. This Actor fetches whichever ESA mirror
responds (all three are tried, in order, for resilience) and parses that PDF; it cannot
report fields the ESAs themselves do not publish.

No personal data: every entry is a company/legal-entity name (the DORA criticality
criteria under Article 31(2) only apply to large-scale undertakings). As a second-layer
guard, any parsed entry that structurally resembles a bare personal name rather than a
company name is withheld from output entirely, not logged, not shipped.

## Why you might need this

Banks, insurers, investment firms and their own ICT vendors have had DORA's third-party
risk-management obligations in force since January 2025. If one of your vendors is
designated a CTPP, it becomes subject to direct ESA oversight (inspections, binding
recommendations) under Articles 33-39 -- something your own third-party risk register
and board reporting should reflect promptly, not discover by chance months later. The
ESAs reportedly intend to review and republish this list annually; this Actor exists to
catch that review's outcome without you manually re-reading a PDF on a schedule.

## Input

```json
{
  "watchlist": ["Microsoft", "Amazon Web Services", "Google Cloud"]
}
```

`watchlist` is optional (default: empty, meaning "report the full list, no highlighting").
Each entry is matched against every currently-designated provider name, case-insensitive,
as a substring either direction -- so "Microsoft" matches "Microsoft Ireland Operations
Limited" without needing the exact published legal name.

## Sample output

One row per provider, per run:

```json
{
  "providerName": "Microsoft Ireland Operations Limited",
  "matchState": "UNCHANGED",
  "firstSeenAt": "2026-10-10T20:44:30.000Z",
  "lastSeenAt": "2026-11-18T09:12:04.000Z",
  "checkedAt": "2026-11-18T09:12:04.000Z",
  "sourceName": "EBA",
  "matchesWatchlist": true
}
```

A provider dropped since your last check comes back the same way with
`"matchState": "REMOVED"` and `lastSeenAt` frozen at the run it was last present.

## Change tracking across runs

Run this Actor on a schedule (monthly is more than enough given the ESAs' own annual
review cadence). Each provider's first-seen/last-seen timestamps are stored per Actor, so
a run after the ESAs' next annual review immediately reports exactly what was added or
removed, not just a re-dump of the current list.

## Pricing

Pay per run started, plus per provider record produced (currently-designated or
removed-since-last-check). See the Pricing tab for exact rates. Worked example at the
proposed rate (USD 0.05/run + USD 0.01/provider record): a monthly scheduled run against
the full list (today, 19 providers) costs USD 0.05 + 19 x USD 0.01 = **USD 0.24/month**;
weekly instead of monthly is **about USD 0.96/month**. The list barely grows year to year,
so cost does not scale with your watchlist size, only with how often you check.
