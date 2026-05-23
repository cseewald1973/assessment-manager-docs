# Local Installation Guide — Assessment Date Manager
# ════════════════════════════════════════════════════════════════════
# This guide installs and runs the app on your Windows 11 laptop
# in LOCAL TEST MODE. No internet connection to Azure or SharePoint
# is needed. Everything runs on your machine using mock data.
#
# When IT admin completes the Azure App Registration, a separate
# guide will cover connecting the app to SharePoint.
# ════════════════════════════════════════════════════════════════════

## What you need before starting

- [x] Windows 11
- [x] VS Code installed
- [x] Node.js v20 or higher installed (check: node --version in terminal)
- [x] Claude Code installed (check: claude --version in terminal)
- [x] The AssessmentManager folder open in VS Code
- [x] CLAUDE.md saved into the AssessmentManager folder

---

## Step 1 — Open the terminal in VS Code

1. Open VS Code
2. Make sure the **AssessmentManager** folder is open
   (you should see it in the left panel — if not, File → Open Folder)
3. Press **Ctrl + `** (backtick, top-left of keyboard) to open the terminal
4. The terminal should show your folder path, something like:
   `PS C:\Users\Christopher\Documents\AssessmentManager>`

---

## Step 2 — Start Claude Code

In the terminal, type this and press Enter:

```
claude
```

Claude Code will start. You will see a welcome message and a prompt
waiting for your input.

---

## Step 3 — Give Claude Code its instructions

When Claude Code is waiting, paste this message exactly:

```
Read CLAUDE.md thoroughly. We are running in LOCAL MOCK MODE — there is no Azure or SharePoint connection. Start with Phase 0b only (the complete mock data layer). After Phase 0b is complete and the app runs in the browser, confirm with me before moving to Phase 1. Do not generate all files at once.
```

Press Enter. Claude Code will now read the spec and start building.

**What to expect:**
- Claude Code will read CLAUDE.md (takes a few seconds)
- It will then start creating files — you will see them appear in the
  left panel of VS Code one by one
- It will ask you questions occasionally — answer them as best you can
  or say "use the default from the spec"
- This will take 10–20 minutes

Do not close VS Code or the terminal while it is working.

---

## Step 4 — Create the environment file

When Claude Code finishes Phase 0b, it will tell you.

Before running the app, you need to create one settings file.

1. In VS Code, look at the left panel (the file list)
2. You should see a file called `.env.example`
3. Right-click it and click **Copy**
4. Right-click in an empty space in the file list and click **Paste**
5. Rename the pasted copy to exactly: `.env`
   (just a dot, then env — no other characters)
6. Open the `.env` file by clicking on it
7. Make sure it contains this line:

```
VITE_USE_MOCK=true
```

If it does, save the file (Ctrl + S) and move to Step 5.

---

## Step 5 — Install the app dependencies

In the terminal (press **Ctrl + `** if it closed), type this and press Enter:

```
npm install
```

You will see a lot of text scroll past — this is normal. It is
downloading the packages the app needs. Wait until it stops and
the cursor returns.

This only needs to be done once.

---

## Step 6 — Run the app

In the terminal, type this and press Enter:

```
npm run dev
```

You will see something like:

```
  VITE v5.x.x  ready in 500ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
```

---

## Step 7 — Open the app in your browser

1. Open **Chrome** or **Edge**
2. In the address bar, type exactly:

```
http://localhost:5173
```

3. Press Enter

You should see the Assessment Manager login screen with a
**[DEV MODE]** amber banner at the top and a dropdown to select
a test user.

**If you see this — the app is working. **

---

## Choosing a test user

The dropdown on the login screen has six test users:

| User | Role | What they can do |
|---|---|---|
| Mr Christopher Uren | Teacher | Book assessments for Grade 10 and 11 |
| Ms Sarah Dlamini | Teacher | Book assessments for Grade 10 |
| Mr James Mokoena | Teacher | Book assessments for Grade 8 and 9 |
| New Teacher | Teacher | Incomplete profile — tests first-login flow |
| HOD Admin | Admin | Everything teachers can + manage cycle tests |
| Principal | Management | Everything + policy, archive, reports |

Start with **Mr Christopher Uren** to see the main teacher experience.

---

## Stopping the app

When you are done testing, go back to the terminal and press:

```
Ctrl + C
```

The app will stop. Your files are all saved — nothing is lost.

---

## Starting the app again next time

You only need to do Steps 6 and 7 next time:

1. Open VS Code with the AssessmentManager folder
2. Open the terminal (Ctrl + `)
3. Type `npm run dev` and press Enter
4. Open `http://localhost:5173` in your browser

---

## Common problems and fixes

**"npm : File cannot be loaded because running scripts is disabled"**
Run this in the terminal, then try again:
`Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned`

**"Port 5173 is already in use"**
The app is already running in another terminal window. Either use that
window, or close it and run `npm run dev` again.

**"Cannot find module" error**
Run `npm install` again, then `npm run dev`.

**The browser shows a blank white page**
Open the browser console (F12 → Console tab) and tell the developer
what error message appears in red.

**Changes I made in the app disappeared**
The mock data resets every time you stop and restart the app.
This is expected during local testing — data persistence comes
when the app connects to SharePoint.

---

## What the mock data includes

The app comes pre-loaded with realistic test data so you can explore
all features immediately:

- 4 assessments already booked across Grade 10 classes
- 5 cycle test dates across Grades 8–11
- All four term windows for 2026 configured
- South African public holidays for 2025 and 2026
- 6 test teachers with different profiles and confirmation states
- One teacher (Ms Dlamini) has a stale confirmation — good for testing

---

## Next steps after local testing

Once you and your IT admin are happy with how the app works:

1. IT admin completes the **Azure App Registration** (30 minutes)
2. Run `node scripts/setupSharePoint.js` to create the SharePoint lists
3. Update the `.env` file with the Azure credentials
4. Set `VITE_USE_MOCK=false`
5. Run `npm run build` and deploy to SharePoint

A separate deployment guide covers these steps.

