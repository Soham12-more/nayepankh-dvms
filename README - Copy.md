# NayePankh DVMS — Donation & Volunteer Management System

A desktop application that helps the **NayePankh Foundation** run its day-to-day
operations: recording donations, tracking campaigns, managing volunteers,
generating **80G tax receipts**, and producing reports — all from a single,
clean Java Swing interface backed by a real SQLite database.

> Built as a focused, production-style submission for the NayePankh Foundation
> Java Development internship. It is intentionally small in scope but engineered
> the way real software is: a layered architecture, a persistent database,
> parameterised queries, search, analytics, and exportable reports.

---

## Why this project fits NayePankh

NayePankh is an 80G & 12A–certified, UP-Government-registered NGO that runs
donation-funded drives — mid-day meals, menstrual hygiene (Project Garima),
winter clothing, education support and health camps. A foundation like that
lives and dies by three things: **money in, where it goes, and the people who
make it happen.** This application models exactly those three things and adds
the one document every Indian donor asks for — a **Section 80G receipt** with
the amount written out in words.

The sample data that ships with the app mirrors NayePankh's real campaign themes,
so the dashboard tells a believable story the moment you launch it.

---

## Features

**Dashboard** — at-a-glance KPIs (total raised, donations, donors, volunteers,
active drives), a funds-by-campaign bar chart drawn with Java2D, per-campaign
progress bars (raised vs goal), and a top-donors leaderboard.

**Donations** — record donations against a donor and campaign; multi-field
search (keyword, campaign, amount range, date range); one-click **80G receipt**
generation with the amount spelled out in Indian-format words; CSV export of the
current filtered view.

**Campaigns** — full create / edit / delete with category, goal, start date and
lifecycle status (Active / Completed / Paused).

**Volunteers** — searchable roster (by keyword, skill and status), with
assignment to a campaign and full create / edit / delete.

**Reports** — formatted donation-summary, campaign-progress and volunteer-roster
reports, each saveable to a text file in the `reports/` folder.

**Indian-context details that show domain research**
- Currency formatted as `₹2,73,000` using the `en-IN` locale (lakh/crore grouping).
- Amounts converted to words: `₹50,000 → "Fifty Thousand Rupees Only"`.
- Receipt numbers auto-generated as `NPF/<year>/<6-digit-id>`.
- PAN captured per donor (required on 80G receipts).

---

## Screenshots

| | |
|---|---|
| **Dashboard** | ![Dashboard]<img width="717" height="464" alt="image" src="https://github.com/user-attachments/assets/069dc316-9222-4113-9df8-a38d9be4e45c" />
 |
| **Donations + search** | ![Donations]<img width="422" height="274" alt="image" src="https://github.com/user-attachments/assets/6bf4877f-0dc6-4159-8557-1f5ad7f0a98a" />
 |
| **Campaigns** | ![Campaigns]<img width="717" height="461" alt="image" src="https://github.com/user-attachments/assets/030cfe9c-bd70-4cb1-9333-0b0d9790fb09" />
 |
| **Volunteers** | ![Volunteers]<img width="719" height="466" alt="image" src="https://github.com/user-attachments/assets/622ecb1b-0157-428d-9b08-2e8e26b2f38a" />
 |
| **Reports** | ![Reports]<img width="720" height="460" alt="image" src="https://github.com/user-attachments/assets/1dbb6fd6-1d2b-4589-b96f-d8a8b04943dd" />
 |

---

## Architecture

The code is organised into clear layers, each depending only on the one beneath
it. The UI never touches the database directly — it goes through a single
service facade. This is the same separation used in real backend systems and
makes the code easy to test and extend.

