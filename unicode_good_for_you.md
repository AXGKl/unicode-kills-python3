## About the Unicode Sandwich ##

Sorry, minirant follows, this misconception is driving me nuts.

This 'unicode sandwich' paradigm, which you can't avoid without getting crazy
in Py3, is why
the industry can't migrate - and this is where the money is, in the end. Money,
which pays developers, who, at least in their leasure time, can contribute to
the language they have to use between 9 to 5. 

Really, Py3 is a toy language as
long as it is blatantly denying the fact, that the professional IT world is kept
together by globally accepted _standards_, with datamodels and all that.

Check
e.g. here http://www.ietf.org/download/rfc-index.txt and try find *any* standard
which is not positional bytes based (e.g. TCP), using escape seqs or with
identifier value pairs. Which are, when to be _processed_ by software and not
just to be treated opaque, always, really ALWAYS ASCII (by definition of being a
standard, because those are created to work globally).

Where is the unicode
indirection, in e.g. the OSI model?

In other words: Value and payload encodings
are _always_ known, in any standard - the codepage fubar created by the MS hack
back then was really the only exception of significance I know of, where
information went into bytes w/o a meaning - and its long overcome, thanks god.
And was anyway relevant only for human data but not for systems.

Python, with
its clean and powerful map and list operations is ideal to process itself (or
at least control the processing of) standardised data - but who exactly in the
professional world needs or wants a unicode indirection to do that?

Python3, it
seems to me, was designed for novices, with the idea of giving them I/O libs
which spit out and accept unicode objects only. So that those people don't need
to comprehend why len('café') can't be 4.

We are just at the beginning of the
Internet of Things era and those things don't care about unicode, really.

Unicode is NOT good for you - if you want to earn *real* money using Python.


