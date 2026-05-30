# CLAUDE.md — Assessment Date Manager
# ════════════════════════════════════════════════════════════════════
# This file is the full specification for Claude Code to build the
# Assessment Date Manager application.
#
# ── OPENING INSTRUCTION FOR CLAUDE CODE ─────────────────────────────
# When you start Claude Code, paste this exactly:
#
#   "Read CLAUDE.md thoroughly. We are running in LOCAL MOCK MODE —
#    there is no Azure or SharePoint connection. Start with Phase 0b
#    only (the complete mock data layer). After Phase 0b is complete
#    and the app runs in the browser, confirm with me before moving
#    to Phase 1. Do not generate all files at once."
#
# ── IMPORTANT: THIS IS A LOCAL TEST BUILD ───────────────────────────
# VITE_USE_MOCK must be set to true (see .env section below).
# No Azure App Registration is needed.
# No SharePoint access is needed.
# The app runs entirely on your local machine using mock data.
# IT admin does not need to be involved at this stage.
#
# ✏️  = sections you should personalise before running Claude Code
# 🔧  = technical decision points — change if your setup differs
# ════════════════════════════════════════════════════════════════════

---
project: Assessment Date Manager
version: 1.0
author: ✏️ Your Name / School Name
target_users: Teachers (non-technical)
deployment: Local development (mock mode) — SharePoint deployment comes later
backend: Mock data layer (VITE_USE_MOCK=true) — Graph API connected later
---

# ══════════════════════════════════════════════════════════════════
# SECTION 1: PROJECT OVERVIEW
# ══════════════════════════════════════════════════════════════════
# Tell Claude Code what we are building and why.
# The more context here, the better the decisions Claude Code makes.

## What We Are Building

A web application that allows teachers to schedule assessments across
the school calendar. The app enforces two core rules automatically:

**Rule 1 — Maximum 2 assessments per day per student group**
No class or student group may have more than 2 assessments scheduled
on the same calendar day. The app must prevent this at the point of
booking, not just warn about it after.

# ✏️ IMPORTANT: Clarify this rule before running Claude Code.
# The rule as written applies PER CLASS/GROUP (e.g. Grade 10A cannot
# have more than 2 assessments on one day). If you mean school-wide
# (no more than 2 assessments total across the whole school on one day)
# change the comment below and Claude Code will implement accordingly.
# CURRENT SETTING: Per class/group

**Rule 2 — No assessments on formal assessment days**
A separate list of cycle test dates — individual days entered by Admins.
external assessments) exists. No teacher may book a class assessment
on any date that appears in this list.

# ✏️ Define whether "formal assessment days" means:
# (a) The SAME class cannot have an assessment on its own formal exam day
# (b) NO class can have ANY assessment on ANY formal assessment day
# CURRENT SETTING: (b) — school-wide block on formal assessment dates

**Rule 3 — No assessments on South African public holidays**
The app has all South African public holidays pre-loaded for 2025 and
2026. Public holidays are automatically blocked — teachers cannot book
assessments on these dates. The Management Team can view the holiday
list and add future years as needed. If a public holiday falls on a
Sunday, the following Monday is also blocked (as per the Public
Holidays Act of South Africa).

**Rule 4 — No assessments on Sundays**
Sundays are always blocked. No assessment may be booked on a Sunday
regardless of any other setting.

**Rule 5 — Saturdays allowed unless blocked**
Saturdays ARE valid assessment days unless they fall on a public holiday
or a date the Management Team has explicitly blocked. The date picker
must NOT automatically grey out Saturdays.

**Rule 6 — School assessment days (management-controlled)**
The Management Team defines which dates are valid assessment days for
the school year via the School Calendar Manager screen. Only dates
that appear in this list (and pass Rules 3, 4, and 5) are bookable.
Dates outside the school's defined assessment window are blocked.

## Who Uses This App

- **Teachers:** Book, view, edit, and cancel assessments for their classes
- **Administrators:** ✏️ Define admin role — manage cycle test dates, override rules, view all bookings
- **Heads of Department (optional):** ✏️ View assessments for their department only

## Deployment Target

The finished app will be:
1. Built as a React single-page application
2. Hosted on a SharePoint page as an embedded web part
3. Accessed by teachers via their existing SharePoint site URL
4. Authenticated via Microsoft 365 (teachers log in with school credentials)

# 🔧 If you want a standalone HTML file instead of SharePoint hosting,
# tell Claude Code: "Build this as a single standalone HTML file with
# localStorage instead of SharePoint Lists." All SharePoint/Graph API
# sections below would then be replaced with localStorage equivalents.

# ══════════════════════════════════════════════════════════════════
# SECTION 2: TECHNOLOGY STACK
# ══════════════════════════════════════════════════════════════════
# Claude Code must use exactly these technologies in this order.
# Do not deviate unless a section is marked as flexible.

## Required Technologies

```
Frontend:     React 18 (functional components, hooks only — no class components)
Styling:      Tailwind CSS + CSS custom properties (design-system.css)
Fonts:        DM Serif Display, DM Mono, Outfit (Google Fonts — loaded in design-system.css)
Animation:    Motion (formerly Framer Motion) for React component animations
Calendar UI:  react-big-calendar (for the main calendar view)
Date Handling: date-fns (for all date calculations — no moment.js)
Export PDF:   jspdf + jspdf-autotable (for all PDF exports including student report)
Export Excel: xlsx (for all Excel exports)
Auth:         Microsoft Authentication Library (MSAL) for React (@azure/msal-react)
API:          Microsoft Graph API (for all SharePoint List operations)
Build Tool:   Vite
Package Mgr:  npm
```

## Project Structure Claude Code Must Create

```
assessment-app/
├── CLAUDE.md                    ← this file
├── package.json
├── vite.config.js
├── index.html
├── src/
│   ├── main.jsx                 ← entry point
│   ├── App.jsx                  ← root component, auth wrapper
│   ├── authConfig.js            ← MSAL configuration ✏️
│   ├── graphConfig.js           ← Graph API endpoints
│   ├── components/
│   │   ├── CalendarView.jsx           ← split-screen: grade switcher + calendar + term grid
│   │   ├── TermGrid.jsx               ← right panel: grade-scoped week-by-week grid
│   │   ├── BookingForm.jsx            ← slide-in form with grade-aware field scoping
│   │   ├── TeacherProfile.jsx         ← profile setup: grades, classes, subjects taught
│   │   ├── ConfirmSchedule.jsx        ← teacher confirmation per grade per term
│   │   ├── ConfirmationReport.jsx     ← management: confirmation status PDF report
│   │   ├── PolicyManager.jsx          ← management: edit Rule 1 & Rule 2 settings
│   │   ├── TermArchiveManager.jsx     ← management: end-of-term archive & clear
│   │   ├── CycleTestManager.jsx      ← admin: manage blocked periods
│   │   ├── SchoolCalendarManager.jsx  ← management: term windows + public holidays
│   │   ├── ReferenceDataManager.jsx   ← management: manage all dropdowns
│   │   ├── AssessmentList.jsx         ← list/table view of assessments
│   │   ├── ConflictWarning.jsx        ← fallback modal for save-time rule violations (belt-and-braces)
│   │   ├── ArchiveBanner.jsx          ← persistent banner shown when archive is due
│   │   ├── FilterBar.jsx              ← subject/status filter within a grade view
│   │   ├── ExportButton.jsx           ← download to PDF/Excel
│   │   ├── AccessDenied.jsx           ← shown when role lacks permission
│   │   └── NavBar.jsx                 ← role-aware navigation + profile link
│   ├── hooks/
│   │   ├── useAssessments.js      ← fetch/create/update/delete assessments
│   │   ├── useCycleTests.js      ← fetch formal blocked dates
│   │   ├── usePublicHolidays.js   ← fetch SA public holidays list
│   │   ├── useSchoolCalendar.js   ← fetch school term windows
│   │   ├── useAppSettings.js      ← fetch & cache rule configuration from AppSettings
│   │   ├── useReferenceData.js    ← fetch & cache all dropdown options
│   │   ├── useUserRole.js         ← check SharePoint group membership & role
│   │   ├── useTeacherProfile.js   ← fetch/save teacher profile (grades/classes/subjects)
│   │   ├── useConflictChecker.js  ← rule engine (reads live settings)
│   │   ├── useArchive.js          ← archive trigger detection, export, clear, log
│   │   ├── useConfirmation.js     ← fetch/manage teacher confirmation state per grade per term
│   │   ├── useCycleTestConfirmations.js ← fetch/manage cycle test confirmation per grade per term
│   │   ├── useTermGrid.js         ← computes grade-scoped week columns and cell data
│   │   └── useGraph.js            ← Graph API wrapper
│   ├── utils/
│   │   ├── conflictRules.js     ← pure functions for rule checking
│   │   ├── dateHelpers.js       ← date formatting utilities
│   │   └── exportHelpers.js     ← PDF/Excel export logic
│   └── styles/
│       └── calendar.css         ← minimal overrides for react-big-calendar only
├── public/
│   └── favicon.ico
├── scripts/
│   ├── setupSharePoint.js   ← Phase 0: creates all lists, library, seeds data
│   ├── seedReferenceData.js ← populates ReferenceData list
│   ├── seedAppSettings.js   ← populates AppSettings list
│   └── seedPublicHolidays.js ← populates PublicHolidays list
└── .env.example                 ← environment variable template ✏️
```

# ══════════════════════════════════════════════════════════════════
# SECTION 3: SHAREPOINT LISTS (DATABASE STRUCTURE)
# ══════════════════════════════════════════════════════════════════
# Claude Code must create instructions for setting up these SharePoint
# Lists, AND write the Graph API calls to read/write them.
# The setup instructions go in a SETUP.md file Claude Code generates.

## List 1: Assessments

**SharePoint List Name:** `Assessments`

| Column Name      | Type           | Required | Notes                                                        |
|------------------|----------------|----------|--------------------------------------------------------------|
| Title            | Single line    | Yes      | Assessment name/description                                  |
| Subject          | Single line    | Yes      | Populated from ReferenceData (Category: Subject)             |
| ClassGroup       | Single line    | Yes      | Populated from ReferenceData (Category: ClassGroup)          |
| AssessmentType   | Single line    | Yes      | Populated from ReferenceData (Category: AssessmentType)      |
| AssessmentDate   | Date only      | Yes      | The scheduled date                                           |
| Duration         | Number         | No       | Duration in minutes                                          |
| TeacherName      | Single line    | Yes      | Auto-populated from logged-in user                           |
| TeacherEmail     | Single line    | Yes      | Auto-populated from Microsoft 365 login                      |
| Term             | Single line    | Yes      | Populated from ReferenceData (Category: Term)                |
| StudyRequired    | Choice         | Yes      | "Requires Study" or "No Study Required". Default: see rules below |
| Notes            | Multiple lines | No       | Optional notes for the assessment. Maximum 300 characters. Enforced in the UI — BookingForm shows a live character counter and blocks save above 300. |
| Status           | Single line    | Yes      | Default: Scheduled                                           |
| CreatedDate      | Date & Time    | Auto     | SharePoint auto-populates                                    |
| ModifiedDate     | Date & Time    | Auto     | SharePoint auto-populates                                    |

### StudyRequired Business Rules

**Rule — Formal assessments always require study:**
When a teacher books an assessment on a date that falls within a
CycleTests period, the StudyRequired field must be
automatically set to "Requires Study" and the field must be
**locked** (read-only, cannot be changed by the teacher).

Show a clear indicator in the BookingForm:
> 🔒 "This is a formal assessment — study is required. This cannot be changed."

**Rule — Non-formal assessments default to "Requires Study":**
For all other assessments, StudyRequired defaults to "Requires Study"
but the teacher CAN change it to "No Study Required" if appropriate
(e.g. a class observation, a homework check, a group project).

**Validation:**
validateBooking() must check and enforce the cycle test rule.
If a cycle test day is detected, StudyRequired is forced to
"Requires Study" regardless of what the teacher selected.

### StudyRequired Display in UI

**In the BookingForm:**
- Toggle switch: "Requires Study" (default ON) / "No Study Required"
- When locked (formal assessment): toggle is greyed out with lock icon
  and the explainer message shown above

**In the Calendar:**
- Assessments that require study: show a small 📚 icon on the calendar event
- Assessments with no study required: no icon
- This lets teachers scan the calendar and immediately see the study load

**In the Term Grid:**
- Badges that require study: show with a subtle top border in amber/gold
- Badges with no study required: standard badge style
- Hover tooltip always shows the study requirement

**In the AssessmentList table:**
- StudyRequired column shown as a coloured pill:
  - "Requires Study" → amber pill
  - "No Study Required" → grey pill

## List 2: CycleTests  ← INDIVIDUAL CYCLE TEST DATES

**SharePoint List Name:** `CycleTests`

# Each row is ONE cycle test on ONE specific date for ONE grade.
# Entered individually by Admins or Management via the CycleTestManager screen.
# Head office sets the actual exam schedule externally — this list covers
# cycle tests only. End-of-year exams are NOT managed in this app.
#
# BLOCKING RULE:
# When a teacher tries to book on a date that matches a CycleTest row for
# the matching grade, that specific DAY is blocked — not the whole week.
# Teachers CAN book continuous assessments on other days in the same week.
# Only the exact cycle test day itself prevents new bookings.
#
# StudyRequired is auto-forced to "Requires Study" and locked for:
# (a) All Cycle Test type entries — always
# (b) Any continuous assessment booked on a cycle test day — locked ON

| Column Name   | Type           | Required | Notes                                                          |
|---------------|----------------|----------|----------------------------------------------------------------|
| Title         | Single line    | Yes      | Auto-set e.g. "Grade 10 Mathematics Cycle Test — 5 Jun 2026"  |
| CycleTestDate | Date only      | Yes      | The specific date of the cycle test                            |
| Grade         | Single line    | Yes      | Which grade this applies to e.g. "Grade 10"                   |
| Subject       | Single line    | No       | Which subject e.g. "Mathematics" — blank = all subjects blocked|
| Notes         | Multiple lines | No       | Admin notes e.g. "Set by head office"                         |
| CreatedBy     | Person         | Auto     | SharePoint auto-populates                                      |

# NOTE: Scope (SchoolWide vs ClassOnly) is still configurable via CYCLE_TEST_SCOPE
# in AppSettings. SchoolWide = all classes blocked that day; ClassOnly = only
# the specified grade is blocked.
#
# Each cycle test is a SINGLE DATE ROW. If a grade has tests on Mon, Wed, and Fri,
# that is THREE rows — allowing teachers to book on Tue and Thu of that same week.

## List 3: PublicHolidays

**SharePoint List Name:** `PublicHolidays`

# This list is pre-seeded by Claude Code with all South African public
# holidays for 2025 and 2026. The Management Team can add future years
# as needed via the School Calendar Manager screen.
# The app NEVER allows bookings on dates in this list.
# If a holiday falls on a Sunday, the substitute Monday is also added.

| Column Name   | Type        | Required | Notes                                              |
|---------------|-------------|----------|----------------------------------------------------|
| Title         | Single line | Yes      | Holiday name e.g. "Youth Day"                      |
| HolidayDate   | Date only   | Yes      | The actual blocked date (including Monday subs)    |
| Year          | Number      | Yes      | The year — for filtering and future-year additions |
| IsSubstitute  | Yes/No      | Yes      | True if this is a Monday substitute for a Sunday holiday |
| OriginalDate  | Date only   | No       | The Sunday date this substitutes (if IsSubstitute = Yes) |
| Source        | Choice      | Yes      | Pre-loaded, Management Added                       |

## Pre-loaded Public Holidays — seed data for PublicHolidays list

### 2025 South African Public Holidays

```json
[
  { "Title": "New Year's Day",        "HolidayDate": "2025-01-01", "Year": 2025, "IsSubstitute": false },
  { "Title": "Human Rights Day",      "HolidayDate": "2025-03-21", "Year": 2025, "IsSubstitute": false },
  { "Title": "Good Friday",           "HolidayDate": "2025-04-18", "Year": 2025, "IsSubstitute": false },
  { "Title": "Family Day",            "HolidayDate": "2025-04-21", "Year": 2025, "IsSubstitute": false },
  { "Title": "Freedom Day",           "HolidayDate": "2025-04-27", "Year": 2025, "IsSubstitute": false },
  { "Title": "Freedom Day (observed)","HolidayDate": "2025-04-28", "Year": 2025, "IsSubstitute": true, "OriginalDate": "2025-04-27" },
  { "Title": "Workers' Day",          "HolidayDate": "2025-05-01", "Year": 2025, "IsSubstitute": false },
  { "Title": "Youth Day",             "HolidayDate": "2025-06-16", "Year": 2025, "IsSubstitute": false },
  { "Title": "National Women's Day",  "HolidayDate": "2025-08-09", "Year": 2025, "IsSubstitute": false },
  { "Title": "Heritage Day",          "HolidayDate": "2025-09-24", "Year": 2025, "IsSubstitute": false },
  { "Title": "Day of Reconciliation", "HolidayDate": "2025-12-16", "Year": 2025, "IsSubstitute": false },
  { "Title": "Christmas Day",         "HolidayDate": "2025-12-25", "Year": 2025, "IsSubstitute": false },
  { "Title": "Day of Goodwill",       "HolidayDate": "2025-12-26", "Year": 2025, "IsSubstitute": false }
]
```

### 2026 South African Public Holidays

```json
[
  { "Title": "New Year's Day",             "HolidayDate": "2026-01-01", "Year": 2026, "IsSubstitute": false },
  { "Title": "Human Rights Day",           "HolidayDate": "2026-03-21", "Year": 2026, "IsSubstitute": false },
  { "Title": "Good Friday",                "HolidayDate": "2026-04-03", "Year": 2026, "IsSubstitute": false },
  { "Title": "Family Day",                 "HolidayDate": "2026-04-06", "Year": 2026, "IsSubstitute": false },
  { "Title": "Freedom Day",                "HolidayDate": "2026-04-27", "Year": 2026, "IsSubstitute": false },
  { "Title": "Workers' Day",               "HolidayDate": "2026-05-01", "Year": 2026, "IsSubstitute": false },
  { "Title": "Youth Day",                  "HolidayDate": "2026-06-16", "Year": 2026, "IsSubstitute": false },
  { "Title": "National Women's Day",       "HolidayDate": "2026-08-09", "Year": 2026, "IsSubstitute": false },
  { "Title": "Day off for Women's Day",    "HolidayDate": "2026-08-10", "Year": 2026, "IsSubstitute": true, "OriginalDate": "2026-08-09" },
  { "Title": "Heritage Day",               "HolidayDate": "2026-09-24", "Year": 2026, "IsSubstitute": false },
  { "Title": "Day of Reconciliation",      "HolidayDate": "2026-12-16", "Year": 2026, "IsSubstitute": false },
  { "Title": "Christmas Day",              "HolidayDate": "2026-12-25", "Year": 2026, "IsSubstitute": false },
  { "Title": "Day of Goodwill",            "HolidayDate": "2026-12-26", "Year": 2026, "IsSubstitute": false }
]
```

# NOTE: Good Friday and Family Day dates are calculated dynamically
# based on Easter. The dates above are correct for 2025 and 2026.
# When adding 2027 and beyond, Good Friday and Family Day dates must
# be recalculated. Claude Code must include a utility function:
# calculateEasterDates(year) that returns Good Friday and Family Day
# for any given year, using the anonymous Gregorian algorithm.

## List 4: SchoolCalendar

**SharePoint List Name:** `SchoolCalendar`

# This list defines which dates are open for assessment booking.
# The Management Team maintains this list via the School Calendar
# Manager screen. Only dates in this list (that also pass Rules 3,
# 4, and 5) can have assessments booked against them.
#
# The Management Team adds date RANGES (e.g. Term 1: 15 Jan – 28 Mar)
# and the app expands these into individual valid days, automatically
# excluding Sundays and public holidays.
#
# Saturdays within the range ARE included unless they fall on a
# public holiday.

| Column Name   | Type        | Required | Notes                                              |
|---------------|-------------|----------|----------------------------------------------------|
| Title         | Single line | Yes      | e.g. "Term 1 2026", "Extra Saturday Sessions"      |
| StartDate     | Date only   | Yes      | First day of the open assessment window            |
| EndDate       | Date only   | Yes      | Last day of the open assessment window             |
| Year          | Number      | Yes      | The academic year                                  |
| Term          | Choice      | Yes      | Term 1 / Term 2 / Term 3 / Term 4 / Extra         |
| IncludeSaturdays | Yes/No   | Yes      | Default: Yes — Saturdays in range are bookable     |
| Notes         | Multiple lines | No    | Admin notes e.g. "Cycle test week excluded separately"   |
| CreatedBy     | Person      | Auto     | SharePoint auto-populates                          |

## List 5: AppSettings  ← CONFIGURABLE RULE DEFAULTS

**SharePoint List Name:** `AppSettings`

# This list stores all configurable business rules for the application.
# The Management Team edits these via the Policy Manager screen.
# The app reads this list on load and applies the current values.
# Defaults are seeded by seedAppSettings.js on first run.
#
# WHY A LIST INSTEAD OF CODE:
# Rules like "max 2 assessments per day" can change — a new principal
# may want to set it to 3, or reduce it to 1 for exam classes.
# Storing rules in code requires a developer and a redeployment.
# Storing them in a SharePoint list means management can change them
# instantly from within the app, with full audit history.

| Column Name   | Type           | Required | Notes                                              |
|---------------|----------------|----------|----------------------------------------------------|
| Title         | Single line    | Yes      | Machine-readable key e.g. "MAX_ASSESSMENTS_PER_DAY"|
| DisplayName   | Single line    | Yes      | Human-readable e.g. "Maximum Assessments Per Day"  |
| Value         | Single line    | Yes      | The current value e.g. "2"                         |
| DefaultValue  | Single line    | Yes      | The original default — shown in UI for reset button|
| DataType      | Choice         | Yes      | Number, Boolean, Text                              |
| Description   | Multiple lines | Yes      | Explains what the rule does and the impact of changing it |
| Category      | Single line    | Yes      | Groups rules: DailyLimit, CycleTests, General     |
| ChangedBy     | Single line    | No       | Auto-populated when value is updated               |
| ChangedDate   | Date & Time    | No       | Auto-populated when value is updated               |

## AppSettings Seed Data

