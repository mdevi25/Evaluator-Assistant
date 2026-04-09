# TM Evaluator Assistant

A quiet companion for sharper Toastmasters evaluations.

This is an AI-powered tool that listens to speeches during club meetings and prepares structured evaluation talking points for evaluators using the Toastmasters CRC (Commend, Recommend, Commend) framework. The evaluator still delivers their own evaluation — the AI provides a cheat sheet of observations to draw from.

**Guiding principle:** AI should elevate the Toastmasters experience, not replace it.

---

## How it works

1. Evaluator opens the app on their phone and enters the club access code
2. Selects the speaker's Pathways project (pathway, level, project)
3. Taps "Start listening" when the speech begins
4. The app transcribes live, counts filler words, and tracks timing
5. When the speech ends, taps "Done — Analyze"
6. AI returns structured talking points: project objectives assessment, commendations with specific moments, actionable recommendations, and a framing note
7. Evaluator glances at the screen and delivers their verbal evaluation in their own words

Nothing is recorded. Nothing is stored. Everything is gone when the tab closes.

---

## What's in this repo

```
tm-evaluator-secure/
  netlify.toml                    ← Netlify configuration
  README.md                       ← This file
  public/
    index.html                    ← The entire app (UI + Pathways data + logic)
  netlify/
    functions/
      evaluate.js                 ← Secure API proxy (API key stays server-side)
```

---

## Tech stack

- **Frontend:** Single HTML file, vanilla JavaScript, no framework
- **Fonts:** DM Sans (body) + Fraunces (headings)
- **Speech-to-text:** Browser Web Speech API (Chrome/Edge required, no audio sent anywhere)
- **AI:** Claude Sonnet via Anthropic API
- **Proxy:** Netlify serverless function (keeps API key off the client)
- **Hosting:** Netlify free tier
- **Storage:** None. Zero. Deliberately.

---

## Pathways knowledge base

The app includes all **six current Toastmasters paths** (sourced from toastmasters.org, April 2026):

| Path | Focus |
|------|-------|
| Dynamic Leadership | Strategic leadership, conflict, change management |
| Engaging Humor | Comedic style, humor techniques, impromptu humor |
| Motivational Strategies | Connection, emotional intelligence, team building |
| Persuasive Influence | Negotiation, conflict resolution, difficult situations |
| Presentation Mastery | Body language, persuasion, audience management |
| Visionary Communication | Communication planning, change, long-term vision |

Each path includes all 5 levels with correct required project names. Legacy paths (Effective Coaching, Innovative Planning, Leadership Development, Strategic Relationships, Team Collaboration) are not included — they are no longer available for new purchases.

The Pathways data lives in the `const PATHWAYS = {...}` object inside `public/index.html`. To update projects or add new paths, edit that object and push to GitHub — Netlify auto-redeploys.

---

## Deployment

### Prerequisites

1. **Anthropic account** — [console.anthropic.com](https://console.anthropic.com)
   - Create an API key (starts with `sk-ant-...`)
   - Add $5 credit under Billing (lasts months for a small club)

2. **GitHub account** (free) — [github.com](https://github.com)

3. **Netlify account** (free) — [app.netlify.com](https://app.netlify.com)
   - Sign up with GitHub for easiest setup

### Step 1: Upload to GitHub

1. Create a new **private** repository
2. Upload all files and folders maintaining the folder structure
3. Commit changes

### Step 2: Deploy on Netlify

1. Go to [app.netlify.com](https://app.netlify.com)
2. Click **Add new site** → **Import an existing project** → **GitHub**
3. Select your repository
4. Verify settings (auto-detected from `netlify.toml`):
   - Publish directory: `public`
   - Functions directory: `netlify/functions`
5. Click **Deploy site**

### Step 3: Add the API key

1. In Netlify → your site → **Site configuration** → **Environment variables**
2. Add a variable:
   - Key: `ANTHROPIC_API_KEY`
   - Value: your API key
3. Go to **Deploys** → **Trigger deploy** → **Deploy site**

### Step 4: Customize your URL

1. **Site configuration** → **Domain management** → **Change site name**
2. Enter a meaningful name (e.g., `tm-evaluator-yourclub`)
3. Your URL: `https://tm-evaluator-yourclub.netlify.app`

### Step 5: Test

1. Open the URL in Chrome on your phone
2. Enter the club access code
3. Select a pathway and project
4. Tap "Start listening" and speak for 20-30 seconds
5. Tap "Done — Analyze"
6. Confirm evaluation results appear within 10 seconds

---

## Access control

The app is protected by a club access code screen. Visitors must enter the correct code before accessing the app. To change the code, edit `public/index.html`, search for `CLUB_PASSWORD`, and update the value. Share the code only with club members through private channels.

---

## Tracking usage

To monitor how many evaluations are being run (useful for justifying costs during experimentation):

1. Log into [app.netlify.com](https://app.netlify.com)
2. Click on your site
3. Click **Logs** in the top navigation
4. Select **Functions** from the dropdown
5. Click **evaluate**

Each log entry = one evaluation = one API call ≈ $0.03. The log shows timestamps for every invocation. No additional setup needed — Netlify logs this automatically.

---

## Updating

### Update the app or Pathways data

1. Go to your GitHub repository
2. Navigate to `public/index.html`
3. Click the pencil icon to edit
4. Make changes → **Commit changes**
5. Netlify auto-redeploys within ~30 seconds

### Change the API key

1. Netlify → Site configuration → Environment variables
2. Edit `ANTHROPIC_API_KEY`
3. Trigger a redeploy

### Change the club access code

1. Edit `public/index.html` in GitHub
2. Find `CLUB_PASSWORD` and change the value
3. Commit — Netlify redeploys automatically
4. Share the new code with members through private channels

---

## Taking the site down

- **Temporarily:** Deploys → Lock to stop auto publishing
- **Permanently:** Site configuration → scroll to bottom → Delete this site

The URL stops working immediately. No data cleanup needed — nothing was stored.

---

## Costs

| Service | Cost |
|---------|------|
| Netlify hosting | Free (125,000 function calls/month, 100GB bandwidth) |
| GitHub private repo | Free |
| Anthropic API | ~$0.02-0.04 per evaluation |
| **Monthly estimate (small club)** | **~$0.15-0.30** |

The $5 minimum Anthropic credit lasts approximately 1.5-2 years at small club usage (4 evaluations/month).

---

## Privacy

- No audio is ever recorded or stored
- Transcripts exist only in browser memory during the session, then are cleared
- The AI analysis is sent via encrypted HTTPS; Anthropic does not retain API request content
- No database, no cookies, no localStorage for any sensitive data
- No user accounts or tracking
- Speakers should be informed that the tool is in use — transparency is a Toastmaster value

---

## Architecture

```
Speaker's voice
    ↓
Phone microphone → Browser Web Speech API (audio to text, on-device)
    ↓
Live transcript (JavaScript variable in browser memory)
    ↓
"Done — Analyze" button pressed
    ↓
Transcript sent to → Netlify function (evaluate.js) → adds API key → Anthropic API
    ↓
Claude Sonnet analyzes against TM criteria + Pathways project objectives
    ↓
JSON response → rendered as CRC talking points on evaluator's screen
    ↓
Transcript cleared from memory. Tab closed = everything gone.
```

No audio leaves the device. Only the text transcript is sent to the API, once, encrypted, and not retained.

---

## License

This is a club experiment tool built for the Toastmasters community. The Pathways project names referenced in the app are from the publicly available Toastmasters International website. Toastmasters International, the Toastmasters International logo, and all other Toastmasters International trademarks and copyrights are the sole property of Toastmasters International.
