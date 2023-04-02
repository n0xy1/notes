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

Perhaps we can manipulate this overflow to point back to the top of our stack, specifically targeting `0x7fffffffec40`

For this to work whats the ascii char for 0x40?

![](_attachments/Pasted%20image%2020230402100619.png)

Change the payload to `A * 126 + @`

![](_attachments/Pasted%20image%2020230402100830.png)`


However this time the stacks been shifted.. 