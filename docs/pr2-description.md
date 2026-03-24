# Baseball API Model Expansion

Expands the data exposed across all baseball model classes to match what the ESPN API actually returns.

## Changes

**`Transaction` / `TransactionItem`**
- Adds `rating`, `execution_type` on `Transaction`
- Adds `fromLineupSlotId`, `toLineupSlotId`, `isKeeper`, `overallPickNumber` on `TransactionItem`
- `date` is now a `datetime` object (converted from ESPN's millisecond timestamp)
- Fixes `KeyError: 'status'` for transaction types that omit the field (e.g. `TRADE_ACCEPT`)

**`Player`**
- Identity: `first_name`, `last_name`, `jersey`, `active`, `droppable`, `laterality`, `stance`
- Ownership: `adp`, `adp_change`, `auction_value`, `auction_value_change`, `percent_owned_change`
- Draft: `draft_ranks` dict keyed by rank type (`STANDARD`, `ROTO`), each with `rank` and `auction_value`
- Roster state: `keeper_value`, `keeper_value_future`, `lineup_locked`, `roster_locked`, `trade_locked`, `on_team_id`, `acquisitionDate`
- News: `last_news_date` (datetime), `season_outlook`
- Handles both data shapes: roster/free-agent entries (`playerPoolEntry` wrapper) and `player_info` card responses (flat structure)

**`Team`**
- Adds `points_for`, `points_against`, `streak_type`, `streak_length`
- Adds home/away/division split records (`home_wins`, `home_losses`, `home_ties`, etc.)
- Adds `current_projected_rank`, `waiver_rank`, `points` (live week score)

**`Settings`**
- Adds `acquisition_limit`, `matchup_acquisition_limit`, `matchup_limit_per_scoring_period`, `minimum_bid`
- Adds `waiver_process_days`, `waiver_process_hour`, `trade_revision_hours`

**Draft**
- `League.draft` is now populated for baseball via `BaseLeague._fetch_draft()`

**Misc**
- Removes stray `import pdb` from `matchup.py` and `team.py`

## Tests
80 unit tests, all passing. New: `test_team.py`. Expanded: `test_player.py`, `test_transaction.py`, `test_settings.py`. All new fields covered including default/missing-value behaviour and both player data shapes.
