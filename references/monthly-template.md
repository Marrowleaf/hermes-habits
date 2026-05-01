# Monthly Habit Report Template

Used to generate a monthly rollup report. Place at:
`Areas/Habits/Reports/Monthly YYYY-MM.md`

Replace placeholders with computed values.

```markdown
---
month: {{MONTH}}
generated: {{GENERATED_DATE}}
---

# Monthly Habit Report — {{MONTH}}

## Overview

- Period: {{MONTH}}
- Total habits tracked: {{TOTAL_HABITS}}
- Overall completion rate: **{{OVERALL_RATE}}%**
- Perfect days: {{PERFECT_DAYS}} / {{TOTAL_DAYS}}

## Per-Habit Summary

| Habit | Category | Done | Missed | Skipped | Rate | Best Streak |
|-------|----------|------|--------|---------|------|-------------|
{{PER_HABIT_ROWS}}

## Category Breakdown

{{CATEGORY_BREAKDOWN}}

## Week-by-Week Trend

| Week | Completion Rate | vs Previous |
|------|----------------|-------------|
{{WEEKLY_TREND_ROWS}}

## Highlights

- 🏆 **Best habit**: {{BEST_HABIT}} ({{BEST_RATE}}%, {{BEST_STREAK}}-day streak)
- 📈 **Most improved**: {{MOST_IMPROVED_HABIT}} ({{PREV_RATE}}% → {{CURR_RATE}}%)
- ⚠️ **Weakest**: {{WEAKEST_HABIT}} ({{WEAKEST_RATE}}%, {{WEAK_STREAK}} days longest gap)
- 📅 **Perfect days**: {{PERFECT_DAYS}} out of {{TOTAL_DAYS}} ({{PERFECT_RATE}}%)
```

## Generating the monthly report

The monthly report aggregates all daily logs for a given month and optionally
references weekly reports for the week-by-week trend.

