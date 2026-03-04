# Playlist Chaos - Code Audit Report

**Date:** March 4, 2026
**Files Reviewed:** 2 (playlist_logic.py, app.py)
**Total Issues Found:** 12 (4 Critical, 5 High, 3 Medium)

---

## Summary

This audit identified **12 bugs and logical errors** across the Streamlit playlist application. Issues range from critical logic errors that will cause crashes or incorrect calculations, to high-priority mutations and inverted search logic, to medium-priority inconsistencies and Streamlit widget behavioral issues.

All issues have been annotated inline in the source files with BUG and FIX comments for developer review.

---

## Critical Issues

### 1. `playlist_logic.py` Line 129: Hype Ratio Calculation Uses Wrong Denominator
**Location:** `compute_playlist_stats()` function
**Issue:** `total = len(hype)` should be `total = len(all_songs)`
**Impact:** The hype_ratio will always equal 1.0 (or 0 if hype is empty), completely misrepresenting the actual proportion of hype songs in the total playlist. For example, if there are 10 hype and 20 total songs, the ratio should be 0.50, but it will show as 1.0.
**Severity:** CRITICAL - Incorrect statistics displayed to user

### 2. `playlist_logic.py` Line 136: Average Energy Calculated Only Over Hype Songs
**Location:** `compute_playlist_stats()` function
**Issue:** `total_energy = sum(song.get("energy", 0) for song in hype)` should include `all_songs`
**Impact:** Average energy statistic is skewed, only reflecting the energy levels of Hype songs while claiming to represent the entire playlist. For a balanced playlist of varied moods, this gives a misleading metric.
**Severity:** CRITICAL - Incorrect statistics, poor user experience

### 3. `playlist_logic.py` Line 212: Random Choice on Empty List Raises IndexError
**Location:** `random_choice_or_none()` function
**Issue:** Function name promises to return None for empty input, but `random.choice([])` raises `IndexError` instead
**Impact:** If "Feeling lucky" button is clicked when a playlist mode has no songs, the app will crash with an unhandled exception instead of showing the warning message.
**Severity:** CRITICAL - Application crash

