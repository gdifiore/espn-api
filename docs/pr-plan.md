# ESPN API Baseball Enhancement PR Plan

---

## PR 1 — Foundation ✅ MERGED
- Fix player position map and `free_agents` position filter
- `Settings` class exposing `position_slot_counts`
- `Transaction` / `TransactionItem` classes
- New league methods: `transactions()`, `player_info()`, `refresh()`, `load_roster_week()`

---

## PR 2 — Model expansion ✅ COMPLETE (current branch)

Expands all model classes to expose the full data returned by the ESPN API.

**`Transaction` / `TransactionItem`**
- `rating`, `execution_type` on `Transaction`
- `fromLineupSlotId`, `toLineupSlotId`, `isKeeper`, `overallPickNumber` on `TransactionItem`
- `date` converted from millisecond timestamp to `datetime`
- Fixed `KeyError: 'status'` for trade transactions

**`Player`**
- Identity: `first_name`, `last_name`, `jersey`, `active`, `droppable`, `laterality`, `stance`
- Ownership: `adp`, `adp_change`, `auction_value`, `auction_value_change`, `percent_owned_change`
- Draft: `draft_ranks` dict keyed by rank type (`STANDARD`, `ROTO`)
- Roster state: `keeper_value`, `keeper_value_future`, `lineup_locked`, `roster_locked`, `trade_locked`, `on_team_id`, `acquisitionDate`
- News: `last_news_date`, `season_outlook`
- Handles both data shapes: roster entries (`playerPoolEntry` wrapper) and `player_info` card (flat)

**`Team`**
- `points_for`, `points_against`, `streak_type`, `streak_length`
- Home/away/division split records
- `current_projected_rank`, `waiver_rank`, `points`

**`Settings`**
- `acquisition_limit`, `matchup_acquisition_limit`, `matchup_limit_per_scoring_period`, `minimum_bid`
- `waiver_process_days`, `waiver_process_hour`, `trade_revision_hours`

**Draft**
- `League.draft` populated via `BaseLeague._fetch_draft()`

**Misc**
- Removed stray `import pdb` from `matchup.py` and `team.py`
- 80 unit tests, all passing

---

## PR 3 — Stat splits (L7 / L15 / L30)
**Branch:** `feat/stat-splits`
**Depends on:** PR 2 merged

- Remove the `stats_split_type` filter in `Player.__init__`
- Restructure `self.stats` to nest by split type:
  ```python
  # Current:   self.stats[scoring_period] = {points, breakdown, ...}
  # Proposed:  self.stats[scoring_period][split_type] = {points, breakdown, ...}
  #   split_type: 'season' | 'last_7' | 'last_15' | 'last_30' | 'box_score'
  ```
- `total_points` / `projected_total_points` continue pulling from `split_type='season'`
- Add `statSplitTypeId` → label constants in `constant.py`

**Breaking change:** `self.stats` structure changes — bump minor version and document migration.

---

## Notes
- PR 3 is the only remaining breaking change — coordinate timing with downstream consumers.
- If upstream maintainers want common `Player` fields extracted to a `BasePlayer`, let them drive that — it requires touching all sports and is out of scope here.
