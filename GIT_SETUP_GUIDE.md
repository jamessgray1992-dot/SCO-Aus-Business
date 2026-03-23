# How to Pull Files to Desktop & Link to GitHub
## Complete Step-by-Step Guide for AUS BD & CRM + DRFA

---

## STEP 1: Download the files from Claude

### Files you need:
1. **drfa.html** — Download from the link Claude provided (click the file, then download)
2. **DRFA_INTEGRATION.md** — Download from the link Claude provided
3. **index.html** — Your existing AUS BD & CRM file (you should already have this)

### Where to put them:
Create a folder on your desktop (or wherever you want your project):

**Windows:**
```
C:\Users\YourName\Documents\AUS-BD-CRM\
├── index.html
├── drfa.html
├── DRFA_INTEGRATION.md
├── sw.js              (if you have a service worker)
├── manifest.json      (if you have a PWA manifest)
└── README.md
```

**Mac:**
```
/Users/YourName/Documents/AUS-BD-CRM/
├── index.html
├── drfa.html
├── DRFA_INTEGRATION.md
└── ...
```

---

## STEP 2: Install Git (if you haven't already)

### Windows:
1. Go to https://git-scm.com/download/win
2. Download and run the installer
3. Use all default settings (just keep clicking Next)
4. When done, open **Git Bash** from the Start menu

### Mac:
1. Open **Terminal** (Cmd + Space, type "Terminal")
2. Type: `git --version`
3. If not installed, it will prompt you to install Xcode Command Line Tools — click Install

### Verify it worked:
```bash
git --version
```
You should see something like `git version 2.40.0`

---

## STEP 3: Create a GitHub account & repository

### If you don't have a GitHub account:
1. Go to https://github.com
2. Click **Sign up**
3. Create your account (free is fine)

