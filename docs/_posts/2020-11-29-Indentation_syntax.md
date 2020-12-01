---
layout: default
comments: true
title: Indentation Syntax
---
{{ page.date | date: "%Y-%m-%d" }}
## {{ page.title }}

Labels: Syntax

### Indentation-defined program structure

I've planned for a long while to make Tuplex program structure indentation-defined like in e.g. Python.
Dispensing with curly braces and semicolons makes the code easier on the eye, and easier to type.
However this required a complete rewrite of the lexical scanner - the first compilation pass that scans the source text -
so I had been putting it off. Now it's done!


### Syntax summary

The scanner will produce tokens INDENT and DEDENT upon increases and decreases in the indent level.
These have the same semantic meaning regarding scope structure as { and } did previously.

> If a decrease doesn't match an earlier indent level an *inconsistent indentation* error is generated.

A sub-scope is preceded by a header, which ends with a colon and a newline, and is demarcated using an increased indent.
(A missing INDENT after COLON NEWLINE is an error.)

    if i > 0:
        do_this()
        do_that()
    else:
        do_other_stuff()
    do_in_any_case()

Semicolons are now optional. A newline is in most cases sufficient statement terminator.

One-liners are written like this:

    if flag: v = 128; else: v = 256


### Drawbacks

Most programming languages do not use significant whitespace. Instead they use explicit printable tokens,
such as begin/end or { }. This makes it easier to implement parsers,
easier to programmatically generate source code (or embed source code within other files and programs),
and reduces risks of ambiguities in the grammar.

Explicit scope / suite enclosure also makes it easier to format statement lines and blocks as the
programmer sees fit. One can distribute a long statement across several lines,
or list several short statements as a concise one-liner. It separates source formatting from program logic.

If indentation is significant it means the tabs-or-spaces question is no longer entirely facetious.
In any given source text one or the other must be chosen. Although the parser can count both tab and space
characters just fine, it is debatable how many spaces shall correspond to a tab,
and above all they are not visually distinguishable.
Mixing tabs and spaces leads to ambiguity for the programmer, and is therefore disallowed.


### Good-bye curly braces

Python popularized the indentation-defined syntax and showed it can be combined with a largely elegant and
unambiguous grammar if the whole language syntax is designed with this approach in mind.

Curly braces are clear in meaning and stand out visually. But above all they are for the benefit of the parser,
not the programmer. Braces (or begin/end keywords) make it significantly easier to parse blocks and program structure.

A good programmer will indent the code consistently anyway. Braces only rarely convey meaning to a human reader
that isn't already apparent from the indentation, which means they are typically redundant to the human.
If they are redundant they add noise that the programmer must look past, and if they also stand out visually
more attention must be spent on it.

In short, braces are for parsers and indentation is for programmers. Tuplex strives to impose a minimum of syntactic
and grammatical artefacts, and eliminate redundancy and boilerplate as much as possible. Thus the choice to 
reduce visual friction rather than make parser implementation easier.


### Good-bye semicolons

Semicolons are optional, for much the same reasons as braces. The newline is in most cases a very natural statement
terminator. Semicolons may still be specified, for example to denote the empty (no-op) statement, or to separate
multiple statements on the same line.

Some statements get very long, for example long argument lists and complex long condition expressions are not that uncommon.
To facilitate this, a line that ends with a comma will have its linebreak ignored.
And in order to write a long expression across multiple lines it can be enclosed in parentheses.

Generally, any source text enclosed in (parentheses), [brackets], or {braces} will ignore line breaks.


### Attempts at supporting both indentation-defined and brace-defined syntax

With the existing grammar parser (built using Bison) there are still some limitations on how smart it can be
parsing the indentation- and newline-defined syntax. This causes some extended one-liner constructs to still
require braces, or an occasional extra semicolon at the end.

I explored a grammar design that would allow for both worlds, both brace-enclosed and
indentation-defined blocks, used and mixed as the programmer sees fit.
However this leads to a bunch of edge cases, and the biggest
problem is that resolving them requires ad-hoc rules for the grammar - there is
no intuitive "obviously correct way" when mixing them.

Consider for example, if allowing brace-enclosed and indentation-defined blocks within each other,
what a consistent treatment of these blocks would be:

    if condition: {
            brace-enclosed body  ## note - INDENT
        }  ## note - DEDENT...
        {
            body part 2 with its own var-scope
        }

    if condition:
        ## can an if-true-body now consist of several brace-enclosed blocks because they have similar indentation?
        {
            body part 1 with its own var-scope
        }
        {
            body part 2 with its own var-scope
        }
    else:
        something_else()

In the end, I enabled support for brace-enclosed scopes but disallowed mixing them with indentation-defined ones.
Within braces, INDENT, DEDENT, and NEWLINE are simply ignored (same as between parentheses and brackets).


#### Consistent rules

A couple of specific rules needed to be defined to avoid ambiguities.

A. Within braces, all statements must be terminated with semicolon. (This makes the syntax similar to C et al.)

B. The colon must be on the same line as the preceding statement in order to prevents this sort of ambiguity:

    if condition
        :            ## INDENT
        something()  ## no INDENT...

    if condition
        : something()   ## INDENT
          other()       ## inconsistent INDENT

C. One-liner syntax can't be mixed with indentation body syntax. This is an error:

    if condition: do_this()
        do_that()

D. Brace-enclosure is needed to support single-line statement suites:

    if cond1: do_a(); else: if cond2: { do_b(); do_c(); } else: do_d()

E. For lambda expressions within statements, an extra semicolon can be necessary:

    localFunc := ( a : Int )->Int : return a * 2; ;

Without a custom context-sensitive grammar / recursive descent parser the last semicolon above is hard to
avoid. In the current Tuplex grammar it is necessary.
The first semicolon terminates the lambda body's statement; the second terminates the assignment statement.


### Implementing a custom scanner

The lexical scanner needs to understand both line structure, scope structure and token syntax. This
was difficult to implement with the Flex scanner Tuplex was originally built with.
So I built a custom scanner from scratch.

It was quite fun, and ended up being fairly efficient. I did manage to avoid writing an entire
regexp-engine while allowing for a decent amount of flexibility in evolving the lexemes and tokens
of Tuplex. Some benefits of the custom implementation include better support for edge cases in
string formatting syntax and in floating point literals.

One thing it doesn't quite support at this stage is unicode. Tuplex shall support proper UTF-8 eventually,
but at that point the scanner will need some tweaking to not assume single-byte characters.

Since Bison expects a Lex/Flex interface I also wrote a facade for the scanner to translate between their
handling of line/positions and semantic values of literals. Also, since the previous Flex scanner
handled file reading, I had to write a file-to-memory reader to replace it.

Long-term I will probably replace the Bison-generated parser with a custom one as well, but I'm putting it
off as long as I can. Bison can handle almost everything Tuplex needs. There are some edge cases around
NEWLINE interpretation (e.g. the extra semicolon requirement above),
but I think Tuplex can live with that for the time being!
