---
description: Tailor a resume to a specific job posting and generate a LaTeX PDF. Accepts either a .pdf CV or a .jsonl resume file as the first argument, plus a job posting URL.
argument-hint: <resume.pdf or resume.jsonl> <job posting URL>
allowed-tools: Read, WebFetch, Write, Bash
---

You are a professional resume writer and LaTeX expert. The user has provided:

```
$ARGUMENTS
```

Parse:
- **Argument 1** = path to a CV file — either `.pdf` or `.jsonl`
- **Argument 2** = job posting URL

Derive the output directory from the CV file's parent folder. All output files go there.

**Check the file extension of Argument 1:**
- If it ends in `.jsonl` → skip Step 0, go directly to Step 1
- If it ends in `.pdf` → run Step 0 first to extract and save a JSONL, then continue to Step 1 using the generated JSONL path

Run the following pipeline:

---

## Step 0 — PDF → JSONL (only if input is a .pdf)

Use the **Read** tool to read the PDF file. It will return all text content from the CV.

Parse the raw text into the structured JSONL format below. Be thorough — extract every piece of information present: all experience entries with every bullet, all education entries, all skills, all publications, all honors/awards, all projects if mentioned.

Output **one JSON object per line**, each with a `"section"` key:

```jsonl
{"section": "personal", "name": "...", "location": "...", "email": "...", "website": "...", "linkedin": "..."}
{"section": "education", "entries": [{"institution": "...", "location": "...", "degree": "...", "gpa": "...", "start": "...", "end": "...", "notes": ["..."]}]}
{"section": "skills", "categories": [{"name": "...", "items": ["...", "..."]}]}
{"section": "experience", "entries": [{"organization": "...", "location": "...", "title": "...", "start": "...", "end": "...", "project": "...", "bullets": ["...", "..."]}]}
{"section": "projects", "entries": [{"name": "...", "url": "...", "description": "...", "tech_stack": ["..."], "key_features": ["..."]}]}
{"section": "publications", "entries": [{"id": 1, "authors": "...", "year": 0000, "title": "...", "venue": "...", "status": "...", "doi": "...", "arxiv": "..."}]}
{"section": "honors", "entries": [{"title": "...", "amount": "...", "date": "..."}]}
```

Rules:
- Every field that exists in the PDF must appear in the JSONL — do not drop any information
- If a field is not present in the PDF (e.g. no website), omit that key entirely
- Keep all original text faithful — do not paraphrase or rewrite at this stage
- For `publications`: extract authors, year, full title, venue/journal, status (published/under review/ongoing), DOI and arXiv links if present
- For `experience`: each bullet point becomes one string in the `bullets` array
- For `projects` mentioned in the CV: extract name, description, tech stack, and any links

Use **Write** to save the JSONL as `resume.jsonl` in the same directory as the PDF.

Print: `✓ Extracted CV → resume.jsonl ([N] sections)`

Set the JSONL path to this newly created file and continue to Step 1.

---

## Step 1 — Load Resume & Fetch Job Posting

Use **Read** to load the JSONL file. Parse each line into a dict keyed by `"section"`. Build a mental model of:
- `personal`: name, email, location, website
- `education`: institutions, degrees, GPAs, dates
- `skills`: all categories and items
- `experience`: all entries with bullets
- `projects`: all entries with tech stack and features
- `publications`: all entries
- `honors`: all entries

Use **WebFetch** on the job URL. Extract:
- Job title, company, location
- Required skills (explicit list)
- Preferred/bonus skills
- Key responsibilities (verbatim phrases matter for ATS)
- Experience and education requirements
- Domain keywords (fraud, analytics, ML, etc.)

Print: `Candidate: [name] → Role: [title] @ [company]`

---

## Step 2 — Match Analysis

Compare the resume against the job. Determine:

**Match score (0–100)** with one-sentence rationale.

**Matched skills**: candidate has, job wants.

**Skill gaps**: job wants, candidate lacks.

**Experience ranking**: sort all experience entries by relevance to this specific role (most → least), with one-line reason each.

**Project ranking**: sort all projects by relevance.

**Reframing tips**: 4–6 specific, actionable suggestions — e.g., "Lead with X because JD uses phrase Y", "Drop Z, it's irrelevant here".

**Tailored summary**: 2–3 sentences positioning the candidate for this exact role. Use JD keywords naturally.

Print the match score and top 3 reframing tips.

---

## Step 3 — Tailor Content

Rewrite the resume content targeting this role. **Rules:**
- Never invent metrics, experiences, or skills the candidate doesn't have
- Reframe existing bullets using the JD's own language and keywords
- Lead every bullet with a strong action verb + quantified outcome
- Include only the **top 3–4 most relevant experience entries**
- Include only the **top 2–3 most relevant projects**
- Order skills: most job-relevant first
- Keep publications and honors unchanged

Produce tailored versions of:
1. **summary** (2–3 sentences)
2. **skills** split into `primary` (directly required) and `secondary` (valuable additions)
3. **experience** entries (3 bullets each, strongest first), only the top 3–4
4. **projects** (1-sentence description + 2 highlight bullets), only top 2–3
5. **education** (unchanged)