```json
[
  {
    "Title": "MAX_ASSESSMENTS_PER_DAY",
    "DisplayName": "Maximum Assessments Per Day",
    "Value": "2",
    "DefaultValue": "2",
    "DataType": "Number",
    "Category": "DailyLimit",
    "Description": "The maximum number of assessments a single class group may have on any one day. Default is 2. Increasing this allows more flexibility but may increase student workload. Minimum allowed value: 1. Maximum allowed value: 5."
  },
  {
    "Title": "DAILY_LIMIT_SCOPE",
    "DisplayName": "Daily Limit Applies To",
    "Value": "ClassGroup",
    "DefaultValue": "ClassGroup",
    "DataType": "Text",
    "Category": "DailyLimit",
    "Description": "Defines whether the daily assessment limit applies per class group (e.g. Grade 10A cannot have more than 2) or school-wide (no more than 2 assessments total in the whole school on one day). Valid values: ClassGroup, SchoolWide."
  },
  {
    "Title": "CYCLE_TEST_SCOPE",
    "DisplayName": "Formal Assessment Date Block Scope",
    "Value": "SchoolWide",
    "DefaultValue": "SchoolWide",
    "DataType": "Text",
    "Category": "CycleTests",
    "Description": "Defines whether blocking a cycle test day prevents bookings for that class only, or for the entire school. Valid values: SchoolWide (no class can book on a cycle test day), ClassOnly (only the specified grade is blocked on that date)."
  },
  {
    "Title": "ALLOW_ADMIN_OVERRIDE",
    "DisplayName": "Allow Admin Rule Override",
    "Value": "true",
    "DefaultValue": "true",
    "DataType": "Boolean",
    "Category": "General",
    "Description": "When true, Administrators and Management Team members can override Rule 1 and Rule 2 violations and save the booking anyway. The override is recorded in the audit log. When false, no one can override any rule."
  },
  {
    "Title": "WARN_BEFORE_CYCLE_TEST",
    "DisplayName": "Warn Days Before Cycle Test",
    "Value": "0",
    "DefaultValue": "0",
    "DataType": "Number",
    "Category": "CycleTests",
    "Description": "Number of school days before a cycle test week during which a warning (but not a block) is shown. Set to 0 to disable. Set to 1 to warn teachers if they book on the day before a cycle test week begins. Set to 5 to warn for the entire week before."
  },
  {
    "Title": "ARCHIVE_DESTINATION_PATH",
    "DisplayName": "Archive Storage Location",
    "Value": "Site Assets/AssessmentArchives",
    "DefaultValue": "Site Assets/AssessmentArchives",
    "DataType": "Text",
    "Category": "Archive",
    "Description": "The SharePoint Document Library path where term archive Excel files are saved. Sub-folders by year are created automatically. Change this to match your school's preferred document structure."
  },
  {
    "Title": "ARCHIVE_SNOOZE_COUNT",
    "DisplayName": "Archive Reminder Snooze Count",
    "Value": "0",
    "DefaultValue": "0",
    "DataType": "Number",
    "Category": "Archive",
    "Description": "Tracks how many times the archive reminder has been snoozed for the current term. Set to 0 automatically after a successful archive. Do not edit manually."
  },
  {
    "Title": "ARCHIVE_SNOOZED_UNTIL",
    "DisplayName": "Archive Reminder Snoozed Until",
    "Value": "",
    "DefaultValue": "",
    "DataType": "Text",
    "Category": "Archive",
    "Description": "ISO datetime string indicating when the archive reminder snooze expires. Blank means not snoozed. Set automatically by the app — do not edit manually."
  },
  {
    "Title": "LAST_ARCHIVED_TERM",
    "DisplayName": "Last Archived Term",
    "Value": "",
    "DefaultValue": "",
    "DataType": "Text",
    "Category": "Archive",
    "Description": "The Title of the SchoolCalendar term that was most recently archived. Used to detect whether a new term needs archiving. Set automatically after each archive — do not edit manually."
  }
]
```

## Archive Trigger Logic (useArchive.js)

Claude Code must implement a `useArchive.js` hook that runs on every
Management Team login with the following logic:

```javascript
/**
 * On Management Team login, check whether an archive is due.
 * Returns archiveRequired: true if:
 * 1. Today's date is after the EndDate of the most recently ended
 *    SchoolCalendar term window, AND
 * 2. That term's Title does NOT match LAST_ARCHIVED_TERM in AppSettings
 *    (meaning it has not yet been archived)
 *
 * Also returns:
 * - snoozedUntil: DateTime | null  (null if not snoozed)
 * - snoozeCount: number
 * - overdueBy: number (days since term ended)
 * - termName: string (e.g. "Term 2 2026")
 */
export function useArchive() {
  return {
    archiveRequired,
    snoozedUntil,
    snoozeCount,
    overdueBy,
    termName,
    snooze,       // function: sets ARCHIVE_SNOOZED_UNTIL to now + 24hrs, increments count
    runArchive,   // function: executes the full archive workflow
  }
}
```

**Archive workflow steps inside runArchive():**
1. Fetch all rows from Assessments list
2. Fetch all rows from CycleTests list
3. Generate Assessments Excel file (two sheets: data + summary)
4. Generate CycleTests Excel file (all cycle test dates for the term)
5. Upload both files to SharePoint Document Library at ARCHIVE_DESTINATION_PATH
6. Confirm upload succeeded (check Graph API response status)
7. Only if step 6 succeeded: delete all rows from Assessments list
8. Only if step 6 succeeded: delete all rows from CycleTests list
#    for the archived term only (other terms remain)
9. Write row to ArchiveLog list
10. Update LAST_ARCHIVED_TERM in AppSettings to current term name
11. Reset ARCHIVE_SNOOZE_COUNT to 0 in AppSettings
12. Clear ARCHIVE_SNOOZED_UNTIL in AppSettings

**SharePoint row deletion:**
Deleting all rows from a large list must use Graph API batch requests
(batches of 20) to avoid throttling. Never delete rows one at a time
in a loop without batching.

1. Fetches all rows from AppSettings on app load
2. Caches as a key-value object: `{ MAX_ASSESSMENTS_PER_DAY: 2, ... }`
3. Exposes a `getSetting(key)` helper that returns the typed value
   (automatically casts to Number, Boolean, or string based on DataType)
4. The rule engine functions receive settings as parameters — they never
   hardcode values like `dailyLimit = 2`

```javascript
// Example in useConflictChecker.js:
const { getSetting } = useAppSettings();
const dailyLimit = getSetting('MAX_ASSESSMENTS_PER_DAY'); // returns 2 (Number)
const scope      = getSetting('DAILY_LIMIT_SCOPE');       // returns 'ClassGroup'
const scope2     = getSetting('CYCLE_TEST_SCOPE');       // returns 'SchoolWide'

// checkDailyLimit now uses the live setting, not a hardcoded constant
checkDailyLimit(newAssessment, existingAssessments, dailyLimit, scope)
```

## List 6: ReferenceData  ← MASTER LOOKUP TABLE

# This is a key architectural decision: instead of hardcoding dropdown
# options (subjects, classes, assessment types, terms) into the app code,
# ALL selectable values live in this single SharePoint list.
#
# WHY THIS IS THE RIGHT APPROACH:
# - An admin can add a new subject or class group without touching the app
# - No developer needed to update dropdown options mid-year
# - One place to maintain = fewer mistakes
# - The app always reflects current school structure
#
# HOW IT WORKS:
# Every row has a Category (what type of option it is) and a Value
# (the actual option). The app fetches all rows on load, then filters
# by category to populate each dropdown.
#
# Example rows:
# | Category        | Value          | SortOrder | Active |
# | Subject         | Mathematics    | 1         | Yes    |
# | Subject         | English        | 2         | Yes    |
# | Subject         | Life Sciences  | 3         | Yes    |
# | ClassGroup      | Grade 8A       | 1         | Yes    |
# | ClassGroup      | Grade 8B       | 2         | Yes    |
# | ClassGroup      | Grade 10A      | 5         | Yes    |
# | AssessmentType  | Test           | 1         | Yes    |
# | AssessmentType  | Assignment     | 2         | Yes    |
# | AssessmentType  | Practical      | 3         | Yes    |
# | Term            | Term 1         | 1         | Yes    |
# | Term            | Term 2         | 2         | Yes    |
# | Status          | Scheduled      | 1         | Yes    |
# | Status          | Cancelled      | 2         | Yes    |

**SharePoint List Name:** `ReferenceData`

| Column Name  | Type        | Required | Notes                                              |
|--------------|-------------|----------|----------------------------------------------------|
| Title        | Single line | Yes      | The display value e.g. "Mathematics", "Grade 10A"  |
| Category     | Single line | Yes      | Groups values: Subject, ClassGroup, AssessmentType, Term, Status |
| SortOrder    | Number      | Yes      | Controls the order items appear in dropdowns        |
| Active       | Yes/No      | Yes      | Default: Yes. Set to No to hide without deleting   |
| Description  | Single line | No       | Optional — e.g. "Includes Physics and Chemistry"   |

## How Claude Code Must Use ReferenceData

Claude Code must implement a `useReferenceData.js` hook that:

1. Fetches all rows from the ReferenceData list on app load (one API call)
2. Caches the result in React state for the session (no repeat calls)
3. Exposes a helper function: `getOptions(category)` that returns a
   sorted array of active values for a given category
4. All dropdowns in the app call `getOptions()` — never use hardcoded arrays

```javascript
// Example usage in BookingForm.jsx:
const { getOptions } = useReferenceData();
const subjects = getOptions('Subject');       // ['English', 'Mathematics', ...]
const classes  = getOptions('ClassGroup');    // ['Grade 8A', 'Grade 8B', ...]
const types    = getOptions('AssessmentType'); // ['Assignment', 'Practical', ...]
const terms    = getOptions('Term');          // ['Term 1', 'Term 2', ...]
```

## Admin: Managing ReferenceData

The CycleTestManager.jsx admin screen must include a second tab:
**"Reference Data"** — a simple table showing all ReferenceData rows,
grouped by Category, with:

- Add new value (select Category from fixed list, enter Title, set SortOrder)
- Toggle Active/Inactive per row (so old classes can be hidden without deletion)
- Reorder rows within a category using SortOrder
- Admin-only — teachers cannot see or edit this tab

## Seeding ReferenceData on First Run

Claude Code must generate a `seed-data.json` file and a
`seedReferenceData.js` script that populates the list with starter
values. The SETUP.md must include instructions to run this script once
after the SharePoint list is created.

The seed file must contain sensible defaults that the school replaces:

```json
{
  "ReferenceData": [
    { "Category": "Subject",        "Title": "Mathematics",    "SortOrder": 1, "Active": true },
    { "Category": "Subject",        "Title": "English",        "SortOrder": 2, "Active": true },
    { "Category": "Subject",        "Title": "Life Sciences",  "SortOrder": 3, "Active": true },
    { "Category": "Subject",        "Title": "Physical Sciences", "SortOrder": 4, "Active": true },
    { "Category": "Subject",        "Title": "History",        "SortOrder": 5, "Active": true },
    { "Category": "Subject",        "Title": "Geography",      "SortOrder": 6, "Active": true },
    { "Category": "Subject",        "Title": "Accounting",     "SortOrder": 7, "Active": true },
    { "Category": "ClassGroup",     "Title": "Grade 8A",       "SortOrder": 1, "Active": true },
    { "Category": "ClassGroup",     "Title": "Grade 8B",       "SortOrder": 2, "Active": true },
    { "Category": "ClassGroup",     "Title": "Grade 9A",       "SortOrder": 3, "Active": true },
    { "Category": "ClassGroup",     "Title": "Grade 9B",       "SortOrder": 4, "Active": true },
    { "Category": "ClassGroup",     "Title": "Grade 10A",      "SortOrder": 5, "Active": true },
    { "Category": "ClassGroup",     "Title": "Grade 10B",      "SortOrder": 6, "Active": true },
    { "Category": "ClassGroup",     "Title": "Grade 11A",      "SortOrder": 7, "Active": true },
    { "Category": "ClassGroup",     "Title": "Grade 11B",      "SortOrder": 8, "Active": true },
    { "Category": "ClassGroup",     "Title": "Grade 11C",      "SortOrder": 9, "Active": true },
    { "Category": "ClassGroup",     "Title": "Grade 12A",      "SortOrder": 10, "Active": true },
    { "Category": "ClassGroup",     "Title": "Grade 12B",      "SortOrder": 11, "Active": true },
    { "Category": "ClassGroup",     "Title": "Grade 12C",      "SortOrder": 12, "Active": true },
    { "Category": "AssessmentType", "Title": "Cycle Test",           "SortOrder": 1, "Active": true },
    { "Category": "AssessmentType", "Title": "Quiz",           "SortOrder": 2, "Active": true },
    { "Category": "AssessmentType", "Title": "Assignment",     "SortOrder": 3, "Active": true },
    { "Category": "AssessmentType", "Title": "Practical",      "SortOrder": 4, "Active": true },
    { "Category": "AssessmentType", "Title": "Oral",           "SortOrder": 5, "Active": true },
    { "Category": "AssessmentType", "Title": "Project",        "SortOrder": 6, "Active": true },
    { "Category": "Term",           "Title": "Term 1",         "SortOrder": 1, "Active": true },
    { "Category": "Term",           "Title": "Term 2",         "SortOrder": 2, "Active": true },
    { "Category": "Term",           "Title": "Term 3",         "SortOrder": 3, "Active": true },
    { "Category": "Term",           "Title": "Term 4",         "SortOrder": 4, "Active": true },
    { "Category": "Status",         "Title": "Scheduled",      "SortOrder": 1, "Active": true },
    { "Category": "Status",         "Title": "Cancelled",      "SortOrder": 2, "Active": true },
    { "Category": "Status",         "Title": "Completed",      "SortOrder": 3, "Active": true }
  ]
}
```

# ✏️ Edit the seed data above to match your school's actual
# subjects, class groups, and term structure before running Claude Code.
# Everything in this list can also be changed later via the admin UI.

## List 7: ReferenceDataAuditLog  ← ACCOUNTABILITY TRAIL

**SharePoint List Name:** `ReferenceDataAuditLog`

# Every change to ReferenceData or AppSettings writes a row here automatically.
# Never written to directly by users. Never delete rows from this list.

| Column Name      | Type              | Required | Notes                                                        |
|------------------|-------------------|----------|--------------------------------------------------------------|
| Title            | Single line       | Yes      | Auto-set by app e.g. "Subject: Mathematics changed"          |
| ListChanged      | Single line       | Yes      | Which list was changed: ReferenceData or AppSettings         |
| Category         | Single line       | Yes      | The Category or Setting key that changed                     |
| OldValue         | Single line       | No       | Value before the change (blank for new additions)            |
| NewValue         | Single line       | Yes      | Value after the change                                       |
| Action           | Choice            | Yes      | Added / Edited / Deactivated / Reactivated / Deleted / Setting Changed |
| ChangedBy        | Single line       | Yes      | Display name of the user who made the change                 |
| ChangedByEmail   | Single line       | Yes      | Email of the user who made the change                        |
| ChangedDate      | Date and Time     | Yes      | Timestamp — set by the app, not SharePoint                   |
| ItemId           | Number            | No       | SharePoint ID of the changed row (for traceability)          |
| Created          | Date and Time     | Auto     | SharePoint creates automatically                             |

## List 8: ArchiveLog  ← END-OF-TERM ARCHIVE RECORD

**SharePoint List Name:** `ArchiveLog`

# Written to automatically by the app after each successful archive operation.
# Never written to directly by users. Never delete rows from this list.

| Column Name        | Type              | Required | Notes                                                    |
|--------------------|-------------------|----------|----------------------------------------------------------|
| Title              | Single line       | Yes      | Auto-set e.g. "Term 2 2026 Archive"                      |
| TermName           | Single line       | Yes      | e.g. "Term 2"                                            |
| Year               | Number            | Yes      | Academic year e.g. 2026                                  |
| AssessmentsFile    | Single line       | Yes      | Full SharePoint path to Assessments Excel file           |
| CycleTestsFile    | Single line       | Yes      | Full SharePoint path to CycleTests Excel file           |
| AssessmentsCount   | Number            | Yes      | Number of rows exported from Assessments                 |
| CycleTestsCount   | Number            | Yes      | Number of rows exported from CycleTests       |
| ArchivedBy         | Single line       | Yes      | Display name of management user who ran archive          |
| ArchivedByEmail    | Single line       | Yes      | Email of management user who ran archive                 |
| ArchivedDate       | Date and Time     | Yes      | Exact timestamp of when the archive completed            |
| ClearSucceeded     | Yes/No            | Yes      | Whether both lists were successfully cleared             |
| Notes              | Multiple lines    | No       | Optional management notes                                |
| Created            | Date and Time     | Auto     | SharePoint creates automatically                         |

## List 9: TeacherProfiles  ← TEACHER PROFILE DATA

**SharePoint List Name:** `TeacherProfiles`

# One row per teacher. Created on first profile save, updated on edit.
# Stores which grades, classes, and subjects each teacher is responsible for.
# Drives: default grade calendar on login, grade switcher tabs, and
# field scoping in the BookingForm (class/subject dropdowns).
#
# GradesTaught, ClassesTaught, and SubjectsTaught are stored as JSON
# strings in Multiple lines of text columns. The app serialises on save
# and deserialises on load. This avoids junction tables in SharePoint.

| Column Name      | Type                            | Required | Default | Notes                                                      |
|------------------|---------------------------------|----------|---------|------------------------------------------------------------|
| Title            | Single line of text             | Yes      | —       | Teacher's display name (auto-populated from Microsoft 365) |
| TeacherEmail     | Single line of text             | Yes      | —       | Primary key — teacher's school email. Used to look up profile on login |
| GradesTaught     | Multiple lines of text (Plain)  | Yes      | —       | JSON string: '["Grade 10","Grade 11"]'                     |
| ClassesTaught    | Multiple lines of text (Plain)  | Yes      | —       | JSON string: '["Grade 10A","Grade 10B","Grade 11A"]'       |
| SubjectsTaught   | Multiple lines of text (Plain)  | Yes      | —       | JSON string: '["Mathematics","English"]'                   |
| DefaultGrade     | Single line of text             | Yes      | —       | Which calendar opens on login e.g. "Grade 10"              |
| ProfileComplete  | Yes/No                          | Yes      | No      | False until teacher first saves profile. Triggers first-login redirect. |
| Created          | Date and Time                   | Auto     | —       | SharePoint creates automatically                           |
| Modified         | Date and Time                   | Auto     | —       | SharePoint creates automatically                           |

This list starts empty. No seed script needed. Teachers populate it themselves
on first login via the TeacherProfile screen.

## List 10: ConfirmationLog  ← TEACHER SCHEDULE CONFIRMATION

**SharePoint List Name:** `ConfirmationLog`

# One row per teacher per grade per term.
# Written when a teacher confirms their schedule, updated if they un-confirm.
# Read by the Management PDF report to show confirmed vs not yet confirmed.
# Never delete rows — they form a permanent audit trail of who confirmed when.
#
# CONFIRMATION RULES:
# - A teacher can only confirm once they have at least 1 Scheduled assessment
#   for that grade in the current term
# - Once confirmed, all their assessments for that grade/term are locked
#   (read-only — cannot be edited or cancelled) until they un-confirm
# - Un-confirming resets their ConfirmedAt to null and unlocks assessments
# - Management can unlock (un-confirm) on a teacher's behalf
# - Assessments added AFTER confirmation automatically flag the teacher as
#   needing to re-confirm (IsStale = true) so Management can see the discrepancy

| Column Name        | Type              | Required | Notes                                                        |
|--------------------|-------------------|----------|--------------------------------------------------------------|
| Title              | Single line       | Yes      | Auto-set e.g. "Mr C. Uren — Grade 10 — Term 1 2026"         |
| TeacherEmail       | Single line       | Yes      | Teacher's school email — links to TeacherProfiles            |
| TeacherName        | Single line       | Yes      | Display name at time of confirmation                         |
| Grade              | Single line       | Yes      | e.g. "Grade 10"                                              |
| Term               | Single line       | Yes      | e.g. "Term 1 2026" — matches SchoolCalendar Title            |
| IsConfirmed        | Yes/No            | Yes      | True when teacher has confirmed. False if un-confirmed.      |
| ConfirmedAt        | Date and Time     | No       | Timestamp of confirmation. Blank if not confirmed.           |
| AssessmentCount    | Number            | Yes      | Number of Scheduled assessments locked at time of confirmation |
| IsStale            | Yes/No            | Yes      | True if assessments were added/changed after confirmation    |
| UnconfirmedAt      | Date and Time     | No       | Timestamp if teacher un-confirmed after confirming           |
| UnconfirmedReason  | Single line       | No       | Optional reason entered when un-confirming                   |
| Created            | Date and Time     | Auto     | SharePoint creates automatically                             |
| Modified           | Date and Time     | Auto     | SharePoint creates automatically                             |

This list starts empty. Rows are created by the app when a teacher
first views the Confirm Schedule screen for a grade/term combination.

## List 11: CycleTestConfirmations  ← CYCLE TEST SCHEDULE CONFIRMATION

**SharePoint List Name:** `CycleTestConfirmations`

# One row per grade per term.
# Written when Management confirms the cycle test schedule for a grade.
# Until a grade's cycle tests are confirmed, teachers CANNOT book any
# assessments for that grade — the booking form is locked.
# Teachers can still VIEW cycle test dates on the calendar at all times.
#
# WORKFLOW:
# 1. Management enters all cycle test dates for a grade via CycleTestManager
# 2. Management clicks "Confirm Grade X Cycle Tests" on CycleTestManager
# 3. A CycleTestConfirmations row is created/updated for that grade + term
# 4. Teachers see a login notification: "Grade X cycle tests confirmed"
# 5. Booking is now open for that grade
#
# If Management needs to change a cycle test after confirming:
# 1. Management un-confirms (IsConfirmed → false)
# 2. Teachers immediately see "Grade X cycle tests not yet confirmed"
#    and booking is blocked again for that grade
# 3. Management edits the cycle test date
# 4. Management re-confirms → booking re-opens
#
# STALE:
# If a cycle test is added, edited, or deleted after confirmation,
# IsStale is set to true automatically. Management must re-confirm.

| Column Name      | Type          | Required | Notes                                                              |
|------------------|---------------|----------|--------------------------------------------------------------------|
| Title            | Single line   | Yes      | Auto-set e.g. "Grade 10 — Term 1 2026 — Cycle Test Schedule"      |
| Grade            | Single line   | Yes      | e.g. "Grade 10"                                                    |
| Term             | Single line   | Yes      | e.g. "Term 1 2026" — matches SchoolCalendar Title                  |
| IsConfirmed      | Yes/No        | Yes      | True when Management has confirmed cycle tests for this grade/term |
| ConfirmedAt      | Date and Time | No       | Timestamp of confirmation                                          |
| ConfirmedBy      | Single line   | No       | Display name of Management user who confirmed                      |
| CycleTestCount   | Number        | Yes      | Number of cycle tests for this grade at time of confirmation       |
| IsStale          | Yes/No        | Yes      | True if cycle tests were changed after confirmation                |
| UnconfirmedAt    | Date and Time | No       | Timestamp if Management un-confirmed                               |
| UnconfirmedBy    | Single line   | No       | Display name of user who un-confirmed                              |
| Created          | Date and Time | Auto     | SharePoint creates automatically                                   |
| Modified         | Date and Time | Auto     | SharePoint creates automatically                                   |

This list starts empty. Rows are created/updated by CycleTestManager
when Management confirms or un-confirms a grade's cycle test schedule.

# ══════════════════════════════════════════════════════════════════
# SECTION 4: MICROSOFT AZURE APP REGISTRATION
# ══════════════════════════════════════════════════════════════════
# The app needs to be registered in Azure Active Directory so
# teachers can log in with their school Microsoft 365 accounts.
# Claude Code must generate a SETUP.md with step-by-step instructions
# for your IT administrator to complete this registration.

## Azure App Registration Requirements

Claude Code must generate a SETUP.md file containing step-by-step
instructions for the school IT administrator to:

1. Go to portal.azure.com → Azure Active Directory → App registrations
2. Create a new registration:
   - Name: ✏️ "School Assessment Manager"
   - Supported account types: "Accounts in this organisational directory only"
   - Redirect URI: ✏️ the SharePoint page URL where the app will be hosted
3. Note the Application (client) ID and Directory (tenant) ID
4. Under API Permissions, add:
   - Microsoft Graph → Delegated → Sites.ReadWrite.All
   - Microsoft Graph → Delegated → User.Read
5. Grant admin consent for the permissions
6. Copy the client ID and tenant ID into the .env file

## Environment Variables (.env file)

Claude Code must create a `.env.example` file with these variables:

