# Playlist Chaos - Project Analysis Memory

## Project Overview
- **Type:** Streamlit web application (Python)
- **Domain:** Smart playlist classification and management system
- **Framework:** Streamlit for UI, custom Python logic for song classification
- **Architecture:** MVC-like with separate logic (playlist_logic.py) and UI (app.py)

## Core Components
- `playlist_logic.py`: Business logic for song classification, playlist building, search, stats
- `app.py`: Streamlit UI with sidebar controls, tabs, search, and stats display
- 22 default songs in various genres for testing

## Key Data Structures
- `Song`: Dict[str, object] with keys: title, artist, genre, energy (1-10), tags, mood
- `PlaylistMap`: Dict[str, List[Song]] mapping mood types to song lists

## Common Bug Patterns Found

### 1. Logic Inversions
- **Issue:** Conditions checked backwards (value in q instead of q in value)
- **Location:** `search_songs()` function, line 185
- **Pattern:** Easy to miss in code review; affected core functionality

### 2. Variable Naming Leading to Wrong Fields
- **Issue:** Using wrong variable for similar checks (title vs genre in classify_song)
- **Locations:** `classify_song()` function, lines 77-80
- **Pattern:** Inconsistent implementations of parallel logic paths
- **Prevention:** Use consistent patterns for similar operations; consider extraction

### 3. Mutation of Input Parameters
- **Issue:** List assignment creates reference, not copy; subsequent .extend() mutates original
- **Location:** `merge_playlists()` function, line 112
- **Pattern:** Python reference semantics; functions not respecting "no side effects" contract
- **Prevention:** Always use `.copy()`, `[:]`, or `list()` when creating new containers

### 4. Hardcoded Indices in Stateful Contexts
- **Issue:** Streamlit selectbox index hardcoded to 0, ignoring session state value
- **Location:** `profile_sidebar()`, line 218
- **Pattern:** Stateful UI not properly reading/restoring state
- **Prevention:** Always recompute index from current value when using index parameter

### 5. Partial Error Handling
- **Issue:** Function promise (return None) doesn't match implementation (raises exception)
- **Location:** `random_choice_or_none()`, line 212
- **Pattern:** Contract mismatch between type hints, docstrings, and actual behavior
- **Prevention:** Validate contracts in code review; test edge cases (empty lists, None)

### 6. Statistics Calculated on Wrong Subset
- **Issue:** Aggregate stats computed using subset of data (hype songs instead of all)
- **Locations:** `compute_playlist_stats()`, lines 129, 136
- **Pattern:** Copy-paste error or incomplete refactoring; variable naming confusion
- **Prevention:** Add assertions/tests verifying stat properties (ratios should be 0-1, etc.)

### 7. Substring Matching Without Boundaries
- **Issue:** Keyword matching using broad "in" checks without word boundaries
- **Location:** `classify_song()`, line 77
- **Pattern:** Works for current data but fragile with extended datasets
- **Severity:** Medium - works now but will break with genre additions

## Type Safety Observations
- Good: Type hints used throughout
- Gap: Inconsistent defensive type checking between similar functions
- Gap: normalize_artist() lacks isinstance check that normalize_title() has

## Input Validation Gaps
- No validation feedback when user adds song without title/artist (silent failure)
- search_songs() doesn't validate that field parameter exists in song dicts
- No protection against duplicate songs in playlist

## Streamlit Specific Issues
- Selectbox index parameter not reactive to state changes (Fix #7)
- Button click handler doesn't catch exceptions; needs explicit error handling
- Session state mutations can have subtle side effects (referenced in Fix #5)

## Code Quality Strengths
- Clear separation of concerns (logic vs UI)
- Good docstrings on most functions
- Type annotations present
- Sensible data structure choices

## Testing Observations
- No test file present in project
- Many bugs could be caught with simple edge case tests:
  - test_search_songs_with_partial_artist_name()
  - test_random_choice_or_none_with_empty_list()
  - test_hype_ratio_property() # Assert 0 <= ratio <= 1
  - test_avg_energy_includes_all_songs()
  - test_merge_playlists_does_not_mutate_input()

## Recommended Code Practices for This Project
1. Always use `[:]` or `list()` when intentionally copying lists
2. Test edge cases: empty lists, None values, empty strings
3. Keep parallel logic consistent (e.g., hype_keyword and chill_keyword checks)
4. Validate function contracts match type hints and docstrings
5. Add assertions for statistical properties (0 <= ratios <= 1, etc.)
6. In Streamlit widgets, always recompute index/value from current state

## Project Structure Notes
- Root directory contains source files (no src/ subdirectory)
- Default songs serve as test data
- No configuration files or environment variables noted
- Project uses virtual environment (.venv/)
