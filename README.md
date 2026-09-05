# MFA Placement Signals

An open specification for identifying low-quality and made-for-advertising (MFA)
placements in Google Display Network and Performance Max campaigns, using only
the data in a standard Google Ads placement report.

No API, no crawling, no vendor lock-in. Everything here can be computed from a
`Campaign URL performance` export in a spreadsheet.

- **[SPEC.md](SPEC.md)** — the signals, weights and thresholds
- **[data/](data/)** — anonymised statistics from real accounts
- Reference implementation: [google-ads-placement-report](https://github.com/MarTechTools-com/google-ads-placement-report)
- Free hosted tool: <https://www.displaygateguard.com/tools/placement-report-cleaner>

## Why this exists

Advertisers running Display and PMAX get a placement report listing every domain
their budget reached. In an unaudited account that list runs to thousands of rows,
and a meaningful share of the spend lands on sites that exist to farm ad revenue
rather than to serve readers.

Most published advice on this problem is either a static blocklist that decays
within weeks, or a vendor's black box. Neither helps you decide anything about
*your* account.

This is the third option: a documented, testable ruleset you can implement
yourself, argue with, and improve.

## Design principles

**1. The advertiser's own performance data outweighs the domain name.**
A name is a hint. A click-through rate is evidence. Signals derived from the
report's own numbers carry roughly twice the weight of signals derived from how
a domain is spelled.

**2. Conversions are exculpatory.**
A placement that converts is working, whatever it happens to be called. Name-based
weight is reduced by 80% at one or more conversions and to zero at three or more.

**3. No single weak signal flags anything.**
Scores accumulate against a threshold. A risky TLD on its own is not enough. This
is the difference between a useful tool and one that cries wolf.

**4. Thresholds are relative to the account, not hardcoded.**
"High cost" and "high CTR" mean different things in different accounts. Bars are
computed from the account's own median spend, click-through rate and cost per
conversion.

**5. Signals must be explainable.**
Every flag states the evidence in plain language. A score you cannot explain to a
client is a score you cannot act on.

## What this cannot do

It reads a spreadsheet. It never visits a website. So it cannot tell you whether a
page is actually made-for-advertising, how much of it is advertising, whether the
content is machine-generated, or whether it is somewhere your brand should not be.
Those require fetching and analysing each page.

Treat every output as a prompt to look, never as a verdict.

## A note on naming domains

This repository deliberately publishes **no blocklist**.

Labelling a specific company's website as "made-for-advertising" is a factual claim
about that business, and getting it wrong causes real harm. While preparing this we
audited a 195-domain list that had been circulating and found a major national news
outlet, a chocolate retailer, and two live-sports sites on it.

Patterns can be published safely. Accusations about named companies cannot. If you
build a blocklist from this spec, verify entries yourself and give site owners a way
to appeal.

## Contributing

Signals that are measurable from a placement report, explainable in one sentence,
and testable against real data are welcome. Signals that amount to "this domain
looks funny" are not — that is exactly what this spec exists to replace.

MIT licensed.
