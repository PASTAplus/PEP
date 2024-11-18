# PASTA Enhancement Proposal (PEP)

PEP stands for **PASTA Enhancement Proposal** and is based on the Python community's *Python Enhancement Proposal* process (see [here](https://peps.python.org/pep-0001)). A PEP serves both EDI and its community as a method to propose significant enhancements or modifications to EDI software, processes, or policies. The [PEP Purpose and Guidelines (PEP-0)](peps/pep-0.md) provides more detailed information about PEPs.


## PEP Index
A list of all PEPs can be found in the PEP [Index](index.md). The PEP Index provides a summary of each PEP, including its PEP number, title, author/contact, current status, and date of its final disposition. 

## Working with PEPs

### Editing PEPs

A PEP document is a plain text file formatted using the [GitHub flavored Markdown](https://github.github.com/gfm/) syntax and can be edited using any text editor. The PEP template is available [here](pep-template.md) and should be used as a starting point for new PEPs.

### Generating a PDF

The best solution to create a printable PDF document of the PEP is to use the universal document converter `pandoc`. Pandoc can be installed using [Conda](https://docs.conda.io/en/latest/), but will require a "PDF engine" like `PDFLaTeX` (for example, see `pdflatex` in  [TeX Live](https://tug.org/texlive/)) to convert a Markdown document to PDF. The following command will generate a PDF from a Markdown file:

```bash
pandoc pep-N.md -o pep-N.pdf \
      --resource-path ./images \
      -f markdown-implicit_figures \
      -V geometry:"margin=1in" \
      -V fontsize="12pt" \
      -V colorlinks
```

