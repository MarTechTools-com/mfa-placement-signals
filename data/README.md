# Data

Anonymised aggregate statistics from real Google Ads placement reports.

**No domain names appear in these files, and none will be added.** These describe
patterns and rates, not companies. See the note on naming domains in the root
README.

## Files

- `account-2026-09.json` — a 5,183-placement account, 49 days, EUR. Spend
  concentration, waste rates, signal frequency, and flag rate by TLD.

## Notable findings

- **62.8%** of placements cost nothing at all. They consumed impressions and were
  never clicked. The long tail of a placement report is mostly noise.
- **36.8%** of spend went to placements with zero conversions. In an unaudited
  account this is typically the largest single bucket.
- The **top 10 placements carried 29.6%** of spend and the top 100 carried 65.1%.
  Auditing is worth starting at the top, but the tail is where the junk hides.
- Flag rate by TLD varies by roughly **4x** between the most and least affected TLDs
  with meaningful volume, which supports weighting TLD as a signal — but at 3.7%,
  even `.com` produces flags, which is why TLD alone never flags a placement.

Contributions of further anonymised account statistics are welcome. Aggregate only.
