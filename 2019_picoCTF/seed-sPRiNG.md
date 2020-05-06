## Writeup for 2019 picoCTF: seed-sPRiNG
Binary Exploitation, 350 points

### Problem
> The most revolutionary game is finally available: seed sPRiNG is open right now! [seed_spring](https://2019shell1.picoctf.com/static/6c73aa391245deeea00aecc8d5a78f88/seed_spring). Connect to it with nc 2019shell1.picoctf.com 4160. 

### Ghidra:
The file seed_spring given is a binary file. To decompile it to C, we could use Ghidra, a reverse engineering tool. Import seed_spring then locate the main function (click on main on the second pane on the screen's left). 

Here's what Ghidra outputs:
```
/* WARNING: Function: __x86.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */

undefined4 main(void)

{
  uint local_20;
  uint local_1c;
  uint local_18;
  int local_14;
  undefined *local_10;
  
  local_10 = &stack0x00000004;
  puts("");
  puts("");
  puts("                                                                             ");
  puts("                          #                mmmmm  mmmmm    \"    mm   m   mmm ");
  puts("  mmm    mmm    mmm    mmm#          mmm   #   \"# #   \"# mmm    #\"m  # m\"   \"");
  puts(" #   \"  #\"  #  #\"  #  #\" \"#         #   \"  #mmm#\" #mmmm\"   #    # #m # #   mm");
  puts(
      "  \"\"\"m  #\"\"\"\"  #\"\"\"\"  #   #          \"\"\"m  #      #   \"m   #    #  # # #    #"
      );
  puts(" \"mmm\"  \"#mm\"  \"#mm\"  \"#m##         \"mmm\"  #      #    \" mm#mm  #   ##  \"mmm\"");
  puts("                                                                             ");
  puts("");
  puts("");
  puts("Welcome! The game is easy: you jump on a sPRiNG.");
  puts("How high will you fly?");
  puts("");
  fflush(stdout);
  local_18 = time((time_t *)0x0);
  srand(local_18);
  local_14 = 1;
  while( true ) {
    if (0x1e < local_14) {
      puts("Congratulation! You\'ve won! Here is your flag:\n");
      get_flag();
      fflush(stdout);
      return 0;
    }
    printf("LEVEL (%d/30)\n",local_14);
    puts("");
    local_1c = rand();
    local_1c = local_1c & 0xf;
    printf("Guess the height: ");
    fflush(stdout);
    __isoc99_scanf(&DAT_00010c9a,&local_20);
    fflush(stdin);
    if (local_1c != local_20) break;
    local_14 = local_14 + 1;
  }
  puts("WRONG! Sorry, better luck next time!");
  fflush(stdout);
                    /* WARNING: Subroutine does not return */
  exit(-1);
}
```

### Solution:

Quick search on Google tells us that rand() in C produces a replicable sequence of random numbers if we know the seed it uses. The seed used is set by srand().  

Note the source code `local_18 = time((time_t *)0x0); srand(local_18);` So our exploit script just needs to set `time((time_t *)0x0` or `time(0)` as the seed. 

1. Here is the **exploit script** seed.c:
```
#include <stdio.h>
#include <time.h>
#include <stdlib.h>

int main() {
        unsigned int n;
        unsigned int seed = time((time_t *)0x0);
        srand(seed);
        for (int i=0; i<30; i++) {
                n = rand() & 0xf;
                printf("%u\n", n);
        }
}
```
2. scp seed.c to the server so you could get the exact time that is on the server (because `srand()` in our problem depends on `time()`). Then gcc to compile seed.c. 
3. Pipe the executable to the game using `./seed | nc 2019shell1.picoctf.com 4160`  

**Flag: picoCTF{pseudo_random_number_generator_not_so_random_24ce919be49576c7df453a4a3e6fbd40}**
