# SharePoint Lists — Setup Reference
# Assessment Date Manager Application
# ════════════════════════════════════════════════════════════════════
# USE THIS DOCUMENT TO BUILD ALL SHAREPOINT LISTS
#
# For each list:
# 1. Go to your SharePoint site
# 2. Click Settings (⚙️) → Site Contents → New → List
# 3. Choose "Blank list", give it the exact name shown below
# 4. Add each column exactly as specified
#
# IMPORTANT NOTES:
# • The "Title" column already exists on every new SharePoint list
#   — do NOT create it again, just rename it if instructed
# • Column names are case-sensitive — use exactly as written
# • "Auto" columns are created by SharePoint automatically
# • Create all 7 lists before running any seed scripts
# ════════════════════════════════════════════════════════════════════

---

# ══════════════════════════════════════════════════════════════════
# LIST 1: Assessments
# ══════════════════════════════════════════════════════════════════
# Purpose: Stores every assessment booking made by teachers
# Who creates rows: Teachers, Admins, Management
# Estimated rows per year: 500–2000 depending on school size

SharePoint List Name (exact): Assessments

| # | Column Name      | SharePoint Column Type          | Required | Default Value | Notes                                      |
|---|------------------|---------------------------------|----------|---------------|--------------------------------------------|
| 1 | Title            | Single line of text             | Yes      | —             | Rename existing Title column. Assessment name/description |
| 2 | Subject          | Single line of text             | Yes      | —             | Populated from ReferenceData (Category: Subject) |
| 3 | ClassGroup       | Single line of text             | Yes      | —             | Populated from ReferenceData (Category: ClassGroup) |
| 4 | AssessmentType   | Single line of text             | Yes      | —             | Populated from ReferenceData (Category: AssessmentType) |
| 5 | AssessmentDate   | Date and Time → Date Only       | Yes      | —             | The scheduled date of the assessment       |
| 6 | Duration         | Number                          | No       | —             | Duration in minutes — optional             |
| 7 | TeacherName      | Single line of text             | Yes      | —             | Auto-populated from Microsoft 365 login    |
| 8 | TeacherEmail     | Single line of text             | Yes      | —             | Auto-populated from Microsoft 365 login    |
| 9 | Term             | Single line of text             | Yes      | —             | Populated from ReferenceData (Category: Term) |
| 10| StudyRequired    | Choice                          | Yes      | Requires Study| Whether students must study for this. Auto-forced to "Requires Study" and locked for formal assessments |
| 11| Notes            | Multiple lines of text (Plain)  | No       | —             | Optional teacher notes. Maximum 300 characters enforced by the app UI. Included in the student distribution report when present. |
| 12| Status           | Single line of text             | Yes      | Scheduled     | Values: Scheduled, Cancelled, Completed    |
| 13| Created          | Date and Time                   | Auto     | —             | SharePoint creates automatically           |
| 14| Modified         | Date and Time                   | Auto     | —             | SharePoint creates automatically           |
| 15| Author           | Person or Group                 | Auto     | —             | SharePoint creates automatically (Created By) |
| 16| Editor           | Person or Group                 | Auto     | —             | SharePoint creates automatically (Modified By) |

For the StudyRequired Choice column, enter these options exactly:
  • Requires Study
  • No Study Required

Column creation order: Create columns 2–12 in order. Columns 13–16 exist automatically.

---

# ══════════════════════════════════════════════════════════════════
# LIST 2: CycleTests
# ══════════════════════════════════════════════════════════════════
# Purpose: Defines exam/cycle test weeks that block bookings
# Who creates rows: Admins and Management via CycleTestManager screen
# Estimated rows per term: 5–20 (one row per cycle test per grade)
# IMPORTANT: Each row is ONE test on ONE specific date — NOT a date range.
# Teachers can book continuous assessments on other days in the same week.

SharePoint List Name (exact): CycleTests

