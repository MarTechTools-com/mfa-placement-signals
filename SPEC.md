# The specification

Each placement receives a score. Signals accumulate; a placement is flagged when
the total crosses a threshold.

```
score = Σ(performance signal weights) + nameMultiplier × Σ(name signal weights)

nameMultiplier = 0.0   if conversions ≥ 3
                 0.2   if conversions ≥ 1
                 1.0   otherwise

flagged HIGH   if score ≥ 55
flagged MEDIUM if score ≥ 30
```

A materiality floor applies: placements with less than 0.10 in spend **and** fewer
than 100 impressions are never flagged, regardless of score. Reviewing them is not
worth anyone's time.

## Account context

Three values are computed from the report itself and used as reference points.

| Value | Definition |
|---|---|
| `medianPaidCost` | Median cost across placements with cost > 0 |
| `accountCtr` | Total clicks ÷ total impressions |
| `accountCpa` | Total cost ÷ total conversions |

## Performance signals

Derived from the advertiser's own numbers. These carry the most weight because
they are evidence rather than inference.

### `implausible-ctr` — weight 50 (20 if the placement has conversions)

Fires when `impressions ≥ 50`, `clicks ≥ 5`, and CTR is above a volume-dependent bar:

| Impressions | CTR bar |
|---|---|
| ≥ 500 | `max(8%, accountCtr × 6)` |
| ≥ 200 | 10% |
| ≥ 50 | 15% |

The bar tightens as volume falls so that a couple of stray clicks on a tiny
placement cannot trip it.

**Rationale.** Display click-through rates are very low — people rarely click
banners deliberately, and most accounts sit well under 1%. A rate several times the
account average almost never indicates interest. It usually indicates ads placed
where they are hit by accident, or non-human clicking. In our reference account this
was the single most reliable indicator of invalid traffic, and it appears at low
impression volumes: one placement recorded 63 clicks on 75 impressions.

### `zero-click-volume` — weight 45

Fires when `impressions ≥ 1000` and `clicks = 0`.

**Rationale.** A low click rate is normal. Zero at four figures of volume usually
means the ad was never genuinely visible — stacked behind other creatives, rendered
below the fold, or served to software. Note this also matches a well-known brand
with a poor creative match, which is why it is weighted below the threshold for HIGH
on its own.

### `spend-no-return` — weight 30

Fires when `conversions = 0`, `clicks ≥ 5`, and `cost ≥ medianPaidCost`.

**Rationale.** Real traffic and real money with nothing returned. This signal
requires judgement rather than a reflex: it is expected behaviour for upper-funnel
and awareness campaigns, for products with long consideration cycles, and for
placements that are simply not where the decision happens. It is also
indistinguishable from a genuinely worthless placement. Never exclude on this signal
alone without checking campaign type, attribution window and conversion tracking.

### `cpa-outlier` — weight 18

Fires when `conversions > 0` and `cost ÷ conversions ≥ accountCpa × 4`.

**Rationale.** The placement works, but expensively. On low volume a single cheap
conversion elsewhere would swing the average, so this is a prompt, not a problem.

## Name signals

Weak priors about how a domain is spelled. Multiplied down or eliminated when the
placement converts. None is sufficient to flag anything on its own.

### `risky-tld` — weight 28

TLDs that are cheap to register in bulk and consequently over-represented on MFA
inventory:

```
top xyz club online site website space live icu buzz click link work fun
cyou rest bar monster quest sbs cfd lol best pw cc gq ml tk ga
```

Plenty of legitimate sites use these. That is why the weight is below the flagging
threshold.

### Vocabulary groups — weight 26

The domain's second-level label is matched against grouped term lists. First match
wins; groups exist so a flag can state which pattern it matched.

| Group | Representative terms |
|---|---|
| Clickbait | viral, buzz, trending, shocking, unbelievable, secret, exposed, revealed, omg, wow, insane, banned, forbidden, jawdrop, mustsee |
| Gossip / celebrity | gossip, celeb, paparazzi, scandal, starnews |
| Money lure | getrich, easymoney, jackpot, payday, lottery, cryptoprofit, forexsignal, winbig, freecash |
| Health lure | weightloss, miraclecure, detox, antiaging, bellyfat, ketodiet, hairloss, testosterone |
| Content-farm vertical | ringtone, klingelton, wallpaper, shayari, whatsappstatus, quotesdaily |
| Download / streaming farm | freedownload, watchfree, fullmovie, modapk, keygen, crackedapk |
| Job / exam alert farm | sarkarijob, govtjob, jobalert, admitcard, examresult |

Bare words common on legitimate publishers — `news`, `tips`, `guide`, `free`,
`kostenlos`, `gratis`, `deals` — are deliberately excluded. They generate false
positives and carry no information.

### `news-farm` — weight 24

A news-style word (`news`, `nachrichten`, `daily`, `times`, `herald`, `gazette`,
`chronicle`, `bulletin`, `report`, …) **combined with** either a risky TLD or two or
more hyphens. The combination is the signal; the word alone is not.

### `listicle` — weight 22

Names built around ranked lists: `top10`, `top-25`, `best-of`, `7-ways`, `5-things`.

### `punycode` — weight 26

Any label beginning `xn--`. Internationalised domains are routinely used to imitate
well-known brands.

### `deep-subdomain` — weight 14

Four or more labels. Content served from a deep subdomain of a larger network.

### `many-hyphens` — weight 12

Three or more hyphens. Two is common and normal; three starts to correlate.

### `long-name` — weight 8

25 or more characters excluding dots and hyphens.

## Deliberately excluded signals

**Digits anywhere in the domain.** An earlier revision scored digit runs and
numeric suffixes. Against real data this flagged `11880.com` — a well-known German
directory service — and `learngerman100.de`, and rated `autoscout24.com` as HIGH.
The `24` suffix in particular is a normal German naming convention: `weiden24.de`,
`amberg24.de`, `bewerbungsratgeber24.de`. A number in a domain name carries no
information about quality, and scoring it produced the single largest source of
false positives. It now counts for nothing.

**Domain age, WHOIS privacy, and hosting provider.** Not present in a placement
report, and each has legitimate uses.

**Backlink authority / PageRank.** Tested and rejected. Link-authority metrics
measure links, not quality: MFA operators buy links and score acceptably, while
small legitimate publishers score near zero. It would clear the bad and flag the
good. It also requires sending the domain list to a third-party API, which
conflicts with processing the report locally.

## Validation

Against a real account of 14,290 placements and EUR 9,905 of spend:

| | |
|---|---|
| Flagged | 444 (3.1%) |
| High severity | 36 |
| Spend at stake | EUR 1,758.82 (17.8%) |
| Converting placements flagged | 1 (on `cpa-outlier`, correctly) |

The previous revision of this model flagged 3.5x as many placements on the same file, 85% of
them from a single weak name signal, including one with 11.5 conversions flagged for
containing two hyphens. Precision matters more than recall here: a list nobody
trusts gets ignored entirely.

Absolute figures throughout are proportionally scaled to protect the advertiser's
identity. Every ratio, rate and share is the measured value.

See [data/](data/) for the full anonymised statistics.
