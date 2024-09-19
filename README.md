# LaTeX

## Processing documents the right way

How can I get a PDF from my LaTeX sources? Why are my references missing? Why does not my new section (or figure, or table) appear in the table of contents (or list of figures, or list of tables)? Why is my bibliography empty? How and when should I process my bibliography? What is the shell escape mode?

### The short story (or I want it now)

Processing LaTeX documents correctly may involve several tools to be invoked in the right order and even require several passes. The whole process can be automated and greatly simplified with `latexmk`:

    latexmk -pdf

If certain special LaTeX packages are used, you may need instead:

    latexmk -pdf -shell-escape

Most auxiliary files can be cleaned up with:

    latexmk -c

Please, use `-C` instead of `-c` to clean the generated PDF file too.

Of course, you can use the facilities available with most editors and integrated development environments instead.

### The long story (or how it works)

In order to produce a PDF file from LaTeX sources (`.tex`) and BibLaTeX references (`.bib`), the sources are processed with PDFLaTeX (`pdflatex`) and Biber (`biber`). XeLaTeX and LuaLaTeX may be used instead of PDFLaTeX with some advantages, particularly in font management and Unicode processing (files are required to be encoded in UTF-8), though your document may require some tweaking, depending on the case.

Let us assume we have two files, namely `document.tex` and `references.bib`, with the former containing `\addbibresource{references.bib}` in the preamble, following the corresponding use of the `biblatex` package. First, we *compile* the `.tex` document with `pdflatex`:

    pdflatex document 	# Compile document.tex into document.pdf

By default, this command outputs a lot of information about the process, similar to this:

    This is pdfTeX, Version 3.14159265-2.6-1.40.20 (TeX Live 2019/Debian) (preloaded format=pdflatex)
     restricted \write18 enabled.
    entering extended mode
    (./document.tex
    LaTeX2e <2020-02-02> patch level 2
    L3 programming layer <2020-02-14>
    (/usr/share/texlive/texmf-dist/tex/latex/base/article.cls
    Document Class: article 2019/12/20 v1.4l Standard LaTeX document class
    (/usr/share/texlive/texmf-dist/tex/latex/base/size11.clo))
    
    ···

    Output written on document.pdf (14 pages, 1444673 bytes).
    Transcript written on document.log.

The last lines confirm that everything was right, with no **errors** stopping the compilation process. In the event of errors, the message corresponding to the first error and the output in its surroundings should be carefully read and analysed. Errors come in different flavours. It is a matter of practice to develop the skills to spot the source of errors and fix them.

Moreover, several intermediate files may be produced:

- `.aux` Auxiliary file
- `.log` Detailed compilation log
- `.toc` Data to produce the table of contents, if `\tableofcontents` is used
- `.lof` Data to produce the list of figures, if `\listoffigures` is used
- `.lot` Data to produce the list of tables, if `\listoftables` is used
- `.out` Bookmark information that is included in the PDF file, if packages `hyperref` or `bookmark` are used
- `.bcf` BibLaTeX control file, if package `biblatex` is used
- `.run.xml` Processing requests in XML format, if package `biblatex` is used

Requests in `.run.xlm` file are recorded through package `logreq`. They indicate pending operations in a format amenable to further processing by certain automated tools.

Nevertheless, this is the *first pass*, i.e. the first time that we process the document, and it is absolutely normal that we get some **warnings** even if everything is correct. Both, errors and warnings, are logged in the `.log` file where you (`grep` is your friend), your editor, or your integrated development environment of choice, can find them.

The following warning:

    LaTeX Warning: Label(s) may have changed. Rerun to get cross-references right.

indicates that we have used labels that have not been processed in a previous pass and saved in the auxiliary file (`.aux`) and so there are undefined references, which also manifest with the following warning: 

    LaTeX Warning: There were undefined references.

Besides, the following warning:

    Package biblatex Warning: Please (re)run Biber on the file:
    (biblatex)                document
    (biblatex)                and rerun LaTeX afterwards.

indicates that we have to process our bibliography next (with `biber`, in this case), so that the information contained in the `.bib` and `.bcf` files can be used to format the bibliography according to the style specified in the document:

    biber    document 	# Compile references.bib with document.bcf into document.bbl

After processing the bibliography, two new intermediate files appear:

- `.bbl` Bibliography auxiliary file
- `.blg` Detailed log 

Now, we have to process the document again to incorporate the information contained in the `.aux` and `.bbl` files. You may need several passes:

    pdflatex document 	# Compile document.{tex,aux,bbl} into document.pdf (second pass)
	
Indeed, we can observe the following warnings in the output or the `.log` file:

	LaTeX Warning: There were undefined references.
	Package biblatex Warning: Please rerun LaTeX.
	
Therefore, an additional pass is needed:

    pdflatex document 	# Compile document.{tex,aux,bbl} into document.pdf (third pass)

Once no critical warnings are issued, we are done. The rest of the time you can go on with just one `pdflatex` pass, unless you include new references in the document, modify the bibliography, etc.

### Shell escape

Some packages require the execution of external commands. Notably, `minted` executes `pigmentize` to achieve its characteristic beautiful syntax highlighting. This is accomplished through a low level TeX instruction named `\write` (applied to the special output stream `18`), that can be preceded by `\immediate` for immediate execution instead of deferring execution to the end of the page, which is the default behaviour.

For example, we could use the following system-dependent code in LaTeX, which uses `bc` to compute a 40-digit approximation of the golden ratio:

    \begin{displaymath}
      \immediate\write18{echo 'scale=64; (1 + sqrt(5)) / 2' | bc -l > result.tmp}
      \phi \approx \input{result.tmp}
    \end{displaymath}

However, using `\write18` for arbitrary command execution can be dangerous, depending on which commands are used. Therefore, `\write18` is usually disabled and must be enabled by supplying `pdflatex` and the like with the `-shell-escape` option:

    pdflatex -shell-escape document
Restricted `\write18` support is sometimes available by default.