| # | Column Name      | SharePoint Column Type          | Required | Default Value | Notes                                      |
|---|------------------|---------------------------------|----------|---------------|--------------------------------------------|
| 1 | Title            | Single line of text             | Yes      | —             | Auto-set by app e.g. "Grade 10 Mathematics Cycle Test — 5 Mar 2026" |
| 2 | CycleTestDate    | Date and Time → Date Only       | Yes      | —             | The specific date of the cycle test — one day only, not a range |
| 3 | Grade            | Single line of text             | Yes      | —             | Which grade e.g. "Grade 10" |
| 4 | Subject          | Single line of text             | No       | —             | Which subject e.g. "Mathematics". Leave blank to block all subjects for that grade on that day |
| 5 | Notes            | Multiple lines of text (Plain)  | No       | —             | Admin notes e.g. "Set by head office"      |
| 6 | Created          | Date and Time                   | Auto     | —             | SharePoint creates automatically           |
| 7 | Author           | Person or Group                 | Auto     | —             | SharePoint creates automatically (Created By) |

Column creation order: Create columns 2–5. Columns 6–7 exist automatically.

---

# ══════════════════════════════════════════════════════════════════
# LIST 3: PublicHolidays
# ══════════════════════════════════════════════════════════════════
# Purpose: Stores South African public holidays — blocks booking on these dates
# Who creates rows: Seed script (first run), Management adds future years via app
# Estimated rows: 13 per year × number of years loaded

SharePoint List Name (exact): PublicHolidays

| # | Column Name      | SharePoint Column Type          | Required | Default Value | Notes                                      |
|---|------------------|---------------------------------|----------|---------------|--------------------------------------------|
| 1 | Title            | Single line of text             | Yes      | —             | Rename existing Title column. Holiday name e.g. "Youth Day" |
| 2 | HolidayDate      | Date and Time → Date Only       | Yes      | —             | The actual blocked date (including substitute Mondays) |
| 3 | Year             | Number (0 decimal places)       | Yes      | —             | The calendar year e.g. 2025, 2026          |
| 4 | IsSubstitute     | Yes/No                          | Yes      | No            | Yes if this is a Monday substituting for a Sunday holiday |
| 5 | OriginalDate     | Date and Time → Date Only       | No       | —             | The Sunday date this substitutes — only filled when IsSubstitute = Yes |
| 6 | Source           | Choice                          | Yes      | Pre-loaded    | Choices: Pre-loaded, Management Added      |
| 7 | Created          | Date and Time                   | Auto     | —             | SharePoint creates automatically           |
| 8 | Author           | Person or Group                 | Auto     | —             | SharePoint creates automatically           |

For the Source Choice column, enter these options exactly:
  • Pre-loaded
  • Management Added

Column creation order: Create columns 2–6. Columns 7–8 exist automatically.

---

# ══════════════════════════════════════════════════════════════════
# LIST 4: SchoolCalendar
# ══════════════════════════════════════════════════════════════════
# Purpose: Defines which date ranges (terms) are open for assessment booking
# Who creates rows: Management only via School Calendar Manager screen
# Estimated rows per year: 4–8 (one per term, possibly split windows)

SharePoint List Name (exact): SchoolCalendar

| # | Column Name        | SharePoint Column Type          | Required | Default Value | Notes                                    |
|---|--------------------|---------------------------------|----------|---------------|------------------------------------------|
| 1 | Title              | Single line of text             | Yes      | —             | Rename existing Title column. e.g. "Term 1 2026" |
| 2 | StartDate          | Date and Time → Date Only       | Yes      | —             | First day of the assessment window       |
| 3 | EndDate            | Date and Time → Date Only       | Yes      | —             | Last day of the assessment window        |
| 4 | Year               | Number (0 decimal places)       | Yes      | —             | The academic year e.g. 2026              |
| 5 | Term               | Single line of text             | Yes      | —             | Populated from ReferenceData (Category: Term) |
| 6 | IncludeSaturdays   | Yes/No                          | Yes      | Yes           | Whether Saturdays in this window are bookable |
| 7 | Notes              | Multiple lines of text (Plain)  | No       | —             | Admin notes e.g. "Excludes sports day week" |
| 8 | Created            | Date and Time                   | Auto     | —             | SharePoint creates automatically         |
| 9 | Author             | Person or Group                 | Auto     | —             | SharePoint creates automatically         |

Column creation order: Create columns 2–7. Columns 8–9 exist automatically.

---

# ══════════════════════════════════════════════════════════════════
# LIST 5: AppSettings
# ══════════════════════════════════════════════════════════════════
# Purpose: Stores configurable rule values for Rule 1 and Rule 2
# Who creates rows: Seed script (first run only — do not add rows manually)
# Who edits rows: Management only via Policy Manager screen
# Total rows: Exactly 5 (one per setting — never more, never less)

