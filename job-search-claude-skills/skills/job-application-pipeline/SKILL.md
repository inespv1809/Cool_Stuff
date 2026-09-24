---
name: "job-application-pipeline"
description: "Use when you share a job description (link, text, or file), ideally with a base resume, to tailor a resume or apply: saves the JD as a PDF, files it in a company folder, tailors the resume, and logs the application in your tracker."
---

# Job Application Pipeline

End-to-end flow from a job posting URL to a tailored resume and a logged application. Keep chat replies short: one or two sentences per step, no recaps.

## Inputs
- Job description (required): a posting URL, or the JD pasted as text or attached as a file
- Base resume (recommended): the version closest to this type of role, attached or taken from the recruiting folder. Starting from the closest version gives the best result; if none is given, use the most recent tailored resume in the recruiting folder
- Recruiting folder: `<YOUR_RECRUITING_FOLDER_PATH>` — a local folder (e.g. `/Users/<you>/Documents/Job Search`) or a Google Drive / cloud folder (request access if not connected)

## Step 1 - Save the JD as a PDF
0. If the JD was pasted or attached rather than linked, save that text as the PDF and skip to step 3.
1. Try to render the page directly (Playwright to PDF).
2. If the site is blocked, read it with WebFetch (verbatim sections: title, location, about, responsibilities, requirements, comp and benefits, additional info) and rebuild a clean PDF. Tell the user it is a rebuild and wording may differ slightly from the original.
3. Title: role + company. Include the source URL (tracking params such as `?gh_src=` removed) and the capture date.
4. Name the file `<Company>_<Role-short>_JD.pdf`.

## Step 2 - File it
- Create `<YOUR_RECRUITING_FOLDER_PATH>/<Company>/` if it does not exist. Save the JD PDF there.

## Step 3 - Review and recommend (do not edit yet)
1. Run the `resume-review` skill on the base resume against the JD. It owns grading, the ACR check, verb rules, and any style guide / action-words reference; do not duplicate those rules here.
2. Add a JD-alignment pass:
   - Pull the JD's key terms and requirements and note which the resume already covers and which are missing.
   - **Recommend** bullet swaps or rewrites that better match the JD, as before -> after, each tagged with the JD requirement it targets. Pull only from the experience bank (`<YOUR_MASTER_RESUME_DOC>`); never invent facts or metrics. Say which existing bullet each one would replace to keep the page at one page.
   - Flag any requirement not met (e.g., years of experience) plainly; never inflate to close the gap.
   - Flag JD tools and skills the user might have but hasn't listed, and ask before adding any.
3. **Wait for approval** on which recommendations to apply. Do not add bullets unilaterally.

## Step 4 - Build the tailored resume
- Apply only the approved changes. Keep any standing rules the user has given you (e.g. one page, one lead verb per bullet, no repeated or synonym lead verbs across the resume, banned words, bullets fit on one line).
- Update the location line to match the job location if desired (e.g., "City, State / City, State").
- Save as an editable Word file `<Your_Name>_<Company>_<Role-short>.docx` in the company folder (docx skill). Render a preview and check for one page and no wrapped bullets before delivering.

## Step 5 - Revisions
- Before every edit, list the company folder and check the resume's modified time and size. If the user has edited it or has it open in Word (a `~$` lock file), edit **their saved copy in place** so their changes are kept. Tell them to close Word without saving and reopen.
- After changes, remind them that any exported PDF is now stale.

## Step 6 - Log the application (confirm first)
1. Ask: have they applied (and when), and where did they find the role?
2. Show the proposed entries as a table and **wait for OK** before writing anything.
3. Log to `<YOUR_TRACKING_SYSTEM>` — pick the backend that matches your setup:
   - **Notion:** a Companies database (create the company if missing: Name, Industry, Career Board URL) and an Applications database (Position, Type, Status, URL, Applied Via, Job Found On, Submission Date, relation to the company page, and a page body linking the posting plus where the resume/JD are saved).
   - **Google Sheet / Excel:** one row per application in your tracker's Applications tab, with `HYPERLINK` cells pointing at the JD PDF and resume in the company folder.
4. Reply with a link to the logged entry (Notion page, or the tracker row/sheet).

## Don'ts
- No decision log for this workflow (unless you've added the `decision-log` skill and want it applied here too).
- Don't overwrite a file the user has edited since your last write.
- Don't state a total number of years of experience unless the user confirms it adds up; offer to drop it instead.

<!--
FILL IN BEFORE USE:
- <YOUR_RECRUITING_FOLDER_PATH>  — where JD PDFs and tailored resumes get filed, one subfolder per company
- <YOUR_MASTER_RESUME_DOC>       — where your experience bank / bullet bank lives (must match resume-review's reference)
- <Your_Name>                    — used in the tailored resume's file name
- <YOUR_TRACKING_SYSTEM>         — Notion, a Google Sheet, Airtable, etc. — pick one path in Step 6 and delete the other
-->
