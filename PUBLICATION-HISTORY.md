# Publication history

Background on how *Safe REXX* came to be written and published,
reconstructed from the author's own CompuServe mail archive
(`SAFEREXX.SAV`/`SAFEREXX.SCR`, TAPCIS format, 1993-94), his BITNET
mailing-list subscription record (`TSO-REXX`/`LISTSERV`, TAPCIS
format, 1993), and his later OS/2-era mail archive (MR/2 ICE,
folders `F028 Safe REXX` and `F805 CPCUG`, 1993-2013). None of this
belongs in the paper's own body text (see the project's own
no-provenance-in-body-text convention) — it's kept here instead, for
the record.

## Origin (1993)

The article began as a direct pitch. On 11-Dec-1993, in a one-line
message to Dick Conklin (editor, *OS/2 Developer* magazine, published
by Miller Freeman): "How about an article on Safe REXX?" Conklin liked
the title immediately and pointed him at the magazine's new "Advanced
Novice" section.

The outline sent back on 19-Dec-1993 is close to the paper's structure
more than thirty years later:

```
I.    Introduction
      A.  What is REXX
      B.  Summary of pitfalls
II.   Specific examples and recommended avoidance tactics
      A.  Constants
      B.  Continuation
      C.  Labels and signal
      D.  PROC
      E.  Variable references
III.  Recap
```

The working title carried a full subtitle — *..., or Will They Still
Respect My Code in the Morning?* — used loosely for whichever "Safe
REXX" piece was under discussion at the time (attached to "Safe REXX
on the Desktop" in the Zeichick correspondence below). It did survive
to publication, but ended up attached to the *Enterprise* variant
instead: *Safe REXX in the Enterprise, or Will They Still Respect My
Code in the Morning?*, in NaSPA's *Technical Support* (see below).

His own later, more self-deprecating account of how the pitch came
about (email to a personal correspondent, 1-Oct-1998): "I made the
mistake of suggesting that somebody write an article on Safe REXX, and
got nominated by the editor as that 'somebody'. I should have been
more precise and said 'somebody else'."

There was a prior article: *The PC Neophyte's Guide to Self Defense*
(copyright retained), published in NaSPA's *PC Systems & Support* in
October 1993. He offered it to Conklin as a reprint candidate at the
same time as the Safe REXX pitch — rejected as too elementary and not
OS/2-specific for that audience.

Alongside the Conklin pitch, he raised two policy questions with
NaSPA directly: whether they'd permit the same or a similar article to
run in both places, and whether they'd permit its dissemination
through nonprofit BBSs and user groups. Both were already-tested
ground — NaSPA had allowed exactly that kind of BBS/user-group
distribution for *The PC Neophyte's Guide to Self Defense* the first
time around.

## The copyright sticking point, and the actual publication venue

Cathy Passage (publisher, *OS/2 Developer*) answered the copublication
question directly on 25-Dec-1993: submitting to both *OS/2 Developer*
and NaSPA was fine as long as both were told; republishing an accepted
article elsewhere needed the original publisher's written permission
plus full accreditation, and while nonprofit reuse (a user-group BBS,
for instance) was easier to clear than a competing commercial reprint,
it would still need a lag of several months after first publication.
He pushed back a few days later that not informing both publishers
"would clearly be fraud" — and noted the nonprofit-BBS question was
academic unless both NaSPA and OS/2 Developer actually wanted the
piece once it was drafted.

Negotiations with Conklin and Passage over copublication/reprint terms
went smoothly for the general policy question, but the *OS/2
Developer* placement itself stalled: by mid-1994 the article was
accepted in principle but sitting in limbo, and by late September 1994
he was explicit with a different editor about why — "I was negotiating
with OS/2 Developer on that, but I wanted to retain the copyright and
they wanted me to assign it to them." The same message notes he was
"currently in contact with NaSPA on the Safe REXX article" as a
parallel track, and that NaSPA, unlike OS/2 Developer, was flexible on
copyright.