SharePoint List Name (exact): AppSettings

| # | Column Name      | SharePoint Column Type          | Required | Default Value | Notes                                      |
|---|------------------|---------------------------------|----------|---------------|--------------------------------------------|
| 1 | Title            | Single line of text             | Yes      | —             | Rename existing Title column. Machine-readable key e.g. "MAX_ASSESSMENTS_PER_DAY" |
| 2 | DisplayName      | Single line of text             | Yes      | —             | Human-readable name shown in Policy Manager |
| 3 | Value            | Single line of text             | Yes      | —             | Current value — always stored as text, app casts to correct type |
| 4 | DefaultValue     | Single line of text             | Yes      | —             | Original default — used by Reset to Default button |
| 5 | DataType         | Choice                          | Yes      | —             | How the Value field should be interpreted  |
| 6 | Description      | Multiple lines of text (Plain)  | Yes      | —             | Explains the setting and impact of changing it — shown in Policy Manager |
| 7 | Category         | Single line of text             | Yes      | —             | Groups settings in Policy Manager: DailyLimit, CycleTests, General |
| 8 | ChangedBy        | Single line of text             | No       | —             | Auto-populated by app when value is updated (not by SharePoint) |
| 9 | ChangedDate      | Date and Time                   | No       | —             | Auto-populated by app when value is updated (not by SharePoint) |
| 10| Created          | Date and Time                   | Auto     | —             | SharePoint creates automatically           |
| 11| Author           | Person or Group                 | Auto     | —             | SharePoint creates automatically           |

For the DataType Choice column, enter these options exactly:
  • Number
  • Boolean
  • Text

The 5 rows this list will contain (added by seedAppSettings.js — do not create manually):
  Row 1 Title: MAX_ASSESSMENTS_PER_DAY
  Row 2 Title: DAILY_LIMIT_SCOPE
  Row 3 Title: CYCLE_TEST_SCOPE
  Row 4 Title: ALLOW_ADMIN_OVERRIDE
  Row 5 Title: WARN_BEFORE_CYCLE_TEST

Column creation order: Create columns 2–9. Columns 10–11 exist automatically.

---

# ══════════════════════════════════════════════════════════════════
# LIST 6: ReferenceData
# ══════════════════════════════════════════════════════════════════
# Purpose: Master lookup table — every dropdown value in the app comes from here
# Who creates rows: Seed script (first run), Management via Reference Data Manager
# Estimated rows: 30–60 initially, grows as school adds subjects/classes

SharePoint List Name (exact): ReferenceData

| # | Column Name      | SharePoint Column Type          | Required | Default Value | Notes                                      |
|---|------------------|---------------------------------|----------|---------------|--------------------------------------------|
| 1 | Title            | Single line of text             | Yes      | —             | Rename existing Title column. The display value e.g. "Mathematics", "Grade 10A" |
| 2 | Category         | Single line of text             | Yes      | —             | Groups values — must be exactly one of: Subject, ClassGroup, AssessmentType, Term, Status |
| 3 | SortOrder        | Number (0 decimal places)       | Yes      | —             | Controls order in dropdowns — lower number appears first |
| 4 | Active           | Yes/No                          | Yes      | Yes           | Set to No to hide from dropdowns without deleting |
| 5 | Description      | Single line of text             | No       | —             | Optional note e.g. "Covers Physics and Chemistry" |
| 6 | Created          | Date and Time                   | Auto     | —             | SharePoint creates automatically           |
| 7 | Modified         | Date and Time                   | Auto     | —             | SharePoint creates automatically           |
| 8 | Author           | Person or Group                 | Auto     | —             | SharePoint creates automatically           |

Column creation order: Create columns 2–5. Columns 6–8 exist automatically.

Valid Category values (the app filters by these exactly — spelling and capitalisation matter):
  • Subject
  • ClassGroup
  • AssessmentType
  • Term
  • Status

Rows are added by seedReferenceData.js on first run.

---

# ══════════════════════════════════════════════════════════════════
# LIST 7: ReferenceDataAuditLog
# ══════════════════════════════════════════════════════════════════
# Purpose: Automatic audit trail — records every change to ReferenceData and AppSettings
# Who creates rows: The application automatically — never add rows manually
# Who reads rows: Management via Audit Log tabs in Reference Data Manager and Policy Manager
# Estimated rows: Grows continuously — never delete rows from this list

