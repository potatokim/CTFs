## Writeup for 2018 picoCTF: strings
General Skills, 100 points

### Problem
> Can you find the flag in this [file](https://2018shell.picoctf.com/static/22ef75638cf590f5fad3db45463883bb/strings) without actually running it? You can also find the file in /problems/strings_2_b7404a3aee308619cb2ba79677989960 on the shell server.

Funny I didn't read the hint and was playing around with `xxd` so here's a solution using it.  
The first command reveals the first part of the flag, as well as its location `0001aae0`. This means to find the remaining parts of the flag, we just need to find the locations right after. 
### Solution:
```
xxd strings | grep "pico"
0001aae0: 7069 636f 4354 467b 7354 7249 6e67 535f  picoCTF{sTrIngS_
--- --- --- 
xxd strings | grep 0001aaf0
0001aaf0: 7341 5665 535f 5469 6d65 5f33 6637 3132  sAVeS_Time_3f712
--- --- ---
xxd strings | grep 0001ab00
0001ab00: 6132 387d 0000 0000 0000 0000 0000 0000  a28}............
```
**Flag: picoCTF{picoCTF{sTrIngS_sAVeS_Time_3f712a28}**
