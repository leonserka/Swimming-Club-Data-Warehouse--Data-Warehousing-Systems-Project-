# Swimming Club Data Warehouse

**Course:** Sustavi za skladištenje podataka (Data Warehousing Systems)  
**Status:** Work in progress (WIP)

A seminar project that develops a relational database and a data warehouse for a swimming club. The analytical focus is swimmers' progress across three seasons and its relationship with coaches, age, initial performance, training attendance, and group level. The operational database also covers membership fees and payments. The current warehouse implementation still contains attendance facts only; performance analysis requires a warehouse extension.

## Project steps and current progress

1. **Create the relational database** — Define the operational tables, relationships, and constraints, including swimming disciplines and timed performance measurements.
2. **Insert test data** — Existing fictional data covers September–November 2025 in the 2025/2026 season. Three-season data and performance measurements remain to be added.
3. **Review the database diagram** — The existing diagram shows the original 12 tables; the two new tables still need to be included.
4. **Design the star schema** — Define an attendance fact table and member, group, coach, and date dimensions in a separate `dw` schema.
5. **Define the time dimension** — Create the date dimension with day, month, quarter, year, and ISO weekday attributes.

SQL scripts for steps 1–2 and the structures for steps 4–5 are provided. A database diagram and a draft presentation are also included. Loading the warehouse dimensions and fact table is still pending; the warehouse script currently creates empty tables only.

## Analytical questions

- Are there differences in swimmers' progress under different coaches across three seasons, taking age, initial performance, and training attendance into account?
- Is more regular training attendance associated with greater improvement in swimming performance across three seasons, and does this association differ by coach and group level?

The Croatian questions are listed in `2pitanja.txt`. Membership fee analysis is no longer one of the two main analytical questions; its source tables remain part of the operational model.

### Measurement definitions

- **Performance:** time in seconds for a specific stroke and distance, recorded with the pool length (25 m or 50 m). Compare results only for the same discipline and pool length.
- **Baseline and progress:** for each swimmer, season, discipline, and pool length, use the first valid measurement as the baseline and the last as the final result. Require at least two distinct measurement dates. Improvement (%) = `(initial_time - final_time) / initial_time * 100`; positive values mean faster swimming. Follow the same swimmers over all three seasons and report seasonal changes as well as the overall trend.
- **Age:** calculate age from `clan.datum_rodenja` on the baseline measurement date, rather than using today's age.
- **Attendance:** attended sessions divided by eligible held sessions in the measurement interval, multiplied by 100. Include only sessions during an active enrollment; exclude cancelled sessions. Both `odsutan` and `opravdano` count as non-attendance. Missing attendance records must be validated before calculating a percentage.
- **Coach and group:** obtain them from the enrollment associated with each measurement. Retain separate enrollments and season-specific groups when a swimmer changes group or coach; do not overwrite historical assignments. Split comparisons into periods with a consistent coach and group level rather than attributing all progress to the latest coach.

Aggregate attendance and performance separately at the same analytical grain before joining them, to avoid multiplying measurements by attendance rows. Compare progress by coach while accounting for baseline age, initial time, attendance, and discipline; examine the attendance relationship within coach and group-level categories. These are associations, not evidence that a particular coach or attendance level caused improvement. The planned analysis uses fictional data and cannot support conclusions about real swimmers.

## Repository contents

| File | Purpose |
| --- | --- |
| `01_plivacki_klub_relacijska_baza.sql` | Creates 14 operational tables, including disciplines and timed results. |
| `02_plivacki_klub_testni_podaci.sql` | Inserts the existing one-season fictional data; new result tables remain empty. |
| `03_plivacki_klub_dijagram_pregled.png` | Original 12-table diagram; update pending. |
| `04_05_plivacki_klub_skladiste.sql` | Creates the warehouse star schema and date dimension structure. |
| `2pitanja.txt` | Two analytical questions about progress, coaches, and attendance across three seasons. |
| `Plivacki_klub.pptx` | Draft seminar presentation. |
| `Zadatak - Seminar.pdf` | Original seminar assignment. |

## Data model

The operational model contains 14 tables: `sezona`, `clan`, `trener`, `bazen`, `grupa`, `upis`, `termin`, `trening`, `dolaznost`, `disciplina`, `rezultat`, `tip_clanarine`, `obracun_clanarine`, and `uplata`.

Members enroll in groups. Each group belongs to a season and is associated with a coach and a pool. Scheduled time slots lead to individual training sessions, with attendance recorded for each enrollment. Membership fee charges are linked to enrollments and fee types, while payments are linked to individual charges.

