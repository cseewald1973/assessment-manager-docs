# Local Installation Guide — Assessment Date Manager
# ════════════════════════════════════════════════════════════════════
# This guide installs and runs the app on your Windows 11 laptop
# in LOCAL TEST MODE. No Azure or SharePoint connection needed.
# Everything runs on your machine using mock data.
#
# When IT admin completes the Azure App Registration, a separate
# guide will cover connecting the app to SharePoint.
# ════════════════════════════════════════════════════════════════════

## What you need before starting

- [x] Windows 11
- [x] VS Code installed
- [x] Node.js v20 or higher (check: type `node --version` in terminal)
- [x] Claude Code installed (check: type `claude --version` in terminal)
- [x] The AssessmentManager folder open in VS Code
- [x] CLAUDE.md saved into the AssessmentManager folder

---

## Starting the app (after first install)

Every time you restart your machine:

1. Open **VS Code**
2. Open the **AssessmentManager** folder (File → Open Folder)
3. Press **Ctrl + `** to open the terminal
4. Type this and press Enter:
   ```
   cd assessment-app
   ```
5. Type this and press Enter:
   ```
   npm run dev
   ```
6. Wait until you see: `Local: http://localhost:5173/`
7. Open **Chrome** and go to `http://localhost:5173`

---

## First-time installation (run once only)

### Step 1 — Install Node.js
Go to **nodejs.org**, click **Get Node.js**, download the
**Windows Installer (.msi)** and run it. Click Next on every screen.

### Step 2 — Fix PowerShell script permissions
Open VS Code terminal and run:
```
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```
Type Y and press Enter when prompted.

### Step 3 — Install Claude Code
```
npm install -g @anthropic-ai/claude-code
```
Note: If on school network and you get a certificate error, use your
phone's mobile hotspot for this step only.

### Step 4 — Open your project folder
- File → Open Folder → select your AssessmentManager folder
- Make sure CLAUDE.md is visible in the left panel

### Step 5 — Start Claude Code and build the app
In the terminal type:
```
claude
```
Then paste this instruction:
```
Read CLAUDE.md thoroughly. We are running in LOCAL MOCK MODE — there is no Azure or SharePoint connection. Start with Phase 0b only (the complete mock data layer). After Phase 0b is complete and the app runs in the browser, confirm with me before moving to Phase 1. Do not generate all files at once.
```
Keep accepting when prompted. This takes 2-4 hours across multiple sessions
(Claude Code has rate limits that reset every 5 hours).

### Step 6 — Create the .env file
When Claude Code finishes Phase 0b, find `.env.example` in the left
panel, copy it, rename the copy to `.env`, and make sure it contains:
```
VITE_USE_MOCK=true
```

### Step 7 — Install dependencies and run
```
cd assessment-app
npm install
npm run dev
```
Open `http://localhost:5173` in Chrome.

---

## Test users

| User | Role | What they can do |
|---|---|---|
| Mr Christopher Uren | Teacher | Grade 10, 11, 12 — booking open for Gr10 & 12, blocked for Gr11 |
| Ms Sarah Dlamini | Teacher | Grade 10 — confirmed but stale |
| Mr James Mokoena | Teacher | Grade 8 and 9 |
| New Teacher | Teacher | Incomplete profile — tests first-login flow |
| Mrs Patricia Nkosi | Admin | Cycle Test Manager, can override rules |
| Mr David van Wyk | Management | All screens including Policy, Archive, Confirmations |

---

## Mock data included

- 37 assessments across all grades and teachers
- 6 cycle test dates across Grades 8–12
- 5 cycle test confirmation states (confirmed, not confirmed, stale)
- All 4 term windows for 2026
- SA public holidays 2025 and 2026
- Full reference data: 7 subjects, class groups Gr8–Gr12, 7 assessment types

---

## Assessment types (teacher-bookable)

| Type | Abbreviation | Icon | Study Required |
|---|---|---|---|
| Class Test | ClsT | ✏️ | Auto-locked ON |
| Quiz | Qz | ⚡ | Default ON, can toggle |
| Assignment | Asn | 📎 | Default ON, can toggle |
| Practical | Pr | 🔬 | Default ON, can toggle |
| Oral | Or | 🎤 | Default ON, can toggle |
| Project | Prj | 📐 | Default ON, can toggle |
| Other | OT | 📋 | Default ON, can toggle |

Note: Cycle Tests are entered by Management via Cycle Test Manager only.

---

## Key features to test

1. **Login notification banner** — shows cycle test confirmation status per grade
2. **Booking gate** — try booking Grade 11 (blocked) vs Grade 10 (open)
3. **Multi-select classes** — tick multiple classes when booking
4. **Live conflict indicator** — select a date to see slots used
5. **Cycle test day block** — try booking on a cycle test date
6. **Confirm Schedule** — confirm a grade and see assessments lock
7. **Cycle Test Manager** — confirm/reopen per grade (Management login)
8. **Legend button** — top right of nav bar, explains all colours and icons
9. **Confirmation Report** — shows confirmed/not confirmed/stale teachers
10. **Term Archive Manager** — 4-step end-of-term workflow

---

## Common problems

**App not loading after restart**
You need to run `npm run dev` every time. See "Starting the app" above.

**Port already in use**
The app is already running. Just open `http://localhost:5173` in Chrome.
If that doesn't work, try `http://localhost:5174` or `http://localhost:5175`.

**npm certificate error on school network**
Use your phone's mobile hotspot for npm install commands only.

**Claude Code says "rate limit reached"**
Normal — limits reset every 5 hours. Select option 1 (wait) and
type `continue` when ready.

**Changes disappeared after restart**
Expected — mock data resets on restart. Data persists once connected to SharePoint.

---

## Sharing the app on your local network (no install needed on other machines)

To let colleagues view the app on the same wifi:

```
cd assessment-app
npm run build
npm run preview -- --host
```

This gives a network address like `http://192.168.1.45:4173` that anyone
on the same wifi can open in their browser — no installation required.

---

## Next steps — connecting to SharePoint

Once IT admin completes the Azure App Registration:

1. Run `node scripts/setupSharePoint.js` to create all 11 SharePoint lists
2. Update `.env` with Azure credentials provided by IT admin
3. Set `VITE_USE_MOCK=false` in `.env`
4. Run `npm run build` to create the production build
5. Upload the `dist` folder to SharePoint
6. Teachers access via the SharePoint page URL

