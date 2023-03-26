
Source code

```C
/*
 * phoenix/stack-zero, by https://exploit.education
 *
 * The aim is to change the contents of the changeme variable.
 *
 * Scientists have recently discovered a previously unknown species of
 * kangaroos, approximately in the middle of Western Australia. These
 * kangaroos are remarkable, as their insanely powerful hind legs give them
 * the ability to jump higher than a one story house (which is approximately
 * 15 feet, or 4.5 metres), simply because houses can't can't jump.
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

char *gets(char *);

int main(int argc, char **argv) {
  struct {
    char buffer[64];
    volatile int changeme;
  } locals;

  printf("%s\n", BANNER);

  locals.changeme = 0;
  gets(locals.buffer);

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

As per the banner, you want to corrupt the locals struct. Specifically the changeme integer.

The binary is located at: `/opt/phoenix/amd64/stack-zero` within your vm.

# Solution

As you can see, the locals structure is defined with a 64-byte buffer, which is declared right before the `changeme` int.  The order that these variables are declared is important. If we manage to overflow the `buffer` variable, it will bleed into the `changeme` variable, changing it's value.

The program sets the `changeme` variable to one BEFORE the user input is saved into the `buffer` with `gets()`. In order to change the `changeme` variable after the program sets its value, we simply have to fill up this buffer with more than 64 characters.

The `gets()` function does not check for input length, and simply takes EVERYTHING you give it and saves it into `locals.buffer`

The easiest way to fill this buffer up would be to spam the keyboard until you win, however for a simple solution I used:

```bash
python -c 'print("A"*64+"B")'
```

This way it'll fill up the buffer, and have an extra "B" in there, which changes the value of `changeme` whilst also giving us familiar hex to examine within the debugger. 
A = 0x41, B = 0x42.


## Checking the solution worked

Whack it open in gdb, step to the main function and you see where it allocates the stack for the locals in the disassembly window:


![[../res/Pasted image 20230319181047.png]]

Asyou can see, I have stepped past the instruction that subtracts 0x60 from $rsp (grows the stack) and now the stack pointer is pointing to the address `0x..e5d0`

This is where the 64 bytes for the `buffer` is, as well as the 4 bytes for the `changeme` int.

The extra space (`0x60` = 96 bytes) could have been allocated by the compiler so it's nice and neat / mapped correctly in memory.. idk yet.

Stepping through the solution after I've overflowed the buffer and corrupted the `changeme` variable shows its now got a B in it:

![[../res/Pasted image 20230319183017.png]]


The upper two arrows point to the `mov` and `test` instructions, these are basically the equivalents of `if somevar == 0`

TEST will set the zero flag if the register is zero, in this case its NOT zero, so the ZF never gets set:

![[../res/Pasted image 20230319183621.png]]

Let the program continue, and theres your answer

![[../res/Pasted image 20230319183726.png]]

# Summary

- The stack is SUBTRACTED to grow. (Grows from higher addresses to lower ones)
- `gets()`  does no checking on input
- GDB (With GEF) is extremely useful for linux binaries, and validating what you're doing.