---

## Step 4 — Generate LaTeX

Using the tailored content from Step 3, write a complete LaTeX file based on Jake's Resume template.

Use **Write** to save it as `tailored_[company]_[title].tex` in the output directory.

**Page length rules (strictly enforce):**
- Target **1 page** for most roles; use **2 pages** only if the candidate has 5+ years of experience or the role explicitly values depth (e.g., research/senior positions).
- If content fits comfortably on 1 page, use 1 page. Never leave a second page with less than ~40% content.
- To control length, use these levers in order:
  1. Reduce bullets per experience to 2 (drop the weakest bullet before trimming words)
  2. Shorten bullet text — aim for 1 line per bullet, 2 lines max
  3. Reduce `\textheight` margin slightly (e.g., `\addtolength{\textheight}{1.2in}` instead of `1.0in`)
  4. Reduce font size to `10pt` if still overflowing
- Never add `\vspace`, `\bigskip`, or blank lines to pad whitespace. If a section is short, let it sit tight.


The LaTeX file must follow this exact structure:

```latex
%-------------------------
% Resume in LaTeX — Based on Jake's Resume (github.com/jakegut/resume)
%-------------------------
\documentclass[letterpaper,11pt]{article}
\usepackage{latexsym}
\usepackage[empty]{fullpage}
\usepackage{titlesec}
\usepackage{marvosym}
\usepackage[usenames,dvipsnames]{color}
\usepackage{verbatim}
\usepackage{enumitem}
\usepackage[hidelinks]{hyperref}
\usepackage{fancyhdr}
\usepackage[english]{babel}
\usepackage{tabularx}

\pagestyle{fancy}
\fancyhf{}
\fancyfoot{}
\renewcommand{\headrulewidth}{0pt}
\renewcommand{\footrulewidth}{0pt}

\addtolength{\oddsidemargin}{-0.5in}
\addtolength{\evensidemargin}{-0.5in}
\addtolength{\textwidth}{1in}
\addtolength{\topmargin}{-.5in}
\addtolength{\textheight}{1.0in}

\urlstyle{same}
\raggedbottom
\raggedright
\setlength{\tabcolsep}{0in}

\titleformat{\section}{
  \vspace{-4pt}\scshape\raggedright\large
}{}{0em}{}[\color{black}\titlerule \vspace{-5pt}]

\newcommand{\resumeItem}[1]{\item\small{{#1 \vspace{-2pt}}}}
\newcommand{\resumeSubheading}[4]{
  \vspace{-2pt}\item
    \begin{tabular*}{0.97\textwidth}[t]{l@{\extracolsep{\fill}}r}
      \textbf{#1} & #2 \\
      \textit{\small#3} & \textit{\small #4} \\
    \end{tabular*}\vspace{-7pt}
}
\newcommand{\resumeProjectHeading}[2]{
    \item
    \begin{tabular*}{0.97\textwidth}{l@{\extracolsep{\fill}}r}
      \small#1 & #2 \\
    \end{tabular*}\vspace{-7pt}
}
\renewcommand\labelitemii{$\vcenter{\hbox{\tiny$\bullet$}}$}
\newcommand{\resumeSubHeadingListStart}{\begin{itemize}[leftmargin=0.15in, label={}]}
\newcommand{\resumeSubHeadingListEnd}{\end{itemize}}
\newcommand{\resumeItemListStart}{\begin{itemize}}
\newcommand{\resumeItemListEnd}{\end{itemize}\vspace{-5pt}}

\begin{document}

% HEADING
\begin{center}
    \textbf{\Huge \scshape [NAME]} \\ \vspace{1pt}
    \small [LOCATION] $|$
    \href{mailto:[EMAIL]}{\underline{[EMAIL]}} $|$
    \href{[WEBSITE]}{\underline{[WEBSITE_DISPLAY]}} $|$
    \href{https://github.com/Jasper0122}{\underline{github.com/Jasper0122}}
\end{center}

% EDUCATION
\section{Education}
  \resumeSubHeadingListStart
    % One \resumeSubheading per institution
    % Add \resumeItemListStart / \resumeItemListEnd for notes (fellowship, advisor)
  \resumeSubHeadingListEnd

% EXPERIENCE
\section{Experience}
  \resumeSubHeadingListStart
    % One \resumeSubheading per entry
    % 3 \resumeItem bullets each
  \resumeSubHeadingListEnd

% PROJECTS
\section{Projects}
  \resumeSubHeadingListStart
    % \resumeProjectHeading with \textbf{Name} $|$ \emph{Stack} $|$ \href{URL}{\underline{GitHub}}
    % 2 \resumeItem bullets each
  \resumeSubHeadingListEnd

% SKILLS
\section{Technical Skills}
 \begin{itemize}[leftmargin=0.15in, label={}]
    \small{\item{
     \textbf{Primary}{: [comma-separated primary skills]} \\
     \textbf{Additional}{: [comma-separated secondary skills]}
    }}
 \end{itemize}

% PUBLICATIONS
\section{Publications}
  \begin{itemize}[leftmargin=0.15in, label={}, itemsep=2pt]
    \small{
      % \item \textbf{[N]} authors (year). \textit{title.} venue. (status if not published)
      % Add \href for DOI/arXiv links where available
    }
  \end{itemize}

% HONORS
\section{Honors \& Awards}
  \begin{itemize}[leftmargin=0.15in, label={}, itemsep=1pt]
    \small{
      % \item \textbf{title} (amount) --- date \hfill date
    }
  \end{itemize}

\end{document}
```

