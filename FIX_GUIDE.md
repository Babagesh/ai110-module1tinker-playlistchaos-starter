# Fix Guide for Playlist Chaos Bugs

This document provides detailed fixes for each of the 12 bugs found in the audit.

---

## Critical Fixes (Must Fix)

### Fix #1: Search Logic Inverted (playlist_logic.py, line 185)

**Current Code:**
```python
for song in songs:
    value = str(song.get(field, "")).lower()
    if value and value in q:  # BUG: Backwards logic
        filtered.append(song)
```

**Fixed Code:**
```python
for song in songs:
    value = str(song.get(field, "")).lower()
    if value and q in value:  # FIX: Search term in field value
        filtered.append(song)
```

**Explanation:** The condition was backwards. We want to find songs where the query string (q) is contained within the field value, not the other way around.

---

### Fix #2: Random Choice Empty List Crash (playlist_logic.py, line 212)

**Current Code:**
```python
def random_choice_or_none(songs: List[Song]) -> Optional[Song]:
    """Return a random song or None."""
    import random
    return random.choice(songs)  # BUG: Crashes on empty list
```

**Fixed Code:**
```python
def random_choice_or_none(songs: List[Song]) -> Optional[Song]:
    """Return a random song or None."""
    import random

    if not songs:  # FIX: Handle empty list
        return None
    return random.choice(songs)
```

**Explanation:** `random.choice()` raises `IndexError` on empty sequences. The function name and type hint promise to return None, so we must honor that contract.

---

### Fix #3: Average Energy Wrong (playlist_logic.py, line 136)

**Current Code:**
```python
if all_songs:
    total_energy = sum(song.get("energy", 0) for song in hype)  # BUG: Only hype songs
    avg_energy = total_energy / len(all_songs)
```

**Fixed Code:**
```python
if all_songs:
    total_energy = sum(song.get("energy", 0) for song in all_songs)  # FIX: All songs
    avg_energy = total_energy / len(all_songs)
```

**Explanation:** The statistic should represent the average energy across ALL playlists, not just Hype. Currently summing only hype songs while dividing by total count, giving a skewed metric.

---

### Fix #4: Hype Ratio Wrong Denominator (playlist_logic.py, line 129)

**Current Code:**
```python
total = len(hype)  # BUG: Should be all songs
hype_ratio = len(hype) / total if total > 0 else 0.0
```

**Fixed Code:**
```python
total = len(all_songs)  # FIX: Use all songs as denominator
hype_ratio = len(hype) / total if total > 0 else 0.0
```

**Explanation:** Currently calculates `len(hype) / len(hype)` which always equals 1.0 (or 0 if empty). The ratio should show what fraction of the TOTAL playlist is hype songs.

---

## High Priority Fixes

### Fix #5: Merge Playlists Mutates Input (playlist_logic.py, line 112)

**Current Code:**
```python
def merge_playlists(a: PlaylistMap, b: PlaylistMap) -> PlaylistMap:
    """Merge two playlist maps into a new map."""
    merged: PlaylistMap = {}
    for key in set(list(a.keys()) + list(b.keys())):
        merged[key] = a.get(key, [])  # BUG: Reference, not copy
        merged[key].extend(b.get(key, []))
    return merged
```

**Fixed Code (Option 1: Slice Copy):**
```python
def merge_playlists(a: PlaylistMap, b: PlaylistMap) -> PlaylistMap:
    """Merge two playlist maps into a new map."""
    merged: PlaylistMap = {}
    for key in set(list(a.keys()) + list(b.keys())):
        merged[key] = a.get(key, [])[:]  # FIX: Create copy with slice
        merged[key].extend(b.get(key, []))
    return merged
```

**Fixed Code (Option 2: list() Constructor):**
```python
def merge_playlists(a: PlaylistMap, b: PlaylistMap) -> PlaylistMap:
    """Merge two playlist maps into a new map."""
    merged: PlaylistMap = {}
    for key in set(list(a.keys()) + list(b.keys())):
        merged[key] = list(a.get(key, []))  # FIX: Create copy with list()
        merged[key].extend(b.get(key, []))
    return merged
```