`disciplina` identifies a stroke (`slobodno`, `ledno`, `prsno`, `leptir`, or `mjesovito`) and a positive distance in metres. `rezultat` links a discipline to an enrollment and stores the measurement date, a positive time in seconds (three decimal places), and the pool length. One official result is allowed per enrollment, discipline, date, and pool length. Separate disciplines or pool lengths must not be compared directly. Enrollment links each result to its swimmer, group, coach, and season. Measurement dates must be validated against the enrollment and season dates during data entry or loading; this cross-table rule is not enforced by the new table constraints.

The current warehouse contains:

- `dw.factdolaznost` — One attendance record for one member enrollment at one held training session.
- `dw.dimdate` — Calendar attributes for analysis over time.
- `dw.dimclan` — Member details.
- `dw.dimgrupa` — Group details, including season and pool attributes.
- `dw.dimtrener` — Coach details.

The fact table stores record, attendance, and absence counts. Both absent (`odsutan`) and excused (`opravdano`) statuses count as absences in the current design.

To answer the revised questions, the warehouse still needs performance facts and discipline attributes, including pool length and measurement dates. The existing attendance schema alone cannot measure swimmers' progress. The final warehouse design must also be checked against the assignment requirements before implementation.

## PostgreSQL in VS Code

1. Install [PostgreSQL for Windows](https://www.postgresql.org/download/windows/) locally and remember the password you set for the `postgres` user. Make sure the PostgreSQL service is running.
2. Open this project folder in VS Code and install Microsoft's [PostgreSQL extension](https://marketplace.visualstudio.com/items?itemName=ms-ossdata.vscode-pgsql) (`ms-ossdata.vscode-pgsql`).
3. Open the PostgreSQL sidebar, add a connection, and enter your local settings:

   | Setting | Typical local value |
   | --- | --- |
   | Host | `localhost` |
   | Port | `5432` |
   | User | `postgres` |
   | Password | Your installation password |
   | Database | `postgres` initially, then `swimming_club` |

4. While connected to `postgres`, create the seminar database once. Run this statement on its own, outside a transaction:

   ```sql
   CREATE DATABASE swimming_club;
   ```

   Skip this step if the database already exists. Use an empty database for the initial setup.
5. Connect to `swimming_club` and check the active connection:

   ```sql
   SELECT current_database(), version();
   ```

6. Open and execute each complete SQL file in the order listed below, using the extension's query execution action. Include the `BEGIN` and `COMMIT` statements in each file. Keep the same `swimming_club` connection for all three scripts.
7. Refresh the database explorer to see the operational tables in `public` and the five warehouse tables in `dw`. The warehouse tables remain empty until a loading process is implemented.

Alternatively, run the commands below in VS Code's integrated terminal. This requires `psql` on your PATH; if it is not recognized, add your PostgreSQL installation's `bin` directory to PATH and restart the terminal.

## Running the SQL scripts

Use PostgreSQL and a new, empty database. Execute these scripts in order, in the same database, using a SQL client such as pgAdmin or `psql`:

```sh
psql -h localhost -p 5432 -U postgres -d swimming_club -v ON_ERROR_STOP=1 -f 01_plivacki_klub_relacijska_baza.sql
psql -h localhost -p 5432 -U postgres -d swimming_club -v ON_ERROR_STOP=1 -f 02_plivacki_klub_testni_podaci.sql
psql -h localhost -p 5432 -U postgres -d swimming_club -v ON_ERROR_STOP=1 -f 04_05_plivacki_klub_skladiste.sql
```

Create `swimming_club` beforehand and adjust the connection settings for your local PostgreSQL installation. Run the scripts once: they are not designed for repeated execution against existing tables or data. The final script lists the five warehouse tables as a structural check.

If the original 12-table database already exists, do not rerun the entire `01` script. Execute only the new `CREATE TABLE disciplina` and `CREATE TABLE rezultat` statements, in that order and in one transaction, provided those tables do not yet exist. This updates the structure without removing existing data. No running database is changed merely by editing the SQL file.

## Remaining work

- Extend test data to three seasons, tracking the same swimmers, historical enrollments, attendance, and comparable initial/final performance measurements.
- Populate `disciplina` and `rezultat` and validate measurement dates against enrollment and season dates.
- Update the relational diagram to include the two new tables.
- Extend the warehouse for performance analysis and implement loading and transformation of operational data.
- Populate and validate the date dimension.
- Validate warehouse data, attendance denominators, baseline results, and improvement measures against the source records.
- Implement the two revised analyses, accounting for age and initial performance and comparing attendance relationships by coach and group level.
- Complete the presentation and final seminar documentation against the assignment requirements.

All test data is fictional. SQL identifiers and the source materials retain their original Croatian names.

