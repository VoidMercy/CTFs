# Pwn 100 - Bad Seed

Here, we are given a netcat server, ```nc 40.76.33.15 31338``` and a 64 bit binary. Here is the pseudocode from IDA Hexrays:

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  unsigned int v3; // eax
  int v5; // [rsp+4h] [rbp-Ch]
  unsigned __int64 v6; // [rsp+8h] [rbp-8h]

  v6 = __readfsqword(0x28u);
  setbuf(stdin, 0LL);
  setbuf(_bss_start, 0LL);
  printf("Something something time is an illusion? ", 0LL);
  __isoc99_scanf("%d", &v5);
  v3 = time(0LL);
  srand(v3);
  if ( rand() != v5 )
    return 0;
  puts("You are a true master!\n");
  system("cat flag");
  return 0;
}
```

So, reading and reversing the code, we see that it receives an integer input from the user with scanf %d, seeds srand with time(0), which is the current second I believe, and if our input is the same as the rand() number it generates, then we get the flag.

The solution is simply to use the same seed the program uses, then provide to the server first number generated.

Here is my C code to generate a random number seeded by the current time in seconds:

```C
int main() {
	srand(time(0LL));
	printf("%d", rand());
}
```

Now here is the python code to interact with both the server, and call this C program to generate a random number:

```
from pwn import *
import os
r = remote("40.76.33.15", 31338)
os.system("./rand > res")
sice = open("res", "r").read()
r.sendline((sice))
r.interactive()
```

This program contacts the server, generates a random number and pipes the output into a file, then reads the number from that file into python, then sends that number, then whala! We have our flag!

```
alex@WINDOWS-0A3VRFK:/mnt/c/Users/alex/Desktop/CTF$ python badseed.py
[+] Opening connection to 40.76.33.15 on port 31338: Done
[*] Switching to interactive mode
Something something time is an illusion? You are a true master!

b0ctf{c0ach_tuck3r_sh0ws_how_t0_t1me_4_dbbl3_l3g}

https://www.youtube.com/watch?v=gpgzIR4Y7Bk
https://www.youtube.com/watch?v=oFiDcazicdk
[*] Got EOF while reading in interactive
```

## Flag
### b0ctf{c0ach_tuck3r_sh0ws_how_t0_t1me_4_dbbl3_l3g}
