# Dependency setup: submodule first, ZIP without Git

Run from the **directory containing the report**, resolved to an absolute path.
Do not install the class into the skill repository. The included upstream logo
is `brand/avient-logo.pdf`; neither method needs a separate logo download.

First check `command -v git`. If available, use **A**; otherwise use **B**.
Merge the ignore template linked from `SKILL.md` into the report's `.gitignore`,
preserving existing rules and authored files. Distinct paths prevent confusion:

| Method | Report-local path | Parent repository tracks |
| --- | --- | --- |
| Submodule | `avient-latex-class/` | Gitlink (commit reference) and `.gitmodules`, not upstream contents |
| ZIP | `.avient-latex/` | Nothing inside this ignored directory |

Never flatten upstream into the report directory: that can overwrite authored
files and would require ignoring names such as `README.md` or `example.tex`.

## A. Git available: use a submodule

Inspect `git rev-parse --show-toplevel`, `git status --short`, and any existing
`.gitmodules`. Ensure the report belongs to the **intended** repository, not
accidentally a home-directory/ancestor repository. If the report has no intended
repository, initialize one in its directory with `git init`; do not reinitialize
an existing project or silently operate in an unrelated ancestor repository.

For a new submodule, after confirming the target path is unused:

```sh
git submodule add --depth 1 \
  https://github.com/Avient-Protective-Materials/avient-latex-class.git \
  avient-latex-class
```

This stages the gitlink and the repository-root `.gitmodules`. Do **not** ignore
`avient-latex-class/` or `.gitmodules`. If an old ignore rule blocks the path,
remove only that conflicting rule; do not bypass it using `git add -f`. The
report's ordinary commit should include the gitlink, `.gitmodules`, setup notes,
ignore rules, and authored sources—not individual upstream files.