```
# ✏️ Fill these in after Azure App Registration (see SETUP.md)
VITE_AZURE_CLIENT_ID=your-client-id-here
VITE_AZURE_TENANT_ID=your-tenant-id-here

# ✏️ Your SharePoint site URL
VITE_SHAREPOINT_SITE_URL=https://yourschool.sharepoint.com/sites/yoursite

# ✏️ The SharePoint site ID (get from Graph API explorer after setup)
VITE_SHAREPOINT_SITE_ID=your-sharepoint-site-id-here
```

# ══════════════════════════════════════════════════════════════════
# SECTION 5: CORE BUSINESS RULES ENGINE
# ══════════════════════════════════════════════════════════════════
# This is the most critical section. These rules must be implemented
# as pure functions in src/utils/conflictRules.js so they can be
# tested independently of the UI.

## Rule Engine: conflictRules.js

Claude Code must implement these functions as named exports:

### Function 1: checkDailyLimit

```javascript
/**
 * Checks whether adding a new assessment would exceed the daily limit
 * for a given class group (or school-wide) on a given date.
 * The limit and scope are READ FROM AppSettings — never hardcoded.
 *
 * @param {Object} newAssessment - The assessment being booked
 * @param {string} newAssessment.classGroup - e.g. "Grade 10A"
 * @param {string} newAssessment.assessmentDate - ISO date string
 * @param {Array}  existingAssessments - All current assessments from SharePoint
 * @param {number} dailyLimit - From AppSettings: MAX_ASSESSMENTS_PER_DAY (default: 2)
 * @param {string} scope - From AppSettings: DAILY_LIMIT_SCOPE
 *                         'ClassGroup' (default) or 'SchoolWide'
 * @returns {Object} { allowed: boolean, count: number, conflicting: Array }
 */
export function checkDailyLimit(newAssessment, existingAssessments, dailyLimit, scope)
```

### Function 2: checkCycleTestConflict

```javascript
/**
 * Checks whether the proposed date is a cycle test day for the given grade.
 * Blocks ONLY that specific day — NOT the whole week.
 * Teachers can book continuous assessments on other days in the same week.
 * The scope is READ FROM AppSettings — never hardcoded.
 *
 * @param {string} proposedDate - ISO date string e.g. "2026-06-05"
 * @param {string} grade        - e.g. "Grade 10"
 * @param {Array}  cycleTests   - Array of { cycleTestDate, grade, subject, title }
 * @param {string} scope        - From AppSettings: CYCLE_TEST_SCOPE
 *                                'SchoolWide' — block all grades on any cycle test day
 *                                'ClassOnly'  — block only the grade in the row
 * @returns {Object} {
 *   blocked: boolean,
 *   reason: string | null,   // e.g. "Grade 10 Maths Cycle Test is on this day"
 *   cycleTest: Object | null // the matching CycleTest row if blocked
 * }
 *
 * IMPORTANT: Date comparison is exact (same calendar day).
 * A date that falls in the same WEEK as a cycle test is NOT blocked
 * unless it is the exact same date as the CycleTest row.
 */
export function checkCycleTestConflict(proposedDate, grade, cycleTests, scope)
```

### Function 2b: enforceStudyRequired

```javascript
/**
 * Determines the correct StudyRequired value for a booking and whether
 * the field should be locked (read-only) in the UI.
 *
 * Rules:
 * - If proposedDate falls within ANY CycleTests period:
 *   → studyRequired MUST be "Requires Study", field is LOCKED
 * - If proposedDate is NOT in a cycle test week:
 *   → studyRequired defaults to "Requires Study" but teacher CAN change it
 *
 * This function is called:
 * (a) In BookingForm.jsx whenever the date field changes — to set the
 *     toggle state and lock/unlock it in real time
 * (b) In validateBooking() as a final enforcement check before save —
 *     overrides any incorrect value the teacher may have submitted
 *
 * @param {string} proposedDate - ISO date string
 * @param {Array}  cycleTests  - Array of { startDate, endDate } objects
 * @param {string} currentValue - What the teacher currently has selected
 * @returns {Object} {
 *   value: "Requires Study" | "No Study Required",  // enforced value
 *   locked: boolean,   // true = field is read-only in UI
 *   reason: string | null  // shown as tooltip/banner when locked
 * }
 */
export function enforceStudyRequired(proposedDate, cycleTests, currentValue)
```

### Function 3: checkPublicHoliday

```javascript
/**
 * Checks whether the proposed date is a South African public holiday
 * or a substitute Monday for a Sunday holiday.
 *
 * @param {string} proposedDate - ISO date string
 * @param {Array}  publicHolidays - Array of { holidayDate, title } objects
 * @returns {Object} { blocked: boolean, reason: string | null }
 * Example reason: "Youth Day (16 June 2026)"
 */
export function checkPublicHoliday(proposedDate, publicHolidays)
```

### Function 4: checkSunday

```javascript
/**
 * Checks whether the proposed date falls on a Sunday.
 * Sundays are always blocked regardless of any other setting.
 *
 * @param {string} proposedDate - ISO date string
 * @returns {Object} { blocked: boolean, reason: string | null }
 */
export function checkSunday(proposedDate)
```

### Function 5: checkSchoolCalendar

```javascript
/**
 * Checks whether the proposed date falls within a school assessment
 * window defined by the Management Team in the SchoolCalendar list.
 *
 * Saturdays within a term window ARE valid unless:
 * (a) The SchoolCalendar entry has IncludeSaturdays = false, OR
 * (b) The Saturday is a public holiday (caught by checkPublicHoliday)
 *
 * @param {string} proposedDate - ISO date string
 * @param {Array}  schoolCalendar - Array of { startDate, endDate, term, includeSaturdays } objects
 * @returns {Object} { blocked: boolean, reason: string | null }
 * Example reason: "This date falls outside the school's assessment calendar.
 *                  The next available term window opens on 13 January 2026."
 */
export function checkSchoolCalendar(proposedDate, schoolCalendar)
```

### Function 6: calculateEasterDates

```javascript
/**
 * Calculates Good Friday and Family Day (Easter Monday) for any year
 * using the Anonymous Gregorian algorithm.
 * Used when seeding or adding public holidays for future years.
 *
 * @param {number} year - The year to calculate Easter for
 * @returns {Object} { goodFriday: string, familyDay: string } - ISO date strings
 */
export function calculateEasterDates(year)
```

### Function 7: validateBooking (master validator — updated)

```javascript
/**
 * Runs ALL rule checks for a proposed booking and returns a combined result.
 * Reads live rule configuration from appSettings — no hardcoded values.
 * Checks in this order (fail-fast on first block):
 * 1. checkSunday
 * 2. checkPublicHoliday
 * 3. checkSchoolCalendar
 * 4. checkCycleTestConflict  — blocks only the specific cycle test day
 *                                (uses CYCLE_TEST_SCOPE from appSettings)
 * 5. checkDailyLimit          (uses MAX_ASSESSMENTS_PER_DAY + DAILY_LIMIT_SCOPE)
 * 6. enforceStudyRequired     (never blocks — corrects StudyRequired value if wrong)
 *
 * @param {Object} newAssessment
 * @param {Array}  existingAssessments
 * @param {Array}  cycleTests
 * @param {Array}  publicHolidays
 * @param {Array}  schoolCalendar
 * @param {Object} appSettings - Live settings object from useAppSettings hook
 * @returns {Object} {
 *   valid: boolean,
 *   errors: string[],
 *   warnings: string[],
 *   correctedAssessment: Object  // assessment with StudyRequired enforced correctly
 * }
 */
export function validateBooking(
  newAssessment,
  existingAssessments,
  cycleTests,
  publicHolidays,
  schoolCalendar,
  appSettings
)

## Rule Behaviour

- **If Rule 1 is violated:** The live daily limit indicator in BookingForm
  shows State 3 (red) and disables the Save button BEFORE the teacher
  attempts to save — preventing the conflict proactively.
  For belt-and-braces, validateBooking() also enforces this at save time.
  If somehow a save is attempted with a full slot (e.g. rapid concurrent
  bookings by two teachers), a ConflictWarning modal surfaces naming
  the conflicting assessments and offering a "Change Date" button.
  Admins and Management can override — override is always logged.
  listing which assessments already exist for that class on that date.
  Display the current daily limit (e.g. "Maximum 2 assessments per day
  for Grade 10A — this class already has 2 booked on this date").
  If ALLOW_ADMIN_OVERRIDE is true in AppSettings, show an
  "Override and Save" button for Admin and Management roles only.

- **If Rule 2 is violated:** Block the save. Show the name and date range
  of the conflicting cycle test week. Display the current scope setting
  (e.g. "School-wide block: no assessments may be booked during
  Grade 10 Mid-Year Exams (3–14 Nov 2026)").
  If ALLOW_ADMIN_OVERRIDE is true, show override button for Admin/Management.

- **Warning (not a block):** If WARN_BEFORE_CYCLE_TEST > 0 in AppSettings
  and the proposed date falls within that many school days before a formal
  period, show a yellow warning banner in the booking form but allow saving.
  Example: "Warning: This date is 3 school days before Grade 10 Mid-Year
  Exams begin. Consider whether this is appropriate."

- **Rule values displayed in UI:** The ConflictWarning modal must always
  show the current rule value so teachers understand why they are blocked.
  Never show a generic "booking not allowed" message.

# ✏️ ADDITIONAL RULES — add any school-specific rules here:
# Examples you might want to add:
# - No assessments on the day BEFORE a formal assessment
# - No more than 3 assessments in one week for a class
# - No assessments in the first week of term
# - Certain subjects (e.g. Physical Education) are exempt from the daily limit
#
# For each additional rule, describe it here and Claude Code will
# add a corresponding function to conflictRules.js

# ══════════════════════════════════════════════════════════════════
# SECTION 5b: UI/UX DESIGN SYSTEM
# ══════════════════════════════════════════════════════════════════
# Claude Code must apply the frontend-design skill to every component.
# This section defines the complete visual identity for the application.
# Every component must follow these specifications exactly.
# Generic AI aesthetics are strictly prohibited.

## Design Direction: "Academic Precision"

**Concept:** The visual language of a well-run school — structured,
trustworthy, and quietly confident. Think high-quality printed timetables,
academic diaries, and museum signage. Not corporate SaaS. Not consumer app.

**Tone:** Refined utilitarian. Purposeful density. Institutional authority
softened with warmth. Every element earns its place.

**The one unforgettable thing:** A custom calendar grid that feels like
a physical planning board — tactile, spatial, immediately scannable.

---

## Typography

```css
/* Display font — headings, labels, navigation */
@import url('https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&display=swap');

/* Body font — body text, form fields, data */
@import url('https://fonts.googleapis.com/css2?family=DM+Mono:wght@300;400;500&display=swap');

/* UI font — buttons, badges, small labels */
@import url('https://fonts.googleapis.com/css2?family=Outfit:wght@400;500;600;700&display=swap');

:root {
  --font-display: 'DM Serif Display', Georgia, serif;
  --font-body:    'DM Mono', 'Courier New', monospace;
  --font-ui:      'Outfit', system-ui, sans-serif;
}
```

- **Page titles and screen headers:** DM Serif Display, 28–36px
- **Section headings (H2):** DM Serif Display italic, 20–24px
- **Navigation items, button labels:** Outfit SemiBold, 13–15px
- **Form labels, table headers:** Outfit Medium, 12px, letter-spacing 0.06em, uppercase
- **Body text, notes, descriptions:** Outfit Regular, 14px
- **Dates, codes, IDs, data values:** DM Mono Regular, 13px

---

## Colour Palette

```css
:root {
  /* ── Core ─────────────────────────── */
  --ink:          #1C1917;   /* near-black — primary text */
  --ink-muted:    #57534E;   /* warm grey — secondary text */
  --ink-faint:    #A8A29E;   /* light grey — placeholder, disabled */
  --paper:        #FAFAF9;   /* warm off-white — page background */
  --surface:      #FFFFFF;   /* card and panel background */
  --border:       #E7E5E4;   /* subtle dividers */
  --border-strong:#D6D3D1;   /* prominent borders */

  /* ── Brand ────────────────────────── */
  --cobalt:       #1D4ED8;   /* primary action — buttons, links, focus */
  --cobalt-light: #DBEAFE;   /* cobalt tint — hover states, highlights */
  --cobalt-dark:  #1E3A8A;   /* cobalt pressed state */

  /* ── Semantic ─────────────────────── */
  --amber:        #D97706;   /* study required, warnings */
  --amber-light:  #FEF3C7;   /* amber tint */
  --jade:         #059669;   /* success, no study required */
  --jade-light:   #D1FAE5;   /* jade tint */
  --crimson:      #DC2626;   /* errors, blocked dates, cycle test weeks */
  --crimson-light:#FEE2E2;   /* crimson tint */
  --slate:        #475569;   /* neutral badges, secondary actions */
  --slate-light:  #F1F5F9;   /* slate tint */

  /* ── Calendar date types ──────────── */
  --date-valid:       #FFFFFF;   /* bookable — white */
  --date-saturday:    #EFF6FF;   /* valid Saturday — pale blue */
  --date-sunday:      #F5F5F4;   /* always blocked — warm grey */
  --date-holiday:     #FFF7ED;   /* public holiday — warm peach */
  --date-cycle-test:      #FEF2F2;   /* cycle test week — pale rose */
  --date-outside:     #F5F5F4;   /* outside school calendar — grey */
  --date-today:       var(--cobalt-light); /* today highlight */

  /* ── Study requirement ────────────── */
  --study-required:   var(--amber);
  --study-none:       var(--jade);

  /* ── Elevation ────────────────────── */
  --shadow-sm:   0 1px 2px rgba(0,0,0,0.06);
  --shadow-md:   0 4px 12px rgba(0,0,0,0.08);
  --shadow-lg:   0 8px 24px rgba(0,0,0,0.10);
  --shadow-panel:0 0 0 1px var(--border), 0 4px 16px rgba(0,0,0,0.07);
}
```

---

## Spacing & Layout

```css
:root {
  --radius-sm:  4px;
  --radius-md:  8px;
  --radius-lg:  12px;
  --radius-xl:  16px;

  --space-1:  4px;
  --space-2:  8px;
  --space-3:  12px;
  --space-4:  16px;
  --space-5:  20px;
  --space-6:  24px;
  --space-8:  32px;
  --space-10: 40px;
  --space-12: 48px;
}
```

---

## Motion & Animation

```css
:root {
  --ease-out:    cubic-bezier(0.0, 0, 0.2, 1);
  --ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1);  /* slight overshoot */
  --duration-fast:   120ms;
  --duration-normal: 220ms;
  --duration-slow:   360ms;
}

/* Standard transition applied to all interactive elements */
.interactive {
  transition: all var(--duration-fast) var(--ease-out);
}
```

**Animation principles:**
- Page load: staggered fade-up of main panels (40px translateY → 0, opacity 0 → 1)
- BookingForm slide-in: 300ms ease-out from right (translateX 100% → 0)
- Grade switcher: crossfade calendar and grid content (200ms opacity)
- Badge hover: scale(1.04) with shadow increase
- Toggle (StudyRequired): smooth thumb slide with colour transition
- Row hover in grid: subtle background fill
- Date picker blocked dates: short shake animation on click attempt (3×4px)

**Use `@media (prefers-reduced-motion: reduce)` to disable all animations.**

---

## Component Specifications

### NavBar
- Full-width top bar, height 56px
- Background: `var(--ink)` (near-black)
- School name/logo: DM Serif Display, white, left-aligned
- Nav items: Outfit SemiBold 13px, white at 70% opacity, active = 100%
- Underline indicator on active item: 2px cobalt
- Teacher name + My Profile link: right-aligned, Outfit 13px

### Grade Switcher Tabs
- Pill-style tabs (not underline)
- Inactive: `var(--border)` border, `var(--paper)` background, ink-muted text
- Active: `var(--cobalt)` background, white text, no border
- Transition: background 150ms ease-out
- Tab label: Outfit SemiBold 14px, grade name only (e.g. "Grade 10")

### Calendar (react-big-calendar)
- Override all default react-big-calendar styles
- Month grid cells: 2px `var(--border)` borders
- Today cell: `var(--cobalt-light)` background, cobalt date number
- Colour-coded blocked dates per the `--date-*` variables above
- Assessment event pills: rounded 4px, subject colour background,
  white text, DM Mono 11px, 📚 icon prepended if StudyRequired
- No event dot indicators — full pill labels only
- Month/Week/Agenda switcher: Outfit 13px pill tabs, top-right of calendar

### Term Grid
- Table with sticky first column (class group name)
- Header row background: `var(--ink)` with white DM Mono text
- Week column headers: DM Mono 12px, date range in ink-muted below
- Class group cells (first column): Outfit Medium 13px, `var(--slate-light)` bg
- Assessment badges: rounded 4px, 11px Outfit SemiBold subject abbreviation
  - Normal badge: subject colour background, white text
  - StudyRequired badge: amber top border 2px, slightly warm background
  - No-study badge: standard colour, no special border
- Empty cells: `var(--paper)` bg, subtle dot pattern texture
- Totals row: `var(--slate-light)` bg, bold count, DM Mono

### BookingForm (slide-in panel)
- Width: 380px (fixed — desktop only app)
- Background: `var(--surface)` with `var(--shadow-lg)`
- Left border: 3px `var(--cobalt)` accent line
- Section headings: Outfit Medium 11px uppercase letter-spaced labels
- Input fields: 1px `var(--border)` border, `var(--radius-md)` corners,
  focus ring: 2px `var(--cobalt)` offset 2px
- StudyRequired toggle:
  - ON (Requires Study): amber thumb, amber track tint, 📚 label
  - OFF (No Study): jade thumb, jade track tint, ✓ label
  - Locked state: grey thumb, lock icon, amber "Formal assessment" banner
- Date picker: custom styled, uses `--date-*` colours for cell backgrounds
- Save button: full-width, `var(--cobalt)` bg, white Outfit SemiBold 14px,
  height 44px, `var(--radius-md)`, hover: `var(--cobalt-dark)`

### StudyRequired Toggle (standalone component)

```jsx
// StudyRequiredToggle.jsx
// Props: value ("Requires Study" | "No Study Required")
//        onChange: Function
//        locked: boolean
//        lockReason: string | null

// Locked state renders:
// [🔒 AMBER BANNER] "Formal assessment — study required"
// [greyed-out toggle in ON position]

// Unlocked ON state:
// [📚 Requires Study]  ●────  (amber)

// Unlocked OFF state:
// ────●  [✓ No Study Required]  (jade)
```

### Assessment List Table
- Alternating row backgrounds: white / `var(--paper)`
- StudyRequired pill:
  - "Requires Study" → amber bg `var(--amber-light)`, amber text, 📚 icon
  - "No Study Required" → jade bg `var(--jade-light)`, jade text, ✓ icon
- Formal assessment indicator: small 🔒 icon next to amber pill

### ConflictWarning Modal
- Centred overlay with `var(--shadow-lg)` and backdrop blur
- Header: DM Serif Display 22px, crimson colour
- Rule description: DM Mono 13px in `var(--slate-light)` box
- "Go Back" button: outline style, ink colour
- Override button (admin only): solid crimson

### Empty State
- Illustrated with a simple SVG of a blank calendar grid
- DM Serif Display italic 20px heading
- Outfit 14px body text, ink-muted
- "Book first assessment" CTA button

---

## Tailwind Configuration

Claude Code must configure Tailwind to use the CSS variables above
by extending the theme in `tailwind.config.js`. All component code
must use Tailwind classes that reference these variables, not hardcoded
hex colours. Where Tailwind doesn't support a pattern, use inline
`style={}` props referencing the CSS variables.

---

## File to create: src/styles/design-system.css

Claude Code must create a global CSS file imported in main.jsx that:
1. Imports the three Google Fonts
2. Defines all CSS custom properties from the palette above
3. Sets base body styles: font-family, background-color, color
4. Provides utility classes for StudyRequired display:
   `.study-required` and `.study-none`
5. Provides the animation keyframes and transition utilities

# ══════════════════════════════════════════════════════════════════
# SECTION 6: USER INTERFACE REQUIREMENTS
# ══════════════════════════════════════════════════════════════════

## TeacherProfile.jsx — New Screen (all teachers)

# Every teacher must complete their profile before using the app.
# The profile stores which grades, class groups, and subjects
# the teacher is responsible for. This drives:
# - Which grade calendar opens by default on login
# - Which grades appear in the grade switcher
# - Which class groups are available in the BookingForm Class/Group dropdown
#   (scoped to the teacher's own classes, not the entire school list)
#
# A teacher who has not yet set up their profile is redirected to
# TeacherProfile on first login, with a message:
# "Before you can book assessments, please set up your profile."

### TeacherProfile screen layout

**Section 1 — Grades I teach**
- Multi-select checkboxes: Grade 8 | Grade 9 | Grade 10 | Grade 11 | Grade 12
- At least one grade must be selected

**Section 2 — My classes (per selected grade)**
- For each selected grade, a multi-select list of class groups
  from ReferenceData (Category: ClassGroup), filtered to that grade
- Example: Grade 10 selected → shows Grade 10A, 10B, 10C, 10D
- Teacher ticks only the classes they personally teach
- A teacher teaching multiple subjects to the same class ticks that
  class once — subjects are handled separately

**Section 3 — Subjects I teach**
- Multi-select from ReferenceData (Category: Subject)
- These populate the Subject dropdown in the BookingForm
- Only the teacher's own subjects are shown in the dropdown
  (not the full school list)

**Section 4 — Default grade view**
- If teacher teaches multiple grades: a radio button to select
  which grade calendar opens by default on login
- If teacher teaches one grade only: auto-set, no radio needed

**Save Profile button**
- Validates at least one grade, one class group, and one subject selected
- Saves to TeacherProfiles SharePoint list (List 9 — see below)
- After save, redirects to the CalendarView for their default grade

**Edit Profile link**
- Available in the NavBar at all times (small link next to teacher's name)
- Same form, pre-filled with existing values

---

## useTeacherProfile.js hook

```javascript
/**
 * Fetches and caches the current teacher's profile from TeacherProfiles list.
 * On first login (profile not yet created), returns profileComplete: false.
 *
 * @returns {Object} {
 *   profile: {
 *     gradesTaught: string[],    // e.g. ["Grade 10", "Grade 11", "Grade 12"]
 *     classesTaught: string[],   // e.g. ["Grade 10A", "Grade 10B", "Grade 11A", "Grade 12A"]
 *     subjectsTaught: string[],  // e.g. ["Mathematics", "English"]
 *     defaultGrade: string,      // e.g. "Grade 10"
 *   },
 *   profileComplete: boolean,
 *   isLoading: boolean,
 *   saveProfile: Function,
 * }
 */
