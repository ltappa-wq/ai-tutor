# Distribution and tiers

Locked 2026-09-05. Prices TBD. Limits are the product lock.

Ship after the household loop works for us. Public distribution is Phase 8, not Phase 3.

## Tiers

Household is the billable unit. Every guardian in the household shares one tier.

| | Free | Basic | Pro |
| --- | --- | --- | --- |
| Students | 1 | 3 | 5 |
| Active files (household) | 50 | 200 | 1,500 |
| Storage | 250 MB | 1 GB | 2 GB |
| Manual upload size | 10 MB | 25 MB | 25 MB |
| Schoology crawl | no | yes | yes |
| 11pm digest | yes | yes | yes |
| Review bench (planner + reviewer + referee) | reviewer only, no referee | full bench | full bench |
| Guided sessions / month | 40 | 300 | unlimited fair-use |

Platform hard cap remains **100 MB per file** (Floot). We still reject a 25 MB+ manual upload on Free/Basic/Pro as above; crawl may store larger LMS decks up to 100 MB and they count against storage + file count.

Soft-deleted files do not count after 30 days. Deduped crawl re-downloads do not add a second file.

## Enforcement

- Upload and crawl both check remaining file count and bytes **before** storing.
- Over cap: file is refused, parent home shows "upgrade or delete." Student Today still works on files already in the repo.
- Crawl that would blow the cap stores nothing new that night and says so on the digest.
- `household.tier`: `free` | `basic` | `pro`. Default `free`.

## What we are not pricing yet

Dollar amounts, annual vs monthly, family-vs-student SKUs, school-site licenses. Those wait until we have a month of our own usage. The **limits** are locked so engineering can gate Phase 3 uploads.
