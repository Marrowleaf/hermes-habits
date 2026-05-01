# Habit Categories & Icons Reference

## Default Categories

| Category      | Description                          | Default Icons              |
|---------------|--------------------------------------|---------------------------|
| health        | Nutrition, hydration, sleep, wellness| 💧 🛌 💊 🥗 🦷            |
| fitness       | Exercise, movement, stretching       | 🏋️ 🏃 🧘‍♂️ 🚶 💪            |
| learning      | Reading, studying, creative practice| 📖 📝 🎸 🎨 🎓            |
| productivity | Focus, planning, deep work          | 🎯 📋 ⏱️ 💻 📅             |
| mindfulness   | Meditation, gratitude, reflection    | 🧘 🙏 🌿 🕯️ ✨             |
| personal      | Self-care, social, chores            | 🧹 📞 💬 🛁 🌙             |

## Adding Custom Categories

Users can add custom categories by adding them to the `## Categories` section
of `Areas/Habits/Habits Registry.md`. Add a new bullet point:

```markdown
- **custom-category** — description of what this category covers
```

Then use the category name when adding new habits. The category name is
case-insensitive but should be lowercase for consistency.

## Icon Selection

When adding a new habit, choose an appropriate emoji icon. Default mappings:

| Habit Type          | Suggested Icons  |
|---------------------|-------------------|
| Water/hydration     | 💧                |
| Exercise/workout    | 🏋️                |
| Running             | 🏃                |
| Walking             | 🚶                |
| Yoga                | 🧘‍♂️               |
| Meditation          | 🧘                |
| Reading             | 📖                |
| Writing/journaling | ✍️                |
| Sleeping            | 🛌                |
| Healthy eating      | 🥗                |
| Vitamins/medicine   | 💊                |
| Stretching          | 🤸                |
| Studying            | 📝                |
| Music practice      | 🎸                |
| Deep work           | 🎯                |
| Planning            | 📋                |
| Gratitude           | 🙏                |
| Decluttering        | 🧹                |
| Social/calling      | 📞                |
| Self-care           | 🛁                |
| Flossing            | 🦷                |

If no icon is specified, default to `📌`.

## Habit Target Frequencies

| Target    | Description                                    |
|-----------|------------------------------------------------|
| daily     | Every day (7 days/week)                        |
| weekdays  | Monday through Friday (5 days/week)            |
| weekly    | Once per week (tracked per week, not per day)  |
| 3x/week   | Three times per week — minimum                 |
| custom    | Specific schedule (e.g., "MWF" or "every other day") |

For non-daily targets, daily logs should still have a row, but on off-days
the status should be `⏭️ skipped` (pre-scheduled rest). The completion rate
for weekly targets is computed per-week, not per-day.