```bash
VAULT="${OBSIDIAN_VAULT_PATH:-$HOME/obsidian-vault}"
MONTH="$(date +%Y-%m)"
REPORT="$VAULT/Areas/Habits/Reports/Monthly $MONTH.md"

mkdir -p "$(dirname "$REPORT")"

python3 -c "
import os, re, glob, math
from datetime import datetime, timedelta

vault = os.path.expanduser('${OBSIDIAN_VAULT_PATH:-~/obsidian-vault}')
month = '$MONTH'  # YYYY-MM format
year, mon = map(int, month.split('-'))

# Compute start/end of month
start_date = datetime(year, mon, 1).date()
if mon == 12:
    end_date = datetime(year + 1, 1, 1).date() - timedelta(days=1)
else:
    end_date = datetime(year, mon + 1, 1).date() - timedelta(days=1)

total_days = (end_date - start_date).days + 1

# Read registry for habit list
registry_path = os.path.join(vault, 'Areas', 'Habits', 'Habits Registry.md')
with open(registry_path) as f:
    content = f.read()

habits = []
categories = {}
added_dates = {}
for line in content.split('\n'):
    if line.startswith('|') and '|' in line[1:]:
        parts = [p.strip() for p in line.split('|')]
        if len(parts) >= 5 and parts[1] and parts[1] != 'Habit' and not '---' in parts[1]:
            habit = parts[1]
            category = parts[2]
            added = parts[5] if len(parts) > 5 else str(start_date)
            habits.append(habit)
            categories[habit] = category
            added_dates[habit] = added

# Track per-habit and per-category stats
from collections import defaultdict
habit_data = {h: {'done': 0, 'missed': 0, 'skipped': 0, 'na': 0, 'best_streak': 0, 'current_streak': 0} for h in habits}
category_totals = defaultdict(lambda: {'done': 0, 'possible': 0})
perfect_days = 0
days_checked = 0

# Read each daily log in the month
d = start_date
while d <= end_date:
    fpath = os.path.join(vault, 'Areas', 'Habits', 'Daily', f'{d.isoformat()}.md')
    if os.path.exists(fpath):
        days_checked += 1
        with open(fpath) as f:
            day_content = f.read()

        day_done = 0
        day_total = 0
        for habit in habits:
            # Skip if habit was added after this date
            try:
                if d.isoformat() < added_dates.get(habit, '2000-01-01'):
                    continue
            except:
                pass

            pattern = rf'\|\s*{re.escape(habit)}\s*\|.*?\|\s*(.+?)\s*\|'
            m = re.search(pattern, day_content)
            if m:
                status = m.group(1).strip()
                day_total += 1
                if '✅' in status or status == '✅ done':
                    habit_data[habit]['done'] += 1
                    day_done += 1
                    category_totals[categories[habit]]['done'] += 1
                    category_totals[categories[habit]]['possible'] += 1
                elif '❌' in status or status == '❌ missed':
                    habit_data[habit]['missed'] += 1
                    category_totals[categories[habit]]['possible'] += 1
                elif '⏭️' in status or status == '⏭️ skipped':
                    habit_data[habit]['skipped'] += 1
                    # Skipped doesn't count toward possible for rate
                else:
                    habit_data[habit]['na'] += 1
        if day_total > 0 and day_done == day_total:
            perfect_days += 1
    d += timedelta(days=1)

# Compute best streaks (read all daily logs again, this time tracking streaks)
for habit in habits:
    streak = 0
    best_streak = 0
    d = start_date
    while d <= end_date:
        fpath = os.path.join(vault, 'Areas', 'Habits', 'Daily', f'{d.isoformat()}.md')
        if os.path.exists(fpath):
            with open(fpath) as f:
                dc = f.read()
            pattern = rf'\|\s*{re.escape(habit)}\s*\|.*?\|\s*(.+?)\s*\|'
            m = re.search(pattern, dc)
            if m:
                status = m.group(1).strip()
                if '✅' in status or status == '✅ done':
                    streak += 1
                    best_streak = max(best_streak, streak)
                elif '❌' in status or status == '❌ missed':
                    streak = 0
                # skipped: neutral, don't break but don't extend
        d += timedelta(days=1)
    habit_data[habit]['best_streak'] = best_streak

# Print summary
overall_done = sum(h['done'] for h in habit_data.values())
overall_possible = sum(h['done'] + h['missed'] for h in habit_data.values())
overall_rate = round(overall_done / overall_possible * 100) if overall_possible > 0 else 0

print(f'Month: {month}')
print(f'Days in month: {total_days}')
print(f'Days logged: {days_checked}')
print(f'Overall: {overall_rate}% ({overall_done}/{overall_possible})')
print(f'Perfect days: {perfect_days}/{days_checked}')
print()
for habit in habits:
    h = habit_data[habit]
    total = h['done'] + h['missed']
    rate = round(h['done'] / total * 100) if total > 0 else 0
    print(f'  {habit} ({categories[habit]}): {h[\"done\"]} done, {h[\"missed\"]} missed, {h[\"skipped\"]} skipped, {rate}%, best streak: {h[\"best_streak\"]}')
print()
for cat, ct in category_totals.items():
    rate = round(ct['done'] / ct['possible'] * 100) if ct['possible'] > 0 else 0
    print(f'  {cat}: {rate}% ({ct[\"done\"]}/{ct[\"possible\"]})')
"
```

## Week-by-Week Trend

To compute the week-by-week trend, iterate through each ISO week in the month:

```python
# Given start_date and end_date of month
from datetime import timedelta

weeks = []
current = start_date
while current <= end_date:
    week_start = current
    week_end = min(current + timedelta(days=6 - current.weekday()), end_date)
    # Compute completion rate for this week (same logic as weekly report)
    weeks.append((week_start, week_end))
    current = week_end + timedelta(days=1)
```

Compare each week's rate to the previous week for the trend arrows (⬆️/⬇️/→).

## Notes

- A "perfect day" is when all habits tracked that day are `✅ done`.

- `⬜ pending` entries are excluded from rate calculations (day not over yet).

- For habits added mid-month, only count from the "Added" date in the registry.
  If a habit was added on the 15th, don't penalize days 1-14.

- Missing daily logs for past dates are treated as `❌ missed` for streaks,
  but excluded from rate calculations.

- The "most improved" metric compares this month's rate to last month's rate
  for each habit. Read the previous monthly report if it exists.