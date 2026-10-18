# Lifeline – MySQL version

Browsers cannot talk to MySQL directly, so this version adds a small **PHP (PDO) API** between
the pages and the database. The pages no longer use `localStorage` for data; everything is stored in MySQL.

```
lifeline-mysql/
├── index.html, dashboard.html
├── css/style.css
├── js/script.js            ← now calls the API with fetch()
├── api/                    ← PHP endpoints (auth, incidents, roster, announcements, training, analytics, meta)
│   └── config.php          ← EDIT: database login
└── database/
    ├── 01_schema.sql       ← all tables + the app's MySQL user
    ├── 02_seed.sql         ← buildings/rooms, squads, zones, condition types, default roster, announcements, trainings
    ├── 03_import_existing_cpr.sql  ← template to load your EXISTING CPR members/officers
    └── 04_demo_data.sql    ← OPTIONAL: ~240 fake past incidents so the charts have data
```

## Requirements
PHP 8.1+ with `pdo_mysql`, MySQL 8.0.16+ (or MariaDB 10.5+), Apache/Nginx (XAMPP or Laragon is fine).

## Setup
1. Copy the whole folder into your web root (e.g. `htdocs/lifeline`). Do not expose `database/` publicly (delete it from the server after importing).
2. In `01_schema.sql`, change `ChangeThisPassword!`, then import the files **in order** with UTF-8 (needed for the emoji):
   ```
   mysql --default-character-set=utf8mb4 -u root -p < database/01_schema.sql
   mysql --default-character-set=utf8mb4 -u root -p < database/02_seed.sql
   ```
3. Put the same password in `api/config.php`.
4. Load your real CPR people (next section). Optionally import `04_demo_data.sql`.
5. Open `http://localhost/lifeline/index.html`.

## How the existing CPR database fits in
- CPR members/officers are rows in `users` (role `member`/`officer`) + `cpr_members` (squad, zone, duty status), with **no password yet**.
- They do **not** pick a role. On the Register tab they enter their Student ID and **full name exactly as in your records**, choose a password, and the existing account is activated. Their role comes from the database.
- Self-registration can now only create regular students. The old client-side access codes are gone (they were visible in `script.js`).
- Recommended: give each person a one-time activation code (step 5 in `03_import_existing_cpr.sql`, **uppercase codes**). Without it, anyone who knows a member's ID and name could activate that account first.
- Officers can add new members from the *CPR Team Management* page; the system generates the one-time code and shows it once.

## Where each screen's data lives
| Screen | Tables |
|---|---|
| Login / Register / Profile | `users`, `login_logs` |
| Emergency form dropdowns | `buildings`, `rooms`, `incident_types` |
| Emergency alerts, dispatch queue, "My requests" | `incidents`, `incident_logs` |
| CPR roster / squads / active responders | `cpr_members`, `squads`, `zones` |
| Announcements | `announcements` |
| Training cards & attendance | `training_sessions`, `training_registrations` |
| Analytics & dashboard numbers | computed from `incidents` (nothing hard-coded) |

## Behaviour notes
- The dashboard refreshes every 5 seconds (polling), so it now works across different devices, not just browser tabs. Browsers slow down background tabs, so keep the responder tab visible if you rely on the alert sound.
- Accepting an alert is race-safe: if two medics tap *Accept* together, only the first succeeds.
- Roles are re-read from the database on every request, so demoting/deactivating someone takes effect immediately.
- Analytics use a rolling 30 days for type/location charts and the last 6 calendar months for the trend charts.

## Not built yet
- Screens for officers to create training sessions or mark attendance (the API exists: `training.php?action=create_session` / `mark_attendance`).
- Removing a member from the roster, and password reset.
- Use HTTPS in production; the session cookie is marked `Secure` automatically when served over HTTPS.

## Testing status
PHP files pass `php -l`, `script.js` passes `node --check`, and every element ID/handler used by the script exists in the HTML. **I could not run the SQL or the API against a real MySQL server** (none was available in my sandbox), so do one pass of: register a student → sign in → send an alert → accept it as a responder → check Analytics.
"# lifeline3" 