The piece that actually got published wasn't placed by Conklin/Passage
at all. Alan Zeichick, editor at Miller Freeman's *OS/2 Magazine* (a
sibling publication to *OS/2 Developer*), took it instead — his terms
resolved the sticking point directly: "I'm also flexible on
copyrights; all I buy is 'First Worldwide Serial Rights,' after which
they revert to the author." Accepted 2-Oct-1994, targeted for a
January or February issue — matching the paper's own recorded February
1995 publication in *OS/2 Magazine* exactly.

One direct technical correction from this exchange, given orally and
confirmed by email on 5-Dec-1994: "all references to 'address os2'
should be 'address cmd'."

Zeichick also declined *The PC Neophyte's Guide to Self Defense* for
the same reason Conklin had (too elementary, not OS/2-specific), and
referred it onward to IDG's *DOS World*/*MAXIMIZE* (contacts: Steve
Smith, Michael Comendul) — no record found of whether it was ever
placed there.

A running joke across two separate editors, unprompted by him each
time: both Conklin and Zeichick independently asked whether he'd
consider writing (or finding someone to write) an equivalent "Safe
C++" article. No evidence he ever did.

Once he had a title he liked, he also worked the nonprofit-BBS
distribution channel directly rather than waiting on either magazine:
a CompuServe library-upload script (`SAFEREXX.SCR`) shows him
submitting `SAFEREXX.ZIP` — described in the upload text as "an
article describing pitfalls of the REXX language" — to forum OS2DF1,
Library 6, with a note that "a modified version of this article was
printed in the February 1995 OS/2 Magazine." That is exactly the kind
of nonprofit, several-months-after-first-publication distribution
Cathy Passage had described as needing a time lag.

The manuscript itself (`NASPA/SAFEREXX.ASC` in this repo) carries its
own explicit rights grant, matching what he'd negotiated: "Permission
for reproduction in whole or in part is hereby granted to educational,
non-profit and computer user groups for internal, non-profit use,
provided credit is given and this notice is included." Its closing
biography also pins down his employer at the time: Unisys Corporation,
Senior MVS Systems Programmer on a Federal Government
facility-management contract, 34 years' computing experience, MA in
Mathematics from SUNY Buffalo — predating the NSF and EDS-era
employers that turn up in the later MR/2 ICE correspondence.

## The twenty-year reprint tail (1994-2013)

The MR/2 ICE mail archive (folders `F028 Safe REXX` and `F805 CPCUG`)
picks up the story after publication and shows a genuine two-decade
maintenance history, not a single article that then sat still:

- Two parallel versions maintained from early on — *Safe REXX on the
  Desktop* (OS/2-oriented) and *Safe REXX in the Enterprise*
  (TSO/MVS-oriented) — with recurring correspondence clarifying which
  one a given requester actually wanted. (A 1993 BITNET subscription
  to the `TSO-REXX` mailing list, `LISTSERV@UCF1VM`, fits the same
  TSO/MVS audience, though the surviving subscription/archive-index
  mail there is purely administrative — no article correspondence.)
- Reprint/redistribution requests spanning at least: *OS/2 Spoken
  Here* (1997); a UK correspondent revising it for HTML (1997-98,
  detailed back-and-forth over indentation, a missing copyright
  symbol, broken `<pre>` tags, and typo fixes — "rexx" corrected to
  "REXX", "rang" corrected to "range"); the RexxLA newsletter (1998-99,
  serialized in two parts with editor F. Scott Ophof — reprinting the
  *Desktop* variant specifically, per his own note that "the one at
  REXXLA is the desktop version"; the two parts are still online at
  rexxla.org/Newsletter/9812safe.html and 9901safe.html — with him
  still cross-checking IBM's actual CMS/TSO stream-I/O feature history
  for accuracy mid-revision); a La-Z-Boy IT employee who found it
  cited on IBM-MAIN (2004); and, in 2013, emailing Gerhard Postpischil
  ז״ל to track down the exact volume/issue/page citation for his own
  1995 article, for his own bibliography's sake — the same
  precision-about-citation instinct this whole project applies to
  *other* people's claims, turned on his own decades-old byline.
- The *Enterprise* (TSO/MVS) variant, *Safe REXX in the Enterprise, or
  Will They Still Respect My Code in the Morning?*, ran in NaSPA's
  *Technical Support* — the magazine NaSPA sent its own members,
  distinct from the BITNET-adjacent NASCOM BBS NaSPA also ran —
  "around 1995" by his own recollection when pointing a correspondent
  to it, matching the *OS/2 Magazine* piece's timing rather than
  trailing it as a later reprint.
