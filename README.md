# Avient LaTeX skill

An [Agent Skill](https://agentskills.io/specification) for writing reports with
[Avient's LaTeX class](https://github.com/Avient-Protective-Materials/avient-latex-class).
It teaches an agent to link the template as a Git submodule (or download a ZIP
without Git), use its actual commands, and compile a branded report without
copying upstream file contents into report history. Build outputs are ignored.
The class and brand assets are **not bundled** here.
The upstream project already includes `brand/avient-logo.pdf`; the skill uses
that file directly, without a separate logo download or duplicate copy.

## Install

From a local checkout:

```sh
npx skills add ./ --skill avient-latex
```

Then ask your agent, for example:

> Create an Avient LaTeX report in this project. Use a submodule for the class,
> or download its ZIP if Git is unavailable. Ignore generated files and build
> the PDF. Leave the document ID clearly unassigned for the person filing it.

Or, to avoid a template copy in each report project:

> Use the Avient LaTeX skill with a shared user TeX installation.

Reports need a LaTeX distribution; LuaLaTeX is recommended. Git enables the
default submodule setup. Without Git, use an HTTPS downloader and ZIP extractor
(the shell example uses `curl` and `unzip`). Node.js/npm is only needed for the
Skills CLI, not for LaTeX; its own installation requirements are separate from
the report's Git-free ZIP fallback.
The setup examples target POSIX shells on Linux/macOS. The skill also explains
the Windows/MiKTeX and Overleaf considerations.

## Can a report just reference GitHub?

**Yes, through a Git submodule:** the report repository stores the upstream URL
in `.gitmodules` and a commit reference (gitlink), not copies of upstream files.
LaTeX still needs the referenced files downloaded locally to build.

| Setup | Where the upstream repository lives | Trade-off |
| --- | --- | --- |
| Git submodule (default) | `avient-latex-class/` beside the report | Track only the upstream URL and pinned commit, not its contents |
| ZIP fallback (no Git) | Ignored `.avient-latex/` beside the report | No Git required; record the archive URL and revision |
| Shared user TeX tree | `TEXMFHOME/tex/latex/avient/` | One checkout per user/machine, reused by all reports; updates affect all of them |

Local setups use `TEXINPUTS`; the shared installation uses TeX's search path.
After setup, builds work offline. On a fresh checkout of a report repository,
`git submodule update --init --recursive` retrieves the pinned dependency.
Never ignore the submodule path or `.gitmodules`. ZIP-extracted files, unlike
the submodule reference, are fully ignored.

Builds go into `build/`, so even generated PDFs are ignored without hiding
PDF figures elsewhere in the project. Legacy root-level `avient.cls` and
`brand/` copies are also ignored. Project `.tex` and `.bib` sources remain tracked.

## Document ID

The person filing the report normally assigns its document ID. Until then, use
`\setdocid{DOCUMENT ID NOT ASSIGNED}`: an unmistakable placeholder, not a valid
number. The agent must not invent plausible IDs and must preserve a real assigned
ID if one is provided. The filing person replaces the marker before final filing.

## Publish on skills.sh

1. Review publication rights, choose a license for your own skill instructions,
   and publish this repository to GitHub. No remote is assumed by this project.
2. Check discovery locally:
   ```sh
   npx skills add ./ --list
   ```
3. Users install from the published GitHub URL with
   `npx skills add "$SKILL_REPOSITORY_URL" --skill avient-latex`, where
   `SKILL_REPOSITORY_URL` is the URL of **this skill repository**, not the class
   repository.
4. The [skills.sh FAQ](https://skills.sh/docs/faq) says listing happens through
   installation telemetry from the Skills CLI; there is no separate upload or
   submission step. A local discovery check is not publication or a listing.

The skill lives in `skills/avient-latex/`, with a specification-compliant
`SKILL.md`, a setup reference, a small report example, and an ignore template.

### Upstream rights

The class README labels it **“For internal use.”** Public GitHub visibility is
not a redistribution license for the class or logo. This repository distributes
only instructions and a small usage example, not upstream files. Confirm the
intended audience and permission to use the branding before publishing or
redistributing branded reports. No license for upstream assets is granted here.

## Validation

Instructions target upstream class v0.7.4, commit
`eeb81c504de5cdb70d11c5d59db4a7c27c847e5b`. To validate a change, follow each setup
in a disposable directory, compile `assets/report.tex` from the installed skill
twice, and check the recorder file for **both** `avient.cls` and
`brand/avient-logo.pdf`. Check that the PDF shows `DOCUMENT ID NOT ASSIGNED`.
Verify Git tracks a single mode-`160000` submodule entry, while the ZIP fallback
and build output are ignored and authored sources/PDF figures are not.
