# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed all instances of `save_to_watchlist` to `add_to_watchlist`
**How I verified:** Ran `git grep -rn "save_to_watchlist"` (no results which confirms old name fully removed) and `git grep -rn "add_to_watchlist"` (confirms new name appears in the definition, import, and call site, and nowhere else)

## Comment 2 — Deduplication
**What I did:** Added an `AlreadyInWatchlistError` exception and a duplicate check to `add_to_watchlist()` in `services/watchlist_service.py`, following the same pattern as `add_to_collection()` in `services/collection_service.py`. The function now queries for an existing `WatchlistEntry` with the same `user_id` and `film_id` before inserting, and raises `AlreadyInWatchlistError` if one is found. Updated the docstring's `Raises:` section to document the new exception. Also added a `UniqueConstraint("user_id", "film_id")` to the `WatchlistEntry` model as a DB-level safeguard against the race condition the app-level check alone doesn't close. Updated `add_film()` in `routes/watchlist/watchlist.py` to import and catch `AlreadyInWatchlistError`, returning a 409 response, matching the error-handling pattern in `routes/collection.py`.
**How I verified:** Confirmed via `flask shell` that `Film.id` is a plain auto-incrementing integer (no UUID), ran `git grep` to confirm no other call sites reference the old duplicate-prone behavior, and manually tested `add_film()` — first call succeeds (201), second call with the same `user_id`/`film_id` returns 409 with the expected error message.

## Comment 3 — Missing test
**What I did:**
**How I verified:**

## Comment 4 — Default visibility
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->