SharePoint List Name (exact): ReferenceDataAuditLog

| # | Column Name      | SharePoint Column Type          | Required | Notes                                                        |
|---|------------------|---------------------------------|----------|--------------------------------------------------------------|
| 1 | Title            | Single line of text             | Yes      | Rename existing Title column. Auto-set by app e.g. "Subject: Mathematics changed" |
| 2 | ListChanged      | Single line of text             | Yes      | Which list was changed: ReferenceData or AppSettings         |
| 3 | Category         | Single line of text             | Yes      | The Category or Setting key that was changed                 |
| 4 | OldValue         | Single line of text             | No       | The value before the change (blank for new additions)        |
| 5 | NewValue         | Single line of text             | Yes      | The value after the change                                   |
| 6 | Action           | Choice                          | Yes      | What type of change was made                                 |
| 7 | ChangedBy        | Single line of text             | Yes      | Display name of the user who made the change                 |
| 8 | ChangedByEmail   | Single line of text             | Yes      | Email address of the user who made the change                |
| 9 | ChangedDate      | Date and Time                   | Yes      | Timestamp of the change — set by the app, not SharePoint     |
| 10| ItemId           | Number (0 decimal places)       | No       | SharePoint ID of the row that was changed (for traceability) |
| 11| Created          | Date and Time                   | Auto     | SharePoint creates automatically                             |
| 12| Author           | Person or Group                 | Auto     | SharePoint creates automatically                             |

For the Action Choice column, enter these options exactly:
  • Added
  • Edited
  • Deactivated
  • Reactivated
  • Deleted
  • Setting Changed

Column creation order: Create columns 2–10. Columns 11–12 exist automatically.

---

# ══════════════════════════════════════════════════════════════════
# LIST 8: ArchiveLog
# ══════════════════════════════════════════════════════════════════
# Purpose: Permanent record of every end-of-term archive operation
# Who creates rows: The application automatically after each archive
# NEVER delete rows from this list — it is the permanent audit trail
# Estimated rows: 4 per year (one per term archive)

SharePoint List Name (exact): ArchiveLog

| # | Column Name        | SharePoint Column Type          | Required | Default Value | Notes                                           |
|---|--------------------|---------------------------------|----------|---------------|-------------------------------------------------|
| 1 | Title              | Single line of text             | Yes      | —             | Rename existing Title column. e.g. "Term 2 2026 Archive" |
| 2 | TermName           | Single line of text             | Yes      | —             | e.g. "Term 2"                                   |
| 3 | Year               | Number (0 decimal places)       | Yes      | —             | Academic year e.g. 2026                         |
| 4 | AssessmentsFile    | Single line of text             | Yes      | —             | Full SharePoint path to Assessments Excel file  |
| 5 | CycleTestsFile    | Single line of text             | Yes      | —             | Full SharePoint path to CycleTests Excel file  |
| 6 | AssessmentsCount   | Number (0 decimal places)       | Yes      | —             | Number of rows exported from Assessments        |
| 7 | CycleTestsCount   | Number (0 decimal places)       | Yes      | —             | Number of rows exported from CycleTests |
| 8 | ArchivedBy         | Single line of text             | Yes      | —             | Display name of management user who ran archive |
| 9 | ArchivedByEmail    | Single line of text             | Yes      | —             | Email of management user who ran archive        |
| 10| ArchivedDate       | Date and Time (include time)    | Yes      | —             | Exact timestamp of when the archive completed   |
| 11| ClearSucceeded     | Yes/No                          | Yes      | Yes           | Whether both lists were successfully cleared    |
| 12| Notes              | Multiple lines of text (Plain)  | No       | —             | Optional management notes                       |
| 13| Created            | Date and Time                   | Auto     | —             | SharePoint creates automatically                |

Column creation order: Create columns 2–12. Column 13 exists automatically.

---

# ══════════════════════════════════════════════════════════════════
# LIST 9: TeacherProfiles
# ══════════════════════════════════════════════════════════════════
# Purpose: Stores each teacher's grades, classes, and subjects taught
# Who creates rows: Each teacher via the My Profile screen on first login
# Who edits rows: The teacher themselves via My Profile
# Estimated rows: One per teacher — same as total teaching staff count
# This list starts EMPTY — no seed script needed
#
# IMPORTANT: GradesTaught, ClassesTaught, and SubjectsTaught store
# JSON strings (e.g. '["Grade 10","Grade 11"]') in text columns.
# Do not try to parse or edit these manually in SharePoint.

