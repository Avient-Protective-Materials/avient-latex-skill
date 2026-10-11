---
name: avient-latex
description: Write and compile Avient Protective Materials LaTeX reports using avient.cls and its included avient-logo.pdf. Use when creating or editing an Avient report, linking the template as a Git submodule, downloading its ZIP without Git, leaving a clearly unassigned document ID for filing, or keeping branding and LaTeX outputs out of report history.
compatibility: Requires a LaTeX distribution, preferably LuaLaTeX, and network access for initial setup. Use Git submodules when Git is available, otherwise an HTTPS ZIP download and extractor. Command examples use a POSIX shell on Linux/macOS.
---

# Avient LaTeX reports

Use the upstream class rather than recreating its layout:
https://github.com/Avient-Protective-Materials/avient-latex-class

The upstream repository already contains **`brand/avient-logo.pdf`**. Use that
logo; do not ask the user to supply it, generate a substitute, or download a
second copy. Keep upstream file contents out of the report's Git history and
out of this skill; track only the submodule reference when using Git.

## Workflow

1. Inspect the report directory, existing `.tex` files, logo paths, `.gitignore`,
   build configuration, and available TeX engine. Preserve authored content and
   existing assets. If a project already supplies `avient-logo.pdf` at its root,
   preserve it and use `\setcompanylogo{./avient-logo.pdf}` instead of copying
   another logo there.
2. Read [setup.md](references/setup.md) and choose one installation:
   - **Git available (default):** add/reuse a submodule at `avient-latex-class/`
     beside the report. Track its gitlink and the report repository's
     `.gitmodules`, not the upstream file contents. Do not ignore this path.
   - **No Git:** download the official upstream ZIP and extract it into ignored
     `.avient-latex/`. Do not require Git or install it just for this fallback.
   - Both local methods use `TEXINPUTS` to resolve the class and included logo.
     An explicitly requested shared TeX installation is also documented.
   - A GitHub URL is not a valid `\documentclass` or logo path. A checkout or
     installation must exist on the build machine; no network fetching inside
     TeX and no shell escape are needed.
3. Before downloading, merge the relevant rules from
   [report.gitignore](assets/report.gitignore) into the report directory's
   `.gitignore`. Preserve existing rules, avoid duplicate entries, and review
   project-specific source exceptions. Ignore the ZIP-extracted dependency,
   legacy `brand/` and class copies, and all build output, but **not** the
   submodule path or `.gitmodules`. Remove conflicting old ignore rules only
   for the selected submodule path. Never blanket-ignore
   `*.pdf`: figures may be source inputs. Generate final PDFs in `build/`;
   for an existing root-output workflow add the exact report PDF path instead.
   `.gitignore` does not untrack files already committed: identify them and
   explain before removing only confirmed dependency/output paths from the
   index. Never delete source files or rewrite history as part of setup.
4. Create or update the user's report. [report.tex](assets/report.tex) is a
   complete, compilable demonstration, not a source of actual report metadata.
   Resolve asset paths relative to this installed skill's directory, not the
   report directory. Use the user's title, authors, recipients, date, and
   supplied findings. The document ID is normally assigned by the person filing
   the report, not its author: use `\setdocid{DOCUMENT ID NOT ASSIGNED}` until
   a real assigned ID is supplied. Never fabricate an ID or use plausible
   dummy numbers. Preserve an existing real ID. Do not invent reviewers,
   approvals, data, or confidentiality classifications. Omit optional fields
   when unknown, but keep the explicit unassigned-ID marker visible.
5. Build from the directory containing the report using the selected setup's
   environment and the commands below. Inspect failures and missing-package
   messages; report missing tooling instead of claiming a successful build.
6. Verify the logo was actually read, resolve references, and inspect the PDF
   visually when a viewer is available. The class silently omits missing logos:
   a successful TeX exit alone is not enough. Confirm Git ignores work and
   report the source path, PDF path, dependency location/commit, and any warnings.

## Class interface

