# Trace Comparison: v1 vs v2 vs v3

Analysis of `check_traces.py` output comparing the three `trail_guide` prompt versions across the five test prompts (`day-hike-gear`, `overnight-camping`, `three-day-backpacking`, `trail-difficulty`, `winter-hiking`).

## Summary

| Version | Avg prompt tokens | Avg completion tokens | Avg duration | Total trace duration |
|---|---|---|---|---|
| v1 | ~68 | ~606 | ~5.7s | 28.3s |
| v2 | ~110 | ~1,389 | ~11.4s | 56.9s |
| v3 | ~166 | ~1,465 | ~12.3s | 61.8s |

Each version's prompt got longer than the last, and both completion length and latency grew along with it.

## Per-test data

### v1

| Test | Duration | Total tokens | Prompt | Completion |
|---|---|---|---|---|
| day-hike-gear | 5.56s | 478 | 66 | 412 |
| overnight-camping | 5.96s | 759 | 62 | 697 |
| three-day-backpacking | 6.05s | 806 | 74 | 732 |
| trail-difficulty | 5.15s | 628 | 71 | 557 |
| winter-hiking | 5.57s | 700 | 66 | 634 |

### v2

| Test | Duration | Total tokens | Prompt | Completion |
|---|---|---|---|---|
| day-hike-gear | 7.25s | 1,075 | 108 | 967 |
| overnight-camping | 12.89s | 1,774 | 104 | 1,670 |
| three-day-backpacking | 12.60s | 1,740 | 116 | 1,624 |
| trail-difficulty | 10.91s | 1,293 | 113 | 1,180 |
| winter-hiking | 13.21s | 1,611 | 108 | 1,503 |

### v3

| Test | Duration | Total tokens | Prompt | Completion |
|---|---|---|---|---|
| day-hike-gear | 7.39s | 1,030 | 164 | 866 |
| overnight-camping | 14.28s | 1,842 | 160 | 1,682 |
| three-day-backpacking | 13.89s | 1,711 | 172 | 1,539 |
| trail-difficulty | 11.63s | 1,593 | 169 | 1,424 |
| winter-hiking | 14.55s | 1,976 | 164 | 1,812 |

## Which version produces the most consistent response lengths?

**v1** is the most consistent. Its completion tokens range from 412 to 732 (roughly a 19% spread relative to its mean), and every call finishes in a tight 5.1–6.1s band.

**v2** is noticeably more variable: completions range from 967 to 1,670 tokens (about a 20% relative spread), and durations range from 7.25s to 12.89s.

**v3** is the least consistent of the three: completions swing from 866 to 1,976 tokens (about a 22–23% relative spread). `day-hike-gear` stands out as an outlier — it produced far less output (866 tokens) than the other four v3 tests (1,424–1,812 tokens), despite v3 using the longest prompt of the three versions.

## Which version shows the highest duration?

**v3**, on both measures: highest average per-call duration (~12.3s) and highest total trace duration (61.8s), though only marginally ahead of v2 (56.9s). v1 is far faster than either — roughly half v2's total time and less than half v3's.

## Are there any calls that stand out as unusually slow or verbose?

- `winter-hiking` (v3) is the single slowest and most verbose call recorded: 14,555ms and 1,976 tokens.
- `overnight-camping` (v3) is close behind: 14,281ms and 1,842 tokens.
- Every v1 call is tightly clustered and comparatively fast/short — no outliers.
- Within v2, there's a sharp jump from `day-hike-gear` (7.25s, 1,075 tokens) to the remaining four tests (all 10.9–13.2s, 1,293–1,774 tokens), suggesting `day-hike-gear` is the "easy" prompt across all three versions.

**Infrastructure note:** the two `GET /metadata/identity/oauth2/token` child spans (~177ms each) appear only once, under v1's first call (`v1_day-hike-gear`). This is Azure managed identity fetching and caching an auth token on cold start — it does not recur later in v1 or at all in v2/v3, so it's one-time setup overhead rather than a per-call cost.

## Takeaway

v1 → v2 → v3 trades speed for thoroughness: each version's longer, more detailed prompt produces longer, slower completions, with diminishing returns between v2 and v3 (prompt tokens grew ~50% but total duration grew only ~9%). Whether that trade-off is worth it depends on whether the extra completion content in v2/v3 is more useful to a reader or just more verbose — worth a manual read-through of a couple of matched responses (e.g. `trail-difficulty` v1 vs v3) to judge.
