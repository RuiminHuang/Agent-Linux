<!-- ARIS:BEGIN -->
## ARIS Skill Scope
ARIS skills installed in this project: 83 entries.
Manifest: `.aris/installed-skills.txt` (lists every skill ARIS installed and its upstream target).
For ARIS workflows, prefer the project-local skills under `.claude/skills/` over global skills.
Do not modify or delete files inside any skill that is a symlink (symlinks point into `/data2/hrm/aris_repo`).
Update with: `bash /data2/hrm/aris_repo/tools/install_aris.sh`  (re-runnable; reconciles new/removed skills).
<!-- ARIS:END -->

## ARIS Upstream Version
The ARIS symlinks (`.claude/skills/*`, `.aris/tools`, `.github/agents/*.agent.md`) are committed as link paths only — GitHub holds none of their content, so a fresh clone has dangling links until `/data2/hrm/aris_repo` is restored at this commit:

- Upstream: `https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep`
- Commit: `b8a50974eae105a5d13b75099a6a956a05377e03` (committed 2026-09-11)

```bash
git clone https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep.git /data2/hrm/aris_repo
git -C /data2/hrm/aris_repo checkout b8a50974eae105a5d13b75099a6a956a05377e03
```

Nothing enforces this pin — the links follow whatever `/data2/hrm/aris_repo` has checked out, so a `git pull` there changes the skills immediately. After updating ARIS, replace the commit above with the output of `git -C /data2/hrm/aris_repo rev-parse HEAD`.

## Local Server
- gpu: local
- GPU: 4x RTX 4090 (24GB)
- Conda env: 'Deep-MIST' — Python 3.8.20 + PyTorch 1.10.1+cu113 (see ## Python script Execution)


## Repository Layout
- `Experiments/`: Experiment code and logs.
- `Manuscripts/IEEE_Template/`: Manuscripts for submission (IEEEtran two-column, pdfLaTeX + BibTeX). Run LaTeX build commands from this directory; see [LaTeX Compilation](#latex-compilation).
- `Manuscripts/Notes_and_Outlines/`: Rough drafts, outlines, and writing notes. Not part of the manuscript LaTeX build.
- `Materials/`: Reference materials, including papers in Markdown and PDF formats. Treat this directory as read-only; do not create, modify, or delete files here.
- `research-wiki/`: Persistent knowledge base for ARIS, covering papers, ideas, experiments, and claims. Update this directory only through the `research-wiki` skill.


## Python script Execution

All Python — DL training, MCP servers, ad-hoc scripts — runs on the `Deep-MIST` env:

```bash
/home/hrm/anaconda3/envs/Deep-MIST/bin/python script.py
```

Never use bare `python3` — it resolves to anaconda base (3.12.2), whose torch is broken.

## LaTeX Compilation

Always `cd Manuscripts/IEEE_Template` first — the project references `./figs/` and `./reference.bib` by relative path, so building from anywhere else loses figures and bibliography.

Default to latexmk (decides whether BibTeX is needed and reruns until cross-references converge):

```bash
latexmk -pdf -synctex=1 main.tex
```

`-synctex=1` is not optional — latexmk's built-in engine call is bare `pdflatex %O %S`, so without it no `main.synctex.gz` is written and editor forward/inverse search breaks. latexmk forwards the flag to pdflatex unchanged.

When an explicit pdfLaTeX chain is required, all four steps are needed — pass 1 records `\citation` in `.aux`, BibTeX reads that and writes `.bbl` from `reference.bib`, pass 2 typesets the bibliography, and pass 3 resolves the in-text `[n]` numbers:

```bash
pdflatex -interaction=nonstopmode -file-line-error -synctex=1 main.tex
bibtex   main
pdflatex -interaction=nonstopmode -file-line-error -synctex=1 main.tex
pdflatex -interaction=nonstopmode -file-line-error -synctex=1 main.tex
```

pdflatex exits 0 even on errors in `nonstopmode` — verify by grepping `main.log`, not the exit code. Baseline: 0 errors / 0 warnings / 0 bad boxes.

`latexmk -c` clears the intermediate files and keeps `main.pdf`; `-C` deletes `main.pdf` too. Either unsticks a build blocked by stale state — e.g. a truncated `main.aux` from an interrupted run makes BibTeX report `I found no \citation commands`, which neither reruns nor `-g` clear. Prefer `-c`.

