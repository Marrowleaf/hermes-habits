# Daily Habit Log Template

Used to create a new daily habit log note. Place at:
`Areas/Habits/Daily/YYYY-MM-DD.md`

Replace `{{DATE}}` with the ISO date, `{{DAY}}` with the weekday name,
and `{{HABIT_ROWS}}` with one row per habit from the registry.

```markdown
---
date: {{DATE}}
day: {{DAY}}
---

# Habit Log — {{DATE}} ({{DAY}})

## Today's Habits

{{HABIT_ROWS}}

## Summary

- Completed: 0 / {{TOTAL_HABITS}} (0%)
- Streaks: _(computed on read)_
```

## Row format per habit

Each habit from the registry gets a row:

```markdown
| {{HABIT_NAME}} | {{CATEGORY}} | ⬜ pending |  |
```

Where `{{HABIT_NAME}}`, `{{CATEGORY}}` come from the registry.

## Generating the daily note

```bash
VAULT="${OBSIDIAN_VAULT_PATH:-$HOME/obsidian-vault}"
DATE="$(date +%Y-%m-%d)"
DAY="$(date +%A)"
DAILY="$VAULT/Areas/Habits/Daily/$DATE.md"

mkdir -p "$(dirname "$DAILY")"

python3 -c "
import os, re

vault = os.path.expanduser('${OBSIDIAN_VAULT_PATH:-~/obsidian-vault}')
date = '$DATE'
day = '$DAY'
registry_path = os.path.join(vault, 'Areas', 'Habits', 'Habits Registry.md')
daily_path = os.path.join(vault, 'Areas', 'Habits', 'Daily', f'{date}.md')

# Read registry to get habits
with open(registry_path) as f:
    content = f.read()

# Parse habit rows from the registry table
habits = []
in_table = False
for line in content.split('\n'):
    if line.startswith('|') and 'Habit' in line and 'Category' in line:
        in_table = True
        continue
    if line.startswith('|') and '---' in line:
        continue
    if in_table and line.startswith('|'):
        parts = [p.strip() for p in line.split('|')]
        if len(parts) >= 5:
            habit = parts[1]
            category = parts[2]
            habits.append((habit, category))
    elif in_table and not line.startswith('|'):
        break

# Build daily log
rows = '\n'.join([f'| {h} | {c} | ⬜ pending |  |' for h, c in habits])
total = len(habits)

daily_note = f'''---
date: {date}
day: {day}
---

# Habit Log — {date} ({day})

## Today's Habits

| Habit | Category | Status | Note |
|-------|----------|--------|------|
{rows}

## Summary

- Completed: 0 / {total} (0%)
- Streaks: _(computed on read)_
'''

os.makedirs(os.path.dirname(daily_path), exist_ok=True)
with open(daily_path, 'w') as f:
    f.write(daily_note)

print(f'Created daily log: {daily_path}')
print(f'  Habits: {total}')
"
```

## Notes

- If the daily log already exists for the date, do NOT overwrite it — read and update in place.
- All `⬜ pending` habits should be populated from the registry on creation.
- The streak line is a placeholder; actual streak values are computed dynamically when reading.