# Narnia

Writeup for the wargame Narnia on [OverTheWire](https://overthewire.org/wargames/narnia/).

As the web states, all you need is under `/narnia`. There you will find a bunch
of dangerous binaries (with active SUID) along with the source code:

```bash
narnia0@narnia:~$ ls -al /narnia/
total 116
drwxr-xr-x  2 root    root    4096 Aug 26  2019 .
drwxr-xr-x 27 root    root    4096 Aug 26  2019 ..
-r-sr-x---  1 narnia1 narnia0 7456 Aug 26  2019 narnia0
-r--r-----  1 narnia0 narnia0 1229 Aug 26  2019 narnia0.c
-r-sr-x---  1 narnia2 narnia1 7296 Aug 26  2019 narnia1
-r--r-----  1 narnia1 narnia1 1021 Aug 26  2019 narnia1.c
-r-sr-x---  1 narnia3 narnia2 5048 Aug 26  2019 narnia2
-r--r-----  1 narnia2 narnia2 1022 Aug 26  2019 narnia2.c
-r-sr-x---  1 narnia4 narnia3 5676 Aug 26  2019 narnia3
-r--r-----  1 narnia3 narnia3 1699 Aug 26  2019 narnia3.c
-r-sr-x---  1 narnia5 narnia4 5224 Aug 26  2019 narnia4
-r--r-----  1 narnia4 narnia4 1080 Aug 26  2019 narnia4.c
-r-sr-x---  1 narnia6 narnia5 5588 Aug 26  2019 narnia5
-r--r-----  1 narnia5 narnia5 1262 Aug 26  2019 narnia5.c
-r-sr-x---  1 narnia7 narnia6 5948 Aug 26  2019 narnia6
-r--r-----  1 narnia6 narnia6 1602 Aug 26  2019 narnia6.c
-r-sr-x---  1 narnia8 narnia7 6532 Aug 26  2019 narnia7
-r--r-----  1 narnia7 narnia7 1964 Aug 26  2019 narnia7.c
-r-sr-x---  1 narnia9 narnia8 5132 Aug 26  2019 narnia8
-r--r-----  1 narnia8 narnia8 1269 Aug 26  2019 narnia8.c
```

So we are expected to exploit these binaries to gain access to the next levels.
I recommend you to download the source code and have it open while you do the
challenges, as all of them are about auditing C code.

- [narnia0 -> narnia1](narnia0.md)
- [narnia1 -> narnia2](narnia1.md)
- [narnia2 -> narnia3](narnia2.md)
