<!--
DRAFT STATUS — read this before editing further.

This is a first structural draft of the merged edition, not a
finished paper. What's done:
  - Full merge of "Safe REXX on the Desktop" (1993/1998/2023) and
    "Safe REXX in the Enterprise" (1993/2023) — every pitfall from
    both source papers is here; nothing from either original was
    silently dropped (see "Merge notes" at the end of each section
    that differed materially between the two sources).
  - Checked ALL files in the repo, not just the two 2023 HTML
    sources, specifically to catch anything the 2023 web revision
    itself might have dropped along the way (see "Source-file
    provenance" below) — this found one genuinely missing section,
    now restored (see Figure 12 and its surrounding text).
  - A new ooRexx-guidance callout under most sections, grounded in
    AI-Priming/ooRexx/RULES.md's already-verified rules (not
    invented) — each one traceable back to that file.
  - Figures renumbered into one consistent sequence (1-12; the new
    "ooRexx-specific pitfalls" section's code examples are
    deliberately unnumbered, since that section is new content, not
    part of the renumbered classic-paper sequence).
  - PARSE VERSION's actual ooRexx output string verified empirically
    against a real ooRexx 5.2.0 install before writing about it
    (see the note under "PARSE SOURCE and VERSION").
  - A second review pass caught and fixed several real errors from
    the first draft: OREXX was wrongly grouped with classic-Rexx
    dialects (it's an object-Rexx-family member, older and more
    limited than ooRexx, not classic Rexx at all); the Keywords
    section's classic-Rexx word list was too narrow (expanded using
    rexx-lint's own verified `keyword-as-variable` list, plus the
    three special variables RC/RESULT/SIGL); the ooRexx directive-
    keyword note overclaimed that `CLASS`/`METHOD`/etc. need avoiding
    as plain variable names — they're only reserved immediately after
    `::`, a style courtesy at most, not a parsing pitfall like the
    classic-Rexx words; `CALL ON` was wrongly framed as an "ooRexx
    note" when it's standard ANSI X3.274-1996 Rexx, implemented by
    Regina too (a myth this file itself once repeated, per
    AI-Priming/Rexx/RULES.md's own correction history); and the
    Continuation section's "continues if it would be syntactically
    invalid" framing was an oversimplification, corrected to name the
    actual triggering constructs.
  - New "Platforms and standards conformance" section added, researched
    rather than assumed: confirmed via rexxinfo.org's classic-Rexx
    function comparison chart that z/VM CMS REXX has NOT been upgraded
    to ANSI X3.274-1996 level -- it's listed identically alongside
    z/OS TSO/E REXX as lacking CHANGESTR/COUNTSTR/QUALIFY, the same
    situation, not something CMS has that TSO/E lacks or vice versa.
    IBM's own PDF references (vm.ibm.com, publibfp.dhe.ibm.com,
    ibm.com/docs) were not directly fetchable in this pass (TLS
    certificate errors and 403s from this tooling, not necessarily a
    real access restriction) -- the rexxinfo.org chart was the source
    that actually resolved it, cross-checked against a separate
    Google Groups thread and ooRexx's own SourceForge feature-request
    tracker reporting the identical z/OS-side "UNRECOGNIZED FUNCTION"
    behavior for CHANGESTR.

What's NOT done yet (per the README's stated scope: "classic Rexx,
ooRexx, TSO/E, ISPF, OMVS, System REXX, CMS, OS/2, Linux, Windows,
ArcaOS"):
  - No dedicated ISPF/ISPF-edit-macro guidance yet.
  - No dedicated System REXX section yet (z/OS-console-services REXX
    has its own I/O and environment model, distinct from TSO/E REXX —
    not yet covered at all).
  - ArcaOS/OREXX-specific guidance is limited to what AI-Priming
    already had verified (the `~translate` vs `~upper` OBJREXX 6.00
    compatibility note) — OREXX's fuller divergence from ooRexx isn't
    mapped out yet.
  - Windows-specific ooRexx guidance (path handling, case sensitivity
    of `value()` for environment variables) is present but thin.

