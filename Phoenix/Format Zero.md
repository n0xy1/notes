This level introduces format strings, and how attacker supplied format strings can modify program execution.

**Hints**

-   `man 3 printf`
-   [Exploiting Format String Vulnerabilities](https://www.google.com/search?q= "exploiting+format+string+vulnerabilities")

### Source code

```c
/*
 * phoenix/format-zero, by https://exploit.education
 *
 * Can you change the "changeme" variable?
 *
 * 0 bottles of beer on the wall, 0 bottles of beer! You take one down, and
 * pass it around, 4294967295 bottles of beer on the wall!
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

  if (locals.changeme != 0) {
    puts("Well done, the 'changeme' variable has been changed!");
  } else {
    puts(
        "Uh oh, 'changeme' has not yet been changed. Would you like to try "
        "again?");
  }

  exit(0);
}
```

# Solution

Printf has some funny quirks with unsanitised input you can type `%x` and it spits out memory addresses and things. Do this within gdb to poke around a bit

![](_attachments/Pasted%20image%2020230402120911.png)

so `%x` does 8 characters.. to fill the buffer and overwrite changeme we could do %x%x%x%x%x 

lets have a look


or just a single %32x

![](_attachments/Pasted%20image%2020230402121135.png)