**Explanation:** Assignment doesn't create a new list, just a reference. When we extend, we're modifying the original. Use either slice notation `[:]` or `list()` to create a shallow copy.

---

### Fix #6: Chill Keywords Check Wrong Field (playlist_logic.py, line 80)

**Current Code:**
```python
is_hype_keyword = any(k in genre for k in hype_keywords)
is_chill_keyword = any(k in title for k in chill_keywords)  # BUG: Checks title, not genre
```

**Fixed Code:**
```python
is_hype_keyword = any(k in genre for k in hype_keywords)
is_chill_keyword = any(k in genre for k in chill_keywords)  # FIX: Check genre like hype
```

**Explanation:** Both should check the same field. The genre field is the correct place to look for mood-related keywords like "lofi", "ambient", etc.

---

### Fix #7: Favorite Genre Selectbox Doesn't Persist (app.py, line 215)

**Current Code:**
```python
profile["favorite_genre"] = st.sidebar.selectbox(
    "Favorite genre",
    options=["rock", "lofi", "pop", "jazz", "electronic", "ambient", "other"],
    index=0,  # BUG: Hardcoded to first option
)
```

**Fixed Code:**
```python
options = ["rock", "lofi", "pop", "jazz", "electronic", "ambient", "other"]
current_genre = profile.get("favorite_genre", "rock")
current_index = options.index(current_genre) if current_genre in options else 0

profile["favorite_genre"] = st.sidebar.selectbox(
    "Favorite genre",
    options=options,
    index=current_index,  # FIX: Use current value's index
)
```

**Explanation:** The selectbox needs to know which option is currently selected via the `index` parameter. We find the index of the current favorite_genre in the options list.

---

### Fix #8: Base Playlists Mutation (app.py, line 409)

**Note:** This is a consequence of Fix #5. Once `merge_playlists()` is fixed, this issue resolves automatically.

**Context:** The call itself doesn't need to change:
```python
merged_playlists = merge_playlists(base_playlists, {})
```

Just fix `merge_playlists()` function as shown in Fix #5.

---

## Medium Priority Fixes

### Fix #9: Type Checking Missing in normalize_artist (playlist_logic.py, line 22)

**Current Code:**
```python
def normalize_artist(artist: str) -> str:
    """Normalize an artist name for comparisons."""
    if not artist:
        return ""
    return artist.strip().lower()
```

**Fixed Code:**
```python
def normalize_artist(artist: str) -> str:
    """Normalize an artist name for comparisons."""
    if not isinstance(artist, str):  # FIX: Add type check like normalize_title
        return ""
    return artist.strip().lower()
```

**Alternative Fix (More robust):**
```python
def normalize_artist(artist: str) -> str:
    """Normalize an artist name for comparisons."""
    if isinstance(artist, str):
        return artist.strip().lower()
    return ""
```

**Explanation:** Match the defensive programming pattern used in `normalize_title()`. Prevents crashes if non-string or None values are passed.

---

### Fix #10: Substring Matching Too Broad (playlist_logic.py, line 77)

**Current Code:**
```python
hype_keywords = ["rock", "punk", "party"]
is_hype_keyword = any(k in genre for k in hype_keywords)
```

**Fixed Code (Option 1: Word boundary check):**
```python
import re

hype_keywords = ["rock", "punk", "party"]
is_hype_keyword = any(re.search(r'\b' + k + r'\b', genre) for k in hype_keywords)
```

**Fixed Code (Option 2: Space-separated words):**
```python
hype_keywords = ["rock", "punk", "party"]
genre_words = genre.split()
is_hype_keyword = any(k in genre_words for k in hype_keywords)
```

**Fixed Code (Option 3: Keep simple if data is reliable):**
```python
# If you trust the data and know genres are single words, current approach is OK
# But add a comment documenting this assumption
```

