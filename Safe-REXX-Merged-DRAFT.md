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
and dialects in current use: TSO/E, ISPF, OMVS, System REXX, CMS,
e.g., Classic REXX, Object REXX, ooRexx, Regina.

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
Object Rexx that does not apply to classic Rexx dialects (TSO/E REXX,
CMS REXX, System REXX, Regina, and the like). Note that OREXX (IBM's
original Object REXX for OS/2, the precursor ooRexx implements and
succeeds as an open-source project) is itself an object-Rexx family
member, not a classic-Rexx dialect — but an older, more limited one:
see the specific `~translate` vs `~upper` gap noted below.

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

```rexx
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

Two forms that look plausible by analogy are wrong, and one of them
doesn't even error:

```rexx
say mystem.(i)         /* WRONG: Error 43 -- parsed as a call to a
                           routine literally named MYSTEM */
say mystem[i]          /* WRONG, but raises NO error */
```

The bracket form is where classic Rexx and ooRexx genuinely part ways
— `[]` is an ooRexx operator (the String class's `[]` method), not a
classic-Rexx construct at all, so a classic-Rexx program can't reach
for it by mistake in the first place. In ooRexx code, `mystem[i]`
silently does something else entirely: an unset simple variable
`MYSTEM` evaluates to the string `"MYSTEM"`, and `[]` on a bare string
is ordinary character indexing — this returns `"S"` (character 3 of
`"MYSTEM"`), not the stem element, with no error to flag the mistake.

**ooRexx note**: ooRexx extends indirect tail access with dot-then-
bracket, `mystem.[expr]`, which accepts a genuine *expression* as the
tail — something bare-symbol substitution above cannot do in one step:

```rexx
j = 2
say mystem.[j + 1]     /* CORRECT: 'three' -- ooRexx-only; classic
                           Rexx would need a temp variable first,
                           n = j + 1; say mystem.n */
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

**ooRexx note**: a real `.stem` collection *object* is a further,
genuinely ooRexx-only alternative — plain bracket notation on it is
correct and idiomatic, but it is a **separate namespace** from a
same-named classic compound variable, even when the base name matches:

```rexx
realStem = .stem~new
realStem[3] = 'bar'
say mystem.3            /* still dropped -- "MYSTEM.3" -- unrelated
                            to realStem[3] even if named the same */
```

### <a id="dropped-symbols"></a>Dropped symbols used as constants

ANSI X3.274-1996 distinguishes a *symbol* (§3.1.47: "a sequence of
characters used as a name... [symbols] are used to name variables,
functions, etc.") from the *variable* it may or may not currently
name. A symbol that has never had a value assigned to it — what's
often called "uninitialized" informally — has no variable behind it
yet at all; the standard's own term for this state is *dropped*
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

```rexx
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

Valid I/O redirect types in the `WITH` clause are `NORMAL`, `STEM`,
`STREAM`, and `USING` — `STRING` is not a valid type. Of these, `USING`
— supplying the input value directly, `input using (expr)`, with no
stem or stream needed — goes beyond ANSI X3.274-1996's own `ADDRESS
WITH` semantics, which define only `STREAM` and `STEM` as resource
types; `NORMAL`/`STEM`/`STREAM` are standard, but `USING` is an
ooRexx extension, verified working in ooRexx 5.2.0. Checked against
Regina's own reference manual: its `ADDRESS WITH` syntax diagram lists
only `STREAM`, `STEM`, `LIFO`, and `FIFO` as resource types — `LIFO`
and `FIFO` are Regina's own extensions beyond the ANSI baseline, but
`USING` does not appear anywhere in the manual, so Regina does not
have it. OREXX/OBJREXX not checked directly. To supply empty stdin
(preventing a child process from blocking waiting for input), define
an empty stem and pass it as `INPUT STEM`:

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
be aware that character encoding varies among systems. CMS and TSO
use EBCDIC; most other systems, e.g., OS/2, Linux, Windows, use
ASCII. Even within CMS and TSO there are national-language issues,
and in many systems there are code-page issues. Be aware of the
character sets used in each of your target systems, and program
accordingly. Segregate system-dependent values and code-page-dependent
values to make your code easier to maintain.

Rexx variable names and labels are case-insensitive on every
platform, in every dialect — but two related things are not, and the
platform matters:

- Filenames on Windows (NTFS) or ArcaOS/OS2 (JFS) may be
  case-insensitive in practice, but should still be treated as
  case-sensitive in code that must also run on Linux.
- The `VALUE()` built-in for reading environment variables is
  case-sensitive on Linux but case-insensitive on Windows.

### <a id="io-model"></a>I/O model

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

```rexx
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
values above.

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
standardized, and varies by implementation. OS/2's original RexxUtil
includes Workplace-Shell-specific functions — `SysCreateObject`,
`SysDestroyObject`, `SysSetObjectData`, `SysQueryClassList`, and the
like — that have no counterpart at all outside OS/2/eComStation/ArcaOS,
since they manipulate an object-oriented desktop shell those other
platforms don't have. Checked across three implementations: ordinary
file/system functions such as `SysFileTree`, `SysMkDir`, and
`SysTempFileName` are part of the common core, present in both ooRexx
(verified directly against ooRexx 5.2.0, via `RxFuncQuery` after
`SysLoadFuncs`) and Regina's `RegUtil` package (per its own reference
manual). The Workplace-Shell-specific functions above are a different
story: `RxFuncQuery` reports all four as **not** registered in ooRexx
5.2.0, and none of the four appears anywhere in RegUtil's reference
manual either — neither reimplements the OS/2-shell-specific portion
of the original repertoire, only the platform-independent core.
OBJREXX 6.00 (ArcaOS) is documented elsewhere as having a RexxUtil
function set that differs from ooRexx's, but has not been checked
function-by-function here.

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
directly by the author.

---

## <a id="references"></a>References

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
ooRexx-conventions reference; a small number of specific claims —
`PARSE VERSION`'s actual ooRexx output string, the `EXIT`-versus-
`RETURN` semantics of a called label falling through into a directive
boundary, and the `Error 17` behavior of falling through into a
`PROCEDURE`-led label — were checked directly against a running
ooRexx 5.2.0 interpreter rather than assumed by analogy with classic
Rexx.

Editorial and drafting assistance for this edition was provided by
Claude (Anthropic).