### Create a new repository:
1. Log into GitHub
2. Click the **+** icon (top right) → **New repository**
3. Fill in:
   - **Repository name:** `aus-bd-crm` (or whatever you want)
   - **Description:** `AUS BD & CRM - Australian Business Development & CRM`
   - **Public** or **Private** (your choice — Private means only you can see it)
   - **DO NOT** tick "Add a README file" (we'll push our own)
4. Click **Create repository**
5. You'll see a page with setup instructions — **keep this page open**, you'll need the URL

The URL will look like:
```
https://github.com/YOUR-USERNAME/aus-bd-crm.git
```

---

## STEP 4: Connect your local folder to GitHub

### Open your terminal/command prompt:

**Windows:** Open **Git Bash** (or Command Prompt)
**Mac:** Open **Terminal**

### Navigate to your project folder:

**Windows (Git Bash):**
```bash
cd AUS-BD-CRM
```

**Windows (Command Prompt):**
```bash
cd AUS-BD-CRM
```

**Mac:**
```bash
cd AUS-BD-CRM
```

### Initialize Git and push:

Run these commands one at a time:

```bash
# 1. Initialize the folder as a Git repo
git init

# 2. Set your identity (use your GitHub email)
git config user.name "Your Name"
git config user.email "your-email@example.com"

# 3. Add ALL files in the folder
git add .

# 4. Create your first commit
git commit -m "Initial commit: AUS BD & CRM with DRFA Intelligence module"

# 5. Name the main branch
git branch -M main

# 6. Connect to your GitHub repo (paste YOUR URL from Step 3)
git remote add origin https://github.com/YOUR-USERNAME/aus-bd-crm.git

# 7. Push everything to GitHub
git push -u origin main
```

### First time authentication:
- GitHub will ask you to log in
- A browser window may pop up asking you to authorize Git
- Click **Authorize** and you're done

---

## STEP 5: Verify it worked

1. Go to https://github.com/YOUR-USERNAME/aus-bd-crm
2. You should see all your files listed
3. Click on `drfa.html` or `index.html` to verify the code is there

---

## STEP 6: Daily workflow (Add, Commit, Push)

Every time you make changes, run these 3 commands:

```bash
# 1. Stage your changes (the dot means "everything")
git add .

# 2. Commit with a message describing what you changed
git commit -m "feat: added DRFA scoring improvements"

# 3. Push to GitHub
git push
```

### Common commit message examples:
```bash
git commit -m "feat: add DRFA Intelligence module"
git commit -m "fix: corrected scoring formula weights"
git commit -m "update: added 20 new LGAs to DRFA dataset"
git commit -m "feat: AUS-wide state selector"
git commit -m "fix: mobile responsive layout on DRFA table"
git commit -m "chore: updated Firebase config"
```

---

## STEP 7: Host it live (optional but recommended)

### Option A: GitHub Pages (FREE — easiest)

1. Go to your repo on GitHub
2. Click **Settings** (top tab)
3. Scroll down to **Pages** (left sidebar)
4. Under "Source", select **Deploy from a branch**
5. Select **main** branch, **/ (root)** folder
6. Click **Save**
7. Wait 1-2 minutes
8. Your site will be live at: `https://YOUR-USERNAME.github.io/aus-bd-crm/`

The DRFA tool will be at: `https://YOUR-USERNAME.github.io/aus-bd-crm/drfa.html`

### Option B: Netlify (FREE — auto-deploys)

1. Go to https://netlify.com
2. Sign up with GitHub
3. Click **Add new site** → **Import an existing project**
4. Select your `aus-bd-crm` repo
5. Click **Deploy site**
6. Every time you `git push`, it auto-deploys

### Option C: Firebase Hosting (you're already using Firebase)

```bash
# Install Firebase CLI
npm install -g firebase-tools

# Login
firebase login

# Initialize hosting
firebase init hosting
# Select your Firebase project
# Set public directory to: . (current folder)
# Configure as single-page app: No

# Deploy
firebase deploy --only hosting
```

---

## QUICK REFERENCE CARD

### First time setup (once only):
```bash
cd your-project-folder
git init
git config user.name "Your Name"
git config user.email "you@email.com"
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/USERNAME/REPO.git
git push -u origin main
```

### Every time you make changes:
```bash
git add .
git commit -m "describe what you changed"
git push
```

### Pull latest changes (if you edit on GitHub or another computer):
```bash
git pull
```

### Check what's changed:
```bash
git status
```

### See commit history:
```bash
git log --oneline
```

---

## TROUBLESHOOTING

### "Permission denied" or auth errors:
```bash
# GitHub now uses personal access tokens instead of passwords
# 1. Go to: GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)
# 2. Generate new token with "repo" scope
# 3. Use this token as your password when Git asks
```

### "fatal: not a git repository":
```bash
# You're not in the right folder. Navigate to your project:
cd AUS-BD-CRM
```

### "Updates were rejected":
```bash
# Someone else pushed changes (or you edited on GitHub). Pull first:
git pull --rebase origin main
# Then push again:
git push
```

### Want to undo your last commit (before pushing):
```bash
git reset --soft HEAD~1
```

### Want to see what files are in the repo:
```bash
git ls-files
```

---

## YOUR FILE STRUCTURE (after everything is set up)

```
AUS BD & CRM/
├── .git/                    ← Git tracking folder (hidden, don't touch)
├── index.html               ← Main AUS BD & CRM app
├── drfa.html                ← DRFA Intelligence module (NEW)
├── DRFA_INTEGRATION.md      ← Integration guide (NEW)
├── sw.js                    ← Service worker (if you have one)
├── manifest.json            ← PWA manifest (if you have one)
└── README.md                ← Project description
```

---

## NEXT TIME YOU GET FILES FROM CLAUDE

When I give you updated files:

1. Download the file from Claude
2. Copy/replace it into your `AUS BD & CRM` folder
3. Open terminal, navigate to the folder
4. Run:
```bash
git add .
git commit -m "update: description of what changed"
git push
```

That's it — 3 commands every time.
