# Job Search Claude Skills

A fill-in-the-blanks kit for building a resume tailoring system with Claude. You give Claude a job description and a base resume. It grades the resume like a strict ATS, recommends edits pulled only from your own experience bank, builds a tailored one-page resume once you approve, and logs the application to your tracker (Notion or Google Drive).

**Start with the visual playbook:** [`docs/index.html`](docs/index.html). It walks through the whole system with diagrams. Download the file and open it in a browser, or turn on GitHub Pages (see below) to get a shareable link.

## The one lesson: build the experience bank first

A grader with no source of truth can only rewrite whatever resume you attach. The edits come out generic, it stretches stories too far, and you end up correcting its phrasing every session. When Claude can read the confirmed version of every story you have, it knows what's true and how far a story can be pushed toward a given role.

So build the system in this order:

| Stage | What you build | Where it lives in this repo |
|---|---|---|
| 1. Experience bank | One document holding every achievement: what actually happened, every version you've used on a resume, and what you can't claim | [`templates/master-experience-bank.md`](templates/master-experience-bank.md) |
| 2. Standards | What "good" looks like: Action-Context-Result bullets, one lead verb per bullet with no repeats, your banned words | Your own style guide + action-verb list, referenced by `resume-review` |
| 3. Grader | A strict ATS grade plus the 2 highest-ROI edits | [`skills/resume-review`](skills/resume-review/SKILL.md) |
| 4. Feedback loop | Every correction becomes either a fact (goes into the bank) or a rule (goes into the standards) | Your bank and standards docs |
| 5. Pipeline | Job description + base resume → tailored resume → logged application | [`skills/job-application-pipeline`](skills/job-application-pipeline/SKILL.md) |

## How it works

```
 WHAT YOU KNOW                         INPUT                        WHAT YOU GET
 ─────────────                  Job description +               ─────────────
 Experience bank                base resume (recommended)       Tailored resume
 Standards + verb list                   │                      Application tracker
 Standing rules                          ▼                              ▲
        │                   ┌──────────────────────────┐                │
        └──── (1) reads ───▶│ Claude                   │── (3) writes ──┘
                            │ grades · tailors · logs  │
                            │ you approve each write   │
                            └──────────────────────────┘
        ▲                                                               │
        └──────────── (4) your corrections update the bank and rules ◀──┘
```

1. Claude reads your experience bank, standards and rules.
2. You share a job description and, ideally, the base resume closest to the role.
3. Claude grades the resume and recommends edits. Once you approve, it builds the tailored resume and logs the application.
4. Each correction you make goes back into the bank or the rules, so the next run starts from a better place.

## What's in here

```
├── README.md
├── docs/
│   └── index.html                    Visual playbook (open in a browser)
├── templates/
│   └── master-experience-bank.md     Starting structure for stage 1
└── skills/
    ├── resume-review/                Stage 3: strict ATS grader
    ├── job-application-pipeline/     Stage 5: JD → tailored resume → tracker
    ├── decision-log/                 Optional: records project decisions and who made them
    └── decision-review/              Optional: reads the log back to find patterns
```

| Skill | What it does |
|---|---|
| `resume-review` | Grades a resume like a strict ATS: Action-Context-Result structure, one non-repeated lead verb per bullet, overlapping bullets, and keyword match when a JD is given. Returns a grade plus the 2 highest-ROI edits, not a full rewrite. |
| `job-application-pipeline` | Starts from a job description (link, text or file) plus a recommended base resume. Saves the JD as a PDF in a company folder, runs `resume-review` plus a JD-alignment pass, waits for your approval, builds the tailored `.docx`, and logs the application after you confirm. |
| `decision-log` / `decision-review` | A standalone pair for any project. One records each decision with its reasoning and who originated it; the other reads the log back to show where you lead and where you delegate. |

## Setup

### 1. Build your experience bank

Copy [`templates/master-experience-bank.md`](templates/master-experience-bank.md) into your Claude Project, Google Drive or Notion. Then give Claude every past resume you have, along with any interview-story notes, and use this prompt:

```
Build me a master experience bank from all the resumes and stories I've shared,
using the attached template. Group every bullet by role and achievement. For each
achievement, give the ground-truth story, list every framing I've used and tag it
by role type, and flag anything that conflicts across versions (numbers, dates,
titles) so I can pick the true one. Don't write a resume.
```

Go through the conflicts it flags and settle each one. Only then move on.

### 2. Pick where things live

| | Google Drive | Notion |
|---|---|---|
| Knowledge (bank, standards, verbs) | Google Docs + a PDF in a *Resume System* folder | Notion pages, plus an Interview Stories database that feeds the bank |
| Files (JD PDF, tailored resume) | One folder per company inside an *Applications* folder | One folder per company on your computer |
| Tracker | A Google Sheet, one row per application | Companies + Applications databases |
| Best if | You want everything in one place | You already use Notion or want linked databases and board views |

Connect the matching connector (Google Drive or Notion) in Claude.

### 3. Fill in the placeholders

Every `SKILL.md` ends with an HTML comment listing what to fill in. In short:

- **`resume-review`**: `<YOUR_MASTER_RESUME_DOC>`, `<YOUR_STYLE_GUIDE_LINK>`, `<YOUR_ACTION_WORDS_FILE>` (optional)
- **`job-application-pipeline`**: `<YOUR_RECRUITING_FOLDER_PATH>`, `<YOUR_MASTER_RESUME_DOC>` (use the same one as `resume-review`), `<Your_Name>`, `<YOUR_TRACKING_SYSTEM>`. In Step 6, keep the Notion or the Sheet path and delete the other.
- **`decision-log` / `decision-review`**: `<YOUR_WORKSPACE>` and your own categories

These are plain-English instructions, so filling them in means editing a sentence or two. There's no code to change.

### 4. Install the skills

Put each skill folder where Claude loads skills (your Skills settings, a Claude Project, or a Cowork plugin's skills folder) so each file sits at `<skills-dir>/<skill-name>/SKILL.md`. Claude picks each skill up from its `description` line, so there's no separate registration step.

### 5. Tune before you automate

Run `resume-review` on a few resumes first. Once its output stops surprising you, start using `job-application-pipeline`.

## Pitfalls

- **Grader before bank.** You'll get generic edits. Build the bank first, even a rough one.
- **More than one copy of the truth.** Keep exactly one bank and one rules list, and point both skills at the same bank.
- **Reusing old framings.** Tailor from the ground-truth story every time. Rewording an old tailored bullet produces near-duplicate resumes.
- **Claiming targets as results.** In the bank, mark each number as a measured outcome or a goal.
- **Unscreened sensitive info.** Mark confidential figures and internal names in the bank so they never reach a resume.

## Share the playbook with a link (optional)

In your repo on GitHub, go to **Settings → Pages**. Under *Build and deployment*, choose **Deploy from a branch**, then pick `main` and the `/docs` folder, and save. After a minute or two the playbook is live at `https://<your-username>.github.io/<repo-name>/`.

## License

MIT. See [LICENSE](LICENSE). Use whatever helps; grading opinions come with no warranty.
