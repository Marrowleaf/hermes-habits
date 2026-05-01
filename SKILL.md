---
name: habits
description: "Track daily habits in Obsidian with streaks, completion rates, and weekly/monthly rollups."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [habits, tracking, streaks, obsidian, daily, wellness, fitness, productivity]
    related_skills: [obsidian, fitness-nutrition]
commands:
  - name: log
    description: "Log a habit completion or miss. Parses natural language like 'drank water', 'did workout', 'missed meditation'."
    usage: habits log "drank 2L water" | habits log "missed meditation" | habits log "ran 5km"
    examples:
      - "I drank water today"
      - "did my morning meditation"
      - "missed reading last night"
      - "completed cold shower"
  - name: list
    description: "Show all tracked habits with current streak and today's status."
    usage: habits list
  - name: streak
    description: "Show streak for a specific habit or all habits."
    usage: habits streak [habit-name]
    examples:
      - "habits streak meditation"
      - "habits streak" # all habits
  - name: report
    description: "Generate a weekly or monthly habit rollup report."
    usage: habits report [week|month] [YYYY-MM-DD]
    examples:
      - "habits report week"
      - "habits report month 2026-04"
  - name: add
    description: "Add a new habit to track with optional category and target frequency."
    usage: habits add <name> [--category <cat>] [--target <daily|weekdays|3x-week>]
    examples:
      - "habits add meditation --category wellness --target daily"
      - "habits add gym --category fitness --target week-days"
  - name: remove
    description: "Archive a habit (stops tracking but preserves history)."
    usage: habits remove <name>
  - name: summary
    description: "Quick summary of today's habit status."
    usage: habits summary
---

# Habits

Track daily habits with natural language input, streak counting, and Obsidian-integrated rollup reports.

## When to Use This Skill

- User mentions completing or missing a recurring activity ("I meditated", "forgot my supplements")
- User asks about their habit streaks or completion rates
- User wants a weekly or monthly habit report
- User wants to start tracking a new habit

## File Structure

All habit data lives in the Obsidian vault:

```
~/obsidian-vault/
├── 3-Resources/habits/
│   ├── config.md                    # Habit definitions and categories
│   ├── 2026-05.md                   # Monthly log (auto-created)
│   └── 2026-05-W18.md              # Weekly reports (auto-created)
└── 2-Areas/Health & Fitness/
    └── Habit Reports/
        └── 2026-05.md              # Monthly rollup report
```

## Step 1: Parse Natural Language

When the user says something like "I meditated" or "did my workout", extract:

1. **Habit name** — match against configured habits (fuzzy match allowed)
2. **Status** — completed ✓ or missed ✗ (default: completed if not explicitly "missed" or "forgot")
3. **Date** — default to today; parse "yesterday", "on Monday", etc.
4. **Note** — any extra context ("ran 5km", "2L water")

### Parsing rules:

| Input pattern | Habit | Status | Note |
|---|---|---|---|
| "did [habit]" | habit | ✓ | — |
| "completed [habit]" | habit | ✓ | — |
| "missed [habit]" | habit | ✗ | — |
| "forgot [habit]" | habit | ✗ | — |
| "I [verb] [noun]" | match to nearest habit | ✓ | verb+noun |
| "[number] [unit] [habit]" | habit | ✓ | number+unit |
| "[habit] [x] days in a row" | habit | ✓ (streak claim) | verify against log |

If the habit name doesn't match any configured habit, ask if they want to add it.

## Step 2: Read or Create Monthly Log

Monthly logs are stored at `3-Resources/habits/YYYY-MM.md`:

```markdown
---
type: habit-log
month: 2026-05
habits: [meditation, water, workout, reading, cold-shower]
---

# Habit Log — May 2026

| Date | Meditation | Water | Workout | Reading | Cold Shower | Notes |
|------|-----------|-------|---------|---------|-------------|-------|
| 2026-05-01 | ✓ | ✓ 2L | — | ✓ 30min | ✗ | Good start |
| 2026-05-02 | ✓ | ✓ 3L | ✓ Upper | ✓ 20min | ✓ | |
```

- ✓ = completed
- ✗ = missed
- — = not tracked (e.g., rest day for workout)
- ✓ [detail] = completed with note