Answer: Posted Apr 25, 2015 8:26 UTC (Sat) by peter-b (subscriber, #66996)
[Link]

> Python3, it seems to me, was designed for novices, with the idea of giving
> them I/O libs which spit out and accept unicode objects only. So that those
> people don't need to comprehend why len('café') can't be 4
.
What the hell are you talking about? Have you ever even used Python 3?  From
<https://docs.python.org/3/library/functions.html#open>:

>Python distinguishes between binary and text I/O. Files opened in binary mode
>(including 'b' in the mode argument) return contents as bytes objects without
>any decoding.

The documentation for the "io" module
<https://docs.python.org/3/library/io.html#module-io> explains very clearly that
Python 3 provides raw binary IO, buffered binary IO, and text IO and that you
can use whichever is appropriate.  I've written several binary network protocol
implementations in Python 3 at a previous job. It's really not difficult in the
slightest.


Reply: Posted Apr 25, 2015 16:50 UTC (Sat) by axgk (guest, #102185) [Link]

> I've written several binary network protocol implementations in Python 3 at a
> previous job. It's really not difficult in the slightest.

Peter, you involuntarily confirm my every point.

I explicitly stated that
Python's list and maps in fact are perfect to process standardised industrial
data. Which is pretty much *either* 1. value only, positionally standarised,
like IP and/or 2. using escape seqs or 4. identifier value based.  all with
potential payload, treated opaque, forwarded to upper layers.  Right?  You
worked with one or both of the first two.  And, btw, you did not need to use any
unicode specific API func at all to get that job done, I just know it.

But what
about the third? Identifier/value based Protocols?  Pretty much ALL of them work
with human readable ASCII identifiers these days.  We do have bandwidth,
Identifiers standardised 30 years ago like .1.3.6.1.2.1.2.... are now like this
(random pick): https://www.broadband-forum.org/technical/download/TR-135...  I
could point to HTTP, MIME, whatever, anything on layer 5 upwards. 

*I* was
ranting about Py3's absolute inability to work with type3, due to this crazy
idea that **everything** which is not binary is to represented by a unicode API.
I mean real work, processing identifiers - and acting and consolidating values.
Check your IO module link again, everything which is not raw data is unicode in
this strange world view of localised pizza delivery rest interfaces with NON
Ascii identifier keys, provided you smoked good stuff, creating that API. 

No -
they broke that and they know it:

http://python-notes.curiousefficiency.org/en/latest/pytho...  "What we broke is
a very specific thing: many of the previously idiomatic techniques for
transparently accepting both Unicode text and text in an ASCII compatible binary
encoding no longer work in Python 3."

To be translated to: No more idiomatic
techniques to work with standards based on ASCII text identifiers - in favour of
human text only processing - where, agreed, sometimes a unicode function might
be needed like upper() or counting symbols on a display.

The "very specific
thing" is just the Information Processing Technology "thing", the software for
the systems of this world.  And we learn the reason: "WHY did we break it? (...)
ASSUMING that binary data uses an ASCII compatible encoding and manipulating it
accordingly can lead to silent data corruption if the assumption is incorrect."
(uppercased by myself)

Sorry, Nick: There is nothing to be "assumed". Its not
happening. There ARE no random smiles we need to get decoded by Py3 libs from
TCP Headers and there is no Kanji in HTTP identifiers. Yes, one could send that
stuff over - but it won't hit a python server but the Loadbalancer error log at
max.  

---

Its just one big misconception. 

PS: I would really like to learn why
a commercial company would want you to write binary protocol
parsers in Py3? I mean - why in the world do I need a binary protocol parser in
the same process where I profit from a unicode API ?



Answer: Posted Apr 26, 2015 18:09 UTC (Sun) by peter-b (subscriber, #66996)
[Link]

I don't understand most of your post, sorry. It doesn't seem particularly
coherent to me. :-(

> PS: I would really like to learn why in the world a commercial company would
> want you to write binary protocol parsers in Py3? I mean - why in the world do
> I need a binary protocol parser in the same process where I profit from a
> unicode API ?

R&D context. Small embedded systems were performing automated data collection
and sending each datum as a UDP packet. We needed to get something up and
running quickly to (1) collect and archive the data into HDF5 files for later
analysis and (2) piggy-back onto the protocol for on-the-fly analysis and
debugging. Python 3 was the quickest and most maintainable solution.
Unicode-awareness had nothing to do with the choice.



Reply: Posted Apr 26, 2015 21:33 UTC (Sun) by axgk (guest, #102185) [Link]

> Unicode-awareness had nothing to do with the choice.

sure, so you could have done just as well with Py2. (...) 


Maybe, to illustrate the use
case better - just as an example from my domain:

Devices we manage do most of
the time not send their stuff like in your case as raw data (i.e. parseable by
position and/or maybe with esape seqs), which you worked with as byte arrays I
guess but they speak standards which are more like HTTP headers, e.g. Key-Value.

Your home router has around 2000 of those kvs on board, should you have got it
from your ISP. Like SNMP - just verbose, with human readable string keys and
values, followed often by opaque payload, like a firmware.  Many standards
nowadays use this paradigm.

Every REST/Json interface as well if not sending
lists only.  My point is that in Py3 they just differentiate TWO kinds of data:

Either raw/binary OR text/unicode but NOT byte strings.  We can't work with
neither of them and I wonder how could any company working with standards. We
require byte strings, just as they come in (the keys are always ASCII) and we
don't want/need any library decode ALL that stuff into unicode using some
encoding before we can start consolidating the data.

Because unicode is
something which is ONLY needed for human text - but not for inter-systems
communication.  That does in now way mean we hate unicode or so, we also get
stuff like usernames with funny characters but we handle those non systems
relevant strings just opaque, back and forth between database, Json API and so
on.

We just don't need / want / can decode the WHOLE payload - only because of
those. Example: When doing len() on such a name value, we do want the number of
bytes it takes to store it - and not the number of characters a human user would
perceive on a display. Apropos: Alone RAM consumption is unbearably high if you
create unicode objects out of any string.  If that still is not coherent to you,
maybe Armin describes it better understandable?:

http://lucumr.pocoo.org/2014/1/5/unicode-in-2-and-3/ ?

That guy is pretty well known, author of Flask beyond much other stuff.  --- In
Py2 you can do all three perfectly, even within one process. In Py3 you can only
work with raw bytes OR with text from and for humans. 

---- 

The Internet of Things requires that other string type.








