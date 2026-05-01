# Weekly Habit Report Template

Used to generate a weekly rollup report. Place at:
`Areas/Habits/Reports/Weekly YYYY-WNN.md`

Replace placeholders with computed values.

```markdown
---
week: {{WEEK}}
period: {{START_DATE}} to {{END_DATE}}
generated: {{GENERATED_DATE}}
---

# Weekly Habit Report — Week {{WEEK_NUM}}

## Overview

- Period: {{START_DATE}} → {{END_DATE}}
- Total habits tracked: {{TOTAL_HABITS}}
- Overall completion rate: **{{OVERALL_RATE}}%**

## Per-Habit Summary

| Habit | Category | Done | Missed | Skipped | Rate | Streak |
|-------|----------|------|--------|---------|------|--------|
{{PER_HABIT_ROWS}}

## Category Breakdown

{{CATEGORY_BREAKDOWN}}

## Daily Heatmap

| Day | {{HABIT_COLUMNS}} | Total |
|-----{{HABIT_COLUMN_SEPARATORS}}|-------|
{{DAILY_HEATMAP_ROWS}}

## Highlights

- 🔥 **Longest streak**: {{LONGEST_STREAK_HABIT}} ({{LONGEST_STREAK_COUNT}} days)
- ⬆️ **Most improved**: {{MOST_IMPROVED_HABIT}} (+{{IMPROVEMENT_DELTA}} vs last week)
- ⚠️ **Needs attention**: {{NEEDS_ATTENTION_HABIT}} ({{NEEDS_ATTENTION_RATE}}% rate, {{NEEDS_ATTENTION_STREAK}}-day streak)
```

## Generating the weekly report

```bash
VAULT="${OBSIDIAN_VAULT_PATH:-$HOME/obsidian-vault}"
WEEK="$(date +%G-W%V)"  # ISO week
START_DATE="$(date -d 'last monday' +%Y-%m-%d)"  # or computed from week
END_DATE="$(date -d 'last sunday' +%Y-%m-%d)"     # or computed from week
REPORT="$VAULT/Areas/Habits/Reports/Weekly $WEEK.md"

mkdir -p "$(dirname "$REPORT")"

python3 -c "
import os, re, glob
from datetime import datetime, timedelta

vault = os.path.expanduser('${OBSIDIAN_VAULT_PATH:-~/obsidian-vault}')

# Compute start/end of the week
today = datetime.now().date()
# ISO week starts Monday
start_of_week = today - timedelta(days=today.weekday())
end_of_week = start_of_week + timedelta(days=6)

# Read registry for habit list
registry_path = os.path.join(vault, 'Areas', 'Habits', 'Habits Registry.md')
with open(registry_path) as f:
    content = f.read()

habits = []
categories = {}
for line in content.split('\n'):
    if line.startswith('|') and '|' in line[1:]:
        parts = [p.strip() for p in line.split('|')]
        if len(parts) >= 5 and parts[1] and parts[1] != 'Habit' and not '---' in parts[1]:
            habit = parts[1]
            category = parts[2]
            habits.append(habit)
            categories[habit] = category

# Read daily logs for the week
week_data = {}
for i in range(7):
    d = start_of_week + timedelta(days=i)
    fpath = os.path.join(vault, 'Areas', 'Habits', 'Daily', f'{d.isoformat()}.md')
    day_name = d.strftime('%A')
    if os.path.exists(fpath):
        with open(fpath) as f:
            day_content = f.read()
        day_data = {}
        for habit in habits:
            pattern = rf'\|\s*{re.escape(habit)}\s*\|.*?\|\s*(.+?)\s*\|'
            m = re.search(pattern, day_content)
            if m:
                status = m.group(1).strip()
                day_data[habit] = status
        week_data[day_name] = day_data

# Compute per-habit stats
habit_stats = {}
for habit in habits:
    done = missed = skipped = 0
    for day_data in week_data.values():
        status = day_data.get(habit, 'N/A')
        if '✅' in status or status == '✅ done':
            done += 1
        elif '❌' in status or status == '❌ missed':
            missed += 1
        elif '⏭️' in status or status == '⏭️ skipped':
            skipped += 1
    total = done + missed
    rate = round(done / total * 100) if total > 0 else 0
    habit_stats[habit] = {'done': done, 'missed': missed, 'skipped': skipped, 'rate': rate}

# Output results
for habit, stats in habit_stats.items():
    cat = categories.get(habit, '?')
    print(f'{habit} ({cat}): {stats[\"done\"]} done, {stats[\"missed\"]} missed, {stats[\"skipped\"]} skipped, {stats[\"rate\"]}% rate')
"
```

## Notes

- ISO week format: use `%G-W%V` (year-week), NOT `%Y-W%W`.
- If a daily log is missing for a day in the period, count that habit as N/A (not counted toward done or missed).
- Compare with the previous week's report if it exists for "most improved" calculation.