## Step 3: Update Streaks

Streaks are calculated from the log data, NOT stored separately.

**Streak rules:**
- Current streak: consecutive days with ✓ ending on today or yesterday
- If today is incomplete, streak counts up to yesterday
- Longest streak: max consecutive ✓ days in the month
- Broken streak: any ✗ in the chain resets it
- Rest days (—) do NOT break streaks for habits with target < daily

**Calculation (Python):**
```python
import re
from datetime import date, timedelta

def calculate_streak(log_content: str, habit: str, month: str) -> dict:
    """Parse the markdown table and calculate streaks for a habit."""
    rows = [line for line in log_content.split('\n') if line.startswith('|') and '2026' in line]
    dates = []
    statuses = []
    for row in rows:
        cells = [c.strip() for c in row.split('|')[1:-1]]
        dates.append(cells[0])
        # Find the column index for this habit
        # ... parse and calculate
    # Walk backwards from most recent date
    current = 0
    longest = 0
    temp = 0
    for s in statuses:
        if '✓' in s:
            temp += 1
            current = temp
            longest = max(longest, temp)
        elif '—' in s:
            pass  # skip rest days
        else:
            temp = 0
    return {"current": current, "longest": longest}
```

## Step 4: Generate Reports

### Weekly Report (`3-Resources/habits/YYYY-MM-W##.md`)

```markdown
---
type: habit-report-weekly
week: 2026-W18
period: 2026-04-27 to 2026-05-03
---

# Habit Report — Week 18

## Overview
- **Completion Rate:** 78% (28/36 possible checkmarks)
- **Best Day:** Wednesday (5/5)
- **Worst Day:** Saturday (2/5)

## Habit Breakdown
| Habit | Completed | Rate | Current Streak | Longest Streak |
|-------|-----------|------|---------------|----------------|
| Meditation | 6/7 | 86% | 🔥 12 | 18 |
| Water 2L+ | 7/7 | 100% | 🔥 21 | 21 |
| Workout | 3/7 | 43% | 1 | 5 |
| Reading | 5/7 | 71% | 3 | 10 |
| Cold Shower | 4/7 | 57% | 1 | 8 |

## Highlights
- 🔥 Water streak at 21 days!
- Workout needs attention — only 3 days this week
```

### Monthly Report (`2-Areas/Health & Fitness/Habit Reports/YYYY-MM.md`)

Contains weekly summaries + overall month stats + trend arrows vs previous month.

## Step 5: Habit Configuration

`3-Resources/habits/config.md`:

```markdown
---
type: habit-config
---

# Habit Configuration

| Habit | Category | Target | Icon | Start Date |
|-------|----------|--------|------|-----------|
| Meditation | Wellness | Daily | 🧘 | 2026-01-15 |
| Water 2L+ | Health | Daily | 💧 | 2026-01-15 |
| Workout | Fitness | 5x/week | 💪 | 2026-01-15 |
| Reading 30min | Learning | Daily | 📚 | 2026-02-01 |
| Cold Shower | Wellness | Daily | 🥶 | 2026-03-01 |
| Supplements | Health | Daily | 💊 | 2026-03-15 |
```

## Integration with fitness-nutrition

When both skills are loaded:
- Workout completions in habits cross-reference with fitness-nutrition skill
- Water intake logs can feed into daily wellness tracking
- Monthly fitness + habit reports can be merged

## Common Pitfalls

1. **Don't create duplicate date rows** — Always check if today's row exists before appending
2. **Column order matches config** — Habits appear in the same order as `config.md`
3. **Month boundaries** — When logging for yesterday across month boundaries, write to the correct month file
4. **Timezone** — All dates in BST (UTC+1) per user's location in Oakham, UK
5. **Streaks don't survive archive** — If a month file is missing, streaks reset. Always create next month's file before the 1st.
6. **Markdown table alignment** — Use `|` separators and pad cells to keep tables readable in Obsidian

## Verification

After any habit operation:
1. `cat ~/obsidian-vault/3-Resources/habits/$(date +%Y-%m).md` — verify today's row exists
2. `cat ~/obsidian-vault/3-Resources/habits/config.md` — verify habit is listed
3. Streak calculation against the last 14 days of log data