Source-file provenance, checked across every file in the repo (both
top-level and every subdirectory: NASPA/, OS2DEV/, Attached/, TSM/,
$REXX/, "Safe REXX/"), not just the two obvious 2023 HTML files:
  - safe_rexx_desktop.html and safe_rexx_enterprise.html (both dated
    Jan/Feb 2023) are the correctly-labeled, already-modernized full
    texts — confirmed by reading both in full and diffing them
    against each other section by section.
  - The name "SAFEREXX" is ambiguous, not a mislabeling: both papers
    once shared that working filename before one was later
    disambiguated to "saferexxe". NASPA/SAFEREXX.* is, despite the
    shared name, the Enterprise paper's own archived copy (its actual
    title text, verified by extracting readable runs from the binary
    .DOC, is "Safe REXX in the Enterprise"); OS2DEV/SAFEREXX.* is the
    Desktop paper (confirmed by its embedded "OS/2 Developer" journal
    submission header). The top-level SAFEREXX.DOC and SAFEREXX.sdw
    happen to be copies of NASPA's (Enterprise) files rather than
    OS2DEV's (Desktop) ones, sitting next to the Desktop-sourced
    SAFEREXX.ASC/.FIG/.THD/.W51 at the same top level — worth knowing
    if anyone reaches for "the top-level SAFEREXX.* files" as a set
    expecting them all to be one paper. This draft used only the
    correctly-verified 2023 HTML versions as source text, not any of
    the top-level files, so the ambiguity didn't affect it either way.
  - The two 2023 HTML versions are closely related, not independently
    authored: NASPA/SAFEREXX.DOC's own text says "A slightly different
    version of this article was printed in the February 1995 OS/2
    Magazine" — i.e. Enterprise is the earlier/base text and Desktop
    is a deliberately adapted variant for a PC-focused readership,
    confirmed independently by OS2DEV/SAFEREXX.THD, the author's own
    1993 pitch letter to OS/2 Developer's editor, which mentions
    intending "the same, or a similar, article" for NaSPA too.
    Confirmed by direct diff: same section order throughout, but
    Desktop has an entire extra pitfall (the WITH/VALUE parse-keyword
    confusion, Figures 6-8 below) that Enterprise omits completely,
    while Enterprise has richer dual CMS+TSO worked examples and an
    extra paragraph on TSO/E's stream-I/O limitations that Desktop
    omits. Both differences are preserved below, not merged away.
  - A real, small bug was found in the current Enterprise HTML during
    this comparison: its I/O-model paragraph says "See Figure 10" but
    the actual next heading is "Figure 7" (a leftover, unfixed
    reference from before that paper's figures were last renumbered).
    Moot here since this draft renumbers everything into one sequence
    anyway, but worth fixing in safe_rexx_enterprise.html directly if
    that file is kept in independent circulation going forward.
  - Real gap found by diffing the 2023 HTML against BOTH papers'
    original 1993 plain-text submissions (OS2DEV/SAFEREXX.ASC,
    NASPA/SAFEREXX.ASC) and an intermediate ~1997-98 web-published
    snapshot (Attached/Vpub/safe_rexx.html, "Safe REXX/safe_rexx.html",
    both mirrored again under TSM/ — confirmed as presentation-only
    variants of that same snapshot, not distinct content, by direct
    diff): both papers originally had a subsection, "REXXUTIL and
    Emergency Boot Disks", positioned between "PARSE SOURCE and
    VERSION" and "Variable patterns", that the final 2023 revision
    dropped from *both* papers identically. Not an accident specific
    to one paper or one revision pass — a deliberate-looking cut, made
    the same way twice, of what by 2023 was genuinely obsolete advice
    (OS/2's Presentation Manager was too large for a 1.44MB emergency
    boot floppy, so scripts meant to run from one needed a
    non-RexxUtil-dependent fallback path). The underlying principle —
    don't assume an optional function library is loaded; check and
    fall back — still generalizes today, so it's restored below in
    modernized form (Figure 12) rather than left dropped a second
    time, with the original figure's exact code preserved and the
    obsolete framing swapped for the still-current one.
-->

# Safe REXX in the Enterprise and on the Desktop, or Will They Still Respect My Code in the Morning?

by Shmuel (Seymour J.) Metz

## Publication history

This is a merged and updated edition of two closely related articles
by the same author: *Safe REXX on the Desktop* (© 1993, revised 1998,
originally printed in *OS/2 Magazine*, February 1995) and *Safe REXX
in the Enterprise* (© 1993, the earlier/base text, from which
*Desktop* was adapted for a PC-focused readership). Both were updated
for the web in January/February 2023. This edition merges them into
one text, adds explicit ooRexx guidance throughout (both papers
predate ooRexx and explicitly disclaimed direct experience with
Object REXX), and extends the scope to the fuller range of platforms
and dialects in current use: TSO/E, ISPF, OMVS, System REXX, CMS,
e.g., Classic REXX, Object Rexx, ooRexx, Regina.

Copyright 1993, 1998, 2023, 2026 by Shmuel (Seymour J.) Metz. All rights
reserved. Permission for reproduction in whole or in part is hereby
granted to educational, non-profit and computer user groups for
internal, non-profit use, provided credit is given and this notice is
included. All other reproduction without the author's prior written
permission is prohibited.

---

## Table of contents

1. [Introduction](#introduction)
2. [What is REXX?](#what-is-rexx)
3. [Platforms and standards conformance](#platforms-and-standards)
5. [Summary of pitfalls](#summary-of-pitfalls)
6. [Specific examples and recommended avoidance tactics](#specific-examples)
   - [Abutment](#abutment)
   - [Continuation](#continuation)
   - [Keywords](#keywords)
   - [Labels and SIGNAL](#labels-and-signal)
   - [Parsing](#parsing)
   - [Scoping rules](#scoping-rules)
   - [Type and range checking](#type-and-range-checking)
   - [Uninitialized variables used as constants](#uninitialized-variables)
   - [Variable references](#variable-references)
7. [Compatibility and environmental considerations](#compatibility)
   - [ADDRESS and the default environment](#address)
   - [Environmental factors](#environmental-factors)
   - [I/O model](#io-model)
   - [PARSE SOURCE and VERSION](#parse-source-and-version)
   - [Availability of optional function libraries](#function-library-availability)
   - [Variable patterns](#variable-patterns)
8. [ooRexx-specific pitfalls](#oorexx-specific-pitfalls)
9. [Recapitulation](#recapitulation)
10. [References](#references)
11. [Notes and trademarks](#notes-and-trademarks)
12. [About the author](#about-the-author)

---

<a id="introduction"></a>
## Introduction

<a id="what-is-rexx"></a>
### What is REXX?

REXX is a language that was originally designed to replace the EXEC
and EXEC2 command-macro languages in the CMS component of IBM's
VM/SP. Since then it has spread to a large number of other platforms,
including Unix, and has been designated by IBM as the SAA Procedures
Language. REXX has been used to implement a wide variety of
applications beyond its original problem domain, including many of
substantial size. **Open Object Rexx (ooRexx)** is a widely-used,
open-source, object-oriented extension of classic Rexx: it executes
unmodified classic Rexx programs and adds classes, methods, and
message-send syntax on top. Everything in this edition that applies
to classic Rexx applies unchanged to ooRexx unless a specific note
says otherwise.

<a id="platforms-and-standards"></a>
### Platforms and standards conformance

The American National Standards Institute published a Rexx standard,
ANSI X3.274-1996, adding a number of enhancements beyond Cowlishaw's
original *The Rexx Language* 2nd-edition ("TRL-2") specification —
among them the `CHANGESTR`, `COUNTSTR`, and `QUALIFY` built-in
functions and the `LOSTDIGITS` condition. ooRexx and Regina both
implement these ANSI-1996 enhancements. IBM's mainframe classic-Rexx
interpreters do not: TSO/E REXX (z/OS) and CMS REXX (z/VM) are both
explicitly documented as lacking `CHANGESTR`/`COUNTSTR`/`QUALIFY` —
confirmed directly against rexxinfo.org's classic-Rexx function
reference chart, which lists both mainframe interpreters identically
as "not supported" for these, rather than treating z/VM as somehow
upgraded past z/OS's level. Neither mainframe interpreter has been
brought up to ANSI-1996 level; both remain at essentially the older
TRL-2 function set. This matters when porting code between
mainframe and non-mainframe Rexx: code that leans on `CHANGESTR` or
`COUNTSTR` for convenience will not run unmodified on TSO/E or CMS,
regardless of which one you started on.

TSO/E REXX does have one significant environment-specific carve-out
despite this: it supports stream I/O (`LINEIN`, `LINEOUT`, `STREAM`,
and the like) only when running in the UNIX System Services (OMVS)
shell, not in an ordinary TSO/E address space, which uses `EXECIO`
instead — see [I/O model](#io-model) below for the full detail and a
worked example of both forms.

REXX has a number of features that can trap the unwary. This does not
mean that REXX is a bad language, just that you need to understand it
for what it is, as you must for any other programming language. Some
of these features are just language glitches, while in other cases
they were added as the necessary price for greater expressive power.

One of the easiest ways to run afoul of REXX is to be misled by
superficial similarity with other languages, especially PL/I, TSO
CLISTs, and languages derived from them. Make a conscious effort to
learn REXX on its own terms, without relying on analogies with other
languages. This applies with equal force to ooRexx: its message-send
syntax and class model can look superficially like other
object-oriented languages, but its scoping and activation rules have
their own logic, covered below.

Other areas that may confuse the neophyte are the use of abutment for
concatenation, the use of uninitialized variables as constants, the
rules for continuation, parsing, the block structure, and the way
variable references are passed. Later sections go into detail on each.

You may write REXX code that must run on multiple platforms, or in
different environments on the same platform. REXX has some language
features that may impede portability. It also has some features that
may be exploited to improve portability. Later sections give
guidelines on how to ease migration between environments and between
platforms.

Of course, there are many generic principles of defensive programming
that apply just as much to REXX as to any other language. These
include:

- Use meaningful variable names
- Use judicious comments
- Use a consistent indentation style

Although this edition discusses only issues and solutions specific to
REXX and ooRexx, those generic principles are of equal importance in
avoiding programming errors.

Sections marked **ooRexx note** cover behavior specific to Open
Object Rexx that does not apply to classic Rexx dialects (TSO/E REXX,
CMS REXX, System REXX, Regina, and the like). Note that OREXX (IBM's
original Object REXX for OS/2, the precursor ooRexx implements and
succeeds as an open-source project) is itself an object-Rexx family
member, not a classic-Rexx dialect — but an older, more limited one:
see the specific `~translate` vs `~upper` gap noted below.

---

<a id="specific-examples"></a>
## Specific examples and recommended avoidance tactics

Although REXX has a number of features that lend themselves to fast
prototyping, it has a few pitfalls that can beset the unwary.

<a id="abutment"></a>
### Abutment

Although REXX has a conventional concatenation operator (`||`), it
also supports two other concatenation operators: abutment with white
space and abutment without white space (see Figure 1). With abutment
an expression is abutted against a second expression. If there is
white space (e.g., blanks, tabs) between the two, the resulting value
is formed by concatenating a **single** blank between the other two
values; otherwise the result is formed by simple concatenation. It is
a common beginner's error to add or remove a blank that appears to be
irrelevant to the program's semantics, only to change the output.

Another common error is to abut a literal string with a single
character variable name. If the variable name is a valid suffix for a
literal string, e.g., `X` (for hexadecimal) or `B` (for binary), it
will be treated as part of the literal string, not as a variable
reference. For this reason, among others, it is best not to use
one-character names for your variables.

It is so easy to misuse abutment that some recommend not to use it at
all. That position is extreme, since abutment is so convenient and
readable, but exercise caution and good judgement in its use.

#### Figure 1: Concatenation operators

```rexx
/* Explicit (conventional) concatenation */
dog = "Peke"
say "Tom's " || dog || "s" /* output is "Tom's Pekes" */

/* Abutment */
dog = "Peke"
say "Dick's "dog"s"        /* output is "Dick's Pekes" */

/* Abutment with white space */
dog = "Peke"
say "Harry's" dog "s"      /* output is "Harry's Peke s" */

/* Incorrect abutment of X -- the risk depends on your platform's
   native character encoding */
x = 'unknown'
say '41'X                  /* ASCII platforms (OS/2, Linux, Windows):
                               displays "A", not "41unknown" */
say 'C1'X                  /* EBCDIC platforms (CMS, TSO/E, System
                               REXX): displays "A", not "C1unknown" */
```

<a id="continuation"></a>
### Continuation

REXX's continuation rules are more nuanced than "an incomplete line
continues." Implicit continuation happens at specific trailing
constructs — a comma awaiting the next argument, a binary operator
awaiting its right operand, an unclosed parenthesis or bracket — not
merely because a line "looks incomplete" in some general sense. A
line that ends in the middle of an unterminated quoted string is a
different failure altogether (a lexical error), not a continuation
case at all: REXX has no line-continuation mechanism for a string
literal split across lines. Explicit continuation is requested with a
trailing comma, which is itself context-sensitive — see the argument-
separator pitfall just below, where a comma already meaningful in its
own right (an argument separator) has to be followed by a *second*,
purely continuation-marking comma. This presents two common pitfalls
for the unwary.

If you break a procedure invocation after a comma, the trailing comma
will be treated as an explicit continuation request rather than as an
argument separator. In this situation you **must** add an additional
comma as an explicit continuation request in order to allow the
separator to be recognized. See Figure 2.

#### Figure 2: Continuation after argument separator

```rexx
/* Example for CMS types */
say value('X',,'LASTING FOO')   /* retrieves X with no side effects */
say value('X',,                 ,
        'LASTING FOO')          /* same as above */
say value('X',,
        'LASTING FOO')          /* displays current value of X and then
                                   sets X to 'LASTING FOO' */

/* Example for TSO types */
say outtrap('mystem',,'CONCAT') /* retrieves output to stem mystem */
say outtrap('mystem',,          ,
        'CONCAT')               /* same as above */
say outtrap('mystem',,
        'CONCAT')               /* error: parameter 2 is not numeric! */

/* Example for OS/2 (and other environment-variable-based) types */
say value('X',,'OS2ENVIRONMENT') /* retrieves X with no side effects */
say value('X',,                  ,
          'OS2ENVIRONMENT')      /* same as above */
say value('X',,
          'OS2ENVIRONMENT')      /* displays current value of X and
                                    then sets X to OS2ENVIRONMENT */
```

If you break an expression after a literal or variable that is not
enclosed in parentheses, the statement will be treated as complete
and the next line will be treated as a new statement. In this
situation you **must** supply a trailing comma as a continuation
request. See Figure 3.

#### Figure 3: Continuation after expression

The `ECHO` command used in these examples is present in various PC
operating systems and in the Unix subsystems of MVS and VM.

```rexx
/* Example for CMS types */
'ECHO' 'DIR'                     /* displays 'DIR' */
'ECHO'                      ,
'DIR'                            /* same as above */
'ECHO'
'DIR'                            /* displays directory */

/* Example for TSO types */
'HELP' 'LISTC'                   /* displays help for LISTC */
'HELP'                      ,
'LISTC'                          /* same as above */
'HELP'                           /* displays generic help */
'LISTC'                          /* displays catalog */
```

Note that although in some cases REXX will recognize a syntax error
when you omit a required explicit continuation character, in other
cases you will get incorrect results with no error message.

<a id="keywords"></a>
### Keywords

Avoid the use of variables with the same name as a REXX keyword. If
you use such names you risk having statements misinterpreted or
rejected as invalid. See Figure 4. This is similar to the problem of
one-character variable names being misinterpreted when abutted to
literal strings. Even if you are careful to write code that does what
you want, use of those names will confuse whoever has to modify your
code, possibly including yourself.

The words that most often cause real *parsing* trouble — because they
double as `PARSE` subkeywords, so the parser genuinely reads them
differently depending on position — are:

```
ARG             PULL            VAR
EXTERNAL        SOURCE          VERSION
NUMERIC         VALUE           WITH
```

Figures 5 through 7 below show exactly how `VALUE`, `VAR`, and `WITH`
can misparse. Beyond that narrower set, avoid the *fuller* list of
REXX instruction keywords, keyword clauses, and common subkeywords as
plain variable names too — legal in every case, and the parser
resolves each occurrence correctly by position, but it is a real
readability hazard for a human reader, not just the parser:

```
ADDRESS   ARG        CALL       DO         DROP      ELSE
END       EXIT       EXPOSE     IF         INTERPRET ITERATE
LEAVE     LOOP       NOP        NUMERIC    OPTIONS   OTHERWISE
PARSE     PROCEDURE  PULL       PUSH       QUEUE     RETURN
SAY       SELECT     SIGNAL     THEN       TRACE     UPPER
WHEN      BY         FOR        FOREVER    TO        WHILE
UNTIL     WITH       ON         OFF        VALUE     CONDITION
DIGITS    FORM       FUZZ
```

Also avoid REXX's three special variables — `RC`, `RESULT`, `SIGL` —
as names for your own ordinary variables, for the same reason:
they already carry a specific meaning the language sets on your
behalf (`RC` from host commands, `RESULT` from `CALL`/function
invocations, `SIGL` the line number of the most recent `SIGNAL` or
`CALL`), and naming your own variable the same thing invites exactly
the kind of silent confusion this section is about. See
[Variable references](#variable-references) below for the `RC`-vs-`RESULT`
distinction in more detail, and the ooRexx-specific `RESULT` trap
under [Uninitialized variables](#uninitialized-variables).

#### Figure 4: Misinterpreted keyword

```rexx
text = 'tom dick harry'
with = 'Ada Emmy Gracie Lise'
/* we want to parse 'tom dick harry Ada Emmy Gracie Lise'
   with 'with first rest' */
parse value text with first rest      /* wrong ! */
parse value text with with first rest /* also wrong ! */
```

Also, be careful about your use of the keywords `VALUE`, `VAR`, and
`WITH`. The code in Figures 5 and 6 will produce quite unexpected
results, and was probably meant to behave like the code in Figure 7.
In general, use `VAR` for simple parsing.

#### Figure 5: Misuse of WITH

```rexx
stg = abc
parse var stg with x +1 y +1 z
/* sets with='ABC' x='' y='' z='' */
```

#### Figure 6: Misuse of VALUE

```rexx
stg = abc
parse value stg x +1 y +1 z
/* Error 38! */
```

#### Figure 7: Correct use of WITH and VALUE

```rexx
stg = abc
parse value stg with x +1 y +1 z
/* sets x='A' y='B' z='C' */
/* Equivalent to, and better form in this case: */
parse var stg x +1 y +1 z
```

**ooRexx note**: ooRexx's `::CLASS`, `::METHOD`, `::ROUTINE`,
`::REQUIRES`, `::ATTRIBUTE`, `::CONSTANT`, `::OPTIONS`, `::RESOURCE`,
and `::PACKAGE` directives introduce keywords that are reserved
*only* immediately after `::` — `::foo` being a directive keyword
does not make bare `foo` one. `class = 5` elsewhere in the same file
parses perfectly and unambiguously as an ordinary variable assignment;
it is not a parsing pitfall the way the classic-Rexx words above are.
It is still worth avoiding in a file that also defines real classes
with `::CLASS`, purely as a courtesy to the human reader — which is
why rexx-lint's own `keyword-as-variable` check flags it as a style
warning, not an error. Body keywords that genuinely do appear as bare
words inside `::METHOD`/`::ROUTINE` bodies — `EXPOSE`, `GUARD`,
`FORWARD`, `USE` — belong on the avoid-as-variable-name list in the
same way the classic-Rexx keywords above do.

<a id="labels-and-signal"></a>
### Labels and SIGNAL

The `SIGNAL` statement in REXX looks very much like a `GOTO` in PL/I
and other block-structured languages, but its semantics are very
different. Do not attempt to use `SIGNAL <labelname>` as a substitute
`GOTO` or you will cause yourself serious difficulties. Although the
form `SIGNAL <labelname>` will cause a jump to the code with that
label, it also flushes the control stack. A subsequent `END` statement
will be detected as an error (see Figure 8). It is best to use
`SIGNAL` strictly for its intended purpose of indicating exceptional
conditions.

#### Figure 8: SIGNAL errors

```rexx
do forever
   signal BELL
   whatever
   BELL:
   end                              /* an error will be detected here
                                        because the SIGNAL logically
                                        terminated the DO */
```

`SIGNAL ON <condition>` and `CALL ON <condition>` both arm condition
traps, but they are not interchangeable, and this is standard ANSI
Rexx (X3.274-1996) behavior, not an ooRexx extension — `CALL ON` is
implemented by Regina too, and any claim that it's ooRexx-only is a
myth worth retiring. The two forms differ in resumability: `SIGNAL
ON` behaves like the unconditional `SIGNAL` above and flushes the
call stack when it fires, with no return to the point of the
condition. `CALL ON` calls the trap routine as a subroutine and
*returns* to the point right after the guarded instruction once the
trap routine finishes. If a guarded host command or block should be
able to resume afterward, use `CALL ON`, not `SIGNAL ON`.

ANSI supports both forms for `ERROR`, `FAILURE`, `HALT`, and
`NOTREADY`; `SYNTAX` and `NOVALUE` are `SIGNAL ON`-only conditions —
there is no `CALL ON SYNTAX` or `CALL ON NOVALUE` in the standard. Not
every platform implements every condition identically: TSO/E REXX
does not support `NOTREADY` at all, for instance. Check your target
dialect's own reference before assuming a given condition/form
combination is portable.

<a id="parsing"></a>
### Parsing

The parsing facilities of REXX have several features that may be
confusing to the neophyte.

REXX has keywords for abbreviated forms of `PARSE`, e.g., `ARG` is
short for `PARSE UPPER ARG`. Beginners often forget that these
abbreviated forms will translate all data to upper case.

When using `PARSE` or its abbreviations, it is important that you
remember that the last variable or period (`.`) is treated
differently from all of the others; in general its value will include
leading **and** trailing blanks. Use the `STRIP` function or a
trailing period to remove these if they are unwanted.

**ooRexx note**: when the parse source is already a variable, prefer
`PARSE VAR` over `PARSE VALUE ... WITH` — it is more readable, saves a
keyword, and makes the source obvious at a glance:

```rexx
parse var foo template          /* preferred when foo is a variable */
parse value foo || bar with template   /* PARSE VALUE reserved for
                                           genuine expression sources */
```

**ooRexx note**: in a parse template, a variable **not** enclosed in
parentheses is a *receiver* — it is assigned the next parsed token,
and does **not** match against the variable's current value. A
variable enclosed in parentheses, `(foo)`, is a *match pattern* —
Rexx uses `foo`'s current value as a literal string to scan for.
Confusing the two is a silent logic error, not a syntax error:

```rexx
parse var line word rest     /* word RECEIVES the first token */
parse var line (delim) rest  /* Rexx SCANS for delim's current value */
```

<a id="scoping-rules"></a>
### Scoping rules

Although superficially REXX appears to be a block-structured
language, it is actually a hybrid between dynamic and static scoping.
It is possible, although bad form, to call a label inside a `DO` from
code outside the `DO`. It is possible to invoke code at an arbitrary
label as both a call and as a function invocation. It is incumbent
upon the programmer to supply the discipline that the language omits.

The scope of a procedure is determined strictly dynamically; there is
no static terminator such as `END`.

Do not write code intended to serve as both inline and out-of-line
code; programs in which you both call and fall through into the same
code are notoriously error prone. Precede each internal subprocedure
with a statement that will prevent accidentally falling into it,
e.g., `EXIT`; if your logic permits, begin the procedure with a
`PROCEDURE` statement, which must be the first statement after the
label. See Figure 9.

#### Figure 9: Procedure isolation

```rexx
saytime: PROCEDURE /* here I can get away with hiding all variables */
   say time
   return
/* Note that there is no END statement ! */

exit /* In case of fallthrough, since I can't use PROCEDURE */
putdata:
   parse arg name .
   say name'='value(name)
   return
/* Note that there is no END statement ! */

badstyle: PROCEDURE                 /* This entry has no access */
badform:                            /* This entry has access    */
...
return
/* Don't ever do this; it is an extremely dangerous style */
```

The `PROCEDURE` statement hides all variables except those explicitly
listed in an `EXPOSE` clause. If your subroutine accesses the
caller's variables and constructs those variable names from its
arguments, then you must not use the `PROCEDURE` statement. This is
the only situation in which you should omit it. See Figure 9.

It is possible to write procedures with overlapping scope in which
one procedure hides variables with a `PROCEDURE` statement and the
other procedure leaves all variables exposed by default. This is a
dangerous practice, and should be avoided.

**ooRexx note**: `EXPOSE` has two genuinely different meanings
depending on where it appears, and they are easy to conflate.
`PROCEDURE EXPOSE` (used inside a classic internal subroutine, as
above) exposes the *caller's* local variables. `EXPOSE` used as the
first statement of an `::METHOD` body exposes that object's *instance*
variables — a completely different variable pool, private to the
object, not the caller's locals. And **`EXPOSE` is not legal at all
inside a `::ROUTINE`** — a routine has no access to any caller's
variable pool the way an internal subroutine does. Attempting it
raises a parse-time error (Error 27.1) *before any instruction in the
calling scope executes at all* — which means a `SIGNAL ON SYNTAX`
trap set up in the caller will never fire, because the whole program
fails to parse before execution ever begins. If code invoking such a
routine is launched as a child process, the symptom is a silent
failure with a non-zero return code and nothing in any in-process
trap; the real diagnostic is only in the child's captured stderr.

<a id="type-and-range-checking"></a>
### Type and range checking

Unlike most other languages, REXX has neither variable typing nor
arrays. Arrays are often simulated using compound variables. This
leads to several possible types of undetected errors.

When you assign a value to a variable, there is no check that the
value is consistent with the intended type. If your logic requires
any constraints on the values that can be assigned, it is your
responsibility to code explicit checks using, e.g., the `DATATYPE`
function.

When you use a variable name as part of a compound variable in order
to simulate an access to an array element, REXX does not check that
the index is within the array extents, or even that it is an integer.
If your logic requires enforcing such constraints, you must code them
explicitly. Note that even an uninitialized variable can be used as
an "index" for a compound variable.

**ooRexx note**: ooRexx provides real collection classes — `.Array`,
`.Directory`, `.Table`, `.Set`, `.Bag`, `.Queue`, `.OrderedCollection`,
and others — as a genuine alternative to stem-simulated arrays, with
actual bounds/type behavior rather than the silent-anything-goes
behavior of a compound variable:

```rexx
arr = .Array~of('a', 'b', 'c')      -- construct-and-populate in one call
do item over arr
    say item
end
```

Prefer `do item over collection` to `do i = 1 to stem.0` when the
data doesn't need positional indexing at all.

**ooRexx note — indirect/computed stem access has three forms, and
only one is correct per context; the wrong ones don't always error.**
This is a real trap precisely because two of the three wrong
combinations look plausible:

```rexx
/* 1. Classic compound-variable indirect tail access: dot, then bracket */
mystem.1 = 'one'; mystem.2 = 'two'; mystem.3 = 'three'
i = 3
say mystem.[i]         /* CORRECT: 'three' */
say mystem.(i)         /* WRONG: Error 43 -- parsed as a call to a
                           routine literally named MYSTEM */
say mystem[i]          /* WRONG, but raises NO error -- an unset
                           simple variable MYSTEM evaluates to the
                           string "MYSTEM", and [] on a bare string is
                           ordinary character indexing: this silently
                           returns "S" (character 3 of "MYSTEM"), not
                           the stem element at all */

/* 2. A real Stem/collection object: plain bracket notation is correct
   here -- but it is a SEPARATE namespace from same-named classic
   compound variables, even when the base name matches */
realStem = .stem~new
realStem[3] = 'bar'
say mystem.3            /* still uninitialized "MYSTEM.3" -- unrelated
                            to realStem[3] even if named the same */

/* 3. Using another compound variable directly as a tail component
   looks like it should nest, but the tail is split on periods into
   independent pieces BEFORE any substitution happens -- a piece is
   never itself re-parsed as a compound-variable reference */
orphans.0 = 0
orphans.0 = orphans.0 + 1
orphans.orphans.0 = 'first'      /* WRONG: not "orphans.1" -- every
                                     iteration clobbers the SAME fixed
                                     derived tail ORPHANS.ORPHANS.0 */
```

The safe pattern in every case: copy the index into a plain simple
variable first, then use that variable as the tail (`n = orphans.0;
orphans.n = value`) — or, better, use a real collection object and
plain bracket notation throughout rather than simulating one with
compound variables.

<a id="uninitialized-variables"></a>
### Uninitialized variables used as constants

When you refer to an uninitialized variable, its value is by default
its name in upper case. This is frequently a convenient alternative
to the use of literal strings. However, if you inadvertently use that
name for a true variable elsewhere in the program, you may get
incorrect and apparently inexplicable results. It is best to adopt
conventions for your variable names that minimize the risk of such
problems.

Some recommend always using explicit literal strings for constants.
Although well meant, this advice can lead to programs that are harder
to read. Use uninitialized variables, but judiciously. If you choose
to not exploit the default behavior for uninitialized variables,
place a `SIGNAL ON NOVALUE` at the beginning of your program to
detect violations of that decision.

**ooRexx note — never name your own variable `result`.** This is the
same "uninitialized variable reverts to its own name" behavior above,
but with a genuinely surprising trigger in ooRexx: `result` isn't only
set by `CALL`. *Any* bare message-send statement — a whole clause, its
return value not assigned to anything — is handled the same way as a
`CALL` to a routine with no `RETURN` value: if the invoked method
returns nothing at all (not even `.nil` — several Collection methods,
e.g. `~put`, are defined to return no result object), `result` is
*dropped* back to uninitialized. This breaks the moment any bare
message-send whose method returns nothing executes — including the
ordinary case of building up your *own* local variable named `result`
via repeated bare sends to it:

```rexx
result = .Directory~new        -- fine: plain assignment
result~put('', 'INTERPRETER')  -- runs; but ~put returns no result
                                   object, so RESULT reverts to
                                   uninitialized right after this line
result~put('', 'DIALECT')      -- Error 97.1: Object "RESULT" does not
                                   understand message "PUT" -- the
                                   PREVIOUS line already reverted it
```

Verified directly, in order: `putReturn = d~put('v','k')` raises
"Message did not return a result," proving `~put` truly returns
nothing; a single bare `result~put(...)` statement reproducibly drops
`result` immediately afterward; an identically-shaped sequence using
any other variable name is never affected. Pick any other name
(`info`, `found`, `outcome`, ...) — there is no scope in which reusing
`result` as your own variable buys anything.

<a id="variable-references"></a>
### Variable references

Classic Rexx does not allow passing parameters to procedures by name
or by reference. However, you can often get similar results by
passing constants and using them to construct names. This is an
extremely common and powerful technique, especially in conjunction
with compound variables. However, there are a few pitfalls.

If you call a procedure that has an `EXPOSE` clause on the
`PROCEDURE` statement, it will only have access to the variables that
you exposed. If you pass an argument containing the name of some
other variable, the code will only be able to access a local version
of that variable.

If you call a procedure that requires a variable name as a parameter,
and use an uninitialized variable to represent its own name for that
parameter, you will probably get incorrect results on your second
time through.

**ooRexx note**: ooRexx closes the by-reference gap classic Rexx
disclaims above, with a real `USE ARG` statement:

```rexx
::routine adjustBalance
  use arg account      -- account is a genuine mutable reference to
                           the caller's object, not a copy
  account~balance = account~balance - fee
```

`USE ARG` is for mutable *objects* passed by reference — it does not
turn a plain string or number into a by-reference parameter the way
some other languages' reference parameters do, since Rexx strings and
numbers are themselves immutable values.

**ooRexx note**: `rc` and `result` are set by different things, and
conflating them is a real, easy-to-make bug. `rc` is set **only** by
host commands — a bare host-command clause, `ADDRESS foo 'expr'`, or
similar. It is **not** set by `CALL` or a function/method invocation.
`result` is set by `CALL` and by any unassigned function invocation
(see the caution about naming your own variable `result`, above).
Reading `rc` after `CALL SysFileCopy` reads the *previous* host
command's return code (or the literal string `'RC'` if none has run
yet), not the routine's own outcome:

```rexx
call SysFileCopy src, dst
copyRc = result           /* CORRECT -- SysFileCopy is a routine call */

address system 'some-command'
cmdRc = rc                /* CORRECT -- a host command sets rc */
```

---

<a id="compatibility"></a>
## Compatibility and environmental considerations

REXX has some specific features that you can exploit to make your
programs more compatible across platforms, or between environments on
the same platform. REXX also has some features that hinder
compatibility.

<a id="address"></a>
### ADDRESS and the default environment

If you write a command file that issues host commands — OS/2, CMS,
TSO, DOS commands — do not assume that the default environment is
that of the host itself. By including, e.g., `ADDRESS TSO` (on a
mainframe) or `ADDRESS CMD` (on OS/2/Windows), you enable the routine
for use from within other environments too, e.g., the ISPF/PDF editor
on a mainframe, or an editor that uses REXX as its macro language on
a PC.

**ooRexx note**: ooRexx (and standard Rexx generally, not an
ooRexx-only extension) can capture a child process's stdout and
stderr directly into stems, with no temp files or pipes needed:

```rexx
address system 'some-command' with output stem out. error stem err.
cmdRc = rc
do i = 1 to out.0
    say out.i
end
do i = 1 to err.0
    say '  [stderr]' err.i
end
```

Valid I/O redirect types in the `WITH` clause are `NORMAL`, `STEM`,
`STREAM`, and `USING` — `STRING` is not a valid type. To supply empty
stdin (preventing a child process from blocking waiting for input),
define an empty stem and pass it as `INPUT STEM`:

```rexx
noIn.0 = 0
address system cmd with output stem out. error stem err. input stem noIn.
```

Do not wrap the command itself in `cmd /c "..."` on Windows, even
though it may seem like it should be needed — `ADDRESS SYSTEM`
already dispatches straight to the platform's native shell. An extra
`cmd /c` layer is redundant at best, and actively wrong once the
wrapped command itself contains its own quoted arguments (a path with
spaces, a commit message with spaces): `cmd.exe`'s quote parser does
not reliably handle the resulting nested quoting.

<a id="environmental-factors"></a>
### Environmental factors

REXX does not shield you from the underlying environment; in writing
a REXX program you must understand the behavior of your operating
system and user interface if you want to avoid nasty surprises. As an
example, if you invoke a REXX program in an OS/2 CMD file and scan
the argument looking for the string `/Q`, you will not find it,
because `CMD.EXE` will have taken the string `/Q` to be a "quiet"
option and removed it.

If you must use binary or hexadecimal constants for character data,
be aware that character encoding varies among systems. CMS and TSO
use EBCDIC; most other systems, e.g., OS/2, Linux, Windows, use
ASCII. Even within CMS and TSO there are national-language issues,
and in many systems there are code-page issues. Be aware of the
character sets used in each of your target systems, and program
accordingly. Segregate system-dependent values and code-page-dependent
values to make your code easier to maintain.

**ooRexx note**: ooRexx variable names and labels are case-insensitive
regardless of platform, but two related things are not, and the
platform matters:

- Filenames on Windows (NTFS) or ArcaOS/OS2 (JFS) may be
  case-insensitive in practice, but should still be treated as
  case-sensitive in code that must also run on Linux.
- The `VALUE()` built-in for reading environment variables is
  case-sensitive on Linux but case-insensitive on Windows.

<a id="io-model"></a>
### I/O model

The I/O model in REXX is based on the file system in the CMS
component of IBM's VM/SP. Most other systems, e.g., DOS, Linux, OS/2,
Windows, do not have an orientation towards line numbers. TSO/E in
MVS does not support the REXX I/O model at all; it uses a subset of
`EXECIO`. For this reason, the `CHARS` and `LINES` functions on those
non-CMS systems only return values of 0 and 1. See Figure 10. Code
that expects an exact number of characters or lines from those
functions may fail on those systems, and an implementation that
*did* return exact numbers on, e.g., Linux, could be horribly
inefficient.

#### Figure 10: I/O examples

```rexx
/* In standard REXX */
do lines(myfile)
   myline = linein(myfile)
   ...
   end
/* In OS/2 this would only read one line ! */

/* In OS/2 SAA REXX */
do while lines(myfile) /= 0
   myline = linein(myfile)
   ...
   end

/* In CMS and TSO REXX */
'EXECIO *' name '(STEM MYSTEM. FINIS'
do i = 1 to mystem.0
   parse var mystem.i myline
   ...
   end
```

TSO/E REXX in MVS does not support stream I/O at all, except in the
UNIX System Services subsystem (originally called OpenEdition, or
Open MVS); code that must run in environments supporting stream I/O
and also in, e.g., TSO, can use conditional logic to select `EXECIO`
on the TSO side (see [PARSE SOURCE and VERSION](#parse-source-and-version)
below for how to detect which side you're on).

REXX provides no good way to detect end-of-file. You could use
`STREAM(file,"State")` and check for a value of `"NOTREADY"`, but
there is no guarantee that end-of-file is the only condition causing
`NOTREADY`.

The safest thing is to encapsulate your input/output code and then
take advantage of whatever facilities may exist in each target
system, e.g., `EXECIO` with the `STEM` option, or a third-party
library such as REXXLIB. Any such code should be thoroughly
documented. Be aware that `EXECIO` in TSO/E supports only the stem
and stack forms, not the variable-name form; even in CMS, it is
usually best to use the stem form of `EXECIO`.

**ooRexx note — `LINEOUT` opens in append mode by default; a
full-file overwrite needs an explicit replace first.** This is a real
bug pattern, not a hypothetical: a script that deletes a file and then
writes it fresh with repeated `LINEOUT` calls will silently duplicate
content the moment the delete step ever fails (a locked file, a
permission issue), because `LINEOUT` doesn't know the delete was
supposed to have happened:

```rexx
/* WRONG -- if the delete silently fails, this appends instead of
   replacing, duplicating old content underneath the new */
call SysFileDelete path
call lineout path, newContent

/* CORRECT -- explicit replace, independent of whether a prior
   delete succeeded */
call stream path, 'C', 'OPEN WRITE REPLACE'
call lineout path, newContent
call stream path, 'C', 'CLOSE'
```

ooRexx also offers stream *methods* on a `.Stream` object as an
alternative to the classic built-in functions above, preferred in new
ooRexx code:

```rexx
s = .Stream~new(path)
s~command('OPEN WRITE REPLACE')
s~lineout(newContent)
s~close()
```

<a id="parse-source-and-version"></a>
### PARSE SOURCE and VERSION

The `PARSE SOURCE` statement allows your code to determine the
operating system and file from which it was invoked, as well as the
type of invocation. You can take advantage of this in order to
maintain a single version of a REXX program for two different
systems, to detect inappropriate invocations, to select character
encoding, etc. If you have data files that, by default, should be in
the same directory as your code, you can use this statement to locate
them. See Figure 11.

The `PARSE VERSION` statement allows you to determine the language
level of REXX that your program has available. This allows you to
write code that exploits new features of REXX, yet include alternate
code that will be used when running on an older platform.

#### Figure 11: PARSE SOURCE and PARSE VERSION examples

```rexx
parse source system invocation origin

select
   when system = 'OS/2' then do
     ...
     end
   when system = 'TSO' then do
     ...
     end
   otherwise do
     say system 'is not supported by' origin
     exit
     end
   end

parse version name level date1 date2 date3 .
select
   when name = 'REXXSAA' then do
      parse var level int '.' frac
      if int > 3 then do
         /* fast code for SAA level 4 goes here */
         end
      else do
         /* slower code for older SAA level goes here */
         end
      end
   when name = 'REXX370' then do
      /* Code for CMS or TSO level of REXX goes here */
      end
   otherwise do
      say name 'is an unsupported REXX implementation'
      exit
   end
   end
```

**ooRexx note — verified against a real ooRexx 5.2.0 install, not
assumed**: the classic `REXXSAA`/`REXX370` name values above are
platform-specific classic-Rexx implementation names; ooRexx reports
neither. A real check:

```
parse version v
say v
```

on ooRexx 5.2.0 produces:

```
REXX-ooRexx_5.2.0(MT)_64-bit 6.06 18 Apr 2026
```

If your `SELECT` on `PARSE VERSION`'s `name` field needs to
distinguish ooRexx from classic implementations, match on a `name`
that begins with `REXX-ooRexx`, e.g., `when name~abbrev('REXX-ooRexx')
then do ... end` — do not assume `name` will be one of the two classic
values above.

<a id="function-library-availability"></a>
### Availability of optional function libraries

Do not assume that an optional function library your code depends on
is actually loaded in every environment it might run in. The original
form of this advice, in both source papers, was framed around a
specific and by-2023 obsolete scenario: OS/2's RexxUtil requires
Presentation Manager, which was too large to fit on a 1.44MB emergency
boot floppy, so code meant to run from one had to avoid depending on
RexxUtil and fall back to a more primitive equivalent — one route
being `BOOTOS2` to build a bootable maintenance partition or emergency
floppy still capable of loading RexxUtil, WPS, or PM sessions on a
minimum boot configuration. See Figure 12.

#### Figure 12: Function-library availability check

```rexx
if REXXUTIL_loaded then do
   stat = SysFileTree(filespec, 'filelist.', 'FSO')
   do i = 1 to filespec.0
      ...
      end
   end
else do
   'DIR' filespec '/F /O > WORK_FILE'
   ...
   end
```

The emergency-boot-floppy scenario itself is long obsolete, but the
underlying principle is not: any function library your code treats as
"just there" — RexxUtil, an ooRexx package pulled in via `::REQUIRES`,
a third-party library like REXXLIB — may genuinely not be loaded in
every environment your code might run in, and checking before you
depend on it is cheap insurance.

**ooRexx note**: the equivalent check before using a RexxUtil function
is `RxFuncQuery`, and the standard way to load the package at all
(needed at least once per process, since it's not autoloaded the way
some built-ins are) is:

```rexx
call RxFuncAdd 'SysLoadFuncs', 'RexxUtil', 'SysLoadFuncs'
call SysLoadFuncs
```

running this unconditionally at the top of a script is the practical,
idiomatic equivalent of Figure 12's availability check for the common
case where you'd rather just load the package than branch on whether
it's there — reach for the explicit `RxFuncQuery` branch only when a
genuine no-RexxUtil fallback path exists, the way the original boot-disk
scenario needed one.

<a id="variable-patterns"></a>
### Variable patterns

If you use variable patterns in the templates of your `PARSE`
statements, be aware that some extremely old implementations of REXX
do not support all forms — e.g., in MVS/XA the form `+(variable)` is
not available. If you need to run on multiple platforms, check which
forms are supported on each and program accordingly.

---

<a id="oorexx-specific-pitfalls"></a>
## ooRexx-specific pitfalls

The pitfalls in this section have no counterpart in classic Rexx —
they arise only from ooRexx's package/class model, and are worth a
dedicated section rather than a callout under an existing classic-Rexx
topic.

### `::CLASS` needs `PUBLIC` to be visible from another package

Every file ooRexx parses — the program you invoke directly, or one
pulled in via `::REQUIRES` — becomes its own `.Package` object. A
class defined with plain `::CLASS Foo` parses cleanly and is usable by
*its own package's* mainline code, but is **not** visible via the
leading-dot environment-symbol lookup (`.Foo~new`) from a *different*
package that reaches it only through `::REQUIRES`. The failure is
silent at `::REQUIRES` time — no error until the first actual
reference, and it reads like the class doesn't exist at all:

```
Error 97.1: Object ".FOO" does not understand message "NEW".
```

Fix: declare the class `PUBLIC`:

```rexx
::class Foo public     -- required for .Foo~new to work from another package
```

`PUBLIC` is also not transitive across a chain of packages. If package
A `::REQUIRES` both B and C, and B's code references a public class
from C, B must `::REQUIRES` C itself — B does not inherit visibility
of C just because A happened to require both.

### `::REQUIRES` with a relative path resolves against the current directory, not the file's own directory

`::REQUIRES '../lib/Foo.cls'` works only when the current working
directory happens to make that relative path correct, and breaks the
moment the program is invoked from anywhere else. A bare filename
with no path prefix, by contrast, is resolved via the program search
path (`PATH`), independent of the current directory — this is how
multi-file ooRexx projects that must run from any invocation location
get away with `::REQUIRES 'Foo.cls'`: they ship a `setenv` script that
adds each directory containing a required file to `PATH`, and every
`::REQUIRES` uses a bare filename, never a relative path.

### Mainline code cannot resume after a directive

A program's non-directive ("mainline") statements must form one
contiguous block at the very start of the file, before the first
`::` directive of any kind. Once a `::CLASS`, `::ROUTINE`, or
`::REQUIRES` directive has appeared, every subsequent top-level clause
must also be a directive — plain executable code cannot resume after
it, even just to call a routine defined below:

```rexx
/* WRONG -- fails with Error 99.916, "Unrecognized directive
   instruction" */
::requires 'Foo.cls'
say .Foo~new~greet

/* WRONG in a subtler way -- parses fine, but does nothing at all:
   defining ::ROUTINE main does not call it */
::requires 'Foo.cls'
::routine main
  say .Foo~new~greet

/* CORRECT -- mainline first, explicitly invoking the entry routine */
parse arg argLine
exit main(argLine)

::requires 'Foo.cls'
::routine main
  use strict arg argLine
  say .Foo~new~greet
```

### Prefer chained methods over nested function calls, including for plain strings

Every ooRexx string is a `.String` object with methods. This isn't
reserved for "real" objects like `.Array`/`.Directory` — it applies
just as much to ordinary character-string manipulation, where
classic-Rexx habit reaches for nested built-in-function calls instead:

```rexx
/* Classic-Rexx style -- reads inside-out */
result = translate(substr(str, 1, 5))
result = strip(space(translate(str)))

/* ooRexx idiom -- reads left-to-right, in actual execution order */
result = str~substr(1, 5)~translate
result = str~translate~space~strip
```

### Debugging: reach for TRACE before guessing from black-box behavior

When an ooRexx program's observed behavior doesn't match what the
source should do — especially anything involving `ADDRESS`,
string-building, or implicit operators — add `TRACE I` (or `TRACE
ALL` for more detail) near the top of the script and run it again,
rather than iterating on black-box hypotheses (rewording the command,
adding or removing quotes, trying alternate constructs) and inferring
the cause from outcomes alone. `TRACE I` prints every clause as it
executes, the intermediate result of each sub-expression, and,
critically, the exact string handed to `ADDRESS` or any other target
— which settles what string your code actually built as a fact
instead of a guess. If a second attempt at explaining unexpected
behavior from outputs alone would just be another guess, that's the
signal to add `TRACE I` instead of guessing a third time.

---

<a id="recapitulation"></a>
## Recapitulation

You can make your use of REXX more enjoyable and productive by
following a few basic rules. Learn REXX on its own terms. Be careful
and consistent in your use of abutment and continuation. Do not use
keywords or single letters as variable names. Use `SIGNAL` only for
error handling — reach for `CALL ON` instead when the guarded code
should be able to resume. Do not attempt to use the same lines as
both inline code and out-of-line code. Place a `PROCEDURE` at the
beginning of every subroutine, and carefully analyze which variables
to expose, especially if you will be passing the names of variables —
and remember that `EXPOSE` means something different again inside an
`::METHOD`, and is not legal at all inside a `::ROUTINE`. Be careful
in your use of uninitialized variables, and never name one of your
own `result`. Adopt a clear and consistent programming style. Prefer
ooRexx's real collection classes to stem-simulated arrays when the
data doesn't need positional-only indexing. Understand the vagaries
of REXX parsing. Try to make your code portable across platforms and
usable in multiple environments.

These rules will not, of course, eliminate all errors, but they will
certainly eliminate many errors that would otherwise be highly likely.
Good luck, and practice Safe REXX!

Note: the portability considerations in this edition are based on
direct experience with REXX in CMS (VM/SP), DOS (Personal REXX), MVS
(TSO/E), OS/2 (SAA REXX and OREXX/ArcaOS), and, since this merged
edition, ooRexx on Windows and Linux. Comments on portability to or
from AREXX or Regina are still welcome, as neither has been used
directly by the author.

---

<a id="references"></a>
## References

- OS/2 Procedures Language 2/REXX Reference, S10G-6268
- OS/2 Procedures Language 2/REXX User's Guide, S10G-6269
- SAA Common Programming Interface Procedures Language Reference, SC26-4358
- TSO Extensions Version 2 REXX Reference, SC28-1883
- TSO Extensions Version 2 REXX User's Guide, SC28-1882
- The REXX Language: A Practical Approach to Programming, 2nd Edition. By Michael F. Cowlishaw (Prentice-Hall, Inc., a division of Simon & Schuster), Englewood Cliffs, New Jersey 07632, ISBN 0-13-780651-5
- Open Object Rexx (ooRexx) Reference, The RexxLA/Open Object Rexx project, <https://www.oorexx.org/>
- Josep Maria Blasco's Rexx Parser (AST/element parser for Rexx, ooRexx, and Executor, written in ooRexx itself), <https://github.com/JosepMariaBlasco/rexx-parser>, also distributed as part of RexxLA's net-oo-rexx
- ANSI X3.274-1996, Information Technology — Programming Language REXX, American National Standards Institute
- Classic Rexx built-in function reference (ANSI-1996/TRL-2/z/OS/z/VM comparison chart), rexxinfo.org, <https://rexxinfo.org/reference/articles/classic_rexx_functions_w_nav_menu.html>

<a id="notes-and-trademarks"></a>
## Notes and trademarks

IBM, MVS/ESA, OS/390, z/OS, OS/2, VM/SP, VM/ESA, and z/VM are
trademarks of IBM Corporation. Unix is a trademark of The Open Group.
Open Object Rexx (ooRexx) is an open-source project distributed under
the Common Public License (CPL); it is not a trademark of IBM or any
other single organization. Slightly different versions of the two
source articles for this edition appeared in print in the 1990s; the
2023 web revisions were merged and updated with ooRexx guidance in
2026.

<a id="about-the-author"></a>
## About the author

Shmuel (Seymour J.) Metz. Mr. Metz is a Senior MVS Systems Programmer
supporting a Federal Government contract. He has worked with
computers for over half a century. He has been involved in the
development of two different operating systems. He has experience on
a wide variety of languages and platforms, and has used REXX on more
than four of them. Mr. Metz has an MA in Mathematics from the State
University of New York at Buffalo.
