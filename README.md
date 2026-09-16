# Safe-REXX

Merged and updated edition of *Safe REXX on the Desktop* (1993/1995)
and *Safe REXX in the Enterprise* (1993/2023) by Shmuel (Seymour J.) Metz.

Scope: platforms and dialects in current use: TSO/E REXX (the only
REXX on z/OS, running under TSO, ISPF, the OMVS shell, batch, and
System REXX), CMS, Classic REXX, Object REXX, ooRexx, Regina.

## Status

`Safe-REXX-Desktop-Enterprise.md` is the merged edition — see its own
header comment for exactly what's done (full merge of both source
papers, checked against every file in this repo including the
original 1993 submissions and an intermediate 1997-98 web snapshot,
not just the two 2023 HTML files, which found one genuinely dropped
section and restored it; ooRexx guidance grounded in
`AI-Priming/ooRexx/RULES.md`). The ISPF, System REXX, and
OS/2/eComStation/ArcaOS (Classic REXX and OREXX) sections are all now
done — the OREXX guidance is drawn from the Object REXX Reference
manuals rather than a live interpreter, since OREXX has been out of
IBM support for years and no live copy was available to test against.
The original
source files, in several formats and archival copies from both
publication venues, remain in this repo's subdirectories
(`NASPA/`, `OS2DEV/`, `Attached/`, `TSM/`, `$REXX/`, `Safe REXX/`) for
provenance — see the merged edition's header comment for the full
provenance notes, including an ambiguous filename worth knowing about:
`NASPA/SAFEREXX.*` is actually the *Enterprise* paper's own archived
copy, not Desktop's, despite the two papers once sharing that working
filename before one was later disambiguated to `saferexxe`.

See [PUBLICATION-HISTORY.md](PUBLICATION-HISTORY.md) for the fuller
origin and reprint story (the 1993 pitch, the copyright dispute that
moved the piece from one Miller Freeman editor to another, and the
twenty-year reprint tail through 2013) — reconstructed from the
author's own mail archive, not something that belongs in the paper's
own body text.

## License

MIT (same as [AI-Priming](https://github.com/shmuelmetz/AI-Priming)),
provisionally — subject to change once submission terms are discussed
with RexxLA.
