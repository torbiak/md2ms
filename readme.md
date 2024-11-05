# NAME

md2ms - Markdown to troff -ms macros

# SYNOPSIS

**md2ms** \[ footnote_links=1 \] \[ eqndelim=xy \] \[ *file*\... \]

# DESCRIPTION

*Md2ms* reads the named Markdown input *files* and converts them into
*troff*(1) input. The *troff* input generated by *md2ms* uses macros
defined in the *ms*(7) macro package.

*Md2ms* recognizes Markdown as defined in the original *Daring Fireball*
syntax guide, augmented with a few extensions from Pandoc\'s Markdown.
They are:

Inline math

:   If **eqndelim=***xy* is given on the command line, the pair of
    characters *xy* is understood to delimit inline *eqn*(1) input. The
    text within and including these delimiters is passed verbatim to the
    output. If *y* is missing, it is taken to be *x*; if *x* is missing,
    inline math is disabled. *Md2ms* always outputs an *eqn* block
    setting the inline delimiter to *xy* (or **off**) when *md2ms*
    starts processing an input file, so the user needn\'t specify it
    separately for *eqn* when *md2ms* is used in a *troff* pipeline.

Citations

:   Words matching the regular expression *@\[A-Za-z0-9\'-\]+* are
    output as *refer*(1) citations. One can optionally bracket such a
    word with **\[\]**. Several citations can also be grouped within a
    single pair of brackets by separating them from each other with
    semicolons.

Inline footnotes

:   Input *text* of the form **\^\[***text***\]** is output as *ms*
    footnotes. *Md2ms* additionally defines the number register *FN*,
    which is used to keep a running count of the footnotes. The in-text
    label of a footnote is *FN* wrapped by the string registers *\[.*
    and *.\]*, which are also used by *refer*.

Smart punctuation

:   Double hyphens and triple hyphens are output as the *troff* escape
    sequences for en dashes and em dashes respectively. Typewriter
    single and double quotes are output as the troff sequences for their
    curved counterparts. Multiple spaces are converted into line breaks:
    if two spaces (or a newline) follow each sentence of input, *troff*
    can typeset end-of-sentence spacing better. Multiple spaces at the
    end of a line still cause, as per Markdown, a hard line break
    (**.br**.)

Intra-word emphasis

:   Underscores in the middle of a word won\'t trigger emphasis, but
    asterisks still will.

*Md2ms* passes *troff* requests and escapes through verbatim, just as
Markdown does with HTML. This is the preferred means of customizing
*md2ms* output: redefine the *troff* macros and string and number
registers it uses at the start of a file. Nonempty lines immediately
following *troff* requests are also not processed, so *troff*
preprocessor commands can be embedded in your Markdown source.

*Troff* and Markdown are very different languages, so a 1:1 mapping
between them is impossible. See the extensive **BUGS** section for what
is lost and altered in translation.

Links are turned into hyperlinks using pdfmark by default, but can be rendered
as footnotes instead by setting **footnote_links** to a truthy value.


# SEE ALSO

*eqn*(1), *refer*(1), *troff*(1), *ms*(7)\
J. Gruber, \`\`Markdown Syntax Documentation\'\', *Daring Fireball*.

\
J. MacFarlane, \`\`Pandoc User\'s Guide\'\', section *Pandoc\'s
markdown*.

# BUGS

*Ms* only defines nesting for the **.IP** macro. *Md2ms* uses it to
implement lists, so they are the only block elements that can be safely
nested. To implement strong emphasis, *md2ms* uses the macro **.BI** to
format spans wrapped in triple asterisks or underscores.

*Ms* does not define a horizontal rule macro, so *md2ms* does it
instead:

    .de HR
    .br
    .tl ''* * *''
    ..

*Troff* only supports embedding PostScript images. *Md2ms* uses the
**.PDFPIC** macro in *pdfpic* to do this.

Markdown makes no distinction between numbered and unnumbered headings,
or indented and left-blocked paragraphs. *Md2ms* opts to output every
heading with the **.NH** macro and every paragraph with the **.LP**
macro.

A backslash followed by an asterisk in the input is ambiguous, as it
both escapes emphasis in Markdown and begins the interpolation of a
string register in *troff*. In *md2ms*, it only performs the former
function.

Ordered lists do not have their ordinals computed on the go: they are
output as they are in the source. Likewise, unordered lists use as
bullet points the characters they were invoked with in the source.

*Md2ms* is implemented as a single-pass formatter without lookahead, so
reference-style images and links and setext-style headers cannot be
supported. There is also no graceful backtracking from unclosed span
elements.

*Md2ms* is an *awk*(1) script, so the command line parsing rules
described on its manual page also apply here. As such, take care not to
give *md2ms* arguments that resemble options or variable assignments not
defined above.

*Md2ms* is unlikely to parse edge cases identically to any other
Markdown implementation.