Start with `\documentclass{avient}`. It is an A4, 11pt `article`-based class;
`apm-report` is the existing default. Keep class defaults rather than duplicating
geometry, headers, fonts, or section styling in each report.

| Command | Purpose |
| --- | --- |
| `\title{...}`, `\subtitle{...}` | Title and optional subtitle |
| `\author{...}` | From; semicolons separate stacked names |
| `\setreviewer{...}` | Optional reviewer; omitted/empty hides the line |
| `\setto{...}`, `\setcc{...}` | To and Copy; semicolons stack entries |
| `\setdocid{DOCUMENT ID NOT ASSIGNED}` | Visible non-ID placeholder; filing person replaces it with the assigned ID |
| `\date{...}` | Date; defaults to `\today` |
| `\setconfidential{...}` | Optional small tag above the logo |
| `\setcompanylogo{brand/avient-logo.pdf}` | Included upstream logo; resolved through the selected setup |
| `\setcompanyname{...}`, `\setcompanyaddress{...}` | Override default Avient identity; `\\` separates address lines |
| `\setbrandmain{2A7DE1}` | Main color, hex without `#` |
| `\setbrandlight{51D4FF}`, `\setbranddark{003478}` | Accent colors |

Inside the document, call `\maketitle`, then use
`\begin{executivesummary}...\end{executivesummary}` and normal sections.
The executive summary is unnumbered and added to the table of contents.
`\thispagestyle{plain}` suppresses the logo on that page; do not apply it to
pages that should show the branding.

The class loads `graphicx`, `xcolor`, `hyperref`, and other layout packages.
Do not reload them with conflicting options. It prefers the `FiraSans` package
under all engines. Without it, LuaLaTeX/XeLaTeX try system Fira Sans Light, then
fall back to Helvetica; pdfLaTeX falls back directly to Helvetica. LuaLaTeX is
recommended. Do not edit the downloaded class merely to change report content.

## Build and verify

With setup complete, run from the report directory (adapt `report.tex` to the
actual filename). Keep the working directory here even when output goes to
`build/`, so relative figure paths remain valid.

```sh
mkdir -p build
lualatex -interaction=nonstopmode -halt-on-error -file-line-error -recorder \
  -output-directory=build report.tex
lualatex -interaction=nonstopmode -halt-on-error -file-line-error -recorder \
  -output-directory=build report.tex
```

If `latexmk` is installed, use
`latexmk -lualatex -interaction=nonstopmode -halt-on-error -file-line-error -recorder -outdir=build report.tex`
instead. For bibliography/index packages, use the project's existing build
recipe or the necessary extra passes. Do not copy the upstream example's
unresolved citation: it refers to an `example.bib` not shipped upstream.

For either local setup, confirm the class and logo were loaded:

```sh
grep -F 'avient.cls' build/report.fls
grep -F 'brand/avient-logo.pdf' build/report.fls
```

With Git, verify the submodule is a single mode-`160000` gitlink, `.gitmodules`
records the URL, and build output is ignored:

```sh
git ls-files --stage -- avient-latex-class
git submodule status -- avient-latex-class
git check-ignore -v build/report.pdf build/report.log
git status --short
```

Without Git, skip Git checks; retain the ignore rules for future version control
and verify `.avient-latex/` contains the extracted class and logo. For a shared
install, inspect the recorder paths to confirm the intended shared files were
used. For an existing root-level logo, check that actual path instead. Confirm
authored `.tex`, `.bib`, and figure files are **not** ignored. Fix missing
branding rather than silently accepting it. Confirm the unassigned-ID marker
is visible in the PDF unless a real assigned ID was supplied.

## Scope and rights

Checked against upstream v0.7.4, commit
`eeb81c504de5cdb70d11c5d59db4a7c27c847e5b`. Inspect the actual checked-out class
and README if a newer version differs. Record the chosen commit in the report
project's setup notes; never silently update a dependency of an existing report.

Upstream says “For internal use.” Do not interpret a public repository as
permission to redistribute its class, logo, or branded reports. Keep fetched
files local and respect the user's authorized use.