**Explanation:** Current substring matching could cause false positives with multi-word genres. For MVP, if genres are always single words, the current approach is acceptable. But using word boundaries or word lists is more robust.

---

### Fix #11: Missing Input Validation (app.py, line 242)

**Current Code:**
```python
if st.sidebar.button("Add to playlist"):
    raw_tags = [t.strip() for t in tags_text.split(",")]
    tags = [t for t in raw_tags if t]
    # ...
    if title and artist:
        # Add song
```

**Fixed Code:**
```python
if st.sidebar.button("Add to playlist"):
    if not title or not artist:  # FIX: Validate early
        st.sidebar.error("Title and Artist are required")
        st.stop()

    raw_tags = [t.strip() for t in tags_text.split(",")]
    tags = [t for t in raw_tags if t]
    # ... rest of code
```

**Alternative (Non-blocking warning):**
```python
if st.sidebar.button("Add to playlist"):
    if not title or not artist:
        st.sidebar.warning("Please enter both Title and Artist")
    elif tags_text:  # Only proceed if all fields present
        raw_tags = [t.strip() for t in tags_text.split(",")]
        tags = [t for t in raw_tags if t]
        # ... add song code
```

**Explanation:** Provides explicit feedback to user instead of silently ignoring their input. Improves UX significantly.

---

### Fix #12: Incomplete Error Handling Comment (app.py, line 314)

**Current Code:**
```python
if st.button("Feeling lucky"):
    pick = lucky_pick(playlists, mode=mode)
    # BUG: Comment suggests pick could be None safely, but the actual issue is...
    if pick is None:
        st.warning("No songs available for this mode.")
        return
```

**Fixed Code:**
```python
if st.button("Feeling lucky"):
    try:
        pick = lucky_pick(playlists, mode=mode)
    except IndexError:
        st.warning("No songs available for this mode.")
        return

    if pick is None:  # This check becomes valid after random_choice_or_none is fixed
        st.warning("No songs available for this mode.")
        return
```

**Better Solution:** Just fix `random_choice_or_none()` (Fix #2), then this becomes:
```python
if st.button("Feeling lucky"):
    pick = lucky_pick(playlists, mode=mode)
    if pick is None:  # Now this truly prevents crashes
        st.warning("No songs available for this mode.")
        return
```

**Explanation:** Once `random_choice_or_none()` is fixed to return None instead of raising IndexError, the existing check will work correctly.

---

## Implementation Order

1. **Phase 1 (Critical - Do First):**
   - Fix #1: Search logic (1 line)
   - Fix #2: Random choice (2 lines)
   - Fix #3: Average energy (1 line)
   - Fix #4: Hype ratio (1 line)

2. **Phase 2 (High - Do Next):**
   - Fix #5: Merge playlists mutation (1 line)
   - Fix #6: Chill keyword check (1 line)
   - Fix #7: Favorite genre selectbox (3 lines)

3. **Phase 3 (Medium - Polish):**
   - Fix #9: Type checking
   - Fix #10: Substring matching (optional, depends on requirements)
   - Fix #11: Input validation
   - Fix #12: Error handling (automatic after Fix #2)

---

## Testing After Fixes

After implementing fixes, test the following scenarios:

1. **Search:** Search for an artist name (e.g., "Queen") - should find matching songs
2. **Lucky Pick:** Click "Feeling lucky" with all playlist types selected - should never crash
3. **Stats:** Check that hype_ratio is between 0 and 1, not always 1.0
4. **Genre Persistence:** Set favorite genre to "pop", refresh page - should stay on "pop"
5. **Empty Playlists:** Set filters so a playlist is empty, try lucky pick - should show warning, not crash
6. **Song Addition:** Try adding song without title - should show warning or error

---

## Questions to Consider

- Should the app prevent adding duplicate songs?
- Should search be case-insensitive and/or fuzzy (match partial words)?
- Should playlists be filtered based on the profile settings?
- Should the history be persistent across sessions?

These are design questions but worth addressing as you refactor.