export function useTeacherProfile()
```

---

## CalendarView.jsx — Main Teacher Screen (Split Layout)

# CORE ARCHITECTURE: Each grade has its own separate calendar.
# The calendar scope is GRADE-LEVEL — a teacher opening the
# Grade 10 calendar sees ALL Grade 10 assessments from ALL
# subjects and ALL class groups (10A, 10B, 10C, 10D).
#
# TERM-SCOPED CALENDAR:
# The calendar is limited to the CURRENT TERM only.
# Because assessments are archived term by term, there is no reason
# to show dates outside the active term window. This keeps the
# interface focused and prevents teachers accidentally booking
# into a term that has not started or has already been archived.
#
# "Current term" = the SchoolCalendar entry whose StartDate <= today <= EndDate.
# If today falls in a holiday gap between terms, a friendly state
# is shown instead of an empty calendar (see Edge Cases below).
#
# Teachers who teach multiple grades have a GRADE SWITCHER
# in the header. Each grade's calendar is a completely independent view.
#
# The screen uses a SPLIT LAYOUT:
# Left panel (60%): term-scoped grade calendar + booking form slide-in
# Right panel (40%): full-term assessment grid — always visible
#
# DESKTOP ONLY: minimum supported viewport is 1280px wide.
# No responsive behaviour is required. If the browser window is
# narrower than 1280px, the layout may scroll horizontally — this
# is acceptable. Do not add @media breakpoints or mobile layouts.
# A "please use a desktop browser" message is NOT needed — school
# staff access this from a school PC or laptop.

### Grade Switcher (top of screen, above both panels)

- Shown only to teachers who teach multiple grades
- Pill/tab buttons: one per grade the teacher teaches
  e.g. [Grade 10] [Grade 11] [Grade 12]  — active grade is highlighted
- Switching grade replaces BOTH panels with that grade's data
- Returns to default grade on next login
- Teachers who teach only one grade: no switcher shown

### Left Panel — Term-Scoped Grade Calendar

**Scope:**
- Shows ALL assessments for the currently selected grade
  within the CURRENT TERM ONLY (StartDate to EndDate from SchoolCalendar)
- All dates outside the term window — even within the displayed month —
  are greyed out and unclickable
- Calendar opens on the month containing today by default
  (or the first month of the term if today is before the term starts)

**Month navigation (constrained to current term):**
- Back arrow (<): disabled when the displayed month is the same
  month as the term StartDate — cannot navigate before the term
- Forward arrow (>): disabled when the displayed month is the same
  month as the term EndDate — cannot navigate past the term
- If the entire term fits within a single month, both arrows are disabled
- Month label shows the term name:
  e.g. "March 2026  .  Term 1" so the teacher always knows which term

**Colour coding:**
- Dates within the term: white (bookable) or coloured per rules below
- Dates outside the term within the displayed month: grey, no cursor
- Public holidays within the term: orange with holiday name on hover
- Cycle test weeks: red
- Sundays: grey
- Valid Saturdays within the term: pale blue
- Today: cobalt blue highlight

**Other behaviours:**
- Click a valid day to open BookingForm (slides in over left panel)
- Click an existing assessment badge to view or edit it
- Teachers can only EDIT or CANCEL their own assessments —
  other teachers' assessments are read-only (lock icon on hover)
- Today button: jumps to the month containing today within the term

### Login Notification Banner (shown on first screen after login)

When a teacher logs in, a notification banner appears below the nav bar
showing the cycle test confirmation status for each grade they teach:

```
┌─────────────────────────────────────────────────────────────────┐
│  Cycle test schedule status for your grades:                     │
│  ✅ Grade 10 — confirmed, booking open                          │
│  ⚠️  Grade 11 — not yet confirmed, booking blocked              │
│  ✅ Grade 12 — confirmed, booking open                          │
│                                          [Dismiss for this session] │
└─────────────────────────────────────────────────────────────────┘
```

- Green row = confirmed, teacher can book for that grade
- Amber row = not confirmed, booking blocked for that grade
- Banner is dismissible for the session but reappears on next login
- If ALL grades are confirmed, banner shows a single green line:
  "✅ All your cycle tests are confirmed — booking is open"
- If ALL grades are confirmed, banner auto-dismisses after 5 seconds

**Management NavBar badge:**
- A red dot appears on the "Cycle Tests" nav item when any grade in
  the current term has unconfirmed or stale cycle tests
- Cleared when all grades are confirmed

**Edge Case 1: Today is between terms (school holiday)**
When today does not fall within any active SchoolCalendar term window:
- Display an informational state in the calendar area instead of a grid
- Message: "School holidays — no active assessment term."
- Sub-message: "Term [X] opens on [date]." (using the next SchoolCalendar entry)
- Do NOT show an empty calendar or an error screen

**Edge Case 2: No term windows configured**
When the SchoolCalendar list is empty:
- Show a banner in the calendar area:
  "No term windows have been configured yet.
   Please ask Management to set up the school calendar."
- This is an expected first-run state — not an error

**Edge Case 3: Term has ended but not yet archived**
When today is past the most recent term EndDate but archive not yet run:
- Teachers still see the calendar for the most recently ended term
  so no data is hidden while Management completes the archive
- A subtle amber banner: "This term has ended. Awaiting archive by management."
- The ArchiveBanner continues to prompt Management separately

**Edge Case 4: Multiple overlapping term windows**
When SchoolCalendar entries overlap on the same date:
- Use the entry with the latest StartDate
- Log a console warning for the developer to flag to Management

### Right Panel — Grade Term Assessment Grid

# Rows = all class groups in the currently selected grade
# Columns = ALL weeks of the current term (full term, not just visible months)
# Cells = assessment badges for that class group in that week
# Scope = ALL assessments for ALL subjects in the selected grade

**Grid header:**
- Term label above the grid:
  "Term 1 2026  .  14 Jan - 27 Mar  .  11 weeks"
- No term selector dropdown — teachers see current term only
  (Historical terms accessible via Assessment List / My Bookings)
- Current week column highlighted with subtle cobalt left border
- Export button top right

**Grid column headers:**
- One column per week of the current term
- Public holidays within a week shown as a small orange dot

**Grid body rows (one per class group in the selected grade):**
- Class group name in first column (e.g. "10A", "10B", "10C")
- Each week cell shows:
  - A coloured badge per assessment (type icon + subject abbreviation)
  - If 2 assessments in the week cell: show both badges
  - If cell would exceed daily limit (Rule 1): red warning icon
  - Empty cells: light grey

**Interaction:**
- Hover badge: tooltip with full assessment name, date, teacher, type
- Click own badge: opens BookingForm in edit mode
- Click another teacher's badge: read-only detail popover
- Click empty cell in own class row: opens BookingForm pre-filled
- Click empty cell in class not taught: tooltip "You do not teach [10B]"

**Grid footer:**
- Total row: assessment count per week across all class groups

**Export:**
- Exports current grade/term grid as Excel
- Filename: Grade10_Term1_2026_Grid.xlsx

### BookingForm.jsx (slide-in within left panel)

Slides in from the right side of the left panel when triggered.
Term grid remains visible behind the form.

**Grade-aware field scoping:**
- Class/Group: **multi-select checkbox list** showing ONLY the classes
  the teacher teaches in the currently selected grade (from TeacherProfile).
  Teacher can tick one or multiple classes (e.g. 10A + 10B + 10C).
  One assessment record is created per selected class on save.
  The conflict check runs independently for each selected class.
  If one class has a conflict, that class is skipped and a warning shown —
  the other classes save successfully.
  Example warning: "10A was skipped — already has 2 assessments on this day.
  10B and 10C were saved successfully."
- Subject dropdown: shows ONLY the subjects from the teacher's profile
- Date picker: constrained to the current term window only —
  dates outside the term are greyed and unselectable
- Term: auto-populated from the current term — not editable by teacher
- Notes (optional): free-text field, maximum 300 characters.
  Show a live character counter below the field (e.g. "142 / 300").
  Counter turns amber at 250+, red at 280+.
  Save button is disabled if the note exceeds 300 characters.
  validateBooking() also enforces the 300-character limit server-side
  as a safety check before the Graph API write.

**Cycle test confirmation gate (checked before anything else):**

Before the teacher can interact with the booking form at all, the app
checks whether the selected grade's cycle tests have been confirmed
for the current term in CycleTestConfirmations.

If NOT confirmed:
- Date picker is disabled (greyed out)
- Class/Group multi-select is disabled
- Save button is disabled
- A prominent banner appears at the top of the form:
  "🔒 Cycle tests for Grade 10 have not been confirmed yet.
   Dates may still change. Contact management to confirm
   before bookings open for this grade."
- The banner links to the CycleTestManager for Admins/Management

If confirmed but STALE (cycle test changed after confirmation):
- Same locked state as above
- Banner message: "⚠️ The Grade 10 cycle test schedule has changed
   since it was last confirmed. Management must re-confirm before
   bookings can continue."

Admins and Management are NEVER blocked by this gate — they can
always book regardless of confirmation status.

**Live daily limit indicator (fires the moment Class/Group AND Date are both selected):**

This is the primary conflict prevention mechanism. It surfaces the daily limit
check BEFORE save — so teachers see the conflict while choosing a date,
not as a surprise hard block after clicking Save.

The indicator appears directly below the Date field and updates instantly
whenever either Class/Group or Date changes. Reads from assessments already
in memory — no additional API call needed.

Three states based on how many assessments that class already has on that day:

State 1 — No bookings (0 of 2 slots used):
  Green border and background.
  Message: "No assessments booked for [class] on [date]"

State 2 — One booking (1 of 2 slots used):
  Amber border and background.
  Message: "1 of 2 slots used for [class] on [date]"
  Shows the existing booking: type icon + abbreviation + subject + teacher name
  Save button remains ENABLED — one slot is still free.

State 3 — Full (2 of 2 slots used — save DISABLED for teachers):
  Red border and background.
  Message: "[class] already has 2 assessments on [date] — choose a different date"
  Shows both existing bookings: type icon + abbreviation + subject + teacher name
  Save button DISABLED for teachers.
  Save button ENABLED for Admins and Management (override is logged).
  Override banner: "Override mode — saving despite the daily limit. This will be logged."

Example — State 2 indicator layout:
  ┌──────────────────────────────────────────────────────────┐
  │ ⚠  1 of 2 slots used for 10A on Thu 5 Mar               │
  │   ✏️ CT  MA  Mathematics Cycle Test  (Mr C. Uren)         │
  └──────────────────────────────────────────────────────────┘

Example — State 3 indicator layout:
  ┌──────────────────────────────────────────────────────────┐
  │ ✗  10A already has 2 assessments on Thu 19 Mar — full    │
  │   🔬 PR  SC  Physical Sciences Practical  (Mr C. Uren)   │
  │   📎 AS  EN  English Assignment           (Ms S. Dlamini)│
  │   Choose a different date to continue.                   │
  └──────────────────────────────────────────────────────────┘

**Cycle test day indicator (fires when Date is selected):**

Separate indicator shown above the daily limit indicator.
Checks whether the selected date is a cycle test day for the selected grade.

When a cycle test day is selected:
  Rose background, crimson border.
  Message: "Cycle test day — [Grade] [Subject] Cycle Test. No new bookings on this day.
            You can book on other days this week."
  Study Required field is automatically locked to "Requires Study".
  Save button DISABLED for teachers.
  Save button ENABLED for Admins and Management (override is logged).

**Live grid preview:**
- As teacher fills in Class/Group and Date, the target cell in the
  term grid highlights yellow — teacher sees where the assessment lands

**Read-only indicator:**
- If teacher clicks someone else's assessment, the form opens in
  read-only mode with a banner:
  "This assessment belongs to [Teacher Name]. You can view but not edit it."

**Confirmation lock indicator:**
- If the teacher's schedule for this grade has been confirmed, the form
  opens with: "This schedule has been confirmed. Reopen to make changes."
  Save button disabled until teacher reopens their confirmation.

**Save button state summary:**

| Condition                      | Teacher       | Admin / Management        |
|-------------------------------|---------------|---------------------------|
| All valid, no conflicts        | Enabled       | Enabled                   |
| 1 of 2 slots used (State 2)   | Enabled       | Enabled                   |
| 2 of 2 slots used (State 3)   | DISABLED      | Enabled — logged override |
| Cycle test day selected        | DISABLED      | Enabled — logged override |
| Notes > 300 characters         | DISABLED      | DISABLED                  |
| Schedule confirmed             | DISABLED      | Enabled                   |
| Required field missing         | DISABLED      | DISABLED                  |

After a successful save:
- BookingForm slides closed
- Both calendar and term grid update immediately
- No page refresh

### Screen Layout Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│  [Grade 10] [Grade 11] [Grade 12]  <- Grade Switcher            │
├─────────────────────────────────┬───────────────────────────────┤
│  LEFT PANEL (60%)               │  RIGHT PANEL (40%)            │
│                                 │                               │
│  March 2026 . Term 1   [Today]  │  Term 1 . 14 Jan-27 Mar  [↓] │
│  < (disabled)    > (active)     │                               │
│  ┌─────────────────────────┐    │  ┌─────────────────────────┐ │
│  │  Current term dates     │    │  │Class│Wk1│Wk2│Wk3│Total │ │
│  │  only — grey outside    │    │  │ 10A │   │MA │SC │  2   │ │
│  │  the term window        │    │  │ 10B │EN │   │MA │  2   │ │
│  │                         │    │  │ 10C │   │MA │EN │  2   │ │
│  └─────────────────────────┘    │  │Total│ 1 │ 2 │ 3 │      │ │
│                                 │  └─────────────────────────┘ │
└─────────────────────────────────┴───────────────────────────────┘
```

### New component: TermGrid.jsx

Claude Code must implement TermGrid.jsx as a separate component.
Props: `assessments`, `term`, `grade`, `teacherProfile`,
       `schoolCalendar`, `publicHolidays`, `onCellClick`, `onBadgeClick`

### useTermGrid.js hook

```javascript
/**
 * Resolves the current active term from the SchoolCalendar list.
 * Called on app load and whenever SchoolCalendar changes.
 *
 * Returns one of four states:
 *   { state: 'active',    term: {...} }  - today is within a term
 *   { state: 'holiday',   nextTerm: {...} }  - today is between terms
 *   { state: 'ended',     lastTerm: {...} }  - last term ended, not archived
 *   { state: 'unconfigured' }  - SchoolCalendar list is empty
 */
export function getCurrentTerm(schoolCalendar)

/**
 * Computes the term grid data from raw assessments, scoped to one grade
 * and the CURRENT TERM ONLY. No term selector — always current term.
 *
 * @param {Array}  assessments    - All assessments fetched from SharePoint
 * @param {Object} currentTerm    - { title, startDate, endDate } from getCurrentTerm()
 * @param {string} grade          - e.g. "Grade 10" — filters rows and cells
 * @param {Array}  publicHolidays - For marking holiday weeks in column headers
 * @param {Object} teacherProfile - Used to determine which badges are editable
 * @returns {Object} {
 *   weeks:   Array<{ label, startDate, endDate, hasHoliday, isCurrent }>,
 *   rows:    Array<{ classGroup, cells: Array<{ assessments, isOverLimit }> }>,
 *   totals:  Array<number>,
 *   termLabel: string,  // e.g. "Term 1 2026  ·  14 Jan – 27 Mar  ·  11 weeks"
 *   exportGrid: Function
 * }
 */
export function useTermGrid(assessments, currentTerm, grade, publicHolidays, teacherProfile)
```

## SchoolCalendarManager.jsx (Management Team only)

# This screen allows the Management Team to define which date ranges
# are open for assessment booking. It works alongside the public
# holiday list to determine which dates the date picker shows as
# available (white) vs blocked (greyed out).

The School Calendar Manager must be a dedicated full-page screen with:

### Term Window Table

Columns: Term | Start Date | End Date | Saturdays Included | Notes | Actions

- Rows sorted by StartDate ascending
- Edit and Delete buttons per row
- Delete only allowed if no assessments exist within that window

### Add / Edit Term Window Form

Slide-in panel with:
- Term (dropdown from ReferenceData Category: Term)
- Year (number — defaults to current year)
- Start Date (date picker)
- End Date (date picker — must be after Start Date)
- Include Saturdays toggle (default: Yes)
- Notes field
- Preview: after selecting dates, show a summary:
  "This window contains X weekdays and Y Saturdays = Z total bookable days
   (excluding X public holidays that fall within this range)"

### Public Holidays Tab

A second tab within SchoolCalendarManager showing:
- All public holidays grouped by year
- Current year and next year pre-loaded
- "Add Future Year" button — when clicked, Claude Code's
  calculateEasterDates() auto-populates Good Friday and Family Day,
  and the management user confirms and adds remaining fixed holidays
- Each holiday shows: Date | Day of Week | Holiday Name | Substitute?
- Saturdays that are public holidays are visually highlighted
  (these are doubly blocked — public holiday AND would-be Saturday)

### Calendar Date Picker Behaviour (updated)

The date picker in BookingForm.jsx must colour-code dates as follows:

| Date Type | Colour | Clickable? |
|-----------|--------|------------|
| Valid assessment day (Mon–Fri, in school calendar, not blocked) | White | Yes |
| Valid Saturday (in school calendar, not a public holiday) | Light blue | Yes |
| Sunday | Grey, strikethrough | No |
| Public holiday | Orange with holiday name tooltip | No |
| Cycle test week | Red with period name tooltip | No |
| Outside school calendar window | Grey | No |
| Today | Outlined in blue | Yes (if otherwise valid) |

## PolicyManager.jsx (Management Team only)

# This is the dedicated screen for editing Rule 1 and Rule 2 settings.
# It reads from and writes to the AppSettings SharePoint list.
# Only the Management Team can access it — changes take effect immediately
# for all users without any app restart or redeployment.

The Policy Manager must be a dedicated full-page screen with:

### Layout

Two clearly separated sections — one per configurable rule.

### Section A: Daily Assessment Limit (Rule 1)

Displays the current Rule 1 configuration with:

**Maximum Assessments Per Day**
- Current value shown prominently (e.g. large "2")
- Increment / decrement buttons (+ and −)
- Minimum: 1, Maximum: 5 (enforced in the UI — cannot type outside this range)
- Description text pulled from AppSettings.Description field
- Current default shown: "Default: 2"
- "Reset to Default" button

**Daily Limit Scope**
- Toggle switch with two options: "Per Class Group" | "School-Wide"
- Clear explanation of each option shown below the toggle:
  - Per Class Group: "Grade 10A cannot have more than [X] assessments on one day.
    Other classes are unaffected."
  - School-Wide: "The entire school cannot have more than [X] assessments
    on any single day across all classes."
- Current selection highlighted

**Impact Preview**
After changing either value, show a live preview banner:
"With this setting, on a day where Grade 10A already has 2 assessments,
 a new booking for Grade 10A would be [BLOCKED / ALLOWED]."

### Section B: Formal Assessment Date Scope (Rule 2)

**Cycle Test Day Block Scope**
- Radio buttons:
  - "School-Wide Block (Recommended)" — no class can book on a cycle test day
  - "Class-Only Block" — only the class with the formal assessment is blocked
- Description of each option with example
- Current default shown: "Default: School-Wide Block"
- "Reset to Default" button

**Warning Period Before Cycle Test Day**
- Number input: "Show warning [  ] school days before a cycle test week begins"
- Set to 0 to disable warnings entirely
- Helper text: "Teachers will see a yellow warning but will still be able to save."
- Preview: "Currently set to [X] days. Teachers booking within [X] school days
  of a cycle test week will see a warning."

**Allow Admin Override**
- Toggle switch: "Allow Administrators to override rule violations"
- When ON: Admins and Management can save blocked bookings (override is logged)
- When OFF: No one can override any rule — blocks are absolute
- Warning banner shown when toggling OFF:
  "Disabling overrides means no one can book past a rule violation,
   including Management. Are you sure?"

### Save & Audit

- **Save Changes** button — saves all changes to AppSettings list in one operation,
  writes a row to ReferenceDataAuditLog with old values, new values, who changed
  them, and when
- **Cancel** button — reverts unsaved changes
- **Change History** tab — shows audit log entries for AppSettings changes only,
  displaying: Changed By | Date | Setting | Old Value | New Value

### Important: No Retroactive Effect

When a rule value is changed, add a clear notice:
"Rule changes apply to new bookings only. Existing assessments that were
 valid under the previous rules are not affected or removed."

## CycleTestManager.jsx (Admin + Management Team only)

# Admins and Management enter individual cycle test dates here.
# Each entry is ONE test on ONE specific date for ONE grade.
# Head office exams are NOT entered here — only school-run cycle tests.
#
# CRITICAL WORKFLOW:
# Teachers CANNOT book assessments for a grade until Management has
# confirmed the cycle test schedule for that grade.
# This ensures teachers plan around finalised cycle test dates.
#
# The screen is organised by grade — each grade has its own section
# showing its cycle tests and a confirmation button at the bottom.

### Layout

**Info banner at top:**
> ℹ️ Enter each cycle test individually. Teachers can still book
> assignments, practicals, and other assessments on other days in
> the same week. Only the specific date entered here is blocked.
> Confirm each grade's cycle tests when the schedule is finalised
> to open bookings for that grade.

**One collapsible section per grade (Grade 8 through Grade 12):**

```
GRADE 10                                          [+ Add Cycle Test]
──────────────────────────────────────────────────────────────────
  DATE          SUBJECT           NOTES           ACTIONS
  Thu 5 Mar     Mathematics       —               Edit  Delete
  Thu 12 Mar    Physical Sciences —               Edit  Delete

  ✓ Confirmed on Thu 5 Mar 2026 at 09:14 by Mr D. van Wyk
  [Reopen Grade 10 Cycle Tests]  ← secondary button

──────────────────────────────────────────────────────────────────
GRADE 11                                          [+ Add Cycle Test]
──────────────────────────────────────────────────────────────────
  DATE          SUBJECT           NOTES           ACTIONS
  Fri 6 Mar     Mathematics       —               Edit  Delete

  ⚠ Not yet confirmed — teachers cannot book Grade 11 assessments
  [✓ Confirm Grade 11 Cycle Tests]  ← primary cobalt button
──────────────────────────────────────────────────────────────────
```

**Confirmation button states per grade:**

Not confirmed:
  Primary blue button: "✓ Confirm Grade X Cycle Tests"
  Warning text: "Teachers cannot book until this is confirmed"

Confirmed:
  Green confirmed badge: "✓ Confirmed [date] by [name]"
  Secondary grey button: "Reopen Grade X Cycle Tests"

Stale (cycle test changed after confirmation):
  Amber badge: "⚠ Re-confirmation needed"
  Primary button: "✓ Re-confirm Grade X Cycle Tests"

**On confirm:**
- Creates/updates CycleTestConfirmations row for that grade + term
- IsConfirmed = true, ConfirmedAt = now, CycleTestCount = current count
- All teachers who teach that grade immediately see login notification:
  "✅ Grade X cycle tests confirmed — booking is now open"
- NavBar red dot for that grade is cleared

**On reopen (un-confirm):**
- IsConfirmed = false in CycleTestConfirmations
- Teachers immediately see: "⚠ Grade X cycle tests not yet confirmed"
- Booking is blocked again for that grade immediately

**On any cycle test edit/delete after confirmation:**
- IsStale = true automatically
- Grade section shows amber "Re-confirmation needed" badge
- Teachers see stale warning and booking is blocked

**Calendar View tab:**
- Shows all cycle test dates in the current term on a mini calendar
- Colour-coded by grade
- Confirmed grades shown with solid rose dates
- Unconfirmed grades shown with dashed rose border

**Changes take effect immediately for all users.**
**Only Admins and Management can access this screen.**

## ReferenceDataManager.jsx (Management Team only)

# This is a dedicated full-screen management interface for the
# ReferenceData SharePoint list. It replaces the "second tab" approach
# mentioned earlier. It is a proper screen in the navigation, visible
# only to users in the Management Team SharePoint Group.
#
# This screen is the only place where subjects, class groups, assessment
# types, terms, and statuses can be added, edited, or deactivated.
# Teachers have no access to this screen — they only see the values
# it produces in their dropdowns.

The ReferenceDataManager must be a dedicated full-page screen with:

### Layout

- **Left sidebar:** Category selector — clicking a category loads its values
  in the main panel. Categories: Subject | ClassGroup | AssessmentType |
  Term | Status
