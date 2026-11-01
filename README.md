# mkpdf-tex

**Write in Markdown, get a typeset PDF** with LaTeX-quality math, cross-references, and proper Georgian typography.

Mkpdf-tex is a small, Make-driven pipeline that turns Markdown files into PDFs using [Pandoc](https://pandoc.org) and XeLaTeX. Each language gets its own folder with its own settings (fonts, labels, language), so adding a new language or document type stays simple.

## Why mkpdf-tex?

- **Markdown is the natural format for SI-assisted writing.** Language models produce clean Markdown by default, and it's far less verbose than LaTeX, so drafting, editing, and reviewing are faster.
- **LaTeX is still the best engine for math and print typography.** Mkpdf-tex uses Pandoc as a bridge, so you get LaTeX output without writing LaTeX.
- **Georgian needs a specific setup.** Georgian text requires XeLaTeX and a font with Georgian glyphs, plus localized labels for figures. Mkpdf-tex configures this once, so each document only needs a short YAML header.

## Features

- Builds every `.md` file in a language folder into `build/<name>.pdf` with a single `make`.
- LaTeX math, inline (`$x^2$`) and display (`$$E = mc^2$$`).
- Cross-references for figures, tables, equations, and sections via [pandoc-crossref](https://github.com/lierdakil/pandoc-crossref), e.g. `[@fig:label]`.
- SVG images, converted to PDF automatically.
- Localized figure labels and captions (`figureTitle`, `figPrefix`).
- Adjustable margins and font, set in the Makefile or per document.

## Project layout

```
mkpdf-tex/
├── README.md
├── examples
└── src
    ├── ka-GE/              # Georgian
    │   ├── article         # Article class
    │   │   ├── Makefile    # Build rules, fonts, margins
    │   │   ├── article.md  # Source document (YAML + Markdown)
    │   │   ├── images/     # Figures referenced in article.md
    │   │   └── build/      # Generated PDFs (created by make)
    │   └── beamer          # Beamer class (not implemented)
    └── en-US               # English only (not implemented)

```

## Dependencies

| Tool | Purpose | Arch Linux | Debian / Ubuntu |
|---|---|---|---|
| **GNU Make** | Runs the build | `make` | `make` |
| **Pandoc** | Markdown → LaTeX → PDF | `pandoc-cli` | `pandoc` |
| **pandoc-crossref** | Figure, table, equation, and section references | `pandoc-crossref` | [GitHub release binary](https://github.com/lierdakil/pandoc-crossref/releases) |
| **XeLaTeX** (TeX Live) | PDF engine with Unicode and font support | `texlive-xetex` `texlive-latexextra` `texlive-fontsrecommended` | `texlive-xetex` `texlive-latex-extra` `texlive-latex-recommended` `texlive-fonts-recommended` |
| **librsvg** (`rsvg-convert`) | Converts SVG images to PDF | `librsvg` | `librsvg2-bin` |
| **DejaVu fonts** | Default font in the Makefile | `ttf-dejavu` | `fonts-dejavu` |

Tested on Arch Linux with Pandoc 3.11, pandoc-crossref 0.3.20, and TeX Live 2026. The Debian/Ubuntu package names have not been tested. Pandoc and pandoc-crossref versions should match: use a pandoc-crossref release built for your Pandoc version.

### Install on Arch Linux

```bash
sudo pacman -S --needed make pandoc-cli pandoc-crossref texlive-xetex texlive-latexextra texlive-fontsrecommended librsvg ttf-dejavu
```

### Install on Debian / Ubuntu

```bash
sudo apt install make pandoc texlive-xetex texlive-latex-extra texlive-latex-recommended texlive-fonts-recommended librsvg2-bin fonts-dejavu
```

Then install `pandoc-crossref` from its [releases page](https://github.com/lierdakil/pandoc-crossref/releases) and put the binary on your `PATH`.

## Quick start

```bash
git clone <repository-url> mkpdf-tex
cd mkpdf-tex/ka-GE
make
```

The PDF appears in `ka-GE/build/`. Other commands:

```bash
make clean   # Remove the build/ directory
make -C ka-GE   # Build from the project root instead
```

## Writing a document

Create a Markdown file in a language folder, start it with a YAML header, and run `make`. Here is the header from the Georgian example:

```markdown
---
lang: ka-GE
title: "დოკუმენტის სათაური"
author: "ავტორი"
date: "7 დეკემბერი 2026"
figureTitle: "**სურათი**"
figPrefix: "სურ."
header-includes:
  - \usepackage[margin=1.5cm, font=small]{caption}
---
```

| Key | Meaning |
|---|---|
| `lang` | Document language (BCP 47 tag). Controls hyphenation and language-specific typesetting. |
| `title`, `author`, `date` | Shown on the title block. |
| `figureTitle` | Label used in figure captions (e.g. "სურათი"). |
| `figPrefix` | Prefix used when referencing figures (e.g. "სურ. 1"). |
| `header-includes` | Raw LaTeX added to the preamble. Here it tightens figure caption spacing. |

### Math

```markdown
Inline: $\min\left(v\right)$

Display:

$$E = mc^2$$

Numbered equation with a label:

$$y'_n \approx \frac{y_{n+1} - y_{n-1}}{2\Delta x}$$ {#eq:central}
```

Reference it as `[@eq:central]`.

### Figures and cross-references

```markdown
![Caption text.](images/plot.svg){#fig:plot width=80%}

As shown in the figure ([@fig:plot]), ...
```

Pandoc attributes are space-separated: write `{#fig:plot width=80%}`, not `{width=80%, #fig:plot}`. The comma form makes the width value `80%,`, which pandoc doesn't parse.

## Customizing

Build settings live in the `FLAGS` variable of the folder's `Makefile`:

```makefile
FLAGS = --pdf-engine=xelatex -V geometry:margin=2.5cm -V mainfont="DejaVu Serif" --filter pandoc-crossref
```

- **Margins:** change `geometry:margin`.
- **Font:** change `mainfont`. Any font with Georgian glyphs works (for example, Noto Serif Georgian, if installed).
- **Precedence:** values passed with `-V` in `FLAGS` override the same keys in a document's YAML header. To give one document different settings, edit the Makefile or move that document into its own folder.

## Troubleshooting

**`pandoc-crossref: not found`**
The binary isn't on your `PATH`. Check with `command -v pandoc-crossref`.

**`Could not convert image ... conversion from SVG failed`**
Install `librsvg` (Arch) or `librsvg2-bin` (Debian/Ubuntu). Pandoc uses `rsvg-convert` to turn SVG images into PDF.

**Georgian letters show as empty boxes or are missing**
The `mainfont` doesn't contain Georgian glyphs. Switch to a font that does, such as DejaVu Serif or Noto Serif Georgian.

**Changing an image doesn't update the PDF**
The Makefile rebuilds a PDF only when its `.md` source changes. Run `make clean && make` to force a full rebuild.

**`File 'xyz.sty' not found`**
A LaTeX package is missing from your TeX Live installation. Install the `latex-extra` / `latex-recommended` collections listed above.

## License

Mkpdf-tex is free software, licensed under the [GNU General Public License v3.0 or later](LICENSE). The license covers this project's code and documentation, not the documents you write with it or the PDFs they produce.

## Acknowledgments

Built on [Pandoc](https://pandoc.org), [pandoc-crossref](https://github.com/lierdakil/pandoc-crossref), [XeTeX / TeX Live](https://tug.org/texlive/), [librsvg](https://wiki.gnome.org/Projects/LibRsvg), and the [DejaVu](https://dejavu-fonts.github.io) fonts.
