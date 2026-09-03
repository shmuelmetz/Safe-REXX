# Safe REXX in the Enterprise and on the Desktop, or Will They Still Respect My Code in the Morning?

by Shmuel (Seymour J.) Metz (שמואל בן ל״ביש)

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
and dialects in current use: TSO/E REXX (the only REXX on z/OS,
running under TSO, ISPF, the OMVS shell, batch, and System REXX), CMS,
Classic REXX, Object REXX, ooRexx, Regina.

The current source of this document, including its revision history,
is maintained at <https://github.com/shmuelmetz/Safe-REXX>.

Copyright 1993, 1998, 2023, 2026 by Shmuel (Seymour J.) Metz
(שמואל בן ל״ביש). All rights
reserved. Permission for reproduction in whole or in part is hereby
granted to educational, non-profit and computer user groups for
internal, non-profit use, provided credit is given and this notice is
included. All other reproduction without the author's prior written
permission is prohibited.

---

## Table of contents

1. [Introduction](#introduction)
   - [What is REXX?](#what-is-rexx)
   - [Platforms and standards conformance](#platforms-and-standards)
   - [Summary of pitfalls](#summary-of-pitfalls)
2. [Specific examples and recommended avoidance tactics](#specific-examples)
   - [Abutment](#abutment)
   - [Continuation](#continuation)
   - [Keywords](#keywords)
   - [Labels and SIGNAL](#labels-and-signal)
   - [Parsing](#parsing)
   - [Scoping rules](#scoping-rules)
   - [Type and range checking](#type-and-range-checking)
   - [Dropped symbols used as constants](#dropped-symbols)
   - [Variable references](#variable-references)
3. [Compatibility and environmental considerations](#compatibility)
   - [ADDRESS and the default environment](#address)
   - [Environmental factors](#environmental-factors)
   - [I/O model](#io-model)
   - [PARSE SOURCE and VERSION](#parse-source-and-version)
   - [Availability of optional function libraries](#function-library-availability)
   - [Variable patterns](#variable-patterns)
4. [ooRexx-specific pitfalls](#oorexx-specific-pitfalls)
5. [Debugging: reach for TRACE before guessing from black-box behavior](#debugging)
6. [Recapitulation](#recapitulation)
7. [References](#references)
8. [Notes and trademarks](#notes-and-trademarks)
9. [About the author](#about-the-author)
10. [Colophon](#colophon)

---

## <a id="introduction"></a>Introduction

### <a id="what-is-rexx"></a>What is REXX?

REXX is a language designed by Mike Cowlishaw, initially under the
name REX, in 1979; REX saw internal use at IBM before being renamed
REXX — gaining its second X by 1982, to avoid confusion with other
products — and shipping for the first time in VM/SP Release 3, to
replace the EXEC and EXEC2 command-macro languages in the CMS
component of IBM's VM/SP. Since then it has spread to a large number
of other platforms,
including Unix, and has been designated by IBM as the SAA Procedures
Language. REXX has been used to implement a wide variety of
applications beyond its original problem domain, including many of
substantial size. **Open Object Rexx (ooRexx)** is a widely-used,
open-source, object-oriented extension of classic Rexx: it executes
unmodified classic Rexx programs and adds classes, methods, and
message-send syntax on top. Everything in this edition that applies
to classic Rexx applies unchanged to ooRexx unless a specific note
says otherwise.

### <a id="platforms-and-standards"></a>Platforms and standards conformance

The American National Standards Institute published a Rexx standard,
ANSI X3.274-1996, adding a number of enhancements beyond Cowlishaw's
original *The Rexx Language* 2nd-edition ("TRL-2") specification —
among them the `CHANGESTR`, `COUNTSTR`, and `QUALIFY` built-in
functions and the `LOSTDIGITS` condition. ooRexx and Regina both
implement these ANSI-1996 enhancements. IBM's mainframe classic-Rexx
interpreters do not: TSO/E REXX (z/OS) and CMS REXX (z/VM) are both
documented as lacking `CHANGESTR`/`COUNTSTR`/`QUALIFY` — per
rexxinfo.org's classic-Rexx function reference chart, which lists
both mainframe interpreters identically as "not supported" for these,
rather than treating z/VM as somehow upgraded past z/OS's level.
Neither mainframe interpreter has been
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

This document does not discuss NetRexx. NetRexx compiles a
Rexx-derived syntax to Java bytecode (or Java source) rather than
running under a classic-Rexx or ooRexx interpreter, and its typed
variables and Java interop give it a different pitfall profile than
the dialects covered here.

### <a id="summary-of-pitfalls"></a>Summary of pitfalls

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
concatenation, the use of dropped symbols as constants, the rules
for continuation, parsing, the block structure, and the way variable
references are passed. Later sections go into detail on each.

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
Object Rexx that does not apply to classic Rexx dialects (TSO/E
REXX — the same interpreter whether run under TSO, ISPF, the OMVS
shell, batch, or System REXX — CMS REXX, Regina, and the like). Note
that OREXX (IBM's original Object REXX — released for OS/2, Windows,
and AIX — the precursor ooRexx implements and succeeds as an
open-source project) is itself an
object-Rexx family member, not a classic-Rexx dialect — but an older,
more limited one: see the specific `~translate` vs `~upper` gap noted
below.

---

## <a id="specific-examples"></a>Specific examples and recommended avoidance tactics

Although REXX has a number of features that lend themselves to fast
prototyping, it has a few pitfalls that can beset the unwary.

### <a id="abutment"></a>Abutment

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

### <a id="continuation"></a>Continuation

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

### <a id="keywords"></a>Keywords

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
under [Dropped symbols](#dropped-symbols).

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

**ooRexx note**: `EXPOSE`, `GUARD`, `FORWARD`, and `USE` genuinely
appear as bare words inside `::METHOD`/`::ROUTINE` bodies, and `OVER`
appears as a bare word in `DO var OVER collection` anywhere in ooRexx
code, not just inside a directive body — all belong on the
avoid-as-variable-name list alongside the classic-Rexx keywords above.

### <a id="labels-and-signal"></a>Labels and SIGNAL

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

### <a id="parsing"></a>Parsing

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

In a parse template, a variable **not** enclosed in parentheses is a
*receiver* — it is assigned the next parsed token, and does **not**
match against the variable's current value. A variable enclosed in
parentheses, `(foo)`, is a *match pattern* — Rexx uses `foo`'s current
value as a literal string to scan for. Confusing the two is a silent
logic error, not a syntax error:

```rexx
parse var line word rest     /* word RECEIVES the first token */
parse var line (delim) rest  /* Rexx SCANS for delim's current value */
```

When the parse source is already a plain variable, prefer `PARSE VAR`
over `PARSE VALUE ... WITH`. The two are not semantically different
here — the reason is that every extra `WITH`/`VALUE` token in the
source is one more chance to trip over exactly the keyword-confusion
pitfalls covered above (Figures 5-7), or to introduce a stray
keyword-shaped identifier while editing later. Reserve `PARSE VALUE
... WITH` for genuine expression sources, where the `VALUE` keyword is
actually doing something:

```rexx
parse var foo template          /* foo is a plain variable */
parse value foo || bar with template   /* a genuine expression source */
```

**ooRexx note**: for pattern matching that outgrows what a `PARSE`
template can express cleanly — optional pieces, repetition, character
classes, alternatives — ooRexx's `.RegularExpression` class is an
alternative worth reaching for instead of contorting a template. It
uses its own pattern syntax (`|` for alternation, `*`/`+`/`{n}` for
repetition, `[...]` for character sets, `:alpha:`/`:digit:`-style
named classes), not POSIX or PCRE syntax. It is not preloaded; a
`::REQUIRES` is needed:

```ooRexx
str = 'name=John'
re = .RegularExpression~new('[:alpha:]+=[:alpha:]+')
say re~match(str)      -- 1: the whole string matches

::requires "rxregexp.cls"
```

`match` returns 1 or 0 for whether `string` matches; `pos` locates a
match's starting position instead of requiring the whole string to
match. Reserve it for genuine pattern matching — plain fixed-position
or delimiter-based extraction is still clearer with ordinary `PARSE`.

### <a id="scoping-rules"></a>Scoping rules

Although superficially REXX appears to be a block-structured
language, it is actually a hybrid between dynamic and static scoping.
It is possible, although bad form, to call a label inside a `DO` from
code outside the `DO`. It is possible to invoke code at an arbitrary
label as both a call and as a function invocation. It is incumbent
upon the programmer to supply the discipline that the language omits.

The scope of a procedure is determined strictly dynamically; there is
no static terminator such as `END`.

**ooRexx note**: a `::METHOD`, `::ROUTINE`, or `::CLASS` body *is*
closed by a static boundary — the next `::` directive, or end of file.
This doesn't contradict the classic-Rexx rule above (there is still no
explicit terminator statement like `END` inside the body itself), but
it does mean a directive body's extent is fixed by the file's
directive structure, not purely by dynamic control flow the way an
internal-subroutine's scope is.

**Pitfall — falling through a classic-style internal label into a
directive silently ends the whole program.** This is specifically
about a plain `CALL`ed label (not a `::ROUTINE` or `::METHOD` body,
which have their own ordinary no-`RETURN`-means-return-nothing
behavior, unaffected by any of this). When such a label's code has no
explicit `RETURN`, running out of code to execute — whether the next
thing in the file is a `::` directive, or nothing at all (true
end-of-file) — does not return control to the caller the way falling
off the end into more ordinary code would; this is easy to get wrong
by analogy with the classic-Rexx case. Both terminate the **entire
program** cleanly instead — no error condition raised, nothing
returned to the caller, the same as an implicit `EXIT`. Code after
the `CALL` simply never runs, with no diagnostic pointing at why. This
is a real risk specifically in ooRexx files that mix classic-style
internal labels with directives, since a directive now sits exactly
where "just more code" used to be assumed.

Do not write code intended to serve as both inline and out-of-line
code; programs in which you both call and fall through into the same
code are notoriously error prone. Precede each internal subprocedure
with a statement that will prevent accidentally falling into it,
e.g., `EXIT`; if your logic permits, begin the procedure with a
`PROCEDURE` statement, which must be the first statement after the
label. See Figure 9.

**This is not just a style preference — falling through into a label
that starts with `PROCEDURE` is a hard error, not silent
misbehavior.** `PROCEDURE` must be the first instruction actually
*executed* immediately after its own label is reached via `CALL` (or
a function invocation); reaching it any other way — straight-line
fall-through from the code above it — raises `Error 17: Unexpected
PROCEDURE.` as a `SYNTAX` condition, at the `PROCEDURE` line itself.
This is standard Rexx behavior, not ooRexx-specific. It's the
mechanism *behind* the "notoriously error prone" warning above: an
unguarded fall-through into a `PROCEDURE`-led subprocedure doesn't
just risk exposing variables unexpectedly — it crashes outright the
moment it happens, which is at least easier to notice than either of
the other two silent failure modes above (a directive or end-of-file
boundary ending the program, or plain code silently running with the
wrong variable scope).

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
first statement of a `::METHOD` body exposes that object's *instance*
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

### <a id="type-and-range-checking"></a>Type and range checking

Unlike most other languages, REXX has neither variable typing nor
arrays. Arrays are often simulated using compound variables. This
leads to several possible types of undetected errors.

**ooRexx note**: ooRexx's plain variables are exactly as untyped as
classic Rexx's — this doesn't change what's above. Its *objects* are
a different matter: ooRexx has dynamic typing at the object level.
Sending an object a message it doesn't recognize is a real, enforced
error — the `Error 97.1`/"does not understand message" pattern seen
throughout this document — not silent misbehavior. It's late-bound
(checked when the message is sent, not before the program runs)
rather than static, but it is genuine type enforcement, absent a
couple of low-level but documented escape hatches. ("Recognizes,"
not "the class defines," is deliberate above — see the second point
below.) A class can define an `UNKNOWN` method to deliberately accept
and handle any message that would otherwise be rejected, receiving
the message name and its argument list. Separately, an individual
*object's* own recognized-message set isn't fixed by its class alone:
a method can be attached to one specific instance at run time (via
`~setMethod`, callable only from that object's own code, or
`Class~enhanced` at creation time), so two objects of the identical
class can end up recognizing different messages. Both are opt-in
mechanisms a program has to invoke deliberately, not a gap in the
checking.

When you assign a value to a variable, there is no check that the
value is consistent with the intended type. If your logic requires
any constraints on the values that can be assigned, it is your
responsibility to code explicit checks using, e.g., the `DATATYPE`
function.

When you use a variable name as part of a compound variable in order
to simulate an access to an array element, REXX does not check that
the index is within the array extents, or even that it is an integer.
If your logic requires enforcing such constraints, you must code them
explicitly. Note that even a dropped symbol can be used as an "index"
for a compound variable.

**ooRexx note**: ooRexx provides real collection classes — `.Array`,
`.Directory`, `.Table`, `.Set`, `.Bag`, `.Queue`, `.OrderedCollection`,
and others — as a genuine alternative to stem-simulated arrays, with
actual bounds/type behavior rather than the silent-anything-goes
behavior of a compound variable:

```ooRexx
arr = .Array~of('a', 'b', 'c')      -- construct-and-populate in one call
do item over arr
    say item
end
```

Prefer `do item over collection` to `do i = 1 to stem.0` when the
data doesn't need positional indexing at all.

**Indirect/computed stem access has more than one form, and reaching
for the wrong one doesn't always error.** Standard classic Rexx
already handles the common case cleanly: a tail that is a single bare
symbol substitutes that symbol's *current value* directly, with no
bracket of any kind:

```rexx
mystem.1 = 'one'; mystem.2 = 'two'; mystem.3 = 'three'
i = 3
say mystem.i           /* CORRECT: 'three' -- bare-symbol substitution,
                           classic Rexx, no bracket needed at all */
```

One form that looks plausible by analogy is a genuine pitfall in any
Rexx dialect:

```rexx
say mystem.(i)         /* WRONG in any Rexx dialect: Error 43 --
                           parsed as a call to a routine literally
                           named MYSTEM */
```

**ooRexx note**: two more forms that look plausible by analogy —
`mystem[i]` and `mystem.[i]` — are valid *only* in ooRexx; classic
Rexx's own lexer has no defined meaning for `[`/`]` at all, so a
classic-Rexx program can't reach for either by mistake in the first
place. In ooRexx, both parse and run: `[]` is genuinely one uniform
mechanism — bracket notation sends a message named `[]` to the
receiver, with whatever's inside the brackets passed as its argument
list — but what that list *means* is entirely up to the receiving
object's own `[]` method, and the two below interpret it very
differently:

- On a `.String`, `[]` is character/substring extraction (ooRexx
  Language Reference §5.1.7.22): `"abc"[2]` is `"b"`; with a second,
  comma-separated argument, `"abc"[2,4]` is a substring, `"bc"`.
- On a `.Stem`, `[]` is documented separately, under "Evaluated
  Compound Variables" (§1.13.5.1), as an alternate way to *construct a
  compound-variable tail*: each comma-separated expression is
  evaluated to a string, and the results are joined with periods to
  form the tail — `a.[1+2, 3+4]` assigns `a.3.7`, exactly as if you
  had written that dotted tail yourself. It is not positional
  "element N" indexing the way `.Array`'s `[]` is.

```ooRexx
say mystem[i]           /* NOT AN ERROR -- and that's the trap: this is
                            a perfectly legitimate character selection,
                            just not the one intended. 'mystem' with no
                            trailing dot is a plain simple variable; it
                            was never assigned, so it's a dropped
                            symbol and evaluates to its own name, the
                            string "MYSTEM"; [] on a String correctly
                            selects a character -- "S" (character 3 of
                            "MYSTEM"). Nothing here is wrong except the
                            programmer's expectation that this reaches
                            the stem element instead */

say mystem.[i]          /* CORRECT: 'three'. 'mystem.' -- the stem
                            itself, trailing dot, no tail -- is always
                            already bound to a genuine Stem object
                            (ooRexx Language Reference §1.13.4); its []
                            method takes i, evaluates it, and uses the
                            result directly as the tail -- the single-
                            expression case of the same tail-building
                            mechanism as the two-expression example
                            above */
```

A third, unrelated trap: using another compound variable directly as
a tail component looks like it should nest, but the tail is split on
periods into independent pieces *before* any substitution happens — a
piece is never itself re-parsed as a compound-variable reference. This
is standard Rexx behavior in both dialects, no brackets involved:

```rexx
orphans.0 = 0
orphans.0 = orphans.0 + 1
orphans.orphans.0 = 'first'      /* WRONG: not "orphans.1" -- every
                                     iteration clobbers the SAME fixed
                                     derived tail ORPHANS.ORPHANS.0 */
```

The safe pattern here too: copy the index into a plain simple
variable first, then use that variable as the tail (`n = orphans.0;
orphans.n = value`).

> **ooRexx note**: a stem's own item count sidesteps maintaining a
> manual counter tail altogether — `orphans.` is always a genuine Stem
> object (as established above), and `~items` is a read-only query
> reporting how many of its compound variables are currently set.
> `~items` itself does not add, remove, or change anything — it's the
> tail *assignment* that changes the count; `~items` only reports
> whatever that count happens to be at the moment you call it, with
> nothing for the program to track by hand:
>
> ```ooRexx
> orphans.[orphans.~items] = 'first'   -- items was 0; sets tail "0"
> orphans.[orphans.~items] = 'second'  -- items is now 1; sets tail "1"
> orphans.[orphans.~items] = 'third'   -- items is now 2; sets tail "2"
> ```
>
> The stem starts out empty, with `~items` equal to `0` — so the very
> first element written this way lands in tail `"0"`, not tail `"1"`:
> the bracket expression evaluates `~items` first, *then* the
> assignment runs and is what makes that tail exist, bumping the count
> for next time. This differs from the classic convention, where tail
> `0` is reserved for a manually-maintained counter and data start at
> `1`. The two schemes are not interchangeable; pick one and stay
> consistent within a given stem. Add `+1` to get the classic 1-based
> numbering instead:
>
> ```ooRexx
> orphans.[orphans.~items+1] = 'first'   -- items was 0; sets tail "1"
> orphans.[orphans.~items+1] = 'second'  -- items is now 1; sets tail "2"
> orphans.[orphans.~items+1] = 'third'   -- items is now 2; sets tail "3"
> ```
>
> This still relies on the same discipline as the 0-based form: every
> tail has to come from this exact idiom, with nothing added or
> removed out of band, or the numbering silently stops meaning what
> you think it means.
>
> A Stem is an associative array — a string-indexed map — which is not
> the same thing as an array, even when every tail happens to be a
> contiguous integer: that's a simulated, conventional usage built on
> top of a fundamentally different structure, the way a flight
> simulator imitates flying without being an airplane. Nothing stops a
> program from setting `orphans.foo` or `orphans.17` directly alongside
> the sequence above. Don't use `~items` as a loop bound
> (`do i = 1 to orphans.~items`) except in the narrow case where every
> tail was built by exactly this idiom and none was added or removed
> out of band. The general, safe way to visit every populated tail is
> `do tail over orphans.~allIndexes` (the tail names) or `do value over
> orphans.~allItems` (the values directly, when the tail names
> themselves don't matter):
>
> ```ooRexx
> do tail over orphans.~allIndexes
>     say tail':' orphans.[tail]
> end
> ```
>
> If what's actually wanted is array-style append — add an element and
> let the collection work out the next position itself — a real
> `.Array` and its own `~append` method do that directly, with none of
> the discipline the `~items`/`~items+1` idioms above depend on.
> `orphans.[orphans.~items] = value` only imitates the effect, for a
> Stem built consistently one way or the other; it is not itself an
> append operation. An `.Array` also comes with `~first`/`~last` (the
> index of the first/last item, or `.nil` if empty) and
> `~firstItem`/`~lastItem` (the item itself) — reading the ends of the
> collection this way needs no bracket arithmetic and no assumption
> about how it was populated, unlike reaching for tail `"0"` or
> `orphans.~items - 1` on a Stem. Unless the data genuinely needs a
> Stem's string-keyed lookup, prefer the array.

**ooRexx note**: `.stem~new` creates a fresh, otherwise-anonymous Stem
object, and it is a **genuinely different object** from the Stem
object automatically bound to a compound-variable stem of the same
name — assigning the new object to a variable doesn't connect the two,
even though both are ordinary Stem objects and both support the same
bracket notation. `realStem` (no trailing dot) and `realStem.`
(trailing dot, no tail) have always been different variables, even in
classic Rexx with no ooRexx involved at all; what's new here is only
that `realStem.` is bound to a genuine Stem *object*, not just a plain
default value:

```ooRexx
realStem = .stem~new
realStem[9] = 'nine'
say realStem.9          /* still dropped -- "REALSTEM.9" -- the
                            compound-variable route uses its own,
                            separate Stem object, sharing nothing with
                            the one 'realStem' happens to hold */
say (realStem. == realStem)   /* 0 -- confirms they're genuinely
                                  different objects, not aliases */
```

### <a id="dropped-symbols"></a>Dropped symbols used as constants

ANSI X3.274-1996 distinguishes a *symbol* (§3.1.47: "a sequence of
characters used as a name... [symbols] are used to name variables,
functions, etc.") from the *variable* it may or may not currently
name. A symbol that has never had a value assigned to it — what's
often called "uninitialized" informally — has no variable behind it
yet; the standard's own term for this state is *dropped*
(§3.1.16: "a symbol which is in an uninitialized state, as opposed to
having had a value assigned to it, is described as dropped"). When you
refer to a dropped symbol, its value is by default its own name in
upper case. This is frequently a convenient alternative to the use of
literal strings. However, if you inadvertently assign a value to that
same name elsewhere in the program — turning it into a real variable —
you may get incorrect and apparently inexplicable results from code
that still expected it dropped. It is best to adopt naming conventions
that minimize the risk of such problems.

Some recommend always using explicit literal strings for constants.
Although well meant, this advice can lead to programs that are harder
to read. Use dropped symbols as constants, but judiciously. If you
choose to not exploit this default behavior, place a `SIGNAL ON
NOVALUE` at the beginning of your program to detect any reference to a
symbol that's still dropped when your logic expected a real variable.

**ooRexx note**: never name your own variable `result`. This is the
same "a dropped symbol reverts to its own name" behavior above, but
with a genuinely surprising trigger in ooRexx: `result` isn't only set
by `CALL`. *Any* bare message-send statement — a whole clause, its
return value not assigned to anything — is handled the same way as a
`CALL` to a routine with no `RETURN` value: if the invoked method
returns nothing at all (not even `.nil` — several Collection methods,
e.g. `~put`, are defined to return no result object), `result` is
dropped again — the variable you just assigned reverts to being a bare
symbol. This breaks the moment any bare message-send whose method
returns nothing executes — including the ordinary case of building up
your *own* local variable named `result` via repeated bare sends to
it:

```ooRexx
result = .Directory~new        -- fine: plain assignment, RESULT is now
                                   a real variable
result~put('', 'INTERPRETER')  -- runs; but ~put returns no result
                                   object, so RESULT is dropped again
                                   right after this line
result~put('', 'DIALECT')      -- Error 97.1: Object "RESULT" does not
                                   understand message "PUT" -- the
                                   PREVIOUS line already dropped it,
                                   so RESULT is just the string "RESULT"
                                   again by the time this line runs
```

`putReturn = d~put('v','k')` raises "Message did not return a
result," which is the proof that `~put` truly returns nothing; a
single bare `result~put(...)` statement reproducibly drops `result`
immediately afterward, while an identically-shaped sequence using any
other variable name is never affected. Pick any other name (`info`,
`found`, `outcome`, ...) — there is no scope in which reusing `result`
as your own variable buys anything.

### <a id="variable-references"></a>Variable references

Classic Rexx does not allow passing parameters to procedures by name
or by reference. However, you can often get similar results by
passing constants and using them to construct names. This is an
extremely common and powerful technique, especially in conjunction
with compound variables. However, there are a few pitfalls.

**When all you need is to read or set a variable by name, prefer
`VALUE()` to `INTERPRET`.** `VALUE(name)` reads the variable named
`name`; `VALUE(name, newvalue)` sets it and returns the *old* value.
`VALUE` also takes an optional third argument, `selector`, naming a
variable pool other than the program's own — `VALUE(name, newvalue,
'ENVIRONMENT')` reads or sets an environment variable instead of a
Rexx variable, the mechanism behind the `OS2ENVIRONMENT` examples in
Figure 2 above — something `INTERPRET` has no direct equivalent for at
all. The third argument is not universal, though: REXX/VM under GCS
(see [ADDRESS and the default environment](#address) below) does not
support the `selector` argument at all, only the two-argument form.
Building a string and running it through `INTERPRET` can achieve the
same thing, but `INTERPRET` executes whatever Rexx source text it is
handed — not just an assignment to one variable — which matters the
moment `name` is not a value you fully control. Hand the same crafted
string (a syntactically invalid variable name with a second clause
hidden after a semicolon) to each, and the difference shows up
immediately: `VALUE()` simply raises a `SYNTAX` condition on the
malformed name and executes nothing else, while `INTERPRET` begins
executing the hidden second clause (visibly compiling and invoking the
routine it named) before failing only because that call was missing a
required argument — not because the injection itself was ever blocked.
Reserve `INTERPRET` for
genuinely dynamic code — constructing a whole statement or expression
at run time — not as a heavier substitute for a single indirect
variable reference.

If you call a procedure that has an `EXPOSE` clause on the
`PROCEDURE` statement, it will only have access to the variables that
you exposed. If you pass an argument containing the name of some
other variable, the code will only be able to access a local version
of that variable.

If you call a procedure that requires a variable name as a parameter,
and use a dropped symbol to represent its own name for that parameter,
you will probably get incorrect results on your second time through.

**ooRexx note**: ooRexx closes the by-reference gap classic Rexx
disclaims above, with a real `USE ARG` statement:

```ooRexx
::routine adjustBalance
  use arg account      -- account is a genuine mutable reference to
                           the caller's object, not a copy
  account~balance = account~balance - fee
```

`USE ARG` is for mutable *objects* passed by reference — it does not
turn a plain string or number into a by-reference parameter the way
some other languages' reference parameters do, since Rexx strings and
numbers are themselves immutable values.

`rc` and `result` are set by different things, and conflating them is
a real, easy-to-make bug — in classic Rexx as much as ooRexx. `rc` is
set **only** by host commands — a bare host-command clause, `ADDRESS
foo 'expr'`, or similar. It is **not** set by `CALL` or a
function/method invocation.
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

## <a id="compatibility"></a>Compatibility and environmental considerations

REXX has some specific features that you can exploit to make your
programs more compatible across platforms, or between environments on
the same platform. REXX also has some features that hinder
compatibility.

### <a id="address"></a>ADDRESS and the default environment

If you write a command file that issues host commands — OS/2, CMS,
TSO, DOS commands — do not assume that the default environment is
that of the host itself. By including, e.g., `ADDRESS TSO` (on a
mainframe) or `ADDRESS CMD` (on OS/2/Windows), you enable the routine
for use from within other environments too, e.g., the ISPF/PDF editor
on a mainframe, or an editor that uses REXX as its macro language on
a PC.

A platform name alone is too coarse here — the *invocation context*,
not just the OS, decides the default. Only environments actually
documented for that context are listed; a blank cell means none:

| Invocation context | Default environment | Other environments |
|---|---|---|
| OS/2 command prompt (classic REXX) | `CMD` | |
| PC-DOS command prompt (classic REXX) | `COMMAND` | |
| Command prompt (OREXX, ooRexx) | `CMD` | `SYSTEM`, `PATH` on ooRexx |
| Regina command prompt | `SYSTEM` | `COMMAND`, `REXX` |
| TSO/E READY | `TSO` | `MVS`, `CONSOLE`†, the link/attach family, the APPC family |
| ISPF on z/OS | `TSO` | `MVS`, `CONSOLE`†, the link/attach family, the APPC family, `ISPEXEC` |
| ISPF/PDF EDIT on z/OS | `TSO` | `MVS`, `CONSOLE`†, the link/attach family, the APPC family, `ISPEXEC`, `ISREDIT` |
| ISPF on z/VM | `CMS` | `ISPEXEC` |
| ISPF/PDF EDIT on z/VM | `CMS` | `ISPEXEC`, `ISREDIT` |
| OMVS shell | `SH` | `TSO`, `MVS`, `SYSCALL` |
| `IRXJCL` | `MVS` | the link/attach family, the APPC family |
| System REXX | `MVS` (`TSO=NO`) | the link/attach family, `APPCMVS`, `BCPii`, the APPC family; `TSO=YES` adds `TSO`, `ISPEXEC`, `ISREDIT` |
| EDIT macro | `EDIT` | none — `TSO` itself is unavailable until `END` terminates `EDIT` |
| TEST macro | `TEST` | none — `TSO` itself is unavailable until `END` or `RUN` terminates `TEST` |
| IPCS macro | `TSO` | `IPCS` — not available at all in the session's own separate TSO/E mode |
| CMS command line | `CMS` | `COMMAND`, `CP` |
| GCS | `GCS` | `COMMAND` |
| XEDIT macro | `XEDIT` | falls through to `CMS`, then `CP`, automatically |

The link/attach family: `LINK`, `LINKMVS`, `LINKPGM` (link to an
unauthorized program on the same task level), `ATTACH`, `ATTCHMVS`,
`ATTCHPGM` (attach one on a different task level) — available to a
REXX exec in *any* address space, TSO/E or not. The APPC family:
`CPICOMM` (SAA CPI Communications calls), `LU62` (APPC/MVS calls,
SNA LU 6.2) — likewise available in any MVS address space. † `CONSOLE`
needs an active extended MCS console session (started with the TSO/E
`CONSOLE` command) and console command authority; it's available only
in the TSO/E address space, not from batch. `ISREDIT` requires an
active edit session regardless of platform — attempting it outside one
fails at run time even where the environment is nominally available.

Standard Rexx (not an ooRexx-only extension) can capture a child
process's stdout and stderr directly into stems, with no temp files or
pipes needed:

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

TSO/E REXX has no `ADDRESS WITH` at all to do this with — like classic
OS/2 REXX and OREXX, it predates the ANSI-1996 enhancement. Its own
mechanism for capturing host-command output is the `OUTTRAP` built-in
function instead, which traps subsequent command output into a stem
(or the program stack) until turned back off:

```rexx
call outtrap 'mystem.'     /* start trapping into mystem. */
'LISTC LEVEL(MY.DATA)'
call outtrap 'off'         /* stop trapping */
do i = 1 to mystem.0
    say mystem.i
end
```

See [Continuation](#continuation) above (Figures 2 and 3) for the
continuation pitfalls specific to `OUTTRAP`'s own argument list.

Valid I/O redirect types in the `WITH` clause are `NORMAL`, `STEM`,
`STREAM`, and `USING` — `STRING` is not a valid type. Of these, `USING`
— supplying the input value directly, `input using (expr)`, with no
stem or stream needed — goes beyond ANSI X3.274-1996's own `ADDRESS
WITH` semantics, which define only `STREAM` and `STEM` as resource
types; `NORMAL`/`STEM`/`STREAM` are standard, but `USING` is an ooRexx
extension (ooRexx 5.2.0). Regina does not have it: its own reference
manual's `ADDRESS WITH` syntax diagram lists only `STREAM`, `STEM`,
`LIFO`, and `FIFO` as resource types — `LIFO` and `FIFO` are Regina's
own extensions beyond the ANSI baseline, but `USING` appears nowhere
in the manual. Neither classic OS/2 REXX nor OREXX has it either — per
IBM's own OS/2 reference documentation for both (Procedures Language
2/REXX Reference and Object REXX Reference), the `ADDRESS` instruction's
own syntax diagram in *both* is just `ADDRESS [environment]
[expression]`, no `WITH` clause of any kind. Neither ever had *any*
form of `ADDRESS WITH`, `USING` included; the whole I/O redirection
clause is an enhancement neither IBM product picked up.
To supply empty stdin (preventing a child process from blocking
waiting for input), define an empty stem and pass it as `INPUT STEM`:

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

### <a id="environmental-factors"></a>Environmental factors

REXX does not shield you from the underlying environment; in writing
a REXX program you must understand the behavior of your operating
system and user interface if you want to avoid nasty surprises. As an
example, if you invoke a REXX program in an OS/2 CMD file and scan
the argument looking for the string `/Q`, you will not find it,
because `CMD.EXE` will have taken the string `/Q` to be a "quiet"
option and removed it.

If you must use binary or hexadecimal constants for character data,
be aware that character encoding varies among systems, and not just
between EBCDIC and ASCII. CMS and TSO use EBCDIC. Most other systems
use some combination of plain 7-bit ASCII, an 8-bit code page
extending ASCII (e.g. Latin-1, Windows-1252 — the specific extension
matters, since they disagree above code point 127), and Unicode
(typically UTF-8 or UTF-16) — which one depends on the specific
system, its locale/code-page configuration, and the file or stream in
question, not just the OS family. Even within CMS and TSO there are
national-language issues, and in many systems there are code-page
issues. Be aware of the character sets used in each of your target
systems, and program accordingly. Segregate system-dependent values
and code-page-dependent values to make your code easier to maintain.

Rexx variable names and labels are case-insensitive on every
platform, in every dialect — but two related things are not, and the
platform matters:

- Filenames on Windows (NTFS) or ArcaOS/OS2 (JFS) may be
  case-insensitive in practice, but should still be treated as
  case-sensitive in code that must also run on Linux.
- The `VALUE()` built-in for reading environment variables is
  case-sensitive on Linux but case-insensitive on Windows.

**ooRexx note**: for case-insensitive comparisons generally, not just
the platform-specific cases above, ooRexx's `.String` class provides a
`caseless`-prefixed method family directly — `caselessEquals`,
`caselessCompare`, `caselessPos`, `caselessCountStr`,
`caselessChangeStr`, `caselessAbbrev`, `caselessMatch`,
`caselessStartsWith`/`caselessEndsWith`, `caselessWordPos`,
`caselessContains`/`caselessContainsWord` — rather than calling
`TRANSLATE()`/`~upper` on both sides before every comparison. `PARSE`
itself has a `CASELESS` modifier too (alongside `LOWER`) for
case-independent template matching — but unlike `PARSE UPPER`, which
is genuine ANSI X3.274-1996 syntax (spelled out in full; the standard
documents no abbreviated form for it), neither `LOWER` nor `CASELESS`
appears anywhere in the ANSI text: both are extensions, present in
ooRexx and Regina alike but absent from TRL-2-level classic Rexx and
from the ANSI standard itself.

### <a id="io-model"></a>I/O model

The original implementation of REX on CMS used the `EXECIO` command,
later inherited by TSO/E and other platforms; through at least VM/SP
Release 5 (1986), it was CMS's only file I/O mechanism, alongside the
program stack. CMS's `EXECIO` reads and writes through three kinds of
destination — the program stack (`FIFO`/`LIFO`), a stem (`STEM
stem.`), or a single plain variable (`VAR name`, but only for exactly
one line at a time; the count operand must be `1` with `VAR`). Of
these, TSO/E REXX in MVS inherited only a subset: the stack and `STEM`
forms, not `VAR`.

Stream I/O (`LINEIN`, `LINEOUT`, `LINES`, `CHARS`) came later — absent
from VM/SP's own Interpreter Reference through Release 5 (1986). It's
part of the language as defined in Cowlishaw's TRL-2 (1990) and later
formalized by ANSI X3.274-1996 — both are language specifications, not
descriptions of any particular product's implementation, so neither
pins down when a given interpreter actually added it. What isn't
guaranteed, once stream I/O is
available at all, is that `CHARS()` or `LINES()` return an exact
count: ANSI Rexx explicitly permits either one to report only whether
at least one more character or line is available (`0` or `1`) instead
of a real count. Which one actually returns an exact count, for which
kind of stream, is a per-implementation, per-function choice, not a
clean split by platform: on CMS, `LINES()` returns an exact count for
disk files but `CHARS()` never does, even for files; on ooRexx,
`CHARS()` returns an exact count for disk files but its own `LINES()`
returns only `0` or `1`. TSO/E REXX in MVS does not support stream I/O
at all outside the UNIX System Services (OMVS) subsystem, where full
stream I/O is available. OS/2's classic Rexx has stream I/O functions
too, but both `CHARS()` and `LINES()` return only `0` or `1` there,
for every kind of stream. See Figure 10. Code that relies on an exact
count from either function should be checked against that specific
target's own documented behavior — never assumed portable, and an
implementation that *did* return an exact count where none was
expected could be horribly inefficient. Some interpreters still
support `EXECIO` for compatibility with legacy TSO/CMS code, but it is
not the primary I/O model outside TSO/E and CMS themselves.

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

A `CHARS()`/`LINES()` that only ever returns `0` or `1` is not the
same thing as having no way to detect end-of-file: `LINES(file) = 0`
(or `CHARS(file) = 0` for
character-mode reads) is a reliable, portable end-of-file test, and
the OS/2 SAA REXX idiom in Figure 10 above already relies on it. What
is *not* reliable, in any dialect, is using `STREAM(file,"State")` and
checking for `"NOTREADY"` — that state can also result from an I/O
error or other condition, not just end-of-file, and you only see it
after already reading past the end. ooRexx exposes the same `LINES`/
`CHARS` test as `.Stream` methods (`aStream~lines`, `aStream~chars`)
for code written in OO style, with no change in the underlying
end-of-file semantics.

The safest thing is to encapsulate your input/output code and then
take advantage of whatever facilities may exist in each target
system, e.g., `EXECIO` with the `STEM` option, or a third-party
library such as REXXLIB. Any such code should be thoroughly
documented. Be aware that `EXECIO` in TSO/E supports only the stem
and stack forms, not the variable-name form; even in CMS, it is
usually best to use the stem form of `EXECIO`.

**`LINEOUT` opens in append mode by default; a full-file overwrite
needs an explicit replace first.** This is standard Rexx behavior, not
an ooRexx quirk, and a real bug pattern, not a hypothetical: a script
that deletes a file and then writes it fresh with repeated `LINEOUT`
calls will silently duplicate
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

```ooRexx
s = .Stream~new(path)
s~command('OPEN WRITE REPLACE')
s~lineout(newContent)
s~close()
```

### <a id="parse-source-and-version"></a>PARSE SOURCE and VERSION

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

**ooRexx note**: the classic `REXXSAA`/`REXX370` name values above are
platform-specific classic-Rexx implementation names; ooRexx reports
neither.

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
values above. The two classic values themselves are also not the
whole story — the full set of `name`/`level` values you may actually
meet:

| Implementation | `name` | `level` | Source |
|---|---|---|---|
| OS/2 classic REXX (Procedures Language 2/REXX) | `REXXSAA` | `4.00` | IBM's own reference manual |
| OREXX (IBM's Object REXX) | `OBJREXX` | `6.00` | IBM's own reference manual (OS/2 edition) |
| CMS / TSO/E REXX (classic mainframe, "REXX370") | `REXX370` | `4.00` | Widely documented |
| Regina | `REXX-Regina_<version>` (e.g. `REXX-Regina_3.9.6(MT)`) | `5.00` | Regina's own reference manual; ANSI-compliant since Regina 3.1 |
| ooRexx | `REXX-ooRexx_<version>(MT)_<bits>-bit` (e.g. `REXX-ooRexx_5.2.0(MT)_64-bit`) | `6.06` | ooRexx 5.2.0 |

Two things worth noticing in this table. First, `level` is *not* the
interpreter's own version number — it is the Rexx *language level* the
interpreter targets (`4.00` is TRL-2, `5.00` is ANSI X3.274-1996), and
several implementations have historically conflated the two; look for
the interpreter's own version inside the `name` word instead (as
ooRexx and Regina both do) or in `PARSE SOURCE`. Second, `REXXSAA` and
`REXX370` share the exact same `level`, `4.00` — despite one being a
PC/workstation implementation and the other a mainframe one — because
neither was ever brought up to the ANSI-1996 level; see
[Platforms and standards conformance](#platforms-and-standards) above.

### <a id="function-library-availability"></a>Availability of optional function libraries

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

The modern equivalent check before using a RexxUtil function —
standard across classic Rexx and ooRexx alike, not an ooRexx-only
mechanism — is `RxFuncQuery`, and the standard way to load the package
at all (needed at least once per process, since it's not autoloaded
the way some built-ins are) is:

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

**Loading the package is not the same as every function in it being
present.** The repertoire behind the name `RexxUtil` is not itself
standardized, and varies by implementation:

| Function(s) | OREXX | ooRexx 5.2.0 | Regina (`RegUtil`) |
|---|---|---|---|
| `SysFileCopy`, `SysFileMove` | No (Windows edition) | Yes | No — `SysCopyObject`/`SysMoveObject` instead |
| The `SysIsFileXxx` family (`SysIsFile`, `SysIsFileDirectory`, `SysIsFileLink`, and the Windows-only detail variants) | No | Yes | No |
| The Workplace-Shell family (`SysCreateObject`, `SysDestroyObject`, `SysSetObjectData`, `SysQueryClassList`, and related) | Yes, but only in the OS/2 edition | No | No |
| The semaphore family (`SysCreateEventSem`, `SysCreateMutexSem`, and related) | Yes | Yes, but deprecated in favor of the `.EventSemaphore`/`.MutexSemaphore` classes | Yes |
| Unix process functions (`SysFork`, `SysWait`, `SysCreatePipe`) and `SysGetMessage`/`SysGetMessageX` (Unix message catalogs) | Yes, in the AIX edition | Yes, on Unix-like platforms | No |
| `SysWinGetPrinters`, `SysWinGetDefaultPrinter`, `SysWinSetDefaultPrinter`, `SysFormatMessage`, `SysGetLongPathName`, `SysGetShortPathName`, `SysShutdownSystem` | No | Yes | No |
| `SysLoadFuncs`/`SysDropFuncs` | Required, to register the package | Deprecated no-ops since ooRexx 4.0.0 — the package is auto-registered | Required, to register the package |

Ordinary file/directory operations (`SysFileTree`, `SysMkDir`,
`SysRmDir`, `SysSearchPath`, `SysTempFileName`, `SysGetFileDateTime`,
`SysSetFileDateTime`, `SysDriveInfo`, `SysDriveMap`, `SysVolumeLabel`,
`SysWaitNamedPipe`), the macro-space family, console I/O (`SysCls`,
`SysGetKey`, `RxMessageBox`, and related — Windows-only in all three),
and `SysQueryProcess` are present in all three; `SysFileTree` and
`SysQueryProcess` are each documented as behaving differently across
platforms, so test them on each target rather than assuming identical
semantics.

A blanket "is RexxUtil loaded?" check, whether via Figure 12's flag or
`RxFuncQuery('SysLoadFuncs')`, only tells you the package itself
loaded — it says nothing about whether the *specific* function you're
about to call is part of that implementation's repertoire. Guard any
WPS-specific (or otherwise platform-specific) call with its own
`RxFuncQuery` on that function's own name, not just on the package.

### <a id="variable-patterns"></a>Variable patterns

If you use variable patterns in the templates of your `PARSE`
statements, be aware that some extremely old implementations of REXX
do not support all forms — e.g., in MVS/XA the form `+(variable)` is
not available. If you need to run on multiple platforms, check which
forms are supported on each and program accordingly.

---

## <a id="oorexx-specific-pitfalls"></a>ooRexx-specific pitfalls

The pitfalls in this section have no counterpart in classic Rexx —
they arise only from ooRexx's package/class/object model, and are
worth a dedicated section rather than a callout under an existing
classic-Rexx topic.

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

```ooRexx
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

```ooRexx
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

```ooRexx
/* Classic-Rexx style -- reads inside-out */
text = translate(substr(str, 1, 5))
text = strip(space(translate(str)))

/* ooRexx idiom -- reads left-to-right, in actual execution order
   (never name a variable `result` -- see the ooRexx note above) */
text = str~substr(1, 5)~translate
text = str~translate~space~strip
```

---

## <a id="debugging"></a>Debugging: reach for TRACE before guessing from black-box behavior

`TRACE` is standard Rexx, not an ooRexx feature, and the advice below
applies to any Rexx dialect. When a program's observed behavior
doesn't match what the source should do — especially anything
involving `ADDRESS`, string-building, or implicit operators — add
`TRACE I` (or `TRACE ALL` for more detail) near the top of the script
and run it again, rather than iterating on black-box hypotheses
(rewording the command, adding or removing quotes, trying alternate
constructs) and inferring the cause from outcomes alone. `TRACE I`
prints every clause as it executes, the intermediate result of each
sub-expression, and, critically, the exact string handed to `ADDRESS`
or any other target — which settles what string your code actually
built as a fact instead of a guess. If a second attempt at explaining
unexpected behavior from outputs alone would just be another guess,
that's the signal to add `TRACE I` instead of guessing a third time.

---

## <a id="recapitulation"></a>Recapitulation

You can make your use of REXX more enjoyable and productive by
following a few basic rules. Learn REXX on its own terms. Be careful
and consistent in your use of abutment and continuation. Do not use
keywords or single letters as variable names. Use `SIGNAL` only for
error handling — reach for `CALL ON` instead when the guarded code
should be able to resume. Do not attempt to use the same lines as
both inline code and out-of-line code. Place a `PROCEDURE` at the
beginning of every subroutine, and carefully analyze which variables
to expose, especially if you will be passing the names of variables —
and remember that `EXPOSE` means something different again inside a
`::METHOD`, and is not legal at all inside a `::ROUTINE`. Be careful
in your use of dropped symbols, and never name one of your own
`result`. Adopt a clear and consistent programming style. Prefer
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
directly by me.

---

## <a id="references"></a>References

- OS/2 Procedures Language 2/REXX Reference, S10G-6268
- OS/2 Procedures Language 2/REXX User's Guide, S10G-6269
- PC DOS 7 REXX User's Guide and Reference, IBM Corp., S83G-9228 (IBM's own REXX bundled with PC DOS 7, distinct from the third-party Personal REXX by Quercus Systems)
- SAA Common Programming Interface Procedures Language Reference, SC26-4358
- Object REXX Reference, IBM Corp. (the OS/2 edition of the manual for OREXX, IBM's cross-platform Object REXX for OS/2, Windows, and AIX, and the precursor to ooRexx); also consulted directly: Object REXX for Windows Reference, Version 2.1, SH12-6725-00, and Object REXX for AIX Reference, Version 1.1.3, SH12-6386-01 — the RexxUtil repertoire differs by edition, notably the Workplace-Shell-specific functions (OS/2 only) and the Unix process functions (AIX only)
- Regina REXX RegUtil Reference (the RexxUtil-equivalent package bundled with Regina), <https://regina-rexx.sourceforge.io/>
- TSO Extensions Version 2 REXX Reference / z/OS TSO/E REXX Reference, SC28-1883 / SA32-0972 (three editions consulted: SC28-1883-0, December 1988; SC28-1883-4, August 1991; and the current z/OS 2.5 edition, SA32-0972-50, 2021 — the earliest documents a noticeably smaller host command environment table than the other two, which agree word for word)
- TSO Extensions Version 2 REXX User's Guide, SC28-1882
- TSO/E Command Reference, IBM Corp., SC28-1969 (documents `EDIT` and `TEST` as TSO commands, including the specific `EXEC` subcommand behavior each imposes on a REXX exec it launches — a separate manual from the REXX Reference above, which does not cover either command)
- z/OS MVS IPCS User's Guide, IBM Corp., SA23-1384 (documents the `ADDRESS IPCS` instruction and its per-mode availability within an IPCS session — a separate manual from the REXX Reference above, which does not cover IPCS)
- ISPF Dialog Developer's Guide and Reference, IBM Corp., SC34-4821 (does not independently state the REXX host command environment list for ISPF; see the TSO/E REXX Reference above for that)
- z/OS Using REXX and z/OS UNIX System Services, IBM Corp., SA23-2283 (documents TSO/E REXX's behavior in the OMVS shell separately from the TSO/E REXX Reference above)
- z/VM REXX/VM Reference, IBM Corp., SC24-6314
- The REXX Language: A Practical Approach to Programming, 2nd Edition. By Michael F. Cowlishaw (Prentice-Hall, Inc., a division of Simon & Schuster), Englewood Cliffs, New Jersey 07632, ISBN 0-13-780651-5
- Rexx brief history, Michael F. Cowlishaw, <https://speleotrove.com/rexxhist/rexxhistory.html> (source for the REX-to-REXX naming history and early VM/SP release dates)
- Open Object Rexx (ooRexx) Reference, The RexxLA/Open Object Rexx project, <https://www.oorexx.org/>
- Josep Maria Blasco's Rexx Parser (AST/element parser for Rexx, ooRexx, and Executor, written in ooRexx itself), <https://github.com/JosepMariaBlasco/rexx-parser>, also distributed as part of RexxLA's net-oo-rexx
- ANSI X3.274-1996, Information Technology — Programming Language REXX, American National Standards Institute. Section citations in this edition are drawn from RexxLA's hosted copy of the X3J18 committee's document (X3J18-199X, <https://www.rexxla.org/rexxlang/standards/j18pub.pdf>), the last public-review draft before ratification, not the final published ANSI text itself.
- Classic Rexx built-in function reference (ANSI-1996/TRL-2/z/OS/z/VM comparison chart), rexxinfo.org, <https://rexxinfo.org/reference/articles/classic_rexx_functions_w_nav_menu.html>

## <a id="notes-and-trademarks"></a>Notes and trademarks

IBM, MVS/ESA, OS/390, z/OS, OS/2, VM/SP, VM/ESA, and z/VM are
trademarks of IBM Corporation. Unix is a trademark of The Open Group.
Open Object Rexx (ooRexx) is an open-source project distributed under
the Common Public License (CPL); it is not a trademark of IBM or any
other single organization. Slightly different versions of the two
source articles for this edition appeared in print in the 1990s; the
2023 web revisions were merged and updated with ooRexx guidance in
2026.

## <a id="about-the-author"></a>About the author

Shmuel (Seymour J.) Metz (שמואל בן ל״ביש). Mr. Metz is a Senior MVS Systems Programmer
supporting a Federal Government contract. He has worked with
computers for over half a century. He has been involved in the
development of two different operating systems. He has experience on
a wide variety of languages and platforms, and has used REXX on more
than four of them. Mr. Metz has an MA in Mathematics from the State
University of New York at Buffalo.

---

## <a id="colophon"></a>Colophon

This merged edition was compiled in 2026 from the 2023 web revisions
of the two source articles described under Publication History above.
Its ooRexx-specific guidance was cross-referenced against a maintained
ooRexx-conventions reference. Where this edition's text depends on a
specific claim about interpreter behavior rather than an assumption by
analogy — a `PARSE VERSION` output string, an error number, whether a
given feature or environment exists in a given dialect — that claim
rests either on a running ooRexx 5.2.0 interpreter or on the specific
implementation's own primary reference manual, rather than on
secondary sources or assumption; see References for the full list of
manuals consulted, and which claims rest on secondary sources instead
where a primary manual could not be obtained.

Editorial and drafting assistance for this edition was provided by
Claude (Anthropic).
