# Collection of Crazy Py3 Stuff Caused By Trying to Decode W/O Need #



## Std. Out Encoding Guessed From Environ Settings ##

Since they have decoded the input they must encode the output - but have no clue about which encoding to apply.
So they look into environ (!)

    https://bugs.python.org/issue19846

    klessinger@MacBook-Pro ~ ⚘ cat u.py                                    21:19:14
    import sys; print (sys.argv[1])
    klessinger@MacBook-Pro ~ ⚘ python u.py 'Jose<0301>'                    21:19:21
    José
    klessinger@MacBook-Pro ~ ⚘ python3 u.py 'Jose<0301>'                   21:19:32
    José
    klessinger@MacBook-Pro ~ ⚘ export LC_ALL=C                             21:19:43
    klessinger@MacBook-Pro ~ ⚘ python3 u.py 'Jose<00cc><0081>'           21:19:49
    Traceback (most recent call last):
    File "u.py", line 1, in <module>
        import sys; print (sys.argv[1])
    UnicodeEncodeError: 'ascii' codec can't encode character '\u0301' in position 4: ordinal not in range(128)
    klessinger@MacBook-Pro ~ ⚘ python u.py 'Jose<0301>'                    21:20:21
    José



Author: Nick Coghlan (ncoghlan) * (Python committer)
At the moment, setting "LANG=C" on a Linux system fundamentally breaks Python 3, and that's not OK.

But later:
Thanks Victor - I now agree that trying to guess another encoding is a bad idea, and that enabling surrogateescape for the standard streams under the C locale is a better way to go.
ergebniss: print sys.argv[1] bricht immer noch in python wenn LC_ALL = C
crazy, sie sind zu verbohrt zu sehen dass sie's kaputt machen.

victor stinner, telling Nick to educate all linux users via docu:

> At the moment, setting "LANG=C" on a Linux system fundamentally breaks Python 3, and that's not OK.

Getting ASCII filesystem encoding is annoying, but I would not say
that it fundamentally breaks Python 3. If you want to do something,
you should write documentation explaining how to configure properly
Linux.


## Str Unusable ##

http://bugs.python.org/issue24019