### 4. `playlist_logic.py` Line 185: Search Logic Inverted
**Location:** `search_songs()` function
**Issue:** `if value and value in q:` should be `if value and q in value:`
**Impact:** Search will only return matches where the entire artist name is contained within the query string. For example, searching for "The" would not find "The Beatles", but searching for "The Beatles" would find "The". This is backwards from expected behavior.
**Severity:** CRITICAL - Core feature broken (search doesn't work)

---

## High Issues

### 5. `playlist_logic.py` Line 112: In-place List Mutation in merge_playlists()
**Location:** `merge_playlists()` function
**Issue:** `merged[key] = a.get(key, [])` creates a reference, not a copy. The subsequent `.extend()` mutates the original list from parameter `a`.
**Impact:** Calling `merge_playlists(base_playlists, {})` will permanently modify the original playlists dictionary. This causes silent data corruption and unexpected behavior in subsequent uses of the original playlists.
**Severity:** HIGH - Data mutation, silent corruption

### 6. `playlist_logic.py` Line 80: Chill Keywords Checked Against Title Instead of Genre
**Location:** `classify_song()` function
**Issue:** `is_chill_keyword = any(k in title for k in chill_keywords)` should check `genre` like line 77 does
**Impact:** Songs with genres "lofi", "ambient", or "sleep" will not be classified as Chill unless those keywords appear in the title. Example: A song with genre "lofi" and title "Driving" will be misclassified.
**Severity:** HIGH - Incorrect song classification

### 7. `app.py` Line 218: Selectbox Always Resets to Index 0
**Location:** `profile_sidebar()` function, favorite_genre selectbox
**Issue:** `index=0` is hardcoded; should compute the current index from the options list
**Impact:** The "Favorite genre" selector always resets to the first option (rock) on each Streamlit rerun, ignoring the user's saved preference. Users cannot persistently change this setting.
**Severity:** HIGH - User preference not retained, poor UX

### 8. `app.py` Line 409: Mutation of base_playlists via merge_playlists Call
**Location:** `main()` function
**Issue:** `merge_playlists(base_playlists, {})` mutates base_playlists due to bug #5
**Impact:** After merging, the original base_playlists dict is corrupted, though in this specific case (merging with empty dict) the effect is less severe. However, it demonstrates the mutation bug and could cause issues if the pattern is repeated elsewhere.
**Severity:** HIGH - Silent data mutation

---

## Medium Issues

### 9. `playlist_logic.py` Line 22-28: Type Inconsistency in normalize_artist()
**Location:** `normalize_artist()` function
**Issue:** Function signature expects `str`, but no type checking like `normalize_title()` has. Will crash if None or non-string is passed.
**Impact:** If a song dict is created without an "artist" key and the code path calls this function with a non-string, it will raise an `AttributeError` on `.strip()`.
**Severity:** MEDIUM - Defensive programming gap, potential crash with bad input

### 10. `playlist_logic.py` Line 77: Substring Matching is Too Broad
**Location:** `classify_song()` function, hype_keyword check
**Issue:** `any(k in genre for k in hype_keywords)` uses substring matching; "rock" will match genres containing "rock" as a substring
**Impact:** While this works for the current data, a genre like "rockabilly" would match "rock" keyword. More problematically, a future genre like "electrorock" would also match. Not exact word matching.
**Severity:** MEDIUM - Fragile pattern matching, potential misclassification with expanded genre list

### 11. `app.py` Line 254: Missing Input Validation for Song Title/Artist
**Location:** `add_song_sidebar()` function
**Issue:** Validation only checks `if title and artist:`, but provides no user feedback if these are empty
**Impact:** Silent failure - user clicks "Add to playlist" with empty title/artist, nothing happens, no warning shown. Confusing UX.
**Severity:** MEDIUM - Poor user experience, silent failure

### 12. `app.py` Line 314-315: Misleading Comment About None Handling
**Location:** `lucky_section()` function
**Issue:** Comment suggests pick could be None safely, but the actual issue is that `random_choice_or_none()` raises IndexError before returning None
**Impact:** The error handling is incomplete; the check for `if pick is None:` won't prevent the crash from the IndexError in `random_choice_or_none()`.
**Severity:** MEDIUM - Incomplete error handling, confusing logic flow

---

## Summary Table

| Severity | Count | Status |
|----------|-------|--------|
| CRITICAL | 4 | All annotated in source files |
| HIGH     | 5 | All annotated in source files |
| MEDIUM   | 3 | All annotated in source files |
| **TOTAL**| **12** | **Ready for developer review** |

---

## Recommended Fix Priority

1. **Fix #4 (search_songs)** - Core feature is broken; easy 1-line fix
2. **Fix #3 (random_choice_or_none)** - Will crash app; 2-line fix
3. **Fix #2 (avg_energy)** - Incorrect stats; 1-line fix
4. **Fix #1 (hype_ratio)** - Incorrect stats; 1-line fix
5. **Fix #5 (merge_playlists mutation)** - Silent data corruption; 1-line fix
6. **Fix #6 (chill_keyword check)** - Song misclassification; 1-line fix
7. **Fix #7 (favorite_genre selectbox)** - User preference lost; 3-line fix
8. **Fix #8** - Derivative of #5; resolves once #5 is fixed
9. **Fixes #9-12** - Medium priority improvements for robustness and UX

---

## Files Annotated

- `/Users/bjethwani/Downloads/ai110-module1tinker-playlistchaos-starter/playlist_logic.py` (7 bugs)
- `/Users/bjethwani/Downloads/ai110-module1tinker-playlistchaos-starter/app.py` (5 bugs)

All issues are marked with `# BUG:` and `# FIX:` comments in the source code at their location.

---

## Notes on Code Quality

### Positive Observations
- Good use of type hints and docstrings
- Sensible normalization functions for data cleaning
- Proper session state management in Streamlit
- Reasonable data structure choices (dicts for songs)

### Areas of Concern
- Limited input validation and error handling
- Shallow testing of edge cases (empty lists, None values)
- Inconsistent defensive programming patterns across similar functions
- No explicit None/type checking in some functions

---

## Conclusion

The application has 12 identifiable bugs that range from critical (crashes and broken features) to medium (UX issues). Most are single-line fixes or involve adding one conditional check. The code structure is fundamentally sound, but needs refinement in error handling and data validation.
