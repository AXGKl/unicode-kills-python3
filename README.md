# unicode-kills-python3
Arguments For Py2 DefaultEncoding UTF-8 and Against the Py3 Paradigm That Text Equals Unicode


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

Since many (especially younger developers) don't even understand the problem any more, i.e. why the switch _could_ be harmful, here some background information about

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
