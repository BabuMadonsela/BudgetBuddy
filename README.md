# 💰 BudgetBuddy

> A personal finance Android app built in Kotlin — track expenses, set budget goals, attach receipt photos, visualise spending, and earn badges for good habits.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Screenshots & Screens](#screenshots--screens)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Database Schema](#database-schema)
- [Getting Started](#getting-started)
- [Permissions](#permissions)
- [Dark Mode](#dark-mode)
- [Achievements & Badges](#achievements--badges)
- [Data Export](#data-export)
- [Security](#security)
- [Known Limitations](#known-limitations)
- [Future Improvements](#future-improvements)

---

## Overview

BudgetBuddy is a fully offline, single-user Android expense tracker. All data is stored locally in SQLite — no internet connection, no cloud account, no subscriptions required. Users register and log in with a username and password, then track their spending across custom categories, set monthly budget goals, attach receipt photos directly to expenses, and view their spending through graphs and tables.

---

## Features

### 💳 Expense Tracking
- Add, edit, and delete expenses with amount, description, date, and category
- Attach receipt photos — taken from camera or chosen from gallery — stored as compressed BLOBs in SQLite
- View expenses filtered by Today, This Week, This Month, or a custom date range
- Sortable transaction table with columns: Date, Description, Uploads (thumbnail), Category, Amount
- Long-press any row to edit or delete; tap a row with a photo to view the full receipt

### 📊 Spending Graphs
- Daily spending bar chart for any date range
- Category breakdown pie/bar chart with colour-coded legend
- Goal lines on charts showing min/max budget bands per category

### 🎯 Budget Goals
- Set an overall monthly budget limit
- Set per-category Min (minimum expected spend) and Max (spending limit) bands
- Dashboard progress bars turn red with an overspend warning when a category limit is exceeded

### 🏅 Achievements & Badges
- Six earnable badges based on real usage milestones
- Budget Score (0–100) calculated monthly and displayed as a tier: Bronze → Silver → Gold → Platinum
- Streak counter tracking consecutive days with at least one logged expense

### ⚙️ Settings
- Change username and password
- Toggle professional dark mode (applies instantly across all screens)
- Manage expense categories — create, rename, recolour, and delete
- Export all expenses to a timestamped CSV file, shareable via any installed app
- Reset all data (expenses, goals, badges) while keeping the account

### 🔐 Auth
- Local registration and login
- Passwords hashed with SHA-256 before storage — never stored in plain text
- Session persisted in SharedPreferences across app restarts

---

## Screenshots & Screens

| Screen | Description |
|---|---|
| **AuthActivity** | Login / Register — switches between modes on one screen |
| **MainActivity** | Dashboard — budget progress, score, streak, category bars |
| **AddExpenseActivity** | Add or edit an expense with optional receipt photo |
| **ExpenseListActivity** | Transaction table with period filter and category totals |
| **SpendingGraphActivity** | Daily and category spending charts |
| **BudgetGoalsActivity** | Set total and per-category min/max budget bands |
| **AchievementsActivity** | Badge gallery with earn status and budget score |
| **ManageCategoriesActivity** | Create, edit colour, rename, delete categories |
| **SettingsActivity** | Account settings, dark mode, export, danger zone |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Kotlin |
| Minimum SDK | API 24 (Android 7.0) |
| Target SDK | API 34 (Android 14) |
| UI | Native Android Views (XML layouts, no Compose) |
| Database | SQLite via `SQLiteOpenHelper` |
| Image handling | `BitmapFactory` + JPEG compression to BLOB |
| File sharing | `FileProvider` |
| Session | `SharedPreferences` |
| Dark mode | `AppCompatDelegate.setDefaultNightMode()` |
| Password hashing | `MessageDigest` SHA-256 |
| Charts | Custom programmatic drawing (Canvas) |

---

## Project Structure

```
app/
├── src/main/
│   ├── java/com/budgetbuddy/app/
│   │   ├── AuthActivity.kt            # Login & registration
│   │   ├── MainActivity.kt            # Dashboard
│   │   ├── AddExpenseActivity.kt      # Add / edit expense + receipt photo
│   │   ├── ExpenseListActivity.kt     # Transaction table + adapter
│   │   ├── SpendingGraphActivity.kt   # Spending charts
│   │   ├── BudgetGoalsActivity.kt     # Budget goal setting
│   │   ├── AchievementsActivity.kt    # Badges & score
│   │   ├── ManageCategoriesActivity.kt# Category CRUD
│   │   ├── SettingsActivity.kt        # App settings
│   │   ├── DatabaseHelper.kt          # SQLite helper — all DB operations
│   │   ├── SessionManager.kt          # SharedPreferences session
│   │   └── Models.kt                  # Data classes: User, Category, Expense, Badge
│   │
│   └── res/
│       ├── layout/
│       │   ├── activity_auth.xml
│       │   ├── activity_main.xml
│       │   ├── activity_add_expense.xml
│       │   ├── activity_expense_list.xml
│       │   ├── activity_spending_graph.xml
│       │   ├── activity_budget_goals.xml
│       │   ├── activity_achievements.xml
│       │   ├── activity_manage_categories.xml
│       │   └── activity_settings.xml
│       ├── drawable/
│       │   ├── gradient_button.xml
│       │   ├── gradient_header.xml
│       │   ├── input_border_gray.xml
│       │   ├── input_border_orange.xml
│       │   ├── outline_green.xml
│       │   ├── outline_purple.xml
│       │   ├── badge_locked.xml
│       │   ├── badge_unlocked.xml
│       │   ├── chip_unlocked.xml
│       │   ├── circle_blue/green/orange/purple.xml
│       │   ├── dashed_border.xml
│       │   └── spinner_bg_white.xml
│       ├── values/
│       │   ├── strings.xml
│       │   ├── colors.xml
│       │   └── themes.xml
│       └── xml/
│           ├── backup_rules.xml
│           ├── data_extraction_rules.xml
│           └── file_provider_paths.xml
│
└── AndroidManifest.xml
```

---

## Database Schema

BudgetBuddy uses a single SQLite database managed by `DatabaseHelper`. Below are the core tables.

### `users`
| Column | Type | Notes |
|---|---|---|
| `_id` | INTEGER PK | Auto-increment |
| `username` | TEXT UNIQUE | Case-sensitive |
| `password_hash` | TEXT | SHA-256 hex digest |

### `categories`
| Column | Type | Notes |
|---|---|---|
| `_id` | INTEGER PK | Auto-increment |
| `user_id` | INTEGER | Foreign key → users |
| `name` | TEXT | Unique per user |
| `colour` | TEXT | Hex colour e.g. `#4CAF50` |

Five default categories are seeded on first registration: Groceries, Transport, Entertainment, Utilities, Health.

### `expenses`
| Column | Type | Notes |
|---|---|---|
| `_id` | INTEGER PK | Auto-increment |
| `user_id` | INTEGER | Foreign key → users |
| `category_id` | INTEGER | Foreign key → categories |
| `amount` | REAL | Positive decimal |
| `description` | TEXT | Required |
| `date` | TEXT | Format: `yyyy-MM-dd` |
| `photo_blob` | BLOB | JPEG compressed to ≤500KB, nullable |

### `budget_goals`
| Column | Type | Notes |
|---|---|---|
| `user_id` | INTEGER | |
| `category_id` | INTEGER | `-1` = overall monthly budget |
| `monthly_limit` | REAL | Max limit |
| `min_limit` | REAL | Minimum expected |
| `max_limit` | REAL | Maximum allowed |

### `user_prefs`
| Column | Type | Notes |
|---|---|---|
| `user_id` | INTEGER PK | |
| `dark_mode` | INTEGER | `0` = light, `1` = dark |

### `badges`
| Column | Type | Notes |
|---|---|---|
| `user_id` | INTEGER | |
| `badge_id` | TEXT | e.g. `STREAK_7`, `BUDGET_HERO` |

---

## Getting Started

### Prerequisites
- Android Studio Hedgehog (2023.1.1) or newer
- Android SDK API 34
- A physical device or emulator running Android 7.0+

### Build & Run

1. Clone or download the project
2. Open the project in Android Studio (`File → Open`)
3. Let Gradle sync complete
4. Run on a device or emulator (`Shift + F10`)

No API keys, no external services, no `.env` files needed — the app runs entirely offline.

### First Launch
1. The app opens on the **Login** screen
2. Tap **"Don't have an account? Register"** to create your account
3. Enter a username and a password of at least 8 characters
4. After registering, log in — you'll land on the **Dashboard**
5. Head to **Budget Goals** to set your monthly budget, then start adding expenses

---

## Permissions

| Permission | Why it's needed |
|---|---|
| `CAMERA` | Take receipt photos directly within the app |
| `READ_EXTERNAL_STORAGE` | Pick existing photos from gallery (Android ≤ 12) |
| `WRITE_EXTERNAL_STORAGE` | Save CSV exports (Android ≤ 9) |
| `READ_MEDIA_IMAGES` | Pick photos from gallery (Android 13+) |

Camera permission is requested at runtime only when the user chooses "Take Photo". All other permissions are handled via scoped storage where applicable.

---

## Dark Mode

Dark mode is toggled from **Settings → Display → Dark Mode**. It applies instantly to every screen using `AppCompatDelegate.setDefaultNightMode()` — no restart required. The preference is saved per user in the local database and restored automatically on every app launch.

The app uses the system's `DayNight` theme, so colours defined with `?attr/colorSurface`, `?attr/colorOnSurface`, and other Material colour attributes adapt correctly in both modes.

---

## Achievements & Badges

| Badge ID | Name | How to Earn |
|---|---|---|
| `FIRST_EXPENSE` | First Expense 🧾 | Log your very first expense |
| `STREAK_7` | 7-Day Streak 🔥 | Log an expense every day for 7 days |
| `STREAK_30` | 30-Day Streak 🔥🔥 | Log an expense every day for 30 days |
| `STREAK_100` | 100-Day Streak 💯 | Log an expense every day for 100 days |
| `BUDGET_HERO` | Budget Hero 🦸 | Stay within your monthly budget |
| `FIRST_SAVE` | Goal Setter 💰 | Set your first budget goal |

### Budget Score Tiers
| Score | Tier |
|---|---|
| 80–100 | 🥇 Platinum |
| 60–79 | 🥈 Gold |
| 40–59 | 🥉 Silver |
| 0–39 | 🏅 Bronze |

The score is calculated monthly based on how well actual spending aligns with the budget goals set by the user.

---

## Data Export

From **Settings → Export Data to CSV**, all expenses are written to a timestamped `.csv` file in the app's external Documents folder:

```
BudgetBuddy_20260423_103045.csv
```

The CSV includes: `ID, Date, Category, Description, Amount, HasPhoto`

After export, the system share sheet opens so the file can be sent via email, WhatsApp, Google Drive, or any other installed app.

---

## Security

- Passwords are hashed with **SHA-256** before being stored — the plain text password is never written to disk
- Sessions are stored in **private SharedPreferences** — not accessible to other apps
- Receipt photos are stored as **BLOBs inside the app's private SQLite database** — not written to the public gallery
- The `FileProvider` is configured to share exported CSV files safely without exposing the full file system

---

## Known Limitations

- **Single user per device session** — the app is designed for one user at a time; switching accounts requires logging out
- **No cloud sync** — all data lives on-device; uninstalling the app permanently deletes all data unless a CSV export was made beforehand
- **Dark mode colour depth** — screens using hardcoded hex colours (e.g. `#FFFFFF`) do not fully adapt in dark mode; only screens using `?attr/` theme attributes adapt automatically
- **Receipt photo size** — photos are compressed to ≤500KB before storage, which may reduce quality for very large images
- **No recurring expenses** — each expense must be logged manually

---

## Future Improvements

- [ ] Cloud backup and sync (Firebase / Google Drive)
- [ ] Recurring/scheduled expenses
- [ ] Currency selection and formatting
- [ ] Full Material You dynamic colour theming
- [ ] Biometric (fingerprint) login
- [ ] Spending notifications and overspend alerts
- [ ] Widget for quick expense entry from the home screen
- [ ] Multi-currency support
- [ ] Shared budgets between family members

---

## Licence

This project is for educational and personal use. No licence has been formally applied — feel free to fork and adapt with attribution.

---

*Built with ❤️ in Kotlin — BudgetBuddy v1.0*