**Critical LaTeX escaping rules — apply to ALL text:**
- `&` → `\&`
- `%` → `\%`
- `$` → `\$` (except inside math mode `$...$`)
- `#` → `\#`
- `_` → `\_`
- `~` → `\textasciitilde{}`
- Use `$R^2$`, `$p<0.001$`, `$\times$`, `$\geq$` for math symbols
- Use `--` for en-dash, `---` for em-dash

Fill in ALL sections completely with real data. Do not leave placeholder comments in the output file.

---

## Step 5 — Compile PDF with Tectonic

Use **Bash** to compile the LaTeX file:

```bash
cd [output_directory] && [tectonic_path] tailored_[company]_[title].tex 2>&1
```

The tectonic binary is at: `C:\Users\zongr\tectonic_bin\tectonic.exe`

If compilation fails with a LaTeX error, fix the error in the .tex file and recompile.

Also use **Write** to save a match report as `match_report_[company]_[title].md` containing:
- Match score and rationale
- Matched skills and gaps
- Experience/project relevance ranking
- Full reframing recommendations

---

## Step 6 — Generate Cover Letter PDF

### 6a — Write cover letter content

Using the match analysis from Step 2 and the tailored resume from Step 3, write a professional cover letter. Structure:

**Paragraph 1 — Opening (2–3 sentences)**
State the role and company. Open with a specific, genuine hook — reference something concrete about the company (their domain, a project, their mission) that connects to the candidate's actual background. Do not use generic openers like "I am writing to apply for...".

**Paragraph 2 — Why you're a strong fit (3–4 sentences)**
Pick the 2–3 most relevant experiences from the match analysis. Describe them in terms of the job's needs using the JD's own language. Include 1–2 specific quantified achievements that directly address the role's core responsibilities.

**Paragraph 3 — Why this company/role (2–3 sentences)**
Connect the candidate's research interests or career direction to what IntegraFEC (or the specific company) does. Be specific — mention fraud detection, forensic analytics, the industry they serve, or whatever is distinctive about this employer. Avoid generic statements.

**Paragraph 4 — Closing (2 sentences)**
Express enthusiasm, reference availability, and include a call to action.

**Tone:** Professional but direct. No filler phrases ("I am a hard worker", "passion for data"). Every sentence must carry information.

### 6b — Generate cover letter LaTeX

Use **Write** to save `coverletter_[company]_[title].tex`. Use this template:

```latex
\documentclass[letterpaper,11pt]{article}

\usepackage[empty]{fullpage}
\usepackage[hidelinks]{hyperref}
\usepackage[english]{babel}
\usepackage{parskip}

\addtolength{\oddsidemargin}{-0.5in}
\addtolength{\evensidemargin}{-0.5in}
\addtolength{\textwidth}{1in}
\addtolength{\topmargin}{-0.7in}
\addtolength{\textheight}{1.4in}

\urlstyle{same}

\begin{document}

% Header
\begin{flushleft}
\textbf{\Large [Full Name]} \\
[Location] $\cdot$ \href{mailto:[email]}{[email]} $\cdot$ \href{[website]}{[website display]}
\end{flushleft}

\vspace{4pt}
[Today's date, format: Month DD, YYYY] \\[6pt]
Hiring Manager \\
[Company Name] \\[12pt]

\textbf{Re: [Job Title]}

\vspace{6pt}

[Paragraph 1]

[Paragraph 2]

[Paragraph 3]

[Paragraph 4]

\vspace{12pt}
Sincerely, \\[24pt]
\textbf{[Full Name]}

\end{document}
```

Apply the same LaTeX escaping rules as Step 4 (`\&`, `\%`, `\$`, etc.).

### 6c — Compile cover letter PDF

Use **Bash** to compile:
```bash
cd [output_directory] && C:\Users\zongr\tectonic_bin\tectonic.exe coverletter_[company]_[title].tex 2>&1
```

If compilation fails, fix and retry.

After all files are saved, print the final summary:
```
✓ Match report       → match_report_[company]_[title].md
✓ Resume (LaTeX)     → tailored_[company]_[title].tex
✓ Resume (PDF)       → tailored_[company]_[title].pdf
✓ Cover letter (LaTeX) → coverletter_[company]_[title].tex
✓ Cover letter (PDF) → coverletter_[company]_[title].pdf
Match score: [X]/100
```