- **Main panel:** Table of all values in the selected category
- **Top right:** "Add New Value" button

### Main Table (per category)

Columns:
| # | Value | Active | Sort Order | Last Modified By | Last Modified Date | Actions |
|---|-------|--------|------------|-----------------|-------------------|---------|

- Rows sorted by SortOrder ascending
- Active column shows a toggle switch (green = active, grey = inactive)
- Toggling Active immediately updates SharePoint — no save button needed
- **Drag to reorder** rows to change SortOrder (use `@dnd-kit/core` package)
  Updates SortOrder values in SharePoint automatically after drop
- **Edit** (pencil icon) — inline edit of the Title field
- **Delete** (bin icon) — only allowed if the value has never been used
  in an Assessment record. If it has been used, show:
  "This value cannot be deleted because it appears in existing assessments.
   Set it to Inactive instead to hide it from future bookings."

### Add New Value Form

Slide-in panel with:
- Category (pre-selected from the sidebar, but changeable via dropdown)
- Value / Title (text input)
- Sort Order (number — defaults to last position + 1)
- Active (toggle — defaults to Yes)
- Save button — validates that the value does not already exist in
  that category (case-insensitive check) before saving

### Audit Log tab

A second tab within ReferenceDataManager showing a log of all changes:
- Who added/edited/deactivated a value
- What the old value was and what it changed to
- Timestamp
- Populated from a fourth SharePoint list: `ReferenceDataAuditLog`

# This audit requirement means Claude Code must also create:
# List 4: ReferenceDataAuditLog
# Columns: Title (auto), Category, OldValue, NewValue, Action
# (Added/Edited/Deactivated/Reactivated/Deleted), ChangedBy, ChangedDate
# Every write operation to ReferenceData must also write a row here.

### Bulk Import tab

A third tab allowing management to paste or upload a list of values:
- Text area: paste one value per line
- Category selector
- "Import" button — adds all new values, skips duplicates, reports results
- Useful for initial setup (paste the whole class list from a spreadsheet)

## AssessmentList.jsx

A table view (alternative to calendar) showing:

- All assessments in a sortable, filterable table
- Columns: Date | Class | Subject | Type | Teacher | Status
- Export button (see ExportButton.jsx)
- Useful for printing / planning meetings

## TermArchiveManager.jsx (Management Team only)

# This screen handles end-of-term archiving for List 1 (Assessments)
# and List 2 (CycleTests).
#
# WHY THIS IS NEEDED:
# Both lists change every term. Without archiving, they grow indefinitely,
# slowing the app and making the calendar cluttered with past data.
# The archive process exports both lists to Excel, stores the files in
# a SharePoint Document Library for permanent record-keeping, then
# clears the lists for the new term.
#
# WHEN IT TRIGGERS:
# The app checks on every Management Team login whether the current date
# is past the EndDate of the most recent SchoolCalendar term window.
# If so, and if that term has not yet been archived, a persistent banner
# appears at the top of every management screen:
# "Term [X] has ended. Archive and clear assessment data before the new
#  term begins." with a button linking to this screen.
#
# The banner cannot be dismissed — it remains until the archive is
# completed or explicitly snoozed (snooze lasts 24 hours).

The TermArchiveManager must be a dedicated full-page screen with:

### Step 1 — Review What Will Be Archived

Show a summary of the data about to be archived:

| Data | Count | Date Range |
|------|-------|------------|
| Assessments (List 1) | [X] bookings | [earliest date] to [latest date] |
| Formal Assessment Dates (List 2) | [X] periods | [earliest date] to [latest date] |

Display a preview table of the last 10 rows from each list so management
can verify the data looks correct before proceeding.

Include a warning banner:
"⚠️  This action cannot be undone. Once the lists are cleared, the data
 only exists in the archived Excel files. Verify the preview looks correct
 before proceeding."

### Step 2 — Name the Archive

- Archive Name field: pre-filled with e.g. "Term 2 2026 Archive"
  (auto-generated from the most recently ended SchoolCalendar term)
- Management user can edit this name
- Archive date: shown as today's date (read-only)
- Archived by: shown as logged-in user's name (read-only)

### Step 3 — Choose Archive Location

- Destination: SharePoint Document Library path
- Default: pre-filled with "Site Assets/AssessmentArchives/[Year]/"
- ✏️ Management can change the default destination path in AppSettings
  (add a setting: ARCHIVE_DESTINATION_PATH)
- The app creates the folder structure automatically if it doesn't exist

### Step 4 — Archive Button

One large "Archive and Clear" button with a confirmation dialog:

"You are about to:
 1. Export [X] assessments to Excel → [filename]
 2. Export [X] cycle test dates to Excel → [filename]
 3. Save both files to [destination path]
 4. Clear ALL rows from the Assessments list
 5. Clear ALL rows from the CycleTests list

 This cannot be undone. Type CONFIRM to proceed."

The user must type the word CONFIRM in a text box before the button
becomes active. This prevents accidental archiving.

### Archive File Format

Claude Code must generate two Excel files per archive operation.
Both files use the `xlsx` npm package.

**File 1: Assessments_[TermName]_[Year].xlsx**

Sheet 1 — "Assessments" — all columns from List 1:
| Date | Class Group | Subject | Assessment Type | Teacher Name | Teacher Email | Term | Status | Notes | Booked On |

Sheet 2 — "Summary" — pivot-style summary:
| Class Group | Total Assessments | By Subject breakdown | By Type breakdown |

**File 2: CycleTests_[TermName]_[Year].xlsx** (individual cycle test dates for the archived term)

Sheet 1 — "Formal Periods" — all columns from List 2:
| Period Name | Start Date | End Date | Affects Grades | Notes | Created By |

Sheet 2 — "Calendar View" — one row per blocked day (expanded from ranges):
| Date | Day of Week | Period Name |

### Archive Log

After a successful archive, write a row to a new SharePoint list:
**ArchiveLog** (List 8 — see below) recording:
- Archive name
- Term and year
- Files generated (names and SharePoint paths)
- Row counts (how many rows were exported from each list)
- Archived by
- Archive date and time
- Whether the clear operation succeeded

### Step 5 — After Archive Completes

Show a success screen with:
- Green confirmation banner: "Archive complete"
- Links to both Excel files in SharePoint (click to open or download)
- Summary: "[X] assessments and [X] cycle test dates archived to [path]"
- "View Archive Log" button
- "Return to Calendar" button

### Failed Archive Handling

If the Excel export succeeds but the SharePoint upload fails:
- Keep both Excel files available for manual download
- Do NOT clear the lists — only clear after confirmed successful upload
- Show error: "Files generated but could not be uploaded to SharePoint.
  Download the files below and upload manually before clearing the lists."
- Provide manual download buttons for both files
- Provide a "Retry Upload" button

If the upload succeeds but the list clear fails:
- Show error with exact SharePoint error message
- Provide a "Retry Clear" button
- Note: files are already safely archived — clearing can be retried safely

### Snooze Functionality

If management is not ready to archive yet:
- "Remind me tomorrow" button — suppresses the banner for 24 hours
- Snooze is stored in AppSettings as ARCHIVE_SNOOZED_UNTIL (Date/Time)
- Maximum 3 snoozes before the banner becomes un-dismissible
- Snooze count tracked in AppSettings as ARCHIVE_SNOOZE_COUNT

## ConfirmSchedule.jsx (all teachers — own grades only)

# This screen lets teachers review and confirm their assessment schedule
# for the current term, grade by grade.
#
# It is accessible from the navigation bar for all teachers.
# It shows a read-only summary of all their assessments grouped by grade.
# A teacher confirms per grade — one confirmation per grade per term.
#
# WHAT CONFIRMATION DOES:
# - Writes/updates a row in ConfirmationLog for that teacher + grade + term
# - Sets IsConfirmed = true and ConfirmedAt = now
# - Locks all Scheduled assessments for that teacher's classes in that grade
#   (assessments become read-only — cannot be edited or cancelled)
# - Shows a confirmed badge on the Calendar View for that grade
#
# STALE CONFIRMATION:
# If a new assessment is booked for a grade the teacher has already confirmed,
# IsStale is set to true on their ConfirmationLog row. The app shows an amber
# banner: "Your schedule has changed since you last confirmed. Please review
# and re-confirm." The teacher must un-confirm and re-confirm to lock again.

### Screen layout

**Header:**
"Confirm Assessment Schedule — Term 1 2026"

**One section per grade the teacher teaches:**

```
GRADE 10  ·  14 Jan – 27 Mar 2026
──────────────────────────────────────────────────────────
  ✏️  Thu 5 Mar    Mathematics Mid-Term Test       📚 Requires Study
                   Covers chapters 4–6. Bring calculator.
  📎  Fri 6 Mar    English Creative Writing         ✓ No Study Required
  🔬  Thu 5 Mar    Physical Sciences Practical      📚 Requires Study
  ⚡  Fri 20 Mar   History Quiz                     📚 Requires Study

  Total: 4 assessments across Grade 10A and 10B

  [✓ Confirm Grade 10 Schedule]   ← primary button, cobalt
──────────────────────────────────────────────────────────
GRADE 11  ·  14 Jan – 27 Mar 2026
  ...

  [✓ Confirm Grade 11 Schedule]
```

**Once a grade is confirmed, the section shows:**
```
GRADE 10  ·  ✓ Confirmed on Thu 5 Mar 2026 at 09:14
──────────────────────────────────────────────────────────
  [list of assessments — read-only, no changes possible]

  [Reopen Grade 10 Schedule]  ← secondary button, grey
  Reopening will unlock your assessments and reset your confirmation.
```

**Stale confirmation banner (amber):**
```
⚠️  Your Grade 10 schedule has changed since you confirmed.
    A new assessment was added on 12 Mar. Please review and re-confirm.
    [Re-confirm Grade 10]
```

**Empty state (no assessments booked yet):**
```
GRADE 10  ·  No assessments booked yet
──────────────────────────────────────
  You need at least 1 assessment before you can confirm.
  [Go to Calendar]
```

### Behaviour rules

- Teacher can confirm a grade at any time within the current term
- Teacher can un-confirm (reopen) at any time — this unlocks assessments
- Management can un-confirm on a teacher's behalf (from Management screens)
- Cancelled assessments are excluded from the confirmation count and list
- If a teacher has no assessments for a grade, confirm button is disabled

### useConfirmation.js hook

```javascript
/**
 * Fetches and manages confirmation state for the current teacher.
 *
 * @returns {Object} {
 *   confirmations: Array<{
 *     grade: string,
 *     term: string,
 *     isConfirmed: boolean,
 *     confirmedAt: string | null,
 *     assessmentCount: number,
 *     isStale: boolean,
 *   }>,
 *   confirmGrade: async (grade) => void,
 *   unconfirmGrade: async (grade, reason?) => void,
 *   isLoading: boolean,
 * }
 */
export function useConfirmation()
```

### isAssessmentLocked() helper

```javascript
/**
 * Returns true if an assessment is locked due to teacher confirmation.
 * Used by BookingForm and calendar badges to show lock icon.
 *
 * An assessment is locked when:
 * - The teacher who owns it has IsConfirmed = true in ConfirmationLog
 *   for the matching grade + term
 * - AND IsStale = false (stale = temporarily unlocked pending re-confirm)
 *
 * Admins and Management are NEVER blocked by confirmation lock —
 * they can always edit any assessment regardless of confirmation status.
 */
export function isAssessmentLocked(assessment, confirmations, userRole)
```

### Confirmation status badges on Calendar View

When a teacher views the calendar for a confirmed grade, show a subtle
green banner below the grade switcher:
"✓ Grade 10 schedule confirmed — assessments are locked"

When IsStale = true, show amber:
"⚠️ Grade 10 schedule needs re-confirmation — new changes detected"

When not yet confirmed, show no banner (default state).

---

## ConfirmationReport.jsx (Management Team only)

# This screen generates the Management PDF showing confirmation status
# across all teachers for the current term.
# Accessible from the Management navigation as "Confirmation Report".

### Screen layout

A simple screen with:
- Term selector (defaults to current term)
- Grade filter (All Grades or specific grade)
- Summary counts:
  "12 of 18 teachers confirmed  ·  6 not yet confirmed  ·  2 stale"
- Two-section preview table (Confirmed / Not Yet Confirmed)
- "Download PDF Report" button

### Management PDF Report — layout

**Filename:** `Confirmation_Report_Term1_2026.pdf`

**Header (every page):**
```
[School Name]
Assessment Schedule Confirmation Report
Term 1 2026  ·  Generated: [date and time]  ·  Page X of Y
```

**Section 1 — Confirmed ✓**

| Teacher | Grade | Assessments | Confirmed On |
|---------|-------|-------------|--------------|
| Mr C. Uren | Grade 10 | 4 | Thu 5 Mar 2026 09:14 |
| Mr C. Uren | Grade 11 | 3 | Thu 5 Mar 2026 09:15 |
| Ms S. Dlamini | Grade 10 | 6 | Fri 6 Mar 2026 11:32 |
| ... | | | |

**Section 2 — Not Yet Confirmed ✗**

| Teacher | Grade | Assessments | Days Since Term Started |
|---------|-------|-------------|------------------------|
| Mr J. Mokoena | Grade 8 | 5 | 12 days |
| Mr J. Mokoena | Grade 9 | 3 | 12 days |
| ... | | | |

**Section 3 — Confirmed but Stale ⚠️** (if any exist)

| Teacher | Grade | Confirmed On | Change Detected |
|---------|-------|-------------|-----------------|
| Ms T. Pieterse | Grade 10 | Mon 3 Mar | New assessment added 12 Mar |

**Footer summary:**
```
Total teachers with schedules this term: 18
Fully confirmed: 12 (67%)
Not yet confirmed: 6 (33%)
Confirmed but stale: 2
Report generated by: [Management user name]
```

### generateConfirmationReport() function

```javascript
/**
 * Generates the Management PDF confirmation report.
 * Uses jspdf + jspdf-autotable.
 *
 * @param {Array}  confirmations - All ConfirmationLog rows for the term
 * @param {Array}  teachers - TeacherProfiles for context
 * @param {Object} term - { title, startDate, endDate }
 * @param {string} generatedBy - Management user's display name
 * @returns {void} — triggers browser PDF download
 *
 * Filename: Confirmation_Report_Term1_2026.pdf
 */
export function generateConfirmationReport(confirmations, teachers, term, generatedBy)
```

## ExportButton.jsx

A dropdown button with three export options:

1. **Download as Excel (.xlsx)** — exports the currently filtered Assessment
   List view as a spreadsheet. Use the `xlsx` npm package.
2. **Download as PDF schedule** — exports a formatted teacher-facing
   assessment schedule. Use the `jspdf` and `jspdf-autotable` npm packages.
3. **Student Assessment Report (PDF)** — exports the grade-scoped,
   subject-grouped student distribution report (see below).

---

## Student Assessment Report

# This is a student-facing printable PDF generated from the current
# term's assessment data for the selected grade.
# It is distributed to students so they know what assessments are
# coming up, in which subjects, on which dates, and whether they
# need to prepare.
#
# WHO GENERATES IT:
# Any teacher can generate it from the Calendar View using the
# Export dropdown. It is always grade-scoped (matches whichever
# grade the teacher is currently viewing) and always covers the
# current term only — consistent with the term-scoped calendar.
#
# SCOPE:
# - Grade: currently selected grade (e.g. Grade 10)
# - Term: current term only
# - Status: Scheduled assessments only — Cancelled excluded
# - Includes Notes field if present (up to 300 characters)
#
# FORMAT:
# Grouped by Subject, then within each subject, each class group's
# assessments listed in chronological order.

### Report Layout

**Header (top of every page):**
```
[School name]                              [Logo placeholder]
Assessment Schedule — Grade 10 · Term 1 2026
Generated: [date]                          Page X of Y
```

**Body — grouped by Subject:**

```
MATHEMATICS
──────────────────────────────────────────────────────────
  Grade 10A
    Thu 5 Mar   Mid-Term Test          ✏️  📚 Requires Study
    Fri 20 Mar  Quiz                   ⚡  📚 Requires Study

  Grade 10B
    Thu 5 Mar   Mid-Term Test          ✏️  📚 Requires Study
    Fri 27 Mar  Project                📐  📚 Requires Study

ENGLISH
──────────────────────────────────────────────────────────
  Grade 10A
    Fri 6 Mar   Creative Writing       📎  ✓  No Study Required
                Note: Portfolio piece — submit draft by Tue 3 Mar
    Thu 12 Mar  Assignment             📎  ✓  No Study Required

  Grade 10B
    Sat 7 Mar   Oral Presentation      🎤  ✓  No Study Required
                Note: 5-minute individual presentation. Topics assigned in class.
```

### Report column definitions

Each assessment row shows:
- **Date** — day of week + date (e.g. "Thu 5 Mar")
- **Assessment name** — from the Title field
- **Type icon** — ✏️ Cycle Test · ⚡ Quiz · 📎 Assignment · 🔬 Practical · 🎤 Oral · 📐 Project
- **Study indicator** — 📚 Requires Study OR ✓ No Study Required
- **Notes** (if present) — indented on the next line in italics, max 300 chars

### Report ordering rules

1. **Subjects** sorted alphabetically (A → Z)
2. **Class groups** within each subject sorted alphabetically (10A, 10B, 10C)
3. **Assessments** within each class group sorted chronologically by date
4. If a class group has no assessments for a subject in this term,
   that class group is omitted from that subject section entirely

### Report behaviour for cycle test weeks

Assessments that fall within a cycle test week are shown
with a note in the report:
```
  Grade 10A
    Mon 1 Jun   Mid-Year Examination   ✏️  📚 Requires Study
                [Cycle Test Day — 1 Jun to 19 Jun]
```

### Implementation — generateStudentReport()

Claude Code must implement `generateStudentReport(assessments, grade, term, subjectOrder)`
in `src/services/exportHelpers.js` using `jspdf` and `jspdf-autotable`.

```javascript
/**
 * Generates a student-facing PDF assessment schedule.
 *
 * @param {Array}  assessments - All Scheduled assessments for the grade/term
 * @param {string} grade       - e.g. "Grade 10"
 * @param {Object} term        - { title, startDate, endDate }
 * @param {Array}  cycleTests - To annotate cycle test weeks
 * @returns {void} — triggers browser download of the PDF
 *
 * Filename: Grade10_Term1_2026_Assessment_Schedule.pdf
 *
 * Grouping logic:
 * 1. Get unique subjects from assessments, sort A-Z
 * 2. For each subject:
 *    a. Get unique class groups that have assessments in this subject, sort A-Z
 *    b. For each class group:
 *       - Get all Scheduled assessments for this subject + class group
 *       - Sort by AssessmentDate ascending
 *       - Render each row with date, name, type icon, study indicator
 *       - If Notes exists and is non-empty, render as indented italic sub-row
 *       - If date falls in a cycleTests period, append cycle test week annotation
 * 3. Start each new subject on the same page (no forced page break per subject)
 *    unless it won't fit — jspdf-autotable handles pagination automatically
 */
export function generateStudentReport(assessments, grade, term, cycleTests)
```

### Where the button appears

The Student Assessment Report option appears in:

1. **Calendar View Export dropdown** (primary location) — generates for the
   currently selected grade and current term
2. **Assessment List (My Bookings) Export dropdown** — generates using the
   current filter state (grade and term filters applied)

### ExportButton dropdown menu

```
┌─────────────────────────────────────┐
│ ↓ Export                            │
├─────────────────────────────────────┤
│ 📊  Excel — assessment data         │
│ 📄  PDF — assessment schedule       │
│ 📋  PDF — student distribution      │ ← new
└─────────────────────────────────────┘
```

## ConflictWarning.jsx

Modal dialog shown when validateBooking() returns valid: false

Must show:
- Clear heading: "This booking cannot be saved"
- Specific reason (which rule was violated)
- Details of conflicting assessments (if Rule 1) or cycle test week (if Rule 2)
- "Go Back and Change Date" button (primary action)
- ✏️ Optional "Request Admin Override" button — sends email to admin

# ══════════════════════════════════════════════════════════════════
# SECTION 7: AUTHENTICATION & PERMISSIONS
# ══════════════════════════════════════════════════════════════════

## Authentication Flow

1. Teacher visits the SharePoint page containing the app
2. MSAL automatically detects their Microsoft 365 session
3. If not logged in, they are redirected to the Microsoft login page
4. On return, MSAL provides an access token for Graph API calls
5. The app reads the logged-in user's name and email from the token
   and uses this to pre-fill TeacherName and TeacherEmail fields
6. On load, the app checks which SharePoint Groups the user belongs to
   and sets their role accordingly (see Permission Levels below)

## Three-Tier Permission Model

# There are now THREE distinct roles. Each is implemented as a
# SharePoint Group. IT admin creates these three groups and adds
# staff to them. Claude Code checks group membership via Graph API
# on every login and stores the role in React context.
#
# SharePoint Groups to create:
# 1. "Assessment Teachers"  ← all teaching staff
# 2. "Assessment Admins"    ← HODs, deputy heads, coordinators
# 3. "Assessment Management" ← principal, deputy principal, bursar

## Role 1: Teacher (SharePoint Group: "Assessment Teachers")

What teachers CAN do:
- View all assessments on the calendar (read-only for others)
- Create new assessments for their own classes
- Edit or cancel their own assessments only
- Export the calendar to PDF or Excel
- View the assessment list filtered to their own bookings by default

What teachers CANNOT do:
- Edit another teacher's assessments
- Access ReferenceDataManager
- Access CycleTestManager
- See the Teacher column in the assessment list (they see their own only)
- Override any booking rule violation

## Role 2: Administrator (SharePoint Group: "Assessment Admins")

All Teacher permissions, PLUS:
- View ALL assessments across all teachers unfiltered
- Edit or cancel any teacher's assessment
- Access CycleTestManager (add/edit/delete cycle test weeks)
- See the Teacher column in the assessment list
- Override booking rule violations — ConflictWarning shows an
  "Override and Save (Admin)" button for this role only
- View audit logs for assessment changes

What Admins CANNOT do:
- Access ReferenceDataManager
- Add, edit, or deactivate subjects, class groups, or assessment types
- Delete reference data values

## Role 3: Management Team (SharePoint Group: "Assessment Management")

All Administrator permissions, PLUS:
- Full access to ReferenceDataManager
- Add, edit, reorder, activate, deactivate, and bulk-import all
  reference data (subjects, class groups, assessment types, terms)
- Delete reference data values (if unused)
- View the full ReferenceData audit log
- Cannot be blocked from any screen in the application

## Role Checking Implementation

Claude Code must implement a `useUserRole.js` hook that:

1. On login, calls Graph API to get the current user's group memberships
2. Checks for membership in the three named SharePoint Groups
3. Returns a role object stored in React context:

```javascript
// src/hooks/useUserRole.js
// Returns one of: 'teacher' | 'admin' | 'management' | 'unknown'
// 'unknown' shown an "Access Denied — contact your IT administrator" screen

export function useUserRole() {
  // Checks group membership in priority order:
  // Management > Admin > Teacher
  // A person in both Admin and Management gets Management role
  return { role, isTeacher, isAdmin, isManagement, isLoading }
}
```

## Navigation — What Each Role Sees

