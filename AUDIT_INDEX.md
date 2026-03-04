# Audit Documentation Index

## Quick Start

If you're just starting, read in this order:
1. **AUDIT_SUMMARY.txt** - 2 minute overview, critical issues, action items
2. **FIX_GUIDE.md** - Detailed fixes with before/after code
3. **Source files** - Read inline BUG/FIX comments while implementing

## Complete Documentation

### AUDIT_SUMMARY.txt
Quick reference guide with:
- All 12 issues at a glance
- Severity levels
- Problem statement for each bug
- Suggested fix
- Action item checklist
- Effort estimate (2 hours total)

**Best for:** Quick overview, priority decisions, action planning

---

### AUDIT_REPORT.md
Comprehensive audit report with:
- Executive summary
- Critical issues (4) with detailed explanations
- High priority issues (5) with impact analysis
- Medium priority issues (3) with improvement suggestions
- Summary table
- Fix priority ranking
- Code quality observations
- Positive and concerning patterns

**Best for:** Understanding the big picture, understanding each issue deeply, prioritization

---

### FIX_GUIDE.md
Detailed implementation guide with:
- Current code for each bug
- Fixed code (with explanations)
- Alternative solutions where applicable
- Testing scenarios
- Implementation order
- Design questions for future consideration

**Best for:** Actually implementing fixes, understanding trade-offs, testing

---

## Inline Annotations in Source Files

All bugs are marked directly in the source code with format:
```python
# BUG: <what is wrong>
# FIX: <how to fix it>
```

### Files Annotated
- **playlist_logic.py** - 7 bugs annotated
- **app.py** - 5 bugs annotated

**Best for:** Code review, while fixing, understanding context

---

## Bug Locations Quick Reference

### playlist_logic.py

| Line | Issue | Severity |
|------|-------|----------|
| 24-28 | normalize_artist() type checking | MEDIUM |
| 75-77 | Substring matching too broad | MEDIUM |
| 78-80 | Chill keyword checks title not genre | HIGH |
| 110-113 | In-place mutation in merge_playlists | HIGH |
| 127-130 | Hype ratio wrong denominator | CRITICAL |
| 134-137 | Avg energy from hype songs only | CRITICAL |
| 183-185 | Search logic inverted | CRITICAL |
| 210-212 | Empty list handling in random_choice | CRITICAL |

### app.py

| Line | Issue | Severity |
|------|-------|----------|
| 213-218 | Favorite genre selectbox persistence | HIGH |
| 253-259 | Missing song input validation | MEDIUM |
| 284-286 | Search function has broken logic | CRITICAL (propagated) |
| 314-318 | Incomplete error handling | MEDIUM |
| 343-345 | Hype ratio metric wrong | CRITICAL (propagated) |
| 346-348 | Avg energy metric wrong | CRITICAL (propagated) |
| 407-409 | Data mutation from merge_playlists | HIGH |

---

## Summary Statistics

### By Severity
- **CRITICAL:** 4 bugs (must fix - will crash or break core features)
- **HIGH:** 5 bugs (should fix - data issues, incorrect behavior)
- **MEDIUM:** 3 bugs (nice to fix - UX, robustness improvements)
- **Total:** 12 bugs

### By File
- **playlist_logic.py:** 7 bugs
- **app.py:** 5 bugs

### By Type
- **Logic errors:** 5 (inverted search, wrong fields, wrong variables)
- **Data/mutation issues:** 2 (merge mutation, empty list crashes)
- **Calculation errors:** 2 (ratio denominator, energy subset)
- **UI/UX issues:** 2 (selectbox persistence, validation)
- **Type safety:** 1 (missing isinstance check)

---

## Implementation Checklist

### Critical (Do First)
- [ ] Fix search_songs() logic (1 line)
- [ ] Fix random_choice_or_none() (2 lines)
- [ ] Fix hype_ratio denominator (1 line)
- [ ] Fix avg_energy subset (1 line)

### High Priority (Do Second)
- [ ] Fix merge_playlists mutation (1 line)
- [ ] Fix chill_keyword field (1 line)
- [ ] Fix selectbox persistence (3 lines)

### Medium Priority (Polish)
- [ ] Add type check to normalize_artist()
- [ ] Improve keyword matching
- [ ] Add input validation feedback
- [ ] Clean up error handling

---

## How Each Document Helps

| Need | Read This |
|------|-----------|
| "What's broken?" | AUDIT_SUMMARY.txt |
| "How bad is it?" | AUDIT_REPORT.md |
| "How do I fix it?" | FIX_GUIDE.md |
| "What's on line 185?" | playlist_logic.py (with inline comments) |
| "What's the priority?" | AUDIT_SUMMARY.txt or AUDIT_REPORT.md |
| "Where do I start?" | AUDIT_SUMMARY.txt |

---

## Key Insights

### Patterns Identified
1. **Inverted logic** - Search condition backwards
2. **Field confusion** - Checking wrong variables for same check
3. **Mutation** - List assignment creates reference not copy
4. **Subset errors** - Statistics from wrong data subset
5. **State management** - Streamlit UI not reading state correctly
6. **Type safety gaps** - Inconsistent defensive programming

### Root Causes
- Likely copy-paste errors (avg_energy / hype_ratio)
- Variable naming confusion (title vs genre)
- Python reference semantics misunderstanding (mutation)
- Incomplete testing of edge cases (empty lists)
- UI state management gaps (selectbox index)

### Preventative Measures
- Unit tests for edge cases (empty lists, None values)
- Assertions on statistical properties (0 <= ratio <= 1)
- Code review checklist for mutation-prone operations
- Consistent patterns for parallel logic paths
- Test Streamlit interactions with state changes

---

## Questions?

Each document is self-contained and can be read independently:
- AUDIT_SUMMARY.txt = 2 minute read
- AUDIT_REPORT.md = 10 minute read
- FIX_GUIDE.md = 20 minute read
- Inline comments in code = context-aware reference

Start with AUDIT_SUMMARY.txt if unsure.
