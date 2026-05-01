# Habits

> Track daily habits in Obsidian with streaks, completion rates, and weekly/monthly rollups.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://github.com/Marrowleaf/hermes-habits)

## Features

- Natural language habit logging ("I meditated", "missed reading")
- Streak tracking with automatic calculation from log data
- Weekly and monthly habit reports with completion rates
- Custom habit categories and target frequencies (daily, weekdays, 3x/week)
- Obsidian-integrated data storage with PARA structure
- Cross-references with fitness-nutrition skill for workout data

## Installation

```bash
hermes skills install health/habits
```

Or manually clone into `~/.hermes/skills/health/habits/`.

## Usage

```
habits log "drank 2L water"
habits log "missed meditation"
habits streak meditation
habits report week
habits add gym --category fitness --target 5x-week
habits summary
```

## Configuration

Store habit data in your Obsidian vault at `~/obsidian-vault/3-Resources/habits/`. Configure habits and categories in `config.md`. All dates use BST (UTC+1) for Oakham, UK.

## Requirements

- Hermes Agent v0.12+
- Obsidian vault (for data storage)

## License

MIT