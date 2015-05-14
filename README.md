### About

Disclaimer: The title of this repo is based on my subjective opinion and chosen provocative - in order to motivate the reader digging into the arguments. The problem *is* severe and the reader, in order to build his own opinion, is required to invest significant time.


- [setdefaultencoding('utf-8'): risk free](#python2-with-a-working-defaultencoding-we-have-it-all)
- [history: why official python rejects it](#analyzing-the-rejection-of-the-setdefaultencoding-switch)
- [why the unicode sandwich is a bad idea for most appps](#setdefaultencodingutf-8-assessment-of-alternatives)
- [python2 the perfect language for modern text processing](#are-explict-manual-conversions-what-python-users-expect)


*Before putting together arguments against Python3's string type, here a longish explanation why Py2's bytes str type is what we want*



# Python2 With A Working DefaultEncoding: We Have It All. 


Blog: Arguments For Py2 DefaultEncoding UTF-8 and Against the Py3 Paradigm That Text Equals Unicode


## Cognitive Dissonance ##
 
Responsible end users DO see that Python's core people and their multipliers object to change the defaultencoding categorically - but they also ask themselves questions:


### How can widening Py2's implicit type conversion feature into full UTF-8 be ALWAYS wrong when... ###

1. Fedora concluded it is [ALWAYS right!][fedora]

   Fedora proposed, after investigating risks, to change distribution wide, i.e. def.enc.=UTF-8 to be the right thing for *any* Linux user round the world. 
Got [refused][fedthread] by all relevant core people except Guido who did not involve ('not supported', 'strange things will
start to happen', 'you are on your own', 'was forgotten to remove') but w/o hard arguments except potential support nightmares related to users opening textfiles in various encodings and also related to filesystem encodings. Which is presented also as the reason why ASCII [was chosen][why_ascii] in the first place ([by Guido][guido_uni] himself). Sample argument from a highly respected great contributor to the language, which is often cited:

    "[it **will** break Python][will_break]" (bold as in the original quote)
    
    Hard fact presented though WHY Python would break is only the hashing behavior I [listed][mytable], which is never picked up by any other opponent within the core community as a reason to worry about or even by the [same person][loewis], when working on user tickets. 

    [Resume of Fedora][fedresume]: "Unpopular" with core people, inconsistency with previous versions.

1. There are [3000 projects alone at openhub][openhub] doing it 
   
   Slow search frontend but scanning over it, I estimate 98% to UTF-8. Nothing found about nasty surprises.
   
1. There are [18000(!) github master branches][github] having it changed
   
   While the change is "[unpopular][fedresume]" at the core community its *pretty* popular in the user base. Proves nothing you might say, there is tons of crazy stuff out there but see the next one:     

1. There are only [150 bugreports][ghissues] totally at github around that. At a rate of nearly 100% the change is the cure(!) and not the poison (as claimed ever so often)  
   
    I scanned through all of them:

    - Def.enc. to UTF-8 is typically INTRODUCED but not removed in the issue closing process, most often as the final cure. Guess around 90-95% of these. Some bigger ones excusing it as [temporary fix][holo], being aware of the 'bad press' it has, far more bug reporters are [just][ghglad1] [glad][ghglad2] about [that fix][ghglad3].

    - A few (1-5?) projects modified their code doing the type conversions manually, so that they did not need to change the default anymore.

    - In two I see someone claiming, that with def.enc. =UTF-8 he gets ["no print" out][noprint] [anymore](https://github.com/ipython/ipython/issues/8071), without explaining test setup but pointing to the official 'get your encoding right' line.
      I could not verify the claim, tested one and found the opposite to be true. Btw, I find [this one][crawler] a good example of what end users expect from the language, see later.

    - One [claims](https://github.com/fritzy/SleekXMPP/issues/82) his "system" might depend on not changing it but we do not learn why. 

    - One(!) had real reason to curse about it: [ipython][hgipyth], because a 3rd party module or the test runner modified their process in an uncontrolled way (it is never disputed that a def.enc. change is advocated by its _proponents_ only at interpreter setup time, i.e. when 'owning' the process).

    **I found zero indication that the different hashes of 'é'  and u'é'   cause problems out there**


1. Python does NOT break

   As far as I can honestly assure you, the interpreter faces NO problem, there is NO hidden black magic, which "breaks Python", the concluding statement in the Fedora thread, which is often cited. I'll reflect on this statement in more detail down below.
   
   I just ran a _make test_ on a std. AWS 64bit jessie on Py2.7.9. Testoutput diff between def.enc ASCII versus UTF-8 in site.py: 
   
   *(No, did not forget to paste the diff - it is empty. The infamous 'ordinal not in range' exception is manually instantiated to test its output - but not provoked.)*

   This means: No feature of Python under unittests is working differently than w/o the switch. The switch _itself_ though, is not tested at all. Looking at the empirical results, plus statements from developers I'll present below, there is little reason to believe in a remaining risk.

   Not so relevant but still: I also just ran a pretty extensive functional test suite, on a development server within my company, for a larger project with def.enc. changed in every container: Zero deviation to default ASCII.
The setup involves 26 interplaying processes, most Py2, incl. microframeworks but also Zope, running a few decades of man years of custom code on top, written by experts AND ordinary guys like myself.

1. It is advised on bugs.python.org to frustrated users

    (often connected with the official line of warning)

    Examples [here][bugs1], [here][loewis] or [here][bugs2]

    The first one demonstrates pretty well how established the switch is in [Asia](https://www.google.de/?gws_rd=ssl#q=bones7456) (and compare with github repos). 


1. Ian Bicking [published][ian] that for him it is ALWAYS right...
   
   This one is entertaining, also a history lesson: 
 
   Ian's blog from 2005 pushed the idea of changing the defaultencoding to UTF-8 massively:

   *I can make my systems and communications consistently UTF-8, things will just get better. I really don't see a downside. But why does Python make it SO DAMN HARD (...) I feel like someone decided they were smarter than me, but I'm not sure I believe them.*


1. ... and Martijn Fassen, while refuting Ian, [admits][fassen] that ASCII might have been wrong in the first place & nothing breaks

    *"I believe if, say, Python 2.5, shipped with a default encoding of UTF-8, it wouldn't actually break anything. But if I did it for my Python, I'd have problems soon as I gave my code to someone else."*

    I find Martijn's contributions regarding Python outstanding (seems to be the name ;-)). As other core people he brings forward only one argument: Library code can't be shared since def.enc. modifies the config of the interpreter. Which really is 100% relevant for lib developers - but next to zero for end users, who do not intend to share their stuff with the world except, maybe, as standalone tools.

    In any case, back in 2005, a modified Python environment was pretty nasty with respect to packaging and configuration mgmt. indeed - mainly because there was no virtualenv.

    How did Ian react? He [_DID_](http://www.ianbicking.org/projects.html) create virtualenv - and later pip. 

    *Note: Ian was always known for his 'thinking out of the box' attitude, also labelled [crazy hacking](http://carljm.github.io/pipvirtualenv-preso/#1) by Python experts. Somewhat an outsider to the core community, he [left](http://www.ianbicking.org/blog/2014/02/saying-goodbye-to-python.html) Python after a rejected talk for Pycon 2013.*


1. In Python3, they don't "practice what they preach" ;-)

    While opposing any def.enc. change so harshly, because of environment dependent code or implicitness, we learn e.g. [here](http://bugs.python.org/issue19846) about [Python3](http://bugs.python.org/issue9549)'s problems with their ['unicode sandwich'][unipain] paradigm and the corresponding required implicit assumptions - based not on interpreter config but even on shell environment variables.

    Further they created possibilities to write valid Python3 code like:

        >>> from 褐褑褒褓褔褕褖褗褘 import *        
        >>> def 空手(合氣道): あいき(ど(合氣道))
        >>> 空手(う힑힜('👏 ') + 흾)
        💔

    *Identifiers picked randomly, hope they don't by accident mean sth. unpolite ;-)*

    *Note: I do think it will attract end users to Python, so I actually like it and we might even use it (should Py3 once be usable for companies like mine). I just highlight the logical twist in a main argument against changing def.enc: portability. A single environmental deviation, in times of docker, compared to the share-ability of this?*

7. DiveIntoPython [recommends it][dive_into]

   and this even for codecs != UTF-8


7. Most prominent I/O libs are doing implicit UTF-8 type conversions

    I picked two libs essential in _my_ world but there are others. 
   
    Lets take Python2 completely out of the loop regarding implicit conversions, no problem to switch it off:

        >>> import sys; reload(sys); sys.setdefaultencoding('undefined')
        >>> print str(u'') => UnicodeError. # Implicitness removed


    Lets start building:

        >>> import json, redis
        >>> s, u = 'é', u'é' 
        >>> m = {s: 1, u: 2} # this is not about sandwiches but how these libs behave
        >>> len(m), len(json.loads(json.dumps(m)))
        (2, 1)  # data lost  
        >>> r = redis.Redis()
        >>> sval, uval = 'χ', u'ε'
        >>> r.set(s, sval); r.set(u, uval)
        >>> stored = r.get(s) # another node wants value of s. But gets:
        >>> stored == sval or type(stored) == type(uval)
        False # 'χ' is lost, 'ε' returned, as utf-8 encoded bytes ([configurable][redis] only on connection level)



    *I leave it to the reader to count the <i>implicit UTF-8 based type conversions</i> between bytes and unicode done by those not exactly irrelevant libs. Both assume UTF-8 even in danger of potential data loss(!)*


Futher notes I collected:


- [Must Read][guido1] thread, in which Guido himself takes part and [advises][guidoenv] a [professional end user](https://github.com/cjw296) to use a process specific environ with the switch set: "...create a custom Python environment for each project.".

   And further:

   *"The fundamental reason the designers of Python's 2.x standard library
don't want you to be able to set the default encoding in your app, is
that the standard library is written with the assumption that the
default encoding is fixed, and no guarantees about the correct
workings of the standard library can be made when you change it. There
are no tests for this situation. Nobody knows what will fail when. And
you (or worse, your users) will come back to us with complaints if
the standard library suddenly starts doing things you didn't expect."*

   Checking [bugs.python.org][pybugs], searching for  just demonstrates that _end users_ consider the _missing_ of that feature a bug. Also, no one seriously asks for the feature outside process initialization time.

- Jython offers to change it on the fly, even in modules (org.python.core.codecs.setDefaultEncoding("utf-8"))


- PyPy did [not](http://osdir.com/ml/python.twisted.bugs/2008-01/msg00030.html) support reload(sys) - but [brought it back](https://bitbucket.org/pypy/pypy/commits/19e305e27e67) on [user request](https://bitbucket.org/pypy/pypy/issue/1991/reload-sys-error) within a single day w/o questions asked or trying to educate the user (yes, user's process wanted to [change](https://github.com/SiCKRAGETV/SickRage/blob/master/SickBeard.py#L148) the defaultencoding to UTF-8). Compare with the "[you are doing it wrong][mercurial]" attitude of CPython's unicode people, [claiming](http://bugs.python.org/issue9632) w/o prove it is the 'root of evil'.

---
    
Ending this  list I confirm that one *could* construct a module which crashes **because** of a changed interpreter config, doing sth like this:


    def is_clean_ascii(s):
        """ [Stupid] type agnostic checker if only ASCII chars are contained in s"""
        try:
            unicode(str(s))
            # we end here also for NON ascii if the def.enc. was changed
            return True
        except Exception, ex:
            return False    
    
    if is_clean_ascii(mystr):
        <code relying on mystr to be ASCII>


     
I do think though, there is a logical twist here:
The person who wrote this dual type accepting module was obviously aware about ASCII vs. non ASCII strings. But then one would assert that he also should know about .decode and .encode functions, don't you think?

So the question should not be COULD libs crash but rather: DO libs crash caused by changing the default encoding. *My* definite answer (also taking github result into account): No.


Lastly: 

As already pointed out in the previous post, widening the scope of Python's implicit type conversion feature may reveal bugs _not related_ to implicit type conversion (like counting code points when you wanted the number of bytes at a len() operation). How exactly? By crashing before they impact!

I leave it to the reader to judge if this is really an argument against the setdefaultencoding switch _itself_.


![lbbrown][linebreakbrown]



## Analyzing The Rejection of the SetDefaultEncoding Switch ##

*Getting implicit string type conversion done by Python2 seems to be by far the most requested but officially denied feature of the language.*

*Question is **why they oppose it** so rigorously - although so many want it so badly they ignore all the warnings.*

The root cause of the problem is a **software engineering mistake** with terrible consequences - incl. the chasm analyzed within this text.

### Bytes Without a Meaning: The IBM/Microsoft Codepage [Fubar][fubar] - And Its Relevance Today ###

Since many (especially younger developers) don't even understand the problem any more, i.e. why the switch _could_ be harmful, here some background information about it.

### How Global Standards Work ###

Real quick: IT is about processing **information**, computers can just handle **bits and bytes**. So when information is to be stored, it must be converted to bytes, i.e. "encoded". When information is to be restored, so that a system can act or a human can understand it must be "decoded" from bytes.

Note: Text created by and for humans is only a tiny little fraction of information which can be encoded in bytes, by and for _systems_, in the information processing chain. 

If information is *exchanged*, the decoder must know _how_ and _exactly_ how the encoder converted his information to bytes in order to be able to do the reverse thing and get back the information.

In order to allow information exchange with others, [tons of globally agreed standards][rfcs] had been created, defining exactly how encoding of information into bytes has to happen.


Typically [_many_ systems, on different "layers"][osi] are involved when information is **exchanged** - and all of them need to be able only to decode _their_ part in order to act correctly, i.e. forward to the next system and in the end, dependent on use case, maybe display sth. like this text to a human, so he can decode within the final layer, his brain. Hopefully ;-)

Here the main possibilities how standards define how to encode information within their layer (the rest in any arbitrary byte sequence is just treated as "payload", i.e. **opaque** for the current layer).

 - By _position_ of bits (e.g. IP, TCP, ...), then w/o identifiers

 - By identifier / value pairs, where identifiers are either

   - fixed bit sequences

   - numbers

   - alphanumerics up to human readable words (e.g. HTTP headers) - prefixed or wrapped within defined 'special' characters, like '<', '/>' or [ANSI][ansi] escape sequences. 

When it is done with words, those are, by definition for a global standard, **always ASCII**. 


### The Mistake ###

Now the nasty thing which happened long ago: IBM came up with this not so bad idea regarding how _more_ information could be encoded within the same amount of bytes, for a certain type of information: Symbols intended for humans - by mapping unstandardised bit sequences, using various mappings, i.e. [lookup tables][codepages].

IBM did this just for eye candy (text base menus) - but MS, trying to get their OS landed worldwide, for symbols which _humans_ use to encode **text**. Via this, humans could enter or get text including symbols of their local language and e.g. print it out locally - using an affordable OS. This boosted the worldwide perception of the OS with no questions asked, creating a 'de-facto' standard (asian countries were later also [covered](http://en.wikipedia.org/wiki/Shift_JIS) using 2 Byte wide standards).

**But in their de-facto standards for text encoding, they [forgot][gates](?) to specify _within_ the payload the encoding standard applied**(!) (e.g. by defining a byte at position 0, making decoding possible for anybody with JUST the payload, the bytes at hand).
Instead, **they hard coded(!) it** into the "localized versions" of their OS itself.
Result: In global companies people had to put stickers on media, informing the receiver about the codepage used to encode the text...

Fortunately this mess was resolved by the **unicode indirection**, defining ONE intermediate lookup table to encode ALL symbols humans (not systems) globally use to write down information. And the encoding of the keys in THIS table into the final bytes is done via standards like UTF-8.

Unfortunately, though, textual information for humans is often meant to _last_ and is being _stored_, in order to be decoded _often_ over time -  unlike e.g. a proprietary routing protocol or the fancy text menu enhancements of IBM. Those come and go, texts last.

**Only in recent years we overcame the problem and code pages bear next to zero relevance anymore, for most computing tasks**.

And even if they would: It is unacceptable to 'punish' each and any user of a computing language for a domain specific engineering problem.

I have not much knowledge about the windows platform so I quote [wikipedia](http://en.wikipedia.org/wiki/Windows_code_page):
*Windows code pages are sets of characters or code pages (known as character encodings in other operating systems) used in Microsoft Windows from the **1980s and 1990s**. Windows code pages were gradually superseded when Unicode was implemented in Windows, although they are still supported both within Windows and other platforms.*

I consider the case close, the problem WAS huge in the human text processing domain - but today it is practically of no relevance any more.

Note: We still don't see the unicode -> bytes encoding standard _within_ the payload itself ([except][bom] in one of them) but the commonly accepted one [UTF-8][utf8main] is able to [detect 'itself'](http://en.wikipedia.org/wiki/Charset_detection), having just payload at hand. Further, in networking, all wrapping protocols, able to also carry text payload for humans, [do define](http://en.wikipedia.org/wiki/List_of_HTTP_header_fields) identifiers for the decoding standard to apply, should it not be fixed.

----

### Conclusions Regarding SetDefaultEncoding ###


 - When Guido picked ASCII as Python's defaultencoding, it was a time were the codepage problems had been very present. Python, correctly positioned as an universal language to process bytes, had to ALSO deal with _that_ type of bytes: Bytes, encoding information for humans - in often unknown encodings.

    ![guidouni][unipic]

 - These times are pretty much over. Only very few, specialized computing tasks involve human texts created on machines without support for the unicode standard's identifiers. So reason number one is pretty much outdated, number two should only apply for those special tasks but not for the default, while number 3 applies for UTF-8 as a superset of ASCII as well.


 - At time when the PyCore community settled their opinion about setdefaultencoding (and never re-evaluated, unfortunately) we [witness][fedthread] them discussing still mostly problems connected with [file I/O](http://article.gmane.org/gmane.comp.python.devel/110080) and FS encodings.

Quite understandable - at _that_ time. 


![lbbrown][linebreakbrown]

*Following up the previous analysis of the chasm between endusers and core developers incl. reasons, lets go through alternatives and practical recommendations now*


## SetDefaultEncoding=UTF-8: Assessment of Alternatives ##

End users, when facing crashes due to Python2's default refusal to implicitly convert bytes to and from unicode, are educated either to 

 A) explicitly code each and any necessary type conversion

 B) convert to unicode each and every bytestring right at ingress ('[unicode sandwich][unipain]'), often connected with a hint to migrate to [Python3][bugsto3] right [away][guidoenv], where the APIs' native string class resembles Py2's unicode type. 


*This section tries to create understanding why this is _NOT_ a sensible option for a large group of users.*


### The "[Unicode Sandwich][unipain]" - A Fit For _Any_ Use Case? ###

Definitely: No, not at all. And thats a fact of life:

It is natural to develop code with a 'sandwich' model in mind, restricting en/decodings to 'boundaries' - to /from other libraries or the outside world.
Thats a "no brainer" and not a revelation.

Not at all natural it is to derive from this, that the 'meat' in the sandwich MUST be unicode under all circumstances and NOT byte strings.

**Novices** in Py2 start with juggling bytestrings around and they get a long long way with this, longer than in a unicode sandwich where their first piped non ascii print out crashes with funny error messages.  

Also quite some undisputed Python **Experts** seem to prefer to work with bytestrings. Most prominent [Armin Ronacher][armin_1], the whole Zope/Plone community is another, [Mercurial][mercurial] another. Even Ned Batchelder [does face situations][ned]...

I'm NOT an expert - but the Python servers my company sells get bombarded by mgmt. stacks of currently some 30 million little devices around the globe. It would be just plain *wrong* to convert all that traffic to and from unicode before/after consolidating it, given "advantages" like...

    >>> RAM, unic = sys.getsizeof, u' ' * 100000
    >>> round(float(RAM(unic)) / RAM(str(unic)), 2)
    4.0
    
 ... and none of the data consolidation routines require a single unicode function.

Wrong!! This code is misleading - the unicode sandwich proponent claims: We live in a 'global' world and this example is not: Beyond ASCII the factor shrinks to 'only' a 2 or even less. And what has data consolidation to do with string types?

Addressing a major misconception:

**The near irrelevancy of unicode in GLOBAL communication standards and protocols**

*While unicode maps the BIGGEST POSSIBLE set of meaningful information for inter HUMAN communication, those define the smallest necessary one for inter SYSTEMS communication.*

When you check any explanation of the [OSI][osi] model, you'll find that the word 'unicode' appears maximum once - as [_one_][uni2] possibility of payload encoding on the representation layer, from code points to displayed symbols. Not more - not less.

I explained already in the previous post, that any _other_ type of information is standardized via positional values or using ASCII identifier / value pairs - a perfect fit for Pythons' outstanding lists and maps.

And with more and more bandwidth available, higher level protocols did get also really verbose regarding identifiers - what was standardized 30 years ago with identifiers like this: 

    .1.3.6.1.2.1.2.2.1.8 # mapping to: .iso.org.dod.internet.mgmt.mib-2.interfaces.ifTable.ifEntry.ifOperStatus

today goes through the Internet comparable to this: 

![stb][stbtr]


 - There is *a lot* of information to be processed by software and only a tiny subset is for humans. 

 - Unicode has zero relevance for inter systems communication: Those act based on bits, bytes, ASCII character identifiers (Identifiers within global standards _never_ go beyond ASCII words to encode information - by definition).

 - Python is ideally suited to process information directly - or control HOW the direct processing should work.



[Again](http://mike.passwall.com/networking/netmodels/isoosi7layermodel.html#PRE):

*The theory and idea behind having standards accepted, ratified, and agreed upon by nations around the world, is to ensure that the system from Country A will be easily integrated with the system from Country B with little effort.*

=> Writing software to enable *truly global* communication means writing software which can process bits, bytes, ASCII identifiers - but in no way does it mean to decode any string of information as if it was intended for humans.


Since the advent of Py3, the Python core community, as it seems to me, recognizes only undecoded binary data versus 'text' - which must be processed via a unicode API and [they know it][nick]:

"*What we broke is a very specific thing: many of the previously idiomatic techniques for transparently accepting both Unicode text and text in an ASCII compatible binary encoding no longer work in Python 3.*" - with the argument: 

"*assuming that binary data uses an ASCII compatible encoding and manipulating it accordingly can lead to silent data corruption if the assumption is incorrect.*"

This does not happen. Nothing is to be assumed. One can 100% rely on the **identifiers** to be 'stateless' ASCII. The Microsoft caused mistake of storing (human text) byte values, without a positional or identifier based encoding type value, was a major engineering mistake - which could cause so much trouble only because of an exceptional situation: The change into the information age as such, with a vacuum of standards in the domain of text processing.

---

Identifiers have to be 'understood' by the processing software, sliced and diced, hased, compared, concatenated, logged, alerted (...). They are always max. ASCII, if present at all. Values, if not numbers, are typically forwarded to the next layer, which is, maybe a human.

Systems software, when running operations like len() on such opaque values are interested for sure only in the number of bytes those values occupy in the RAM or on the socket - but mostly *not* on the potentially human perceivable symbols. And if so: There are great unicode functions available in the most simple 'pythonic' way. 



If it is still not yet 100% understood why so many ASCII strings have to be processed by the software of this world, with an ideal fit of the Python2 byte str() type: [Here](http://t.co/6ksmZDd4jG) is an example, relevant also for a standard we *all* know well: HTTP. Should they, just to cut the "X-" off 100% guaranteed ASCII header keys, converting them all to unicode first?

**JSON**

Isnt' this a standard allowing unicode identifiers?
No. Json is not an application communication standard but a generic data interchange format, now an RFC https://tools.ietf.org/html/rfc4627 standardized based on a given widely used de-facto standard.

The simple reason why its identifiers (and values) are unicode has a simple reason: In its beginning, Javascript was created to improve human perception of otherwise static HTML. Thats why it is based on unicode and that property got than also the property of their data exchange format with the servers, with the advent of Web 2.0. Yes, it theoretically does allow non ASCII identifiers but, with maybe the exception of your local Pizza delivery services's Rest interface - this has nothing to do with global communication standards.

Its main success factor: The possibility of nesting structures, with extensible keysets. The keys themselves, in standards based on json are always ASCII (i.e. same bytes as Unicode).  

Nevertheless, in the upcoming times of interconnected microservices, based on Json as its lingua franca, sometimes unicode values happen to diffuse into byte sandwich based software.

*Which is a good example of the usefulness of a UTF-8 based defaultencoding.*


--- 

Closing this section I want to remind the reader that the Python3 choice of a unicode based native str type was for sure influenced by the social network hype at that time, connecting humans closer with each other than ever before. That might have influenced them a lot, besides the then perceived problems with code pages, 2 byte encodings, and so on - with unicode as the only rescue.

But times have changed a lot: Encodings are by default reliably at hand and the era which is just starting off is this: **[Internet of Things](https://www.google.com/search?q=Internet+of+Things&ie=utf-8&oe=utf-8&gws_rd=cr&ei=7Gs5VafIF4LXapiJgaAG)**.

Those interconnected things do not sense and act based on unicode text but on ASCII identifier based or positional information exchange protocols.


That in turn, to stay within the picture, lets unicode sandwiches taste a little ... rotten - compared to a byte string sandwich with a UTF-8 based default encoding for normally anyway opaque values. 

And there is **a lot** dynamic business logic involved within that area - and this is where Python really really shines, especially when defining that logic together with domain specific experts - see also the next point. 

**Lets hope the recent [good news](http://legacy.python.org/dev/peps/pep-0461/) about 3.5 do mean that Python gets back to were it already was with version 2 & and a decent default encoding: A perfect language to parse and process the [communication standards][rfcs] of this world.**


![lbbrown][linebreakbrown]

### Are Explict Manual Conversions What _Python_ Users Expect? ###

Zen of Python rule "Explicit is better than implicit" is quoted, when hinting users to solve all problems themselves, instead of letting the interpreter do it, by changing the default encoding.


This is obviously completely infeasible, if the problems an end user is trying to solve are happening in code outside his control, which is often the case, alone due to versioning or know how constraints but lets ignore this for now.

Yes, feasible. And for library writers a MUST - I expect the libs I use to have gotten their encodings in order.

But how about _endusers_ especially those in the industry, with its often insane time constraints?

Even IF the user got all explicit string type conversions right internally - can he be sure about

  - really all third party libs imported?
  - code from colleagues, customers, partners?

Did he carefully check the _return types_ of all third party libs in use? Many accept both types at ingress but deliver only one at egress, some's egress depends on type of ingress. All potential combinations included in tests? 

*Note: The risk to overlook / undertest missing explicit conversions is higher(!) in English speaking countries, with human I/O often ASCII only - by pure "luck"* 

Tip for all users who decide to go this path: Besides stuffing non ASCII data into unit tests, wherever the use case involves data potentially from and for humans, I recommend also to set the default encoding, while testing, to [this codec][ascic] and check the warnings...

--- 

*Some users don't want this, even if they could - [Java][java] is then the better language, yet [more explicit][java]*

[Here is a good read][pissed] regarding how the 'do it yourself' tip is perceived by _many_ end users, incl. myself, especially after investigating about the next to zero risks of the setdefaultencoding switch. 


Many end users like myself deliberately picked Python, since it beats any other language, regarding its low 'noise', purity and expressive power, which leads to ultimate _simplicity_, in the best possible meaning of this word (btw: there is also Zen rule number 3). 

THIS is the reason why many people chose and got happy with the language, imho - for any other computing requirement there are meanwhile better suited ones.

Example?

Only in Python, once a fitting custom API is built within a project, you can take smart problem owners and go with them step by step through the code, finetuning what they want. It is still very satisfying to see how excited they get when they see how transparent their problems can be addressed, compared to other languages.


We should not throw this all away, because of problems which are not present any more - and never were in most computing domains.

Experts should stop irritating and alienating users when they help themselves with this simple switch, then getting a next to perfect language.

I think its time to realize what we have with the Python 2 way of working with text and continue lobby for it in Py3 (there are good things [happening](https://mail.python.org/pipermail/python-dev/2014-March/133621.html) recently), using unicode 'only' as a then appreciated feature - when needed.
But only then.   


**import antigravity**



misc links:

 German: Py3 the future? http://www.sax.de/unix-stammtisch/docs/2014-02/py23.pdf






  






[java]: http://www.oracle.com/technetwork/articles/javase/supplementary-142654.html




[ascic]: http://www.google.de/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0CCcQFjAA&url=http%3A%2F%2Ftwistedmatrix.com%2F~washort%2Fascii_with_complaints.py&ei=i4I5VbvcLMXVapCCgfgK&usg=AFQjCNF2JNBDxzkztGjNGnxxLHUBMdxFpQ&bvm=bv.91427555,d.d2s

![lbbrown][linebreakbrown]

[nick]: http://python-notes.curiousefficiency.org/en/latest/python3/binary_protocols.html


[uni2]: http://carl.sandiego.edu/bus188/osi_model.htm

[ned]: http://nedbatchelder.com/blog/201106/filenames_with_accents.html

[mercurial]: https://bugs.python.org/msg199265

[pissed]: http://bugs.python.org/issue1668295

[bugsto3]: http://bugs.python.org/issue6832

[guido_uni]: https://downloads.egenix.com/python/Unicode-EPC2002-Talk.pdf

[bom]: http://en.wikipedia.org/wiki/Byte_order_mark

[utf8py]: http://utf8everywhere.org/#faq.python

[utf8main]: http://utf8everywhere.org/

[codepages]: https://10kloc.wordpress.com/2013/08/25/plain-text-doesnt-exist-unicode-and-encodings-demystified/

[gates]: http://boldomatic.com/view/post/FEiBYw
 
[ansi]: http://en.wikipedia.org/wiki/ANSI_escape_code

[rfcs]: http://www.ietf.org/download/rfc-index.txt

[fubar]: http://www.allacronyms.com/_military/FUBAR/Fucked_Up_Beyond_Any_Recognition

[osi]: http://www.webopedia.com/quick_ref/OSI_Layers.asp

[bugs1]: http://bugs.python.org/issue6611

[bugs2]: http://bugs.python.org/msg31343

[pybugs]: http://bugs.python.org/issue?%40columns=id%2Cactivity%2Ctitle%2Ccreator%2Cassignee%2Cstatus%2Ctype&%40sort=-activity&%40filter=status&%40action=searchid&ignore=file%3Acontent&%40search_text=setdefaultencoding&submit=search&status=-1%2C1%2C2%2C3

[guido1]: https://mail.python.org/pipermail/python-dev/2009-August/091362.html

[guidoenv]: https://mail.python.org/pipermail/python-dev/2009-August/091418.html

[loewis]: http://bugs.python.org/issue1052098#msg54297

[will_break]: http://python.6.x6.nabble.com/Proposed-downstream-change-to-site-py-in-Fedora-sys-defaultencoding-tc1891360.html#a1891499

[why_ascii]: http://article.gmane.org/gmane.comp.python.devel/110071

[armin1]: http://lucumr.pocoo.org/2014/5/12/everything-about-unicode/

[pep20]: https://www.python.org/dev/peps/pep-0020/

[brandon]: https://www.youtube.com/watch?v=z9Hmys8ojno

[death]: https://www.youtube.com/watch?v=LAJQOeCSs_o

[loewis_evil]: https://mail.python.org/pipermail/python-dev/2009-August/091405.html

[lemburg_cache]: https://mail.python.org/pipermail/python-dev/2009-August/091406.html

[dive_into]: http://www.diveintopython.net/xml_processing/unicode.html

[codecs]: https://docs.python.org/2/library/codecs.html#codecs.open

[redis]: https://github.com/andymccurdy/redis-py/blob/2.10.2/redis/client.py#L398

[unipain]: http://nedbatchelder.com/text/unipain/unipain.html#35

[crawler]: https://github.com/meibenjin/GoogleSearchCrawler/blob/master/gsearch.py

[noprint]: https://github.com/meibenjin/GoogleSearchCrawler/issues/3

[ghglad1]: https://github.com/seecr/weightless-core/issues/2

[ghglad2]: https://github.com/ajs124/autokey/issues/165

[ghglad3]: https://github.com/joeyespo/grip/issues/86

[holo]: https://github.com/ioam/holoviews/issues/29

[laggards]: http://sealedabstract.com/rants/python-3-is-fine/

[nick]: http://python-notes.curiousefficiency.org/en/latest/python3/binary_protocols.html

[fedora]: 
https://fedoraproject.org/wiki/Features/PythonEncodingUsesSystemLocale

[mytable]: http://stackoverflow.com/a/29558302/4583360

[fedthread]: http://thread.gmane.org/gmane.comp.python.devel/109914

[fedresume]: http://article.gmane.org/gmane.comp.python.devel/109970

[openhub]: http://code.openhub.net/search?s=setdefaultencoding

[github]: https://github.com/search?utf8=%E2%9C%93&q=setdefaultencoding+language%3APython+utf-8&type=Code&ref=searchresults

[ghissues]: https://github.com/search?l=python&o=desc&q=setdefaultencoding+language%3APython+utf-8&ref=searchresults&s=updated&type=Issues&utf8=%E2%9C%93

[hgipyth]: https://github.com/ipython/ipython/pull/252

[ian]: http://www.ianbicking.org/illusive-setdefaultencoding.html

[fassen]: http://blog.startifact.com/posts/older/changing-the-python-default-encoding-considered-harmful.html

[linebreakbrown]: http://i.stack.imgur.com/FFxEA.jpg


[unipic]: http://i.stack.imgur.com/krQhC.png


[stbtdr]: http://i.stack.imgur.com/4VTGj.png


 [stbtr]: http://i.stack.imgur.com/rJyw2.png


 [antigravity]: http://i.stack.imgur.com/vN3VT.png



[uni2]: http://carl.sandiego.edu/bus188/osi_model.htm

[ned]: http://nedbatchelder.com/blog/201106/filenames_with_accents.html

[mercurial]: https://bugs.python.org/msg199265

[pissed]: http://bugs.python.org/issue1668295

[bugsto3]: http://bugs.python.org/issue6832

[guido_uni]: https://downloads.egenix.com/python/Unicode-EPC2002-Talk.pdf

[bom]: http://en.wikipedia.org/wiki/Byte_order_mark

[utf8py]: http://utf8everywhere.org/#faq.python

[utf8main]: http://utf8everywhere.org/

[codepages]: https://10kloc.wordpress.com/2013/08/25/plain-text-doesnt-exist-unicode-and-encodings-demystified/

[gates]: http://boldomatic.com/view/post/FEiBYw
 
[ansi]: http://en.wikipedia.org/wiki/ANSI_escape_code

[rfcs]: http://www.ietf.org/download/rfc-index.txt

[fubar]: http://www.allacronyms.com/_military/FUBAR/Fucked_Up_Beyond_Any_Recognition

[osi]: http://www.webopedia.com/quick_ref/OSI_Layers.asp

[bugs1]: http://bugs.python.org/issue6611

[bugs2]: http://bugs.python.org/msg31343

[pybugs]: http://bugs.python.org/issue?%40columns=id%2Cactivity%2Ctitle%2Ccreator%2Cassignee%2Cstatus%2Ctype&%40sort=-activity&%40filter=status&%40action=searchid&ignore=file%3Acontent&%40search_text=setdefaultencoding&submit=search&status=-1%2C1%2C2%2C3

[guido1]: https://mail.python.org/pipermail/python-dev/2009-August/091362.html

[guidoenv]: https://mail.python.org/pipermail/python-dev/2009-August/091418.html

[loewis]: http://bugs.python.org/issue1052098#msg54297

[will_break]: http://python.6.x6.nabble.com/Proposed-downstream-change-to-site-py-in-Fedora-sys-defaultencoding-tc1891360.html#a1891499

[why_ascii]: http://article.gmane.org/gmane.comp.python.devel/110071

[armin1]: http://lucumr.pocoo.org/2014/5/12/everything-about-unicode/

[pep20]: https://www.python.org/dev/peps/pep-0020/

[brandon]: https://www.youtube.com/watch?v=z9Hmys8ojno

[death]: https://www.youtube.com/watch?v=LAJQOeCSs_o

[loewis_evil]: https://mail.python.org/pipermail/python-dev/2009-August/091405.html

[lemburg_cache]: https://mail.python.org/pipermail/python-dev/2009-August/091406.html

[dive_into]: http://www.diveintopython.net/xml_processing/unicode.html

[codecs]: https://docs.python.org/2/library/codecs.html#codecs.open

[redis]: https://github.com/andymccurdy/redis-py/blob/2.10.2/redis/client.py#L398

[unipain]: http://nedbatchelder.com/text/unipain/unipain.html#35

[crawler]: https://github.com/meibenjin/GoogleSearchCrawler/blob/master/gsearch.py

[noprint]: https://github.com/meibenjin/GoogleSearchCrawler/issues/3

[ghglad1]: https://github.com/seecr/weightless-core/issues/2

[ghglad2]: https://github.com/ajs124/autokey/issues/165

[ghglad3]: https://github.com/joeyespo/grip/issues/86

[holo]: https://github.com/ioam/holoviews/issues/29

[laggards]: http://sealedabstract.com/rants/python-3-is-fine/

[nick]: http://python-notes.curiousefficiency.org/en/latest/python3/binary_protocols.html

[fedora]: 
https://fedoraproject.org/wiki/Features/PythonEncodingUsesSystemLocale

[mytable]: http://stackoverflow.com/a/29558302/4583360

[fedthread]: http://thread.gmane.org/gmane.comp.python.devel/109914

[fedresume]: http://article.gmane.org/gmane.comp.python.devel/109970

[openhub]: http://code.openhub.net/search?s=setdefaultencoding

[github]: https://github.com/search?utf8=%E2%9C%93&q=setdefaultencoding+language%3APython+utf-8&type=Code&ref=searchresults

[ghissues]: https://github.com/search?l=python&o=desc&q=setdefaultencoding+language%3APython+utf-8&ref=searchresults&s=updated&type=Issues&utf8=%E2%9C%93

[hgipyth]: https://github.com/ipython/ipython/pull/252

[ian]: http://www.ianbicking.org/illusive-setdefaultencoding.html

[fassen]: http://blog.startifact.com/posts/older/changing-the-python-default-encoding-considered-harmful.html

[linebreakbrown]: http://i.stack.imgur.com/FFxEA.jpg


[unipic]: http://i.stack.imgur.com/krQhC.png


[stbtr]: http://i.stack.imgur.com/4VTGj.png