```
┌─────────────────────────────────────────────────────────┐
│  Navigation Bar                                          │
├──────────────────┬──────────────────────────────────────┤
│  TEACHER sees:   │  ADMIN sees:       MANAGEMENT sees:  │
│  • Calendar      │  • Calendar        • Calendar         │
│  • My Bookings   │  • All Bookings    • All Bookings     │
│  • Confirm       │  • Cycle Tests   • Cycle Tests     │
│  • My Profile    │  • My Profile      • School Calendar  │
│  • Help          │  • Help            • Policy Manager   │
│                  │                    • Term Archive     │
│                  │                    • Reference Data   │
│                  │                    • Confirmation     │
│                  │                    • Audit Log        │
│                  │                    • Help             │
│                  │                    • My Profile       │
└──────────────────┴──────────────────────────────────────┘
```

"Confirm" links to ConfirmSchedule.jsx (teacher's own grades)
"Confirmation" links to ConfirmationReport.jsx (management PDF report)

## Access Denied Handling

If a user navigates directly to a restricted URL (e.g. /reference-data)
without the correct role, show a full-page "Access Denied" component:
- Clear message: "You do not have permission to access this page."
- Their current role displayed
- Contact information: ✏️ "Please contact [IT Admin name/email] to
  request access."
- Button to return to the Calendar

# ══════════════════════════════════════════════════════════════════
# SECTION 8: GRAPH API OPERATIONS
# ══════════════════════════════════════════════════════════════════
# Claude Code must implement all SharePoint List operations using
# the Microsoft Graph API. No direct SharePoint REST API calls.

## Required Graph API Calls (implement in useGraph.js)

```javascript
// Base URL pattern for all SharePoint List calls:
// GET  /sites/{siteId}/lists/{listId}/items
// POST /sites/{siteId}/lists/{listId}/items
// PATCH /sites/{siteId}/lists/{listId}/items/{itemId}
// DELETE /sites/{siteId}/lists/{listId}/items/{itemId}

// Get all assessments (with $expand=fields for SharePoint columns)
GET /sites/{siteId}/lists/Assessments/items?$expand=fields

// Get assessments for a specific class
GET /sites/{siteId}/lists/Assessments/items?$expand=fields&$filter=fields/ClassGroup eq 'Grade 10A'

// Get assessments for a specific date range (for calendar view performance)
GET /sites/{siteId}/lists/Assessments/items?$expand=fields
    &$filter=fields/AssessmentDate ge '2025-01-01' and fields/AssessmentDate le '2025-12-31'

// Create new assessment
POST /sites/{siteId}/lists/Assessments/items
     Body: { fields: { Title, Subject, ClassGroup, AssessmentDate, ... } }

// Update assessment
PATCH /sites/{siteId}/lists/Assessments/items/{itemId}
      Body: { fields: { Status: 'Cancelled' } }

// Get all formal assessment dates
GET /sites/{siteId}/lists/CycleTests/items?$expand=fields

// Get ALL assessments for a specific grade (for grade-scoped calendar)
// Filters by ClassGroup starting with grade prefix e.g. "Grade 10"
GET /sites/{siteId}/lists/Assessments/items?$expand=fields
    &$filter=startswith(fields/ClassGroup,'Grade 10')

// Get teacher profile by email (used on every login)
GET /sites/{siteId}/lists/TeacherProfiles/items?$expand=fields
    &$filter=fields/TeacherEmail eq 'teacher@school.com'

// Get all ConfirmationLog rows for current teacher and term
GET /sites/{siteId}/lists/ConfirmationLog/items?$expand=fields
    &$filter=fields/TeacherEmail eq 'teacher@school.com'
    and fields/Term eq 'Term 1 2026'

// Get all ConfirmationLog rows for a term (Management use — all teachers)
GET /sites/{siteId}/lists/ConfirmationLog/items?$expand=fields
    &$filter=fields/Term eq 'Term 1 2026'

// Create ConfirmationLog row (first confirmation for a grade/term)
POST /sites/{siteId}/lists/ConfirmationLog/items
     Body: { fields: {
       Title: "Mr C. Uren — Grade 10 — Term 1 2026",
       TeacherEmail: "c.uren@school.edu.za",
       TeacherName: "Mr Christopher Uren",
       Grade: "Grade 10",
       Term: "Term 1 2026",
       IsConfirmed: true,
       ConfirmedAt: "2026-03-05T09:14:00Z",
       AssessmentCount: 4,
       IsStale: false
     }}

// Update ConfirmationLog row (un-confirm or mark stale)
PATCH /sites/{siteId}/lists/ConfirmationLog/items/{itemId}
      Body: { fields: { IsConfirmed, ConfirmedAt, IsStale,
                        UnconfirmedAt, UnconfirmedReason, AssessmentCount }}
// Returns empty array if teacher has no profile yet (first login)

// Create teacher profile (first save)
POST /sites/{siteId}/lists/TeacherProfiles/items
     Body: { fields: {
       Title: "Teacher Name",
       TeacherEmail: "teacher@school.com",
       GradesTaught: '["Grade 10","Grade 11","Grade 12"]',
       ClassesTaught: '["Grade 10A","Grade 10B","Grade 11A","Grade 12A","Grade 12B"]',
       SubjectsTaught: '["Mathematics","English"]',
       DefaultGrade: "Grade 10",
       ProfileComplete: true
     }}

// Update teacher profile (subsequent saves)
PATCH /sites/{siteId}/lists/TeacherProfiles/items/{itemId}
      Body: { fields: { GradesTaught, ClassesTaught, SubjectsTaught, DefaultGrade, ProfileComplete } }

// Get current user info (name + email for auto-populating profile and bookings)
GET /me
```

## Error Handling

All Graph API calls must handle these error cases gracefully:

- **401 Unauthorized:** Token expired — trigger MSAL re-authentication silently
- **403 Forbidden:** User lacks permission — show "Contact your administrator"
- **429 Too Many Requests:** Graph API throttling — implement exponential backoff retry
- **Network error:** Show "Unable to connect — check your internet connection"
- **SharePoint List not found:** Show "Setup required — contact your administrator" with link to SETUP.md

# ══════════════════════════════════════════════════════════════════
# SECTION 9: BUILD & DEPLOYMENT
# ══════════════════════════════════════════════════════════════════

## Development Build

Claude Code must include a working development environment:

```bash
npm install          # install all dependencies
npm run dev          # start local dev server at localhost:5173
npm run build        # create production build in /dist folder
npm run preview      # preview production build locally
```

## Deploying to SharePoint

Claude Code must generate a DEPLOY.md file with step-by-step instructions:

1. Run `npm run build` — this creates a `/dist` folder
2. Upload the entire `/dist` folder contents to a SharePoint Document Library
   (e.g. `Site Assets > AssessmentApp`)
3. Note the URL of `index.html` in that library
4. On your SharePoint page, add a "Embed" web part
5. Paste the URL of `index.html` into the embed web part
6. Publish the page — teachers can now access the app via the SharePoint page URL

# ✏️ Alternative deployment: If your school has an IT team who can
# deploy to Azure Static Web Apps, Claude Code can generate the
# azure-static-web-apps.yml workflow file too. Add this instruction:
# "Also generate an Azure Static Web Apps deployment workflow."

## What Teachers Download

# ✏️ "Downloadable" for teachers means one of two things:
# OPTION A: They bookmark the SharePoint page URL — nothing to install
# OPTION B: They export the assessment schedule as PDF/Excel (ExportButton)
#
# If you meant teachers should be able to run the app offline/locally,
# change this section to:
# "Build as a single index.html file with all JS/CSS inlined,
#  using localStorage instead of SharePoint. Teachers download
#  this one file and open it in any browser."
#
# CURRENT SETTING: Option A — SharePoint URL, with PDF/Excel export

# ══════════════════════════════════════════════════════════════════
# SECTION 10: BUILD ORDER
# ══════════════════════════════════════════════════════════════════
# Claude Code must follow this sequence exactly.
# Complete and test each step before moving to the next.
# Do not generate all files at once.

## Claude Code Build Sequence

**Phase 0 — SharePoint Setup (run this BEFORE writing any app code)**

# Phase 0 must be completed and verified before Phase 1 begins.
# The app cannot be tested without a live SharePoint connection.
# Azure App Registration must already exist (see Section 4) and
# the .env file must be filled in before Phase 0 can run.
#
# Phase 0 generates and runs a single script — setupSharePoint.js —
# that creates all SharePoint infrastructure automatically.
# IT admin runs this script ONCE. Never again unless setting up
# a new environment (e.g. a test site or a second school).

**Phase 0 tasks — generate and run setupSharePoint.js:**

0a. Generate `scripts/setupSharePoint.js` — a standalone Node.js script
    that uses the Microsoft Graph API to create all SharePoint infrastructure.
    The script must be self-contained: it handles its own MSAL authentication
    using the credentials in .env, without depending on any other app code.

0b. The script must create the following in sequence, pausing after each
    group to confirm success before continuing:

**Group A — SharePoint Lists (8 lists)**
For each list: create the list, then add every column in the correct order.

```
List 1: Assessments
  Columns: Subject (text), ClassGroup (text), AssessmentType (text),
           AssessmentDate (dateTime, dateOnly), Duration (number),
           TeacherName (text, required), TeacherEmail (text, required),
           Term (text, required),
           StudyRequired (choice, required,
                          choices: ["Requires Study","No Study Required"],
                          default: "Requires Study"),
           Notes (multilineText, plain),
           Status (text, required, default: "Scheduled")

List 2: CycleTests
  Columns: StartDate (dateTime, dateOnly, required),
           EndDate (dateTime, dateOnly, required),
           Subject (text),
           Notes (multilineText, plain)

List 3: PublicHolidays
  Columns: HolidayDate (dateTime, dateOnly, required),
           Year (number, required),
           IsSubstitute (boolean, required, default: false),
           OriginalDate (dateTime, dateOnly),
           Source (choice, required, choices: ["Pre-loaded","Management Added"],
                   default: "Pre-loaded")

List 4: SchoolCalendar
  Columns: StartDate (dateTime, dateOnly, required),
           EndDate (dateTime, dateOnly, required),
           Year (number, required),
           Term (text, required),
           IncludeSaturdays (boolean, required, default: true),
           Notes (multilineText, plain)

List 5: AppSettings
  Columns: DisplayName (text, required),
           Value (text, required),
           DefaultValue (text, required),
           DataType (choice, required, choices: ["Number","Boolean","Text"]),
           Description (multilineText, plain, required),
           Category (text, required),
           ChangedBy (text),
           ChangedDate (dateTime)

List 6: ReferenceData
  Columns: Category (text, required),
           SortOrder (number, required),
           Active (boolean, required, default: true),
           Description (text)

List 7: ReferenceDataAuditLog
  Columns: ListChanged (text, required),
           Category (text, required),
           OldValue (text),
           NewValue (text, required),
           Action (choice, required, choices: ["Added","Edited","Deactivated",
                   "Reactivated","Deleted","Setting Changed"]),
           ChangedBy (text, required),
           ChangedByEmail (text, required),
           ChangedDate (dateTime, required),
           ItemId (number)

List 8: ArchiveLog
  Columns: TermName (text, required),
           Year (number, required),
           AssessmentsFile (text, required),
           CycleTestsFile (text, required),
           AssessmentsCount (number, required),
           CycleTestsCount (number, required),
           ArchivedBy (text, required),
           ArchivedByEmail (text, required),
           ArchivedDate (dateTime, required),
           ClearSucceeded (boolean, required),
           Notes (multilineText, plain)

List 9: TeacherProfiles
  Columns: TeacherEmail (text, required),
           GradesTaught (multilineText, plain, required),
           ClassesTaught (multilineText, plain, required),
           SubjectsTaught (multilineText, plain, required),
           DefaultGrade (text, required),
           ProfileComplete (boolean, required, default: false)
  # Note: Title column already exists on every SharePoint list.
  # Total manually created columns = 6. Title + 2 auto = 9 columns total.

List 10: ConfirmationLog
  Columns: TeacherEmail (text, required),
           TeacherName (text, required),
           Grade (text, required),
           Term (text, required),
           IsConfirmed (boolean, required, default: false),
           ConfirmedAt (dateTime),
           AssessmentCount (number, required, default: 0),
           IsStale (boolean, required, default: false),
           UnconfirmedAt (dateTime),
           UnconfirmedReason (text)

List 11: CycleTestConfirmations
  Columns: Grade (text, required),
           Term (text, required),
           IsConfirmed (boolean, required, default: false),
           ConfirmedAt (dateTime),
           ConfirmedBy (text),
           CycleTestCount (number, required, default: 0),
           IsStale (boolean, required, default: false),
           UnconfirmedAt (dateTime),
           UnconfirmedBy (text)
  # Note: Title auto-set by app. Total manually created columns = 10.
```

**Group B — Document Library**
```
Library: AssessmentArchives
  Type: documentLibrary
  Description: "Stores end-of-term Excel archive files"
```

**Group C — Seed Data (run after lists confirmed created)**
Run the three seed scripts in sequence:
```bash
node scripts/seedReferenceData.js     # populates List 6
node scripts/seedAppSettings.js       # populates List 5 (9 rows)
node scripts/seedPublicHolidays.js    # populates List 3 (2025 + 2026)
```

0c. The script must print a completion report listing every list and
    column created, the document library, and seed row counts:

```
════════════════════════════════════════════
ASSESSMENT APP — SHAREPOINT SETUP COMPLETE
════════════════════════════════════════════
✅  List created: Assessments (10 columns)
✅  List created: CycleTests (4 columns)
✅  List created: PublicHolidays (5 columns)
✅  List created: SchoolCalendar (6 columns)
✅  List created: AppSettings (8 columns)
✅  List created: ReferenceData (4 columns)
✅  List created: ReferenceDataAuditLog (9 columns)
✅  List created: ArchiveLog (11 columns)
✅  List created: TeacherProfiles (6 columns)
✅  List created: ConfirmationLog (10 columns)
✅  Document Library created: AssessmentArchives
✅  Seeded ReferenceData: 27 rows
✅  Seeded AppSettings: 9 rows
✅  Seeded PublicHolidays: 26 rows (2025 + 2026)
════════════════════════════════════════════
All done. Proceed to Phase 1.
```

0d. The script must handle errors gracefully:
    - If a list already exists: skip creation, log "already exists — skipped"
    - If a column already exists: skip, log warning, continue
    - If a seed script fails: report exact error and stop — do not continue
      to next seed script. Partial seeding is worse than no seeding.
    - If any Group A item fails: stop immediately. Do not proceed to Group B
      or C until all 8 lists and the library are confirmed created.

0e. Add a `--verify` flag that can be run at any time to check the current
    state of SharePoint without creating anything:
    ```bash
    node scripts/setupSharePoint.js --verify
    ```
    This reads all 8 lists, checks every column exists with the correct type,
    checks the document library exists, and checks seed data row counts.
    Useful for diagnosing issues after deployment or when setting up a
    test environment.

0f. Add a `--dry-run` flag that prints everything the script WOULD do
    without making any Graph API calls:
    ```bash
    node scripts/setupSharePoint.js --dry-run
    ```
    Useful for IT admin to review before running for real.

**Test (Phase 0 complete when ALL pass):**
- [ ] `node scripts/setupSharePoint.js --dry-run` prints full plan with no errors
- [ ] `node scripts/setupSharePoint.js` runs to completion with green ticks
- [ ] `node scripts/setupSharePoint.js --verify` shows all lists, columns, and seed data
- [ ] Running the script a second time (on already-created lists) skips gracefully
- [ ] AppSettings list contains exactly 9 rows
- [ ] ReferenceData list contains 27 rows
- [ ] PublicHolidays list contains 26 rows (13 per year × 2 years)
- [ ] Log into the SharePoint site and visually confirm all 8 lists exist

# ══ SKIP PHASE 0 ENTIRELY FOR NOW ══════════════════════════════
# Phase 0 creates SharePoint lists — not needed for local testing.
# Go straight to Phase 0b below. Phase 0 will be run later when
# IT admin has completed the Azure App Registration.
# ════════════════════════════════════════════════════════════════

---

**Phase 0b — Mock Data Layer (run this to enable local testing WITHOUT Azure)**

# ════════════════════════════════════════════════════════════════════
# PHASE 0b — LOCAL DEVELOPMENT WITHOUT AZURE OR SHAREPOINT
# ════════════════════════════════════════════════════════════════════
# PURPOSE:
# Allows the full application to be built, run, and tested locally
# without Azure App Registration, SharePoint access, or IT admin involvement.
#
# HOW IT WORKS:
# A service layer sits between the React hooks and the data source.
# When VITE_USE_MOCK=true, all hooks read from local mock data files
# instead of making Graph API calls. The hooks themselves are unchanged —
# only the data source switches. Switching to live SharePoint later
# requires no code changes — only updating the .env file.
#
# WHAT YOU CAN TEST IN MOCK MODE:
# ✅ All UI screens and navigation
# ✅ Grade-per-calendar architecture and grade switcher
# ✅ Term grid with badges, tooltips, and week columns
# ✅ BookingForm with live grid preview
# ✅ StudyRequired toggle (locked/unlocked)
# ✅ All six booking rules (daily limit, cycle test days, public holidays, etc.)
# ✅ Teacher profile screen and first-login redirect
# ✅ All three roles (Teacher / Admin / Management)
# ✅ Policy Manager, Reference Data Manager, School Calendar Manager
# ✅ Term Archive Manager (exports to Excel using local mock data)
# ✅ Archive banner trigger logic
# ✅ Full design system — fonts, colours, animations
#
# WHAT CANNOT BE TESTED IN MOCK MODE:
# ❌ Real Microsoft 365 login (replaced with mock user selector)
# ❌ Data persistence between sessions (resets on browser refresh)
# ❌ SharePoint group membership (roles set via mock user selector)
# ════════════════════════════════════════════════════════════════════

**Phase 0b tasks — generate the complete mock data layer:**

**0b-1. Create `src/services/graphService.js` — the service abstraction layer**

This is the central routing file. Every hook imports from graphService,
never directly from the Graph API or mock data. This means the hooks
are identical in mock and live modes.

```javascript
// src/services/graphService.js
//
// Reads VITE_USE_MOCK from the environment.
// In mock mode: imports from mockDataService.js
// In live mode: imports from graphApiService.js
//
// All hooks call graphService — never the underlying service directly.
// This is the ONLY file that changes behaviour based on VITE_USE_MOCK.

const USE_MOCK = import.meta.env.VITE_USE_MOCK === 'true';

export const graphService = USE_MOCK
  ? await import('./mockDataService.js')
  : await import('./graphApiService.js');
```

**0b-2. Create `src/services/graphApiService.js` — the live Graph API service**

Implements all data operations using the Microsoft Graph API.
Used when VITE_USE_MOCK=false. This is the production service.

Functions to implement (all async, all return the same shape as mock):
```javascript
export async function getAssessments(grade)
export async function createAssessment(data)
export async function updateAssessment(id, data)
export async function deleteAssessment(id)
export async function getCycleTests()
export async function createCycleTest(data)
export async function updateCycleTest(id, data)
export async function deleteCycleTest(id)
export async function getPublicHolidays()
export async function getSchoolCalendar()
export async function createSchoolCalendarEntry(data)
export async function getAppSettings()
export async function updateAppSetting(key, value)
export async function getReferenceData()
export async function createReferenceDataItem(data)
export async function updateReferenceDataItem(id, data)
export async function getTeacherProfile(email)
export async function saveTeacherProfile(data)
export async function getArchiveLog()
export async function createArchiveLogEntry(data)
export async function archiveAndClearLists(termName, year)
export async function getCurrentUser()
export async function getUserRole()
```

**0b-3. Create `src/services/mockDataService.js` — the mock data service**

Implements identical function signatures to graphApiService.js but
reads from and writes to local JavaScript objects (in-memory store).
Data persists within the browser session but resets on refresh.

```javascript
// src/services/mockDataService.js
// In-memory store — mirrors SharePoint List structure exactly
let store = { ...INITIAL_MOCK_DATA };
// All functions read/write store — same signatures as graphApiService
```

**0b-4. Create `src/data/mockData.js` — the initial mock dataset**

Realistic seed data for all 9 SharePoint lists, pre-populated with
enough records to exercise every feature of the application.

Claude Code must generate INITIAL_MOCK_DATA containing:

```javascript
// src/data/mockData.js

export const INITIAL_MOCK_DATA = {

  // ── LIST 1: Assessments ──────────────────────────────────────────
  // 20–30 assessments spread across Grade 8–11, all four terms,
  // multiple subjects and class groups, mix of StudyRequired values,
  // some on dates close to cycle test weeks (to test warnings),
  // some cancelled (to test status display)
  assessments: [
    {
      id: "1", title: "Mathematics Mid-Term Test",
      subject: "Mathematics", classGroup: "Grade 10A",
      assessmentType: "Class Test", assessmentDate: "2026-03-05",
      duration: 60, teacherName: "Mr Christopher Uren",
      teacherEmail: "c.uren@school.edu.za",
      term: "Term 1", studyRequired: "Requires Study",
      notes: "Covers chapters 4-6. Bring a calculator and geometry set.", status: "Scheduled"
    },
    {
      id: "2", title: "English Creative Writing Assignment",
      subject: "English", classGroup: "Grade 10A",
      assessmentType: "Assignment", assessmentDate: "2026-03-12",
      duration: 90, teacherName: "Ms Sarah Dlamini",
      teacherEmail: "s.dlamini@school.edu.za",
      term: "Term 1", studyRequired: "No Study Required",
      notes: "Portfolio piece. Submit draft by Tue 10 Mar for feedback before final submission.", status: "Scheduled"
    },
    {
      id: "3", title: "Physical Sciences Practical",
      subject: "Physical Sciences", classGroup: "Grade 10B",
      assessmentType: "Practical", assessmentDate: "2026-03-05",
      duration: 120, teacherName: "Mr Christopher Uren",
      teacherEmail: "c.uren@school.edu.za",
      term: "Term 1", studyRequired: "Requires Study",
      notes: "", status: "Scheduled"
    },
    {
      id: "5", title: "Mathematics Assignment",
      subject: "Mathematics", classGroup: "Grade 12A",
      assessmentType: "Assignment", assessmentDate: "2026-03-10",
      duration: 60, teacherName: "Mr Christopher Uren",
      teacherEmail: "c.uren@school.edu.za",
      term: "Term 1", studyRequired: "No Study Required",
      notes: "Chapter 5 consolidation task", status: "Scheduled"
    },
    {
      id: "6", title: "Physical Sciences Practical",
      subject: "Physical Sciences", classGroup: "Grade 12B",
      assessmentType: "Practical", assessmentDate: "2026-03-19",
      duration: 120, teacherName: "Mr Christopher Uren",
      teacherEmail: "c.uren@school.edu.za",
      term: "Term 1", studyRequired: "Requires Study",
      notes: "", status: "Scheduled"
    },
    // ✏️ Claude Code must generate 25+ more realistic assessments
    // covering all grades (8–12), all class groups, multiple teachers,
    // both terms 1 and 2, and a mix of StudyRequired values.
    // Include at least 2 assessments per class group per term.
    // Include at least 3 cancelled assessments.
  ],

  // ── LIST 2: CycleTests ───────────────────────────────
  // 4–6 cycle test weeks covering the school year
  cycleTests: [
    { id: "1", title: "Grade 10 Mathematics Cycle Test — 5 Mar 2026",
      cycleTestDate: "2026-03-05", grade: "Grade 10", subject: "Mathematics", notes: "" },
    { id: "2", title: "Grade 10 Physical Sciences Cycle Test — 12 Mar 2026",
      cycleTestDate: "2026-03-12", grade: "Grade 10", subject: "Physical Sciences", notes: "" },
    { id: "3", title: "Grade 11 Mathematics Cycle Test — 6 Mar 2026",
      cycleTestDate: "2026-03-06", grade: "Grade 11", subject: "Mathematics", notes: "" },
    { id: "4", title: "Grade 8 English Cycle Test — 10 Mar 2026",
      cycleTestDate: "2026-03-10", grade: "Grade 8", subject: "English", notes: "" },
    { id: "5", title: "Grade 9 Mathematics Cycle Test — 11 Mar 2026",
      cycleTestDate: "2026-03-11", grade: "Grade 9", subject: "Mathematics", notes: "" },
    { id: "6", title: "Grade 12 Mathematics Cycle Test — 7 Mar 2026",
      cycleTestDate: "2026-03-07", grade: "Grade 12", subject: "Mathematics", notes: "" },
  ],

  // ── LIST 3: PublicHolidays ───────────────────────────────────────
  // Full 2026 SA public holidays (pre-loaded)
  publicHolidays: [
    { id: "1",  title: "New Year's Day",        holidayDate: "2026-01-01", year: 2026, isSubstitute: false },
    { id: "2",  title: "Human Rights Day",      holidayDate: "2026-03-21", year: 2026, isSubstitute: false },
    { id: "3",  title: "Good Friday",           holidayDate: "2026-04-03", year: 2026, isSubstitute: false },
    { id: "4",  title: "Family Day",            holidayDate: "2026-04-06", year: 2026, isSubstitute: false },
    { id: "5",  title: "Freedom Day",           holidayDate: "2026-04-27", year: 2026, isSubstitute: false },
    { id: "6",  title: "Workers Day",           holidayDate: "2026-05-01", year: 2026, isSubstitute: false },
    { id: "7",  title: "Youth Day",             holidayDate: "2026-06-16", year: 2026, isSubstitute: false },
    { id: "8",  title: "National Womens Day",   holidayDate: "2026-08-09", year: 2026, isSubstitute: false },
    { id: "9",  title: "Womens Day (observed)", holidayDate: "2026-08-10", year: 2026, isSubstitute: true },
    { id: "10", title: "Heritage Day",          holidayDate: "2026-09-24", year: 2026, isSubstitute: false },
    { id: "11", title: "Day of Reconciliation", holidayDate: "2026-12-16", year: 2026, isSubstitute: false },
    { id: "12", title: "Christmas Day",         holidayDate: "2026-12-25", year: 2026, isSubstitute: false },
    { id: "13", title: "Day of Goodwill",       holidayDate: "2026-12-26", year: 2026, isSubstitute: false },
  ],

  // ── LIST 4: SchoolCalendar ───────────────────────────────────────
  // Four term windows for 2026
  schoolCalendar: [
    { id: "1", title: "Term 1 2026", startDate: "2026-01-14", endDate: "2026-03-27", year: 2026, term: "Term 1", includeSaturdays: true },
    { id: "2", title: "Term 2 2026", startDate: "2026-04-14", endDate: "2026-06-26", year: 2026, term: "Term 2", includeSaturdays: true },
    { id: "3", title: "Term 3 2026", startDate: "2026-07-14", endDate: "2026-09-25", year: 2026, term: "Term 3", includeSaturdays: true },
    { id: "4", title: "Term 4 2026", startDate: "2026-10-06", endDate: "2026-12-11", year: 2026, term: "Term 4", includeSaturdays: false },
  ],

  // ── LIST 5: AppSettings ──────────────────────────────────────────
  // Default rule configuration — matches the seeded live values
  appSettings: {
    MAX_ASSESSMENTS_PER_DAY:  { value: 2,             dataType: "Number",  displayName: "Maximum Assessments Per Day" },
    DAILY_LIMIT_SCOPE:        { value: "ClassGroup",  dataType: "Text",    displayName: "Daily Limit Applies To" },
    CYCLE_TEST_SCOPE:        { value: "SchoolWide",  dataType: "Text",    displayName: "Formal Assessment Date Block Scope" },
    ALLOW_ADMIN_OVERRIDE:     { value: true,          dataType: "Boolean", displayName: "Allow Admin Rule Override" },
    WARN_BEFORE_CYCLE_TEST:  { value: 0,             dataType: "Number",  displayName: "Warn Days Before Cycle Test" },
    ARCHIVE_DESTINATION_PATH: { value: "Site Assets/AssessmentArchives", dataType: "Text", displayName: "Archive Storage Location" },
    ARCHIVE_SNOOZE_COUNT:     { value: 0,             dataType: "Number",  displayName: "Archive Reminder Snooze Count" },
    ARCHIVE_SNOOZED_UNTIL:    { value: "",            dataType: "Text",    displayName: "Archive Reminder Snoozed Until" },
    LAST_ARCHIVED_TERM:       { value: "",            dataType: "Text",    displayName: "Last Archived Term" },
  },

  // ── LIST 6: ReferenceData ────────────────────────────────────────
  // Subjects, class groups, assessment types, terms, statuses
  referenceData: [
    { id: "1",  category: "Subject",        title: "Mathematics",      sortOrder: 1, active: true },
    { id: "2",  category: "Subject",        title: "English",          sortOrder: 2, active: true },
    { id: "3",  category: "Subject",        title: "Physical Sciences", sortOrder: 3, active: true },
    { id: "4",  category: "Subject",        title: "Life Sciences",    sortOrder: 4, active: true },
    { id: "5",  category: "Subject",        title: "History",          sortOrder: 5, active: true },
    { id: "6",  category: "Subject",        title: "Geography",        sortOrder: 6, active: true },
    { id: "7",  category: "Subject",        title: "Accounting",       sortOrder: 7, active: true },
    { id: "8",  category: "ClassGroup",     title: "Grade 8A",         sortOrder: 1, active: true },
    { id: "9",  category: "ClassGroup",     title: "Grade 8B",         sortOrder: 2, active: true },
    { id: "10", category: "ClassGroup",     title: "Grade 9A",         sortOrder: 3, active: true },
    { id: "11", category: "ClassGroup",     title: "Grade 9B",         sortOrder: 4, active: true },
    { id: "12", category: "ClassGroup",     title: "Grade 10A",        sortOrder: 5, active: true },
    { id: "13", category: "ClassGroup",     title: "Grade 10B",        sortOrder: 6, active: true },
    { id: "14", category: "ClassGroup",     title: "Grade 11A",        sortOrder: 7, active: true },
    { id: "15", category: "ClassGroup",     title: "Grade 11B",        sortOrder: 8, active: true },
    { id: "15b", category: "ClassGroup",   title: "Grade 11C",        sortOrder: 9, active: true },
    { id: "15c", category: "ClassGroup",   title: "Grade 12A",        sortOrder: 10, active: true },
    { id: "15d", category: "ClassGroup",   title: "Grade 12B",        sortOrder: 11, active: true },
    { id: "15e", category: "ClassGroup",   title: "Grade 12C",        sortOrder: 12, active: true },
    // The following are the teacher-bookable types:
    // Cycle Test is NOT here — it is managed via CycleTestManager only
    { id: "16",  category: "AssessmentType", title: "Class Test",  abbreviation: "ClsT", icon: "pencil",     sortOrder: 1, active: true },
    { id: "16b", category: "AssessmentType", title: "Quiz",        abbreviation: "Qz",   icon: "bolt",       sortOrder: 2, active: true },
    { id: "16c", category: "AssessmentType", title: "Assignment",  abbreviation: "Asn",  icon: "paperclip",  sortOrder: 3, active: true },
    { id: "16d", category: "AssessmentType", title: "Practical",   abbreviation: "Pr",   icon: "microscope", sortOrder: 4, active: true },
    { id: "16e", category: "AssessmentType", title: "Oral",        abbreviation: "Or",   icon: "microphone", sortOrder: 5, active: true },
    { id: "16f", category: "AssessmentType", title: "Project",     abbreviation: "Prj",  icon: "ruler",      sortOrder: 6, active: true },
    { id: "16g", category: "AssessmentType", title: "Other",       abbreviation: "OT",   icon: "clipboard",  sortOrder: 7, active: true },
    { id: "17", category: "AssessmentType", title: "Quiz",             sortOrder: 2, active: true },
    { id: "18", category: "AssessmentType", title: "Assignment",       sortOrder: 3, active: true },
    { id: "19", category: "AssessmentType", title: "Practical",        sortOrder: 4, active: true },
    { id: "20", category: "AssessmentType", title: "Oral",             sortOrder: 5, active: true },
    { id: "21", category: "AssessmentType", title: "Project",          sortOrder: 6, active: true },
    { id: "22", category: "Term",           title: "Term 1",           sortOrder: 1, active: true },
    { id: "23", category: "Term",           title: "Term 2",           sortOrder: 2, active: true },
    { id: "24", category: "Term",           title: "Term 3",           sortOrder: 3, active: true },
    { id: "25", category: "Term",           title: "Term 4",           sortOrder: 4, active: true },
    { id: "26", category: "Status",         title: "Scheduled",        sortOrder: 1, active: true },
    { id: "27", category: "Status",         title: "Cancelled",        sortOrder: 2, active: true },
    { id: "28", category: "Status",         title: "Completed",        sortOrder: 3, active: true },
  ],

  // ── LIST 7: ReferenceDataAuditLog ───────────────────────────────
  referenceDataAuditLog: [],   // starts empty in mock

  // ── LIST 8: ArchiveLog ───────────────────────────────────────────
  archiveLog: [],              // starts empty in mock

  // ── LIST 9: TeacherProfiles ──────────────────────────────────────
  // One profile per mock user — matches MOCK_USERS below
  teacherProfiles: [
    {
      id: "1",
      title: "Mr Christopher Uren",
      teacherEmail: "c.uren@school.edu.za",
      gradesTaught: ["Grade 10", "Grade 11", "Grade 12"],
      classesTaught: ["Grade 10A", "Grade 10B", "Grade 11A", "Grade 12A", "Grade 12B"],
      subjectsTaught: ["Physical Sciences", "Mathematics"],
      defaultGrade: "Grade 10",
      profileComplete: true
    },
    {
      id: "2",
      title: "Ms Sarah Dlamini",
      teacherEmail: "s.dlamini@school.edu.za",
      gradesTaught: ["Grade 10"],
      classesTaught: ["Grade 10A", "Grade 10B"],
      subjectsTaught: ["English"],
      defaultGrade: "Grade 10",
      profileComplete: true
    },
    {
      id: "3",
      title: "Mr James Mokoena",
      teacherEmail: "j.mokoena@school.edu.za",
      gradesTaught: ["Grade 8", "Grade 9"],
      classesTaught: ["Grade 8A", "Grade 8B", "Grade 9A"],
      subjectsTaught: ["Mathematics", "Accounting"],
      defaultGrade: "Grade 8",
      profileComplete: true
    },
    {
      id: "4",
      title: "New Teacher",
      teacherEmail: "new.teacher@school.edu.za",
      gradesTaught: [],
      classesTaught: [],
      subjectsTaught: [],
      defaultGrade: "",
      profileComplete: false  // ← triggers first-login redirect
    },
  ],

  // ── LIST 10: ConfirmationLog ─────────────────────────────────────
  // Pre-populated to test all three states in the Management PDF:
  // confirmed, not yet confirmed, and stale
  confirmationLog: [
    {
      id: "1",
      title: "Mr Christopher Uren — Grade 10 — Term 1 2026",
      teacherEmail: "c.uren@school.edu.za",
      teacherName: "Mr Christopher Uren",
      grade: "Grade 10",
      term: "Term 1 2026",
      isConfirmed: true,
      confirmedAt: "2026-03-05T09:14:00Z",
      assessmentCount: 4,
      isStale: false,
      unconfirmedAt: null,
      unconfirmedReason: ""
    },
    {
      id: "2",
      title: "Mr Christopher Uren — Grade 11 — Term 1 2026",
      teacherEmail: "c.uren@school.edu.za",
      teacherName: "Mr Christopher Uren",
      grade: "Grade 11",
      term: "Term 1 2026",
      isConfirmed: false,
      confirmedAt: null,
      assessmentCount: 3,
      isStale: false,
      unconfirmedAt: null,
      unconfirmedReason: ""
    },
    {
      id: "3",
      title: "Ms Sarah Dlamini — Grade 10 — Term 1 2026",
      teacherEmail: "s.dlamini@school.edu.za",
      teacherName: "Ms Sarah Dlamini",
      grade: "Grade 10",
      term: "Term 1 2026",
      isConfirmed: true,
      confirmedAt: "2026-03-06T11:32:00Z",
      assessmentCount: 6,
      isStale: true,  // ← tests stale state: new assessment added after confirmation
      unconfirmedAt: null,
      unconfirmedReason: ""
    },
    {
      id: "3b",
      title: "Mr Christopher Uren — Grade 12 — Term 1 2026",
      teacherEmail: "c.uren@school.edu.za",
      teacherName: "Mr Christopher Uren",
      grade: "Grade 12",
      term: "Term 1 2026",
      isConfirmed: false,
      confirmedAt: null,
      assessmentCount: 2,
      isStale: false,
      unconfirmedAt: null,
      unconfirmedReason: ""
    },
    {
      id: "4",
      title: "Mr James Mokoena — Grade 8 — Term 1 2026",
      teacherEmail: "j.mokoena@school.edu.za",
      teacherName: "Mr James Mokoena",
      grade: "Grade 8",
      term: "Term 1 2026",
      isConfirmed: false,
      confirmedAt: null,
      assessmentCount: 5,
      isStale: false,
      unconfirmedAt: null,
      unconfirmedReason: ""
    },
  ],
  // ── LIST 11: CycleTestConfirmations ─────────────────────────────
  // Mixed states to test all scenarios:
  // Grade 10 confirmed, Grade 11 not confirmed, Grade 12 stale
  cycleTestConfirmations: [
    {
      id: "1", title: "Grade 10 — Term 1 2026 — Cycle Test Schedule",
      grade: "Grade 10", term: "Term 1 2026",
      isConfirmed: true, confirmedAt: "2026-01-10T08:30:00Z",
      confirmedBy: "Mr David van Wyk", cycleTestCount: 2,
      isStale: false, unconfirmedAt: null, unconfirmedBy: ""
    },
    {
      id: "2", title: "Grade 11 — Term 1 2026 — Cycle Test Schedule",
      grade: "Grade 11", term: "Term 1 2026",
      isConfirmed: false, confirmedAt: null,
      confirmedBy: "", cycleTestCount: 1,
      isStale: false, unconfirmedAt: null, unconfirmedBy: ""
    },
    {
      id: "3", title: "Grade 12 — Term 1 2026 — Cycle Test Schedule",
      grade: "Grade 12", term: "Term 1 2026",
      isConfirmed: true, confirmedAt: "2026-01-10T08:35:00Z",
      confirmedBy: "Mr David van Wyk", cycleTestCount: 1,
      isStale: true, unconfirmedAt: null, unconfirmedBy: ""
      // IsStale: new cycle test was added after confirmation
    },
    {
      id: "4", title: "Grade 8 — Term 1 2026 — Cycle Test Schedule",
      grade: "Grade 8", term: "Term 1 2026",
      isConfirmed: true, confirmedAt: "2026-01-09T14:00:00Z",
      confirmedBy: "Mr David van Wyk", cycleTestCount: 1,
      isStale: false, unconfirmedAt: null, unconfirmedBy: ""
    },
    {
      id: "5", title: "Grade 9 — Term 1 2026 — Cycle Test Schedule",
      grade: "Grade 9", term: "Term 1 2026",
      isConfirmed: false, confirmedAt: null,
      confirmedBy: "", cycleTestCount: 1,
      isStale: false, unconfirmedAt: null, unconfirmedBy: ""
    },
  ],
};

**0b-5. Create `src/data/mockUsers.js` — mock user accounts for role testing**

```javascript
// src/data/mockUsers.js
// These users replace Microsoft 365 authentication in mock mode.
// Each user has a role that drives what they can see and do.

export const MOCK_USERS = [
  {
    id: "teacher-1",
    name: "Mr Christopher Uren",
    email: "c.uren@school.edu.za",
    role: "teacher",
    // Has a complete profile — lands on Grade 10 calendar with grade switcher
  },
  {
    id: "teacher-2",
    name: "Ms Sarah Dlamini",
    email: "s.dlamini@school.edu.za",
    role: "teacher",
    // Grade 10 only — no grade switcher shown
  },
  {
    id: "teacher-3",
    name: "Mr James Mokoena",
    email: "j.mokoena@school.edu.za",
    role: "teacher",
    // Grade 8 and 9 — tests grade switcher on lower grades
  },
  {
    id: "teacher-new",
    name: "New Teacher",
    email: "new.teacher@school.edu.za",
    role: "teacher",
    // profileComplete: false — tests first-login redirect to TeacherProfile
  },
  {
    id: "admin-1",
    name: "Mrs Patricia Nkosi",
    email: "p.nkosi@school.edu.za",
    role: "admin",
    // HOD — tests Admin role: Cycle Tests, override capability
  },
  {
    id: "management-1",
    name: "Mr David van Wyk",
    email: "d.vanwyk@school.edu.za",
    role: "management",
    // Principal — tests Management role: all screens including Policy Manager
  },
];
```

**0b-6. Create `src/components/MockLoginSelector.jsx`**

A development-only login screen shown when VITE_USE_MOCK=true.
Replaces the real Microsoft MSAL authentication flow.

```jsx
// MockLoginSelector.jsx
// Shows ONLY when VITE_USE_MOCK=true — never in production
// A simple dropdown + "Login" button
// Selecting a user from MOCK_USERS sets the mock session

// UI:
// ┌─────────────────────────────────────────────┐
// │  🏫  [School Name]                          │
// │  Assessment Date Manager                    │
// │  ─────────────────────────────────────────  │
// │  [DEV MODE — Mock Login]                    │
// │                                             │
// │  Select user:                               │
// │  ┌─────────────────────────────────────┐    │
// │  │ Mr Christopher Uren (Teacher)    ▼  │    │
// │  └─────────────────────────────────────┘    │
// │                                             │
// │  [  Sign In  ]                              │
// │                                             │
// │  Role: Teacher — Grade 10 & 11              │
// └─────────────────────────────────────────────┘

// Styled with the full design system (DM Serif Display heading,
// Outfit body, cobalt button) — identical to production login feel
// except for the [DEV MODE] banner in amber
```

**0b-7. Update `.env.example` with mock variables**

```bash
# ══════════════════════════════════════════════════════════
# MOCK MODE — local testing without Azure or SharePoint
# ══════════════════════════════════════════════════════════
# Set to true to run the full app locally without any Azure
# App Registration or SharePoint access.
# Set to false when deploying to SharePoint with real data.
VITE_USE_MOCK=true

# ══════════════════════════════════════════════════════════
# LIVE MODE — required when VITE_USE_MOCK=false
# ══════════════════════════════════════════════════════════
# Fill these in after Azure App Registration is complete.
# Leave blank while VITE_USE_MOCK=true — they are not read.
VITE_AZURE_CLIENT_ID=
VITE_AZURE_TENANT_ID=
VITE_SHAREPOINT_SITE_URL=
VITE_SHAREPOINT_SITE_ID=
```

**0b-8. Add mock-related files to project structure**

```
src/
├── data/
│   ├── mockData.js          ← initial dataset for all 9 lists
│   └── mockUsers.js         ← mock user accounts for role testing
├── services/
│   ├── graphService.js      ← routes to mock or live based on VITE_USE_MOCK
│   ├── graphApiService.js   ← live Graph API implementation
│   └── mockDataService.js   ← in-memory mock implementation
├── components/
│   └── MockLoginSelector.jsx ← dev-only login selector (mock mode only)
```

**0b-9. Switching from mock to live — the only change needed**

When Azure App Registration is complete and SharePoint lists are created:

```bash
# 1. Open your .env file
# 2. Change one line:
VITE_USE_MOCK=false

# 3. Fill in the Azure credentials:
VITE_AZURE_CLIENT_ID=your-client-id
VITE_AZURE_TENANT_ID=your-tenant-id
VITE_SHAREPOINT_SITE_URL=https://yourschool.sharepoint.com/sites/yoursite
VITE_SHAREPOINT_SITE_ID=your-site-id

# 4. Run Phase 0 to create the SharePoint lists
node scripts/setupSharePoint.js

# 5. Restart the dev server
npm run dev
# The app now reads from and writes to SharePoint
```

Zero code changes. The entire application — every hook, every component,
every rule — is identical in mock and live modes.

**Test (Phase 0b complete when ALL pass):**
- [ ] `npm run dev` starts without errors
- [ ] MockLoginSelector appears at localhost:5173
- [ ] Logging in as "Mr Christopher Uren" shows Grade 10 calendar with grade switcher
- [ ] Logging in as "Ms Sarah Dlamini" shows Grade 10 calendar with NO grade switcher
- [ ] Logging in as "New Teacher" redirects immediately to TeacherProfile screen
- [ ] Logging in as "Mrs Patricia Nkosi" (Admin) shows Cycle Tests in navigation
- [ ] Logging in as "Mr David van Wyk" (Management) shows all navigation items
- [ ] Booking an assessment in mock mode appears on calendar and in term grid
- [ ] Logging in as "Mr Christopher Uren" shows Grade 10 as confirmed (green banner) and Grade 11 as not confirmed
- [ ] Logging in as "Ms Sarah Dlamini" shows Grade 10 as stale (amber banner)
- [ ] ConfirmSchedule screen shows correct assessment count per grade
- [ ] Confirming a grade creates/updates a ConfirmationLog row and locks assessments
- [ ] Management confirmation report shows all three sections correctly
- [ ] Booking on a public holiday (e.g. 2026-04-27) is blocked
- [ ] Booking on a cycle test week date is blocked AND StudyRequired is locked
- [ ] Daily limit rule fires when trying to book a 3rd assessment for same class/day
- [ ] Data resets on browser refresh (confirming in-memory store, not persistence)

---

**Phase 1 — Foundation (do this after Phase 0b — Phase 0 can wait for IT admin)**
1. Scaffold the Vite + React project structure
2. Install all npm dependencies including Motion library
3. Create src/styles/design-system.css — import Google Fonts (DM Serif Display,
   DM Mono, Outfit), define all CSS custom properties from Section 5b,
   set base body styles, create .study-required and .study-none utility classes,
   define animation keyframes
4. Configure tailwind.config.js to extend theme with CSS variable references
5. Create .env.example with all required variables
6. Create authConfig.js and graphConfig.js with placeholder values
7. Create a working MSAL authentication wrapper in App.jsx styled to design system
8. Build StudyRequiredToggle.jsx — standalone component with locked/unlocked states,
   amber/jade colour treatment, 📚 and ✓ icons, smooth thumb animation
9. Test: `npm run dev` shows a login screen styled with DM Serif Display heading
   and Outfit body font — NOT the default browser styling

**Phase 2 — Data Layer (do this second)**
7.  Implement useGraph.js with all Graph API calls
8.  Implement useAssessments.js using useGraph
9.  Implement useCycleTests.js using useGraph
10. Implement usePublicHolidays.js using useGraph
11. Implement useSchoolCalendar.js using useGraph
12. Implement useAppSettings.js using useGraph (with getSetting helper)
13. Implement useReferenceData.js using useGraph (with getOptions helper)
14. Implement useUserRole.js — check SharePoint Group membership on login
15. Implement useTeacherProfile.js — fetch/save teacher profile, detect
    first-time users (profileComplete = false → redirect to TeacherProfile)
16. Implement useArchive.js — archive trigger detection and workflow
17. Implement useConfirmation.js — fetch/write ConfirmationLog rows,
    expose confirmGrade(), unconfirmGrade(), and isStale detection
18. Implement isAssessmentLocked() helper in conflictRules.js —
    checks ConfirmationLog to determine if an assessment is locked
19. Generate seed scripts (seedReferenceData.js, seedAppSettings.js,
    seedPublicHolidays.js) — TeacherProfiles and ConfirmationLog need no seed data
20. Test: all hooks fetch correctly from the lists created in Phase 0
21. Test: getSetting('MAX_ASSESSMENTS_PER_DAY') returns 2 (Number)
22. Test: useTeacherProfile returns profileComplete: false for new teacher
23. Test: useConfirmation returns isConfirmed: false for new grade/term combination
24. Test: isAssessmentLocked returns false when teacher has not confirmed
25. Test: isAssessmentLocked returns true when teacher has confirmed and IsStale is false
26. Test: isAssessmentLocked always returns false for Admin and Management roles

**Phase 3 — Rule Engine (do this third)**
21. Implement conflictRules.js with all rule functions including enforceStudyRequired
22. Implement calculateEasterDates(year) utility function
23. Implement useConflictChecker.js
24. Write unit tests for each rule function in conflictRules.test.js
25. Test: all rule scenarios pass (see test cases in Section 11)
26. Test: calculateEasterDates(2025) → { goodFriday: '2025-04-18', familyDay: '2025-04-21' }
27. Test: checkPublicHoliday correctly blocks 2026-08-10 (Women's Day substitute)
28. Test: enforceStudyRequired returns locked=true for dates within CycleTests
29. Test: enforceStudyRequired returns locked=false for non-cycle-test dates
30. Test: validateBooking correctedAssessment always has StudyRequired="Requires Study"
    when date falls in a cycle test week, regardless of what teacher submitted

**Phase 4 — UI Components (do this fourth)**
31. Build NavBar.jsx — role-aware navigation including Confirm link for teachers,
    Confirmation Report link for Management, My Profile link for all roles
32. Build AccessDenied.jsx — shown for unauthorised route access
33. Build ArchiveBanner.jsx — persistent management banner when archive is due
34. Build TeacherProfile.jsx — profile setup screen
35. Build FilterBar.jsx — subject/status filter within a grade view
36. Build useTermGrid.js hook — grade-scoped week columns and cell data
37. Build TermGrid.jsx — right panel with grade-scoped data
38. Build CalendarView.jsx — term-scoped, grade-scoped split-screen with
    confirmation status banner below grade switcher (green=confirmed,
    amber=stale, none=not yet confirmed)
39. Build BookingForm.jsx — grade-aware field scoping, read-only when
    assessment is locked by confirmation (isAssessmentLocked = true).
    CRITICAL: implement the live daily limit indicator:
    - Triggers as soon as both Class/Group AND Date are selected
    - Reads from in-memory assessments — no extra API call
    - State 1 (0 existing): green indicator, Save enabled
    - State 2 (1 existing): amber indicator showing existing booking, Save enabled
    - State 3 (2 existing): red indicator showing both bookings, Save DISABLED for teachers,
      enabled for Admins/Management with override banner and audit log entry
    Also implement the cycle test day indicator:
    - Triggers when Date is selected, checks CycleTests list for that grade
    - Rose indicator: "Cycle test day — no new bookings on this day. You can book other days this week."
    - Save DISABLED for teachers, enabled for Admins/Management (logged)
    ConflictWarning modal (belt-and-braces) still fires at save time for
    concurrent booking race conditions
40. Build ConfirmSchedule.jsx — per-grade confirmation screen:
    - Read-only assessment list grouped by grade
    - Confirm button per grade (disabled if no Scheduled assessments)
    - Reopen button after confirmation with optional reason field
    - Stale banner when IsStale = true
    - Empty state: "No assessments booked yet — go to Calendar to add some"
41. Build ConfirmationReport.jsx — management screen:
    - Term selector (default current term)
    - Grade filter
    - Summary counts (confirmed / not confirmed / stale)
    - Preview table with two sections
    - Download PDF button (generateConfirmationReport())
42. Build ConflictWarning.jsx
43. Build AssessmentList.jsx
44. Build TermArchiveManager.jsx (management only)
45. Build PolicyManager.jsx (management only)
46. Build CycleTestManager.jsx (admin + management only)
47. Build SchoolCalendarManager.jsx (management only)
48. Build ReferenceDataManager.jsx (management only)
49. Build ExportButton.jsx — three-option dropdown:
    Excel (assessment data), PDF (schedule), PDF (student distribution report)
    Build generateStudentReport() in exportHelpers.js
    Build generateConfirmationReport() in exportHelpers.js
50. Test: teacher with no assessments sees disabled Confirm button
51. Test: teacher confirms Grade 10 — ConfirmationLog row created with IsConfirmed=true
52. Test: confirmed assessment shows lock icon on calendar and grid
53. Test: clicking locked assessment shows "This schedule has been confirmed" banner
54. Test: Admin can edit a locked assessment (confirmation does not block admins)
55. Test: new assessment added after confirmation sets IsStale=true
56. Test: stale confirmation shows amber banner on ConfirmSchedule screen
57. Test: teacher un-confirms — IsConfirmed=false, assessments become editable
58. Test: Confirmation PDF contains all three sections (confirmed / not confirmed / stale)
59. Test: new teacher with no profile is redirected to TeacherProfile on login
60. Test: teacher who teaches Grade 10 and 11 sees grade switcher tabs
61. Test: Grade 10 calendar shows ALL Grade 10 assessments (10A + 10B + 10C)
62. Test: BookingForm Class/Group dropdown shows only teacher's own classes
63. Test: live grid preview highlights target cell yellow as teacher fills form
64. Test: after saving, badge appears immediately in grid and calendar
65. Test: archive trigger fires when today > term EndDate and term not archived
66. Test: full user journey for each of the three roles end to end

**Phase 5 — Polish & Deployment (do this last)**
52. Add loading states to all data-fetching operations
53. Add error boundaries for graceful failure
54. Desktop only — minimum supported width 1280px. No responsive
    breakpoints or mobile layouts required. Skip all @media queries
    except prefers-reduced-motion (accessibility — keep this one).
55. Run `npm run build` and verify the /dist folder is complete
56. Generate `DEPLOY.md` — step-by-step SharePoint deployment guide for IT admin:

```
DEPLOY.md must include:

STEP 1 — BUILD
  Run: npm run build
  This creates the /dist folder. Do not modify any files inside it.

STEP 2 — UPLOAD TO SHAREPOINT
  1. Go to your SharePoint site → Site Contents → Site Assets
  2. Create a new folder: AssessmentApp
  3. Upload the entire contents of /dist into Site Assets/AssessmentApp/
  4. Note the URL of index.html:
     https://[yoursite].sharepoint.com/sites/[site]/SiteAssets/AssessmentApp/index.html

STEP 3 — CREATE THE APP PAGE
  1. Go to your SharePoint site → New → Page
  2. Name it: Assessment Manager
  3. Add an Embed web part to the page
  4. Paste the index.html URL into the Embed web part
  5. Publish the page
  6. Copy the published page URL — this is what teachers will use

STEP 4 — UPDATE THE REDIRECT URI
  1. Go to portal.azure.com → Microsoft Entra ID → App registrations
  2. Open School Assessment Manager
  3. Go to Authentication → Platform configurations → Single-page application
  4. Add the published SharePoint page URL as a new Redirect URI
  5. Save
  (Keep http://localhost:5173 as well for future development)

STEP 5 — ADD TO SHAREPOINT NAVIGATION
  1. Edit the SharePoint site navigation
  2. Add a link: label "Assessment Manager", URL = published page URL
  3. Save navigation

STEP 6 — TEST WITH PILOT GROUP
  Add 3-4 teachers to the Assessment Teachers SharePoint Group
  Ask them to open the SharePoint page and verify:
  - App loads without errors
  - They can see the calendar
  - They can book an assessment
  - The booking appears on the calendar
  - The conflict rules work correctly

STEP 7 — SCHOOL-WIDE ROLLOUT
  Add all teaching staff to the Assessment Teachers SharePoint Group
  Communicate the SharePoint page link to all staff
  Most teachers will already be authenticated via Microsoft 365

UPDATING THE APP IN FUTURE
  1. Make code changes
  2. Run: npm run build
  3. Upload new /dist contents to Site Assets/AssessmentApp/ (overwrite)
  4. Teachers refresh their browser — they are on the latest version immediately
  No re-sharing links. No reinstallation. No downtime.
```

57. Test: deployed app on SharePoint page loads and authenticates correctly
58. Test: redirect URI updated in Azure — no auth errors on SharePoint URL

# ══════════════════════════════════════════════════════════════════
# SECTION 11: TEST CASES
# ══════════════════════════════════════════════════════════════════
# Claude Code must verify these scenarios work correctly.

## Rule Engine Test Cases (conflictRules.test.js)

```javascript
// Test 1: Should ALLOW booking when no other assessments exist on that day
checkDailyLimit({ classGroup: "Grade 10A", assessmentDate: "2025-06-10" }, [], 2)
// Expected: { allowed: true, count: 0 }

// Test 2: Should ALLOW booking when 1 assessment already exists (limit is 2)
checkDailyLimit(
  { classGroup: "Grade 10A", assessmentDate: "2025-06-10" },
  [{ classGroup: "Grade 10A", assessmentDate: "2025-06-10", title: "Maths Test" }],
  2
)
// Expected: { allowed: true, count: 1 }

// Test 3: Should BLOCK booking when 2 assessments already exist
checkDailyLimit(
  { classGroup: "Grade 10A", assessmentDate: "2025-06-10" },
  [
    { classGroup: "Grade 10A", assessmentDate: "2025-06-10", title: "Maths Test" },
    { classGroup: "Grade 10A", assessmentDate: "2025-06-10", title: "English Test" }
  ],
  2
)
// Expected: { allowed: false, count: 2 }

// Test 4: Should ALLOW booking for different class on same day
checkDailyLimit(
  { classGroup: "Grade 10B", assessmentDate: "2025-06-10" },
  [
    { classGroup: "Grade 10A", assessmentDate: "2025-06-10", title: "Maths Test" },
    { classGroup: "Grade 10A", assessmentDate: "2025-06-10", title: "English Test" }
  ],
  2
)
// Expected: { allowed: true, count: 0 }

// Test 5: Should BLOCK booking on a formal assessment date
checkCycleTestConflict(
  "2025-11-05",
  [{ startDate: "2025-11-03", endDate: "2025-11-14", title: "Grade 10 Mid-Year Exams" }]
)
// Expected: { blocked: true, reason: "Grade 10 Mid-Year Exams (3 Nov – 14 Nov 2025)" }

// Test 6: Should ALLOW booking the day AFTER a cycle test week ends
checkCycleTestConflict(
  "2025-11-15",
  [{ startDate: "2025-11-03", endDate: "2025-11-14", title: "Grade 10 Mid-Year Exams" }]
)
// Expected: { blocked: false, reason: null }
```

## UI Test Scenarios (manual verification)

| # | Scenario | Expected Result |
|---|----------|-----------------|
| 1 | Teacher clicks a blocked (formal) date on the calendar | Date picker disables it; cannot be selected |
| 2 | Teacher tries to book a 3rd assessment for Grade 10A on same day | ConflictWarning modal appears; save is blocked |
| 3 | Teacher books an assessment — it appears on calendar immediately | Optimistic UI update before Graph API confirms |
| 4 | Admin navigates to Cycle Test Manager | CycleTestManager renders; regular teacher sees "Access Denied" |
| 5 | Teacher clicks Export → Excel | .xlsx downloads with currently filtered assessments |
| 6 | Teacher clicks Export → PDF | .pdf downloads with formatted schedule |
| 7 | App loads at 1280px viewport width | Split layout renders correctly — left panel 60%, right panel 40% |
| 8 | Network connection drops mid-session | Error message appears; data is not lost |
| 9 | Teacher selects "Grade 10" in the grade filter | Calendar shows only Grade 10 assessments AND grid shows only Grade 10A/B/C rows |
| 10 | Teacher clicks an empty cell in the Grade 10A row, Week 3 column | BookingForm opens pre-filled with ClassGroup=Grade10A and a date within Week 3 |
| 11 | Teacher clicks a badge in the term grid | BookingForm opens in edit mode for that specific assessment |
| 12 | Teacher fills in ClassGroup and Date in BookingForm | The target cell in the term grid highlights yellow before save |
| 13 | Teacher saves a new assessment | Badge appears immediately in the correct grid cell without page refresh |
| 14 | A week has 2 assessments for Grade 10A (at the daily limit) | Grid cell shows two badges; no red warning icon |
| 15 | Data error: somehow 3 assessments exist for same class on same day | Grid cell shows red warning icon; cell is visually flagged |
| 16 | Teacher clicks Export on the term grid | .xlsx downloads with week columns and assessment names in each cell |
| 17 | Teacher changes term selector in grid from Term 1 to Term 2 | Grid columns update to Term 2 week dates; badges update accordingly |
| 18 | A week column contains a public holiday | Orange dot appears in that column header |
| 19 | Brand new teacher logs in for the first time | App redirects to TeacherProfile screen with message "Before you can book, please set up your profile" |
| 20 | Teacher selects Grade 10, Grade 11, and Grade 12 on profile screen | DefaultGrade radio button appears asking which to open first |
| 21 | Teacher saves profile with Grade 10, 11, and 12 selected | On next login, grade switcher tabs show [Grade 10] [Grade 11] [Grade 12] |
| 22 | Teacher with one grade (Grade 10 only) logs in | No grade switcher shown — Grade 10 calendar loads directly |
| 23 | Teacher with two grades clicks Grade 11 or Grade 12 tab | Both calendar AND term grid replace with that grade's data |
| 24 | Teacher clicks My Profile link in NavBar | TeacherProfile form opens pre-filled with current values |
| 25 | Teacher updates their classes and saves | BookingForm Class/Group dropdown immediately reflects the change |
| 26 | Teacher tries to navigate back before the term start | Back arrow (‹) is disabled — cannot navigate outside current term |
| 27 | Teacher tries to navigate past the term end | Forward arrow (›) is disabled — cannot navigate outside current term |
| 28 | Today is between terms (school holiday) | Calendar area shows "School holidays — no active assessment term" and the next term start date |
| 29 | SchoolCalendar list is empty (first run) | Calendar area shows "No term windows have been configured yet" banner |
| 30 | Term has ended but archive not yet run | Teachers still see the ended term's calendar with an amber "Awaiting archive" banner |
| 26 | Teacher books a non-formal assessment | StudyRequired toggle defaults to ON (Requires Study), teacher can switch it OFF |
| 27 | Teacher books an assessment on a formal assessment date | StudyRequired toggle is greyed out and locked ON with amber banner "Formal assessment — study required" |
| 28 | Teacher tries to submit with StudyRequired = "No Study Required" on a cycle test day | validateBooking corrects it to "Requires Study" before save; locked state shown |
| 29 | Assessment saved with StudyRequired=true appears on calendar | 📚 icon visible on the calendar event pill |
| 30 | Assessment saved with StudyRequired=false appears on calendar | No 📚 icon on calendar event pill |
| 31 | Term grid badge for a study-required assessment | Amber top border visible on badge |
| 32 | Assessment list shows StudyRequired pills | "Requires Study" → amber pill with 📚; "No Study Required" → jade pill with ✓ |
| 33 | Formal assessment badge in grid | 🔒 icon next to amber pill in assessment list view |
| 34 | Teacher types 301 characters in Notes field | Character counter turns red, save button disabled |
| 35 | Teacher types exactly 300 characters in Notes | Counter shows "300 / 300" in red, save still allowed |
| 36 | Teacher types 249 characters in Notes | Counter shows "249 / 300" in normal colour |
| 37 | Teacher types 255 characters in Notes | Counter shows "255 / 300" in amber |
| 38 | Teacher generates Student Assessment Report | PDF downloads with assessments grouped by subject (A-Z), then class (A-Z), then date (asc) |
| 39 | Assessment has no notes — student report | No sub-row rendered for that assessment |
| 40 | Assessment has notes — student report | Notes appear as indented italic sub-row below the assessment |
| 41 | Assessment falls in cycle test week — student report | Cycle test week annotation appears below that row |
| 42 | A subject has no assessments for a class group | That class group is omitted from that subject section entirely |
| 43 | Student report generated for Grade 10 in Term 1 | PDF filename is Grade10_Term1_2026_Assessment_Schedule.pdf |
| 44 | Assessment with status Cancelled | Excluded from student report entirely |
| 45 | Teacher selects Class/Group and Date with 0 existing assessments | Green indicator appears: "No assessments booked for 10A on this date" — Save enabled |
| 46 | Teacher selects Class/Group and Date with 1 existing assessment | Amber indicator: "1 of 2 slots used for 10A on [date]" listing existing booking with teacher name — Save enabled |
| 47 | Teacher selects Class/Group and Date with 2 existing assessments | Red indicator: "10A already has 2 assessments on [date] — full" listing both bookings — Save DISABLED |
| 48 | Admin selects Class/Group and Date with 2 existing assessments | Red indicator shown but Save button stays enabled with override banner — override is logged on save |
| 49 | Teacher selects a cycle test day | Rose indicator: "Cycle test day — [name]. No new bookings on this day. You can book other days this week." — Save DISABLED |
| 50 | Teacher changes date from a full day to a free day | Indicator immediately updates from red/rose to green — Save re-enables without page interaction |
| 51 | Teacher changes Class/Group after selecting a full date | Indicator recalculates for the new class — may switch from red to green if new class has free slots |
| 52 | Two teachers book simultaneously filling the last slot | Second teacher's save triggers ConflictWarning modal (server-side belt-and-braces) naming both conflicting assessments |
| 53 | Teacher opens booking form for Grade 11 (cycle tests not confirmed) | Form is locked — date picker and class list disabled, banner: "Cycle tests for Grade 11 not yet confirmed" |
| 54 | Teacher opens booking form for Grade 10 (cycle tests confirmed) | Form is fully interactive — no lock banner shown |
| 55 | Teacher selects multiple classes (10A + 10B + 10C) and saves | Three separate assessment records created, one per class |
| 56 | Teacher selects 10A + 10B where 10A already has 2 assessments that day | 10A skipped with warning, 10B saved successfully |
| 57 | Management confirms Grade 11 cycle tests in CycleTestManager | All Grade 11 teachers immediately see confirmed notification on next interaction |
| 58 | Management adds a cycle test after confirming Grade 10 | Grade 10 entry becomes stale, booking blocked with re-confirm warning |
| 59 | Management re-confirms Grade 10 after editing cycle test | Booking re-opens immediately for Grade 10 teachers |
| 60 | Teacher logs in with Grade 10 confirmed and Grade 11 not confirmed | Login banner shows green row for Grade 10, amber row for Grade 11 |
| 61 | Teacher logs in with all grades confirmed | Single green banner, auto-dismisses after 5 seconds |
| 62 | Management user logs in with unconfirmed grades | Red dot visible on Cycle Tests nav item |

# ══════════════════════════════════════════════════════════════════
# SECTION 12: THINGS TO CUSTOMISE BEFORE RUNNING CLAUDE CODE
# ══════════════════════════════════════════════════════════════════
# A summary of every ✏️ point in this file.
# Work through this checklist with your IT administrator first.

## Pre-Flight Checklist

### Path A — Local testing first (Azure not yet available)
- [ ] Install Node.js v18+ from nodejs.org (free)
- [ ] Create a `.env` file containing only: `VITE_USE_MOCK=true`
- [ ] Tell Claude Code to start with Phase 0b
- [ ] Run `npm run dev` — app opens at localhost:5173 with mock login
- [ ] Complete Phase 1 through Phase 5 using mock data
- [ ] When IT admin completes Azure registration, do Path B below

### Path B — Live SharePoint (when Azure is ready)
- [ ] ✏️ IT admin completes Azure App Registration (Section 4)
- [ ] ✏️ Fill in .env with Azure credentials and set VITE_USE_MOCK=false
- [ ] ✏️ Edit seed data in seedReferenceData.js with actual school subjects and classes
- [ ] ✏️ Run Phase 0 (setupSharePoint.js) — creates all 9 lists automatically
- [ ] ✏️ IT admin creates 3 SharePoint Groups manually:
         "Assessment Teachers", "Assessment Admins", "Assessment Management"
- [ ] ✏️ IT admin adds staff to each group
- [ ] ✏️ Management Team adds school term windows via School Calendar Manager
- [ ] ✏️ Management Team adds cycle test dates via Cycle Test Manager
- [ ] ✏️ After Phase 5, update Azure Redirect URI with the live SharePoint page URL
- [ ] ✏️ Pilot with 3–4 teachers before school-wide rollout

- [ ] ✏️ School name in the author field at the top
- [ ] ✏️ Rules 1–6: all confirmed as specified above ✅
- [ ] ✏️ Edit seed-data in seedReferenceData.js with your actual subjects,
         class groups, assessment types, and term names before running Phase 0
- [ ] ✏️ IT admin completes Azure App Registration (Section 4) — needed BEFORE Phase 0
- [ ] ✏️ Fill in .env file with Azure credentials and SharePoint site URL
         — needed BEFORE Phase 0 can run
- [ ] ✏️ Run Phase 0 (setupSharePoint.js) — this creates all 9 lists,
         the document library, and seeds all reference data automatically
- [ ] ✏️ IT admin creates 3 SharePoint Groups manually (Phase 0 does not create groups):
         "Assessment Teachers", "Assessment Admins", "Assessment Management"
- [ ] ✏️ IT admin adds all teaching staff to "Assessment Teachers"
- [ ] ✏️ IT admin adds HODs/coordinators to "Assessment Admins"
- [ ] ✏️ IT admin adds principal/deputy/management to "Assessment Management"
- [ ] ✏️ After Phase 5, update the Azure Redirect URI with the live SharePoint page URL
- [ ] ✏️ Add contact details for Access Denied screen (Section 7)
- [ ] ✏️ Management Team adds school term windows via School Calendar Manager
- [ ] ✏️ Management Team adds cycle test dates via Cycle Test Manager
- [ ] ✏️ Pilot with 3–4 teachers before school-wide rollout

## After Claude Code Builds the App

- [ ] Fill in the .env file with real Azure credentials
- [ ] Create the SharePoint Lists following SETUP.md
- [ ] Test with a small group of teachers before school-wide rollout
- [ ] Add formal assessment dates for the year to CycleTests list
- [ ] Share the SharePoint page URL with all teachers

# ════════════════════════════════════════════════════════════════════
# END OF CLAUDE.md
# ════════════════════════════════════════════════════════════════════
# WHAT TO SAY TO CLAUDE CODE TO START:
#
# IF YOU ARE WAITING FOR AZURE/IT ADMIN (start here):
# "Read CLAUDE.md thoroughly. Generate Phase 0b first — the complete
#  mock data layer so I can test locally without Azure. After Phase 0b
#  is complete, move to Phase 1. Confirm with me after each phase
#  before moving to the next. Do not generate all files at once."
#
# IF AZURE IS ALREADY REGISTERED (start here):
# "Read CLAUDE.md thoroughly. Begin Phase 0 — generate
#  scripts/setupSharePoint.js and show me the --dry-run output
#  for my approval before running it for real. Also generate Phase 0b
#  in parallel so I can test locally while SharePoint is being set up.
#  Confirm with me after each phase before moving to the next."
#
# IMPORTANT: Have your .env file set to VITE_USE_MOCK=true before
# starting Phase 0b. No other environment variables are needed for
# local mock testing.
# ════════════════════════════════════════════════════════════════════