SharePoint List Name (exact): TeacherProfiles

| # | Column Name      | SharePoint Column Type          | Required | Default Value | Notes                                              |
|---|------------------|---------------------------------|----------|---------------|----------------------------------------------------|
| 1 | Title            | Single line of text             | Yes      | —             | Rename existing Title column. Teacher's display name (auto from Microsoft 365) |
| 2 | TeacherEmail     | Single line of text             | Yes      | —             | Primary key — teacher's school email address. Used to look up profile on every login |
| 3 | GradesTaught     | Multiple lines of text (Plain)  | Yes      | —             | JSON string e.g. '["Grade 10","Grade 11"]'         |
| 4 | ClassesTaught    | Multiple lines of text (Plain)  | Yes      | —             | JSON string e.g. '["Grade 10A","Grade 10B","Grade 11A"]' |
| 5 | SubjectsTaught   | Multiple lines of text (Plain)  | Yes      | —             | JSON string e.g. '["Mathematics","English"]'       |
| 6 | DefaultGrade     | Single line of text             | Yes      | —             | Which calendar opens on login e.g. "Grade 10"      |
| 7 | ProfileComplete  | Yes/No                          | Yes      | No            | Set to Yes after teacher completes first profile save. App checks this to trigger redirect. |
| 8 | Created          | Date and Time                   | Auto     | —             | SharePoint creates automatically                   |
| 9 | Modified         | Date and Time                   | Auto     | —             | SharePoint creates automatically                   |

Column creation order: Create columns 2–7. Columns 8–9 exist automatically.

---

---

# ══════════════════════════════════════════════════════════════════
# LIST 10: ConfirmationLog
# ══════════════════════════════════════════════════════════════════
# Purpose: Records whether each teacher has confirmed their assessment schedule
#          for each grade and term. One row per teacher per grade per term.
# Who creates rows: The app creates rows automatically when a teacher views
#                   the Confirm Schedule screen for a grade/term combination
# Who edits rows: The app updates rows when teachers confirm or un-confirm
# Estimated rows: Number of teachers × number of grades they teach × terms per year
# Never delete rows — they are a permanent audit trail

SharePoint List Name (exact): ConfirmationLog

| # | Column Name        | SharePoint Column Type          | Required | Default Value | Notes                                                      |
|---|--------------------|---------------------------------|----------|---------------|------------------------------------------------------------|
| 1 | Title              | Single line of text             | Yes      | —             | Auto-set by app e.g. "Mr C. Uren — Grade 10 — Term 1 2026" |
| 2 | TeacherEmail       | Single line of text             | Yes      | —             | Teacher's school email. Links to TeacherProfiles list.     |
| 3 | TeacherName        | Single line of text             | Yes      | —             | Display name at time of confirmation                       |
| 4 | Grade              | Single line of text             | Yes      | —             | e.g. "Grade 10"                                            |
| 5 | Term               | Single line of text             | Yes      | —             | e.g. "Term 1 2026" — matches SchoolCalendar Title field    |
| 6 | IsConfirmed        | Yes/No                          | Yes      | No            | True when teacher has confirmed. False if un-confirmed.    |
| 7 | ConfirmedAt        | Date and Time                   | No       | —             | Timestamp of confirmation. Blank if not yet confirmed.     |
| 8 | AssessmentCount    | Number                          | Yes      | 0             | Number of Scheduled assessments locked at confirmation     |
| 9 | IsStale            | Yes/No                          | Yes      | No            | Set to True if assessments changed after confirmation      |
| 10| UnconfirmedAt      | Date and Time                   | No       | —             | Timestamp if teacher later un-confirmed                    |
| 11| UnconfirmedReason  | Single line of text             | No       | —             | Optional reason entered when un-confirming                 |
| 12| Created            | Date and Time                   | Auto     | —             | SharePoint creates automatically                           |
| 13| Modified           | Date and Time                   | Auto     | —             | SharePoint creates automatically                           |

Column creation order: Create columns 2–11. Columns 12–13 exist automatically.

This list starts empty. The app creates and updates rows — no manual entry needed.

---