```
        ┌─────────────────────────────────────────────┐
        │                   UI  (Swing)                │
        │  MainFrame · Dashboard · Donations · ...      │
        └───────────────────────┬─────────────────────┘
                                 │  calls only
        ┌───────────────────────▼─────────────────────┐
        │              Service  (facade + logic)        │
        │  FoundationService · ReportService            │
        │  CurrencyUtil · CampaignProgress              │
        └───────────────────────┬─────────────────────┘
                                 │
        ┌───────────────────────▼─────────────────────┐
        │           DAO  (data access objects)          │
        │  DonationDAO · DonorDAO · CampaignDAO · ...    │
        │  PreparedStatements · dynamic search criteria  │
        └───────────────────────┬─────────────────────┘
                                 │  JDBC
        ┌───────────────────────▼─────────────────────┐
        │          DB  ·  SQLite (data/nayepankh.db)    │
        │  DatabaseManager: schema + first-run seeding   │
        └───────────────────────────────────────────────┘

        model/  ←  plain data objects shared by every layer
```

**Design choices worth calling out**
- **Layered, not monolithic.** `model → db → dao → service → ui`. The UI depends
  on `FoundationService` alone, so the storage layer could be swapped for MySQL
  by rewriting only the DAOs.
- **Safe SQL.** Every query uses `PreparedStatement` with bound parameters — no
  string concatenation, no SQL injection. Donation search builds its `WHERE`
  clause dynamically from whichever filters are set.
- **Referential integrity.** Foreign keys are enforced (`PRAGMA foreign_keys=ON`),
  with `ON DELETE CASCADE` for a campaign's donations and `ON DELETE SET NULL`
  for a deleted campaign's volunteers.
- **Correct domain modelling.** A business rejection (e.g. insufficient stock)
  is never confused with a system failure — a dedicated exception keeps those
  paths separate.

---

## Tech stack

- **Java 17+** (developed and tested on OpenJDK 21), Swing for the UI, Java2D for the chart.
- **SQLite** via the `sqlite-jdbc` driver (bundled in `lib/`).
- **No build tool required** — plain `javac` / `java`. SLF4J jars are included
  only to keep the console clean.

---

## How to run

You only need a JDK (Java 17 or newer) installed. No internet, Maven or Gradle.

**Linux / macOS**
```bash
./run.sh
```

**Windows**
```bat
run.bat
```

The script compiles everything into `out/` and launches the app. On first run the
database is created at `data/nayepankh.db` and seeded with sample campaigns,
donors, donations and volunteers, so there is data to explore immediately.

To compile and run manually:
```bash
# from the project root
mkdir -p out
find src -name "*.java" > sources.txt
javac -d out -cp "lib/*" @sources.txt
java -cp "out:lib/*" com.nayepankh.dvms.Main      # use ; instead of : on Windows
```

---

## Project structure

```
nayepankh-dvms/
├── run.sh / run.bat            # build + launch
├── lib/                        # bundled JDBC + logging jars
├── data/                       # SQLite db (created on first run)
├── reports/                    # saved reports / receipts / CSV exports
├── screenshots/                # UI screenshots used in this README
├── test/
│   └── SmokeTest.java          # headless backend verification
└── src/com/nayepankh/dvms/
    ├── Main.java               # entry point
    ├── model/                  # Donor, Campaign, Donation, Volunteer
    ├── db/                     # DatabaseManager (schema + seed)
    ├── dao/                    # one DAO per entity + search criteria
    ├── service/                # FoundationService, ReportService, CurrencyUtil
    └── ui/                     # MainFrame + 5 panels + Theme + chart
```

---

## Testing

`test/SmokeTest.java` runs the whole backend headlessly — seeding, search,
inserting a donation with an auto-generated receipt number, and rendering an 80G
receipt — and prints the results. It is how the data layer was verified
independently of the UI:

```bash
javac -d out -cp "lib/*" $(find src -name "*.java") test/SmokeTest.java
java -cp "out:lib/*" SmokeTest
```

---

## Possible next steps

Authentication for staff, editable donor records from the UI, PDF receipts via a
library, and a MySQL/Postgres backend for multi-user deployment — each is a small
change thanks to the layered design.

---

*Submitted for the NayePankh Foundation Java Development internship.*