If the submodule is already registered, inspect its URL and local changes and
reuse it; do not call `submodule add` again. On a fresh checkout, initialize it
with (no `--remote`, which would bypass the parent's pinned revision):

```sh
git submodule update --init --recursive -- avient-latex-class
```

If the target exists but is not the intended registered submodule (for example,
an old ignored clone or a ZIP extraction), stop and inspect it. Do not overwrite,
delete, or reset it. Plan any migration separately, preserving local changes.

Verify the recorded dependency:

```sh
git ls-files --stage -- avient-latex-class
git submodule status -- avient-latex-class
git -C avient-latex-class remote -v
git -C avient-latex-class rev-parse HEAD
```

The index should show **one mode-`160000` entry** for the dependency. Its class,
brand assets, README, and examples live in their own repository, not the report's
file history. Git still downloads them locally because LaTeX must read them.

Select the dependency for the build:

```sh
AVIENT_DIR="$PWD/avient-latex-class"
```

## B. Git unavailable: download the ZIP

No Git commands are needed for this method. Use the official GitHub HTTPS
archive, not a third-party mirror. The following POSIX-shell example needs
`curl` and `unzip`. It downloads the reviewed revision for repeatable builds,
uses a temporary directory, refuses an existing target, and verifies the class
and logo before moving the extracted directory into place:

```sh
(
  set -eu
  AVIENT_DIR="$PWD/.avient-latex"
  AVIENT_REF=eeb81c504de5cdb70d11c5d59db4a7c27c847e5b
  AVIENT_ARCHIVE_URL="https://github.com/Avient-Protective-Materials/avient-latex-class/archive/$AVIENT_REF.zip"
  if [ -e "$AVIENT_DIR" ] || [ -L "$AVIENT_DIR" ]; then
    printf '%s\n' "Refusing to overwrite $AVIENT_DIR; inspect and reuse it if valid." >&2
    exit 1
  fi
  AVIENT_TMP="$(mktemp -d)"
  trap 'rm -rf "$AVIENT_TMP"' EXIT
  curl --fail --location --retry 3 --connect-timeout 20 --max-time 180 \
    "$AVIENT_ARCHIVE_URL" -o "$AVIENT_TMP/upstream.zip"
  unzip -q "$AVIENT_TMP/upstream.zip" -d "$AVIENT_TMP/extracted"
  AVIENT_EXTRACTED="$AVIENT_TMP/extracted/avient-latex-class-$AVIENT_REF"
  test -f "$AVIENT_EXTRACTED/avient.cls"
  test -f "$AVIENT_EXTRACTED/brand/avient-logo.pdf"
  mv "$AVIENT_EXTRACTED" "$AVIENT_DIR"
)
```

The subshell keeps temporary variables/traps scoped to this download; only its
own temporary files are removed. Check that the block succeeds before building.
If `curl`/`unzip` are unavailable, use the environment's HTTPS downloader/browser
and ZIP extractor with the same URL and checks—do not require Git as the fallback
for missing Git. If no downloader/extractor exists, report that specific blocker.
If a browser saves the ZIP inside the report project, retain its upstream name
(`avient-latex-class-<revision>.zip`, covered by the ignore template) or add its
exact saved path to `.gitignore`. Do not ignore unrelated ZIP source data.

For a previously extracted `.avient-latex/`, verify provenance/revision against
the report's setup notes and check its class/logo before reusing it. Do not
download over existing files. Keep the whole directory ignored, even if Git is
installed later. The temporary ZIP is not retained in the project.

Select the dependency for the build (outside the download subshell):

```sh
AVIENT_DIR="$PWD/.avient-latex"
```

## Build environment for either local method

In the **same shell** where `AVIENT_DIR` was selected, from the report directory:

```sh
export TEXINPUTS="$AVIENT_DIR//:${TEXINPUTS:-}:"
test -f "$AVIENT_DIR/avient.cls"
test -f "$AVIENT_DIR/brand/avient-logo.pdf"
kpsewhich avient.cls
kpsewhich brand/avient-logo.pdf
```

`//` enables recursive search. The trailing `:` preserves TeX's built-in search
paths. Keep existing `TEXINPUTS` entries; do not globally change the user's shell
profile. An editor or CI build must receive the same environment. Check that
both lookups resolve to the intended dependency, not a stale installation.

The report uses the same preamble in both cases:

```tex
\documentclass{avient}
\setcompanylogo{brand/avient-logo.pdf}
\setdocid{DOCUMENT ID NOT ASSIGNED}
```

Then run the build commands in `SKILL.md`. Stay in the report directory and
write all generated files to `build/`. An existing root-level `avient-logo.pdf`
can be preserved and selected with `\setcompanylogo{./avient-logo.pdf}` instead.
The explicit `./` prevents recursive `TEXINPUTS` lookup from choosing the
upstream logo with the same filename. Do not create that second copy just for
a new report.

## Optional: reuse a shared user TeX installation

Only use this if explicitly requested. A shared installation avoids a checkout
per report but is not a submodule pinned by each report repository.

Inspect `kpsewhich -var-value=TEXMFHOME`, `kpsewhich avient.cls`, and
`kpsewhich brand/avient-logo.pdf` first. Missing-file lookups return nonzero.
Reuse an existing verified installation without overwriting it. For a new one,
confirm `TEXMFHOME` is a single writable absolute directory and place one Git
checkout (or, without Git, ZIP extraction using method B's safeguards) at
`TEXMFHOME/tex/latex/avient/`, containing the class and its `brand/` subdirectory.
Do not copy the logo again. With Git and an unused destination:

```sh
AVIENT_TEXMFHOME="$(kpsewhich -var-value=TEXMFHOME)"
mkdir -p "$AVIENT_TEXMFHOME/tex/latex"
git clone --depth 1 \
  https://github.com/Avient-Protective-Materials/avient-latex-class.git \
  "$AVIENT_TEXMFHOME/tex/latex/avient"
```

TeX Live normally searches the user tree directly. If required, refresh only
this tree using `mktexlsr "$AVIENT_TEXMFHOME"`, with no `sudo`. From the report
directory, without a stale local `TEXINPUTS` override, verify both lookups and
the build recorder paths. Shared builds require no per-project dependency copy
or search setting. Each other machine still needs setup; shared updates affect
all reports using it. Use submodules when reports need different revisions.

## Reproducibility and updates

Record the dependency URL, method, revision, and build command in the report's
README/setup notes. For submodules, the gitlink pins the revision; include it in
the report's normal commit. For ZIPs, record the exact commit archive URL above.
A `main.zip` URL changes over time and is not a revision pin.

Never update the dependency silently before building. For a deliberate submodule
update, inspect its local changes, fetch/check out the chosen upstream commit,
rebuild, and stage the changed gitlink in the parent. Do not reset local work.
ZIP updates likewise require a reviewed revision and preserved existing files.
Do not convert between a ZIP directory and a submodule implicitly.

## Troubleshooting and other environments

- **Missing class:** check `kpsewhich avient.cls` in the actual build environment;
  initialize the submodule or verify the ZIP extraction and `TEXINPUTS`.
- **Missing logo:** the class silently skips absent logos via `\IfFileExists`.
  Verify the included logo with `kpsewhich` and the build's `.fls` file. A
  successful compile without the logo does not pass validation.
- **Missing packages/fonts:** consult the upstream README and installed TeX
  distribution's package manager. Fira is optional; layout packages are not.
  Do not install system packages without appropriate permissions or enable
  shell escape to fetch dependencies.
- **Windows/MiKTeX:** adapt shell/download commands to the available environment;
  use `;` instead of `:` in Windows search paths. For shared installs, register
  the user TEXMF root in MiKTeX Console and refresh its filename database.
- **Overleaf:** it cannot access your local TeX tree. Initialize the submodule
  locally or download the ZIP, then upload `avient.cls` and
  `brand/avient-logo.pdf` to the report root. Keep these export copies ignored
  in the local report repository. A GitHub URL alone cannot supply TeX files.
- **Already tracked artifacts:** ignore rules do not untrack them. Review exact
  paths before removing only confirmed generated/dependency copies from the
  index, preserving working files. Never untrack the intended gitlink or
  `.gitmodules`. Old committed copies remain in history; do not rewrite it.