# ══════════════════════════════════════════════════════════════════
# LIST 11: CycleTestConfirmations
# ══════════════════════════════════════════════════════════════════
# Purpose: Records whether Management has confirmed the cycle test
#          schedule for each grade each term. Teachers CANNOT book
#          assessments for a grade until its cycle tests are confirmed.
# Who creates rows: The app creates/updates rows when Management
#                   confirms or un-confirms via CycleTestManager
# One row per grade per term

SharePoint List Name (exact): CycleTestConfirmations

| # | Column Name      | SharePoint Column Type          | Required | Default | Notes                                                        |
|---|------------------|---------------------------------|----------|---------|--------------------------------------------------------------|
| 1 | Title            | Single line of text             | Yes      | —       | Auto-set e.g. "Grade 10 — Term 1 2026 — Cycle Test Schedule" |
| 2 | Grade            | Single line of text             | Yes      | —       | e.g. "Grade 10"                                              |
| 3 | Term             | Single line of text             | Yes      | —       | e.g. "Term 1 2026" — matches SchoolCalendar Title            |
| 4 | IsConfirmed      | Yes/No                          | Yes      | No      | True when Management has confirmed cycle tests for this grade |
| 5 | ConfirmedAt      | Date and Time                   | No       | —       | Timestamp of confirmation                                    |
| 6 | ConfirmedBy      | Single line of text             | No       | —       | Display name of Management user who confirmed                |
| 7 | CycleTestCount   | Number                          | Yes      | 0       | Number of cycle tests for this grade at time of confirmation |
| 8 | IsStale          | Yes/No                          | Yes      | No      | True if cycle tests changed after confirmation               |
| 9 | UnconfirmedAt    | Date and Time                   | No       | —       | Timestamp if Management later un-confirmed                   |
| 10| UnconfirmedBy    | Single line of text             | No       | —       | Display name of user who un-confirmed                        |
| 11| Created          | Date and Time                   | Auto     | —       | SharePoint creates automatically                             |
| 12| Modified         | Date and Time                   | Auto     | —       | SharePoint creates automatically                             |

Column creation order: Create columns 2–10. Columns 11–12 exist automatically.
This list starts empty. The app creates and updates rows automatically.

---

# ══════════════════════════════════════════════════════════════════
# DOCUMENT LIBRARY: AssessmentArchives
# ══════════════════════════════════════════════════════════════════
# Purpose: Stores the Excel archive files generated at end of each term
# This is a Document Library (not a List) — it stores actual files

How to create:
1. Go to your SharePoint site
2. Click Settings (⚙️) → Site Contents → New → Document library
3. Name it exactly: AssessmentArchives
4. Click Create

No columns need to be added — SharePoint manages files automatically.

The app will create sub-folders automatically:
  AssessmentArchives/
    2026/
      Assessments_Term1_2026.xlsx
      CycleTests_Term1_2026.xlsx
      Assessments_Term2_2026.xlsx
      CycleTests_Term2_2026.xlsx
    2027/
      ...

Permission: Give the "Assessment Management" SharePoint Group
Contribute access to this library so they can view and download archives.

---

| # | List Name                | Columns to Create Manually | Auto Columns | Seeded by Script | Managed via Screen                  |
|---|--------------------------|---------------------------|--------------|------------------|-------------------------------------|
| 1 | Assessments              | 11                        | 4            | No               | Teachers / Admins (calendar)        |
| 2 | CycleTests    | 4                         | 2            | No               | Cycle Test Manager                 |
| 3 | PublicHolidays           | 5                         | 2            | Yes              | School Calendar Manager             |
| 4 | SchoolCalendar           | 6                         | 2            | No               | School Calendar Manager             |
| 5 | AppSettings              | 8                         | 2            | Yes              | Policy Manager                      |
| 6 | ReferenceData            | 4                         | 3            | Yes              | Reference Data Manager              |
| 7 | ReferenceDataAuditLog    | 9                         | 2            | No               | Read-only (Audit Log tabs)          |
| 8 | ArchiveLog               | 11                        | 1            | No               | Read-only (Term Archive Manager)    |
| 9 | TeacherProfiles          | 6                         | 2            | No               | Teachers (My Profile screen)        |
| 10| ConfirmationLog          | 10                        | 2            | No               | App (auto-written on confirmation)  |
| 11| CycleTestConfirmations   | 9                         | 2            | No               | App (auto-written when CT confirmed)|