- His own bibliography, restated at least three times over the years
  with matching wording each time (a 2005 résumé to Gabe Goldberg; a
  2013 note to Osher Lifelong Learning Institute newsletter editor
  Carol Henderson; and elsewhere), lists all three publications
  together: *A PC Neophyte's Guide to Self Defense*, *PC Systems &
  Support* (October 1993); "Practicing Safe REXX," *OS/2 Magazine*,
  Volume 2 Number 2 (February 1995); and "Safe REXX in the Enterprise,
  or Will They Still Respect My Code in the Morning?", *Technical
  Support* (1995).
- Outside technical input incorporated into revisions even from casual
  correspondents, e.g., an Amiga ARexx user's explanation of why most
  ARexx code isn't portable (tied to the Amiga's pre-emptive
  multitasking model).
- `F805 (CPCUG)` is otherwise membership/BBS-access correspondence with
  the Capital PC User Group's members-only system, the MIX
  (`mix.cpcug.org`) — confirms the MIX as CPCUG's own BBS, distinct
  from the CompuServe forum library, and incidentally confirms
  `shmuel@acm.org` as a working address of his (per an ACM
  membership). A keyword search of the surviving ACM mail folder
  itself (`F005`, 69 files) turned up no Safe REXX correspondence, but
  the `shmuel@acm.org` address was actively used for it elsewhere —
  e.g., a reader's direct request for a copy, filed under a "Misc"
  folder rather than under ACM.

## Sourcing note

This file draws on several independent archives rescued the same
session: the CompuServe-era TAPCIS mailbox (`SAFEREXX.SAV`/
`SAFEREXX.SCR`, 1993-94 — the article's actual origin and first-sale
negotiation, plus its direct CompuServe-forum-library distribution);
the TAPCIS-era `TSO-REXX`/`LISTSERV` subscription record (1993 —
administrative only, no article content); and the later MR/2 ICE
mailbox (folders `F028 Safe REXX` and `F805 CPCUG`, 1994-2013 — the
reprint/maintenance tail and BBS-access correspondence). None have
been read in full; `SAFEREXX.SAV` alone is 2,664 lines, and this file
reflects a representative keyword search of each, not an exhaustive
transcript.

Two other MR/2 ICE folders were keyword-searched and came up empty for
Safe REXX material: `F005` (ACM, 69 files) and `F099` (comp.lang.rexx,
9 files). `F805` (CPCUG) is not present in the original rescued MR/2
ICE export (`COMM-rescue/MR2ICE/mail`, folders `F001`-`F108` only) —
it, and a further block of folders (`F110`-`F169`, `F800`-`F834`),
turned up separately in a second export from the same source drive
(`I-zip-missing-folders`).

A full folder-by-folder keyword sweep of that second export (over
100,000 files across 108 folders) turned up "rexx" in about thirty of
them, almost all coincidental (REXX as a listed skill in résumés and
job-search correspondence, mainly folders `F814` "Jobs" and `F808`
"EDS"). Most of these are repeated pastes of the same
Publications-section boilerplate across job applications and personal
correspondence, but two instances of it — `F159` "CompuWare" and
`F160` "Osher Lifelong Learning Institute" — carry the fullest version
of the bibliography seen anywhere, including the *Technical Support*
citation with its exact title and 1995 date, which corrected this
file's earlier guess of a 1998 reprint (based on the RexxLA
correspondence's use of the same magazine name for an unrelated
acknowledgement request). Also found: a 1998 mail-client directory listing (`F828` "Secant")
showing a `safe_rexx.html` file, consistent with the HTML revision
described above. (The "REXXLA is the desktop version" line used above
to attribute the RexxLA reprint is from the original `F028` archive,
not this second export.)

One more source: a backup of his old `H:\NASPA\` working folder
(`Downloads\NaSPA.zip`) turned out to be redundant with the
`SAFEREXX.*`/`saferexxe.doc` files already committed to this repo's
`NASPA/` folder — confirmed byte-identical — except for two files out
of scope for this project (a companion, non-Safe-REXX manuscript and
an unrelated sample document).
