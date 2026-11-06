# Swimming Club Data Warehouse

**Course:** Sustavi za skladištenje podataka (Data Warehousing Systems)  
**Status:** Work in progress (WIP)

A seminar project that develops a relational database and a data warehouse for a swimming club. The operational database covers members, coaches, groups, pools, training sessions, attendance, membership fees, and payments. The current warehouse design focuses on training attendance.

## Project steps and current progress

1. **Create the relational database** — Define the operational tables, relationships, primary and foreign keys, and integrity constraints.
2. **Insert test data** — Populate the database with fictional swimming club data for the 2025/2026 season.
3. **Review the database diagram** — Document the relational model and the relationships between tables.
4. **Design the star schema** — Define an attendance fact table and member, group, coach, and date dimensions in a separate `dw` schema.
5. **Define the time dimension** — Create the date dimension with day, month, quarter, year, and ISO weekday attributes.

SQL scripts for steps 1–2 and the structures for steps 4–5 are provided. A database diagram and a draft presentation are also included. Loading the warehouse dimensions and fact table is still pending; the warehouse script currently creates empty tables only.

## Analytical questions

- What is the training attendance percentage by month, group, and coach?
- Which groups have the most absences, and how does this change throughout the season?
- How much has been billed, paid, and left unpaid in membership fees by month and group?

The first two questions are listed in `2pitanja.txt`. The presentation also includes the membership fee question. Financial data exists in the operational database, but the current warehouse schema contains only attendance facts.

## Repository contents

| File | Purpose |
| --- | --- |
| `01_plivacki_klub_relacijska_baza.sql` | Creates the relational database tables and constraints. |
| `02_plivacki_klub_testni_podaci.sql` | Inserts fictional test data. |
| `03_plivacki_klub_dijagram_pregled.png` | Overview of the relational database diagram. |
| `04_05_plivacki_klub_skladiste.sql` | Creates the warehouse star schema and date dimension structure. |
| `2pitanja.txt` | Two attendance-related analytical questions. |
| `Plivacki_klub.pptx` | Draft seminar presentation. |
| `Zadatak - Seminar.pdf` | Original seminar assignment. |

## Data model

The operational model contains 12 tables: `sezona`, `clan`, `trener`, `bazen`, `grupa`, `upis`, `termin`, `trening`, `dolaznost`, `tip_clanarine`, `obracun_clanarine`, and `uplata`.

Members enroll in groups. Each group belongs to a season and is associated with a coach and a pool. Scheduled time slots lead to individual training sessions, with attendance recorded for each enrollment. Membership fee charges are linked to enrollments and fee types, while payments are linked to individual charges.

The warehouse contains:

- `dw.factdolaznost` — One attendance record for one member enrollment at one held training session.
- `dw.dimdate` — Calendar attributes for analysis over time.
- `dw.dimclan` — Member details.
- `dw.dimgrupa` — Group details, including season and pool attributes.
- `dw.dimtrener` — Coach details.

The fact table stores record, attendance, and absence counts. Both absent (`odsutan`) and excused (`opravdano`) statuses count as absences in the current design.

## Running the SQL scripts

Use PostgreSQL and a new, empty database. Execute these scripts in order, in the same database, using a SQL client such as pgAdmin or `psql`:

```sh
psql -d swimming_club -v ON_ERROR_STOP=1 -f 01_plivacki_klub_relacijska_baza.sql
psql -d swimming_club -v ON_ERROR_STOP=1 -f 02_plivacki_klub_testni_podaci.sql
psql -d swimming_club -v ON_ERROR_STOP=1 -f 04_05_plivacki_klub_skladiste.sql
```

Create `swimming_club` beforehand and adjust the connection settings for your local PostgreSQL installation. Run the scripts once: they are not designed for repeated execution against existing tables or data. The final script lists the five warehouse tables as a structural check.

## Remaining work

- Implement loading and transformation of operational data into the warehouse dimensions and fact table.
- Populate and validate the date dimension.
- Validate warehouse data and attendance measures against the source records.
- Implement analytical SQL queries and present their results.
- Resolve the scope of membership fee analysis and extend the warehouse if required.
- Complete the presentation and final seminar documentation against the assignment requirements.

All test data is fictional. SQL identifiers and the source materials retain their original Croatian names.