**Plus 1 Document Library:**

| # | Library Name          | Purpose                                      | Who accesses               |
|---|-----------------------|----------------------------------------------|----------------------------|
| 1 | AssessmentArchives    | Stores Excel archive files — 2 per term      | Management Team (read/write) |

Total columns to create manually across all 11 lists: 83
Total lists: 11
Total document libraries: 1

---

# ══════════════════════════════════════════════════════════════════
# HOW TO ADD COLUMNS IN SHAREPOINT
# ══════════════════════════════════════════════════════════════════
# Step-by-step for each column type used in this document

## Adding a Single line of text column
1. In your list, click + Add column (or the + at the top of the column headers)
2. Select "Single line of text"
3. Enter the Column Name exactly as shown
4. Set Required: Yes or No as specified
5. Click Save

## Adding a Multiple lines of text column
1. Click + Add column
2. Select "Multiple lines of text"
3. Enter the Column Name
4. Under "Specify the type of text to allow" select "Plain text"
5. Set Required as specified
6. Click Save

## Adding a Number column
1. Click + Add column
2. Select "Number"
3. Enter the Column Name
4. Under "Number of decimal places" select "0" unless specified otherwise
5. Set Required as specified
6. Click Save

## Adding a Date and Time column — DATE ONLY (most date columns)
1. Click + Add column
2. Select "Date and time"
3. Enter the Column Name
4. Under "Include Time" — toggle OFF (date only)
5. Set Required as specified
6. Click Save

## Adding a Date and Time column — DATE AND TIME (ChangedDate, Created, Modified)
1. Click + Add column
2. Select "Date and time"
3. Enter the Column Name
4. Under "Include Time" — toggle ON
5. Set Required as specified
6. Click Save

## Adding a Yes/No column
1. Click + Add column
2. Select "Yes/No"
3. Enter the Column Name
4. Under "Default value" set to Yes or No as specified
5. Click Save

## Adding a Choice column
1. Click + Add column
2. Select "Choice"
3. Enter the Column Name
4. Under "Choices" delete the default options and type each choice
   on a separate line exactly as shown in the list specification above
5. Set Required as specified
6. Set a Default value if specified
7. Click Save

## Renaming the Title column
1. Click on the "Title" column header
2. Select "Column settings" → "Edit"
3. Change the Name to "Title" (it may already be correct)
4. Add a description if helpful
5. Click Save
(Note: in some lists you may want to rename it — but for this app
 "Title" is always correct, so no rename is needed)

---

# ══════════════════════════════════════════════════════════════════
# AFTER CREATING ALL 8 LISTS AND THE DOCUMENT LIBRARY
# ══════════════════════════════════════════════════════════════════

Run these three scripts in this order:

1. node seedReferenceData.js
   → Populates List 6 (ReferenceData) with subjects, classes, types, terms

2. node seedAppSettings.js
   → Populates List 5 (AppSettings) with 9 default settings including
     rule configuration and archive settings

3. node seedPublicHolidays.js
   → Populates List 3 (PublicHolidays) with 2025 and 2026 SA public holidays

These scripts are generated by Claude Code during Phase 2 of the build.
Do not run them until Claude Code has completed Phase 2.

Lists 1, 2, 4, 7, and 8 start empty — they are populated through normal
use of the application. The AssessmentArchives document library also
starts empty and receives files automatically at end-of-term.

---

# ══════════════════════════════════════════════════════════════════
# GRAPH API LIST IDs
# ══════════════════════════════════════════════════════════════════
# After creating all lists, you need to find each list's ID for the .env file.
# Claude Code will need these to make Graph API calls.

To find a list's ID:
1. Go to your SharePoint site
2. Open the list
3. Click Settings (⚙️) → List settings
4. Look at the URL in your browser — the list ID is the GUID after "List="
   Example: .../_layouts/15/listedit.aspx?List=%7B1234abcd-...%7D
5. Copy the GUID (without the %7B and %7D brackets) and add it to your .env file

Alternatively, after Azure App Registration is complete, use Graph Explorer
(graph.microsoft.com) to call:
GET https://graph.microsoft.com/v1.0/sites/{siteId}/lists
This returns all lists with their IDs in one call.

# ════════════════════════════════════════════════════════════════════
# END OF SETUP REFERENCE
# ════════════════════════════════════════════════════════════════════
