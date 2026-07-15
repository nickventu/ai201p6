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
**What I did:** Created `tests/test_watchlist.py` and added `test_add_to_watchlist_nonexistent_film_raises`, adapted from `test_collection.py`'s `test_add_to_collection_nonexistent_film_raises` pattern (same `app`/`sample_user`/`sample_film` fixture structure). Also changed the fake film ID from a UUID string to an integer (`999999`), since `Film.id` is an auto-incrementing `Integer`, not a UUID (confirmed earlier).
**How I verified:** Ran `pytest tests/test_watchlist.py -v` — test passes, correctly raising `FilmNotFoundError` for a nonexistent `film_id`.

## Comment 4 — Default visibility
**My position:** Visibility should default to public.
**Reasoning:** The brief specifies CineLog is a "community film tracking app", indicating that a major aspect of the app is the social aspect. Defaulting to public optimizes for seamless content discovery and sharing; users can immediately see their friends' watchlists without either user needing to do anything. Another consideration is that WatchlistEntry.public is a per-entry field meaning a user can override this default per item.
**Tradeoff acknowledged:** There is a real tradeoff compared to the collection feature: a watchlist exposes current interests and intent rather than a completed action, which can be a meaningful exposure for users who would prefer to keep that private.

## Comment 5 — Sort order
**My position:** Sort by date added
**Reasoning:** Users are more likely to want to know about recently added content than just see it in an essentially random order.
**Engagement with reviewer's point:** I agree with your point, recency is what most users are optimizing for when checking a watchlist. However we should also think about cases where someone may just want to look for a specific title in a watchlist. In this situation alphabetical makes it much easier than date-added. I don't think this argues against date-added as the default, though — it only argues for eventually offering alphabetical as an optional secondary sort or filter, rather than making it the default users see first. I'd keep date-added as default and treat lookup-by-title as a separate, later feature if it turns out to be a common need.

## Comment 6 — Rebase
**What conflicted:** .gitignore(non-substantive), models.py(WatchlistEntry class was written before main's UUID refactor landed; main had no WatchlistEntry at all yet, so the conflict was really "where does this new class go?")
**How I resolved it:**
.gitignore — accepted both sets of entries (union), since neither side's ignores were contradictory
models.py — kept the WatchlistEntry class, but changed film_id from db.Integer to db.String(36) to match the now-UUID Film.id
**How I verified no conflict remains:**
git log --oneline --graph — confirmed linear history, no stray merge commits introduced by the rebase
Manually audited three files that referenced film_id as an int for leftover stale assumptions post-rebase (watchlist_service.py docstring, watchlist.py route docstring, test_watchlist.py fake ID)
Ran pytest tests/test_watchlist.py -v (passes)
## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->