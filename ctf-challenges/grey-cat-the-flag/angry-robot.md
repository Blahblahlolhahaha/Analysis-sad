# Angry Robot

### Challenge Description

You have entered a top secret robot production facilities and your clumsy friend tripped the alarm. You and your friends are about to be "decontaminated".

Luckily, you have unpacked the authentication firmware during previous reconaissance. Can you use them to override the decontamination process?



MD5 (authentication.tar.gz) = 965b5a18735af4bbfa81879e2cedc9cc

* rootkid

nc challs.nusgreyhats.org 10523

### Challenge details:

In this challenge, we were given 100 64 bit elf binaries which we need to find the password for&#x20;

![](<../../.gitbook/assets/image (18) (1) (1) (1).png>)

![](<../../.gitbook/assets/image (12) (1) (1) (1).png>)

Here I send it back to a decompiler and found out that all of these files are similar, with the `main` function taking input from `stdin` and passing it to an encode function and comparing it with another string.&#x20;

![Checking function](<../../.gitbook/assets/image (1) (1).png>)

![Encoding function](<../../.gitbook/assets/image (8) (1) (1).png>)

Basically, the 2 functions can be simplified into the following python script:

```python
yes = (i * x % mod + x + i) % 73 + 48
wheee = (check * x % mod + x + check) % 73 + 48
if(yes == check and wheee == check1):
    return chr(x)
```

where

* i is the current index
* x is input\[i]
* mod is a special number used that is different in each file (in the example above it will be 2784)
* check is str1\[i]&#x20;
  * In the example above, str1= "LwLRcbp34t9glXCPW\[VHj;@7d"
* check1 is str2\[i]
  * In the example above, str2= "F]ZNlR\`?kagq:tOF0hlnjI8Ft"

So to solve this challenge, we will have to find the strings and mod used for each file and brute force the password that satisfies each file.

However, this cannot be achieved by just reading the file as the string length for each file is different, hence another way is needed. In this case, I decided to use radare2 to find the strings and use a script to figure out the strings used for each file and read `mod` which can be found at a specific offset.

### Solution

In radare, i did the following steps:

* Put a breakpoint at the encode function. (address can be found using the disassembler)

![](<../../.gitbook/assets/image (5).png>)

*   Run the code by using dc and then read rsp when the program reaches the breakpoint:



![](<../../.gitbook/assets/image (4).png>)

Here we can see the strings! But to only just read the string we will have to use `ps @rsp+offsetOfString` instead to read the strings. The offsets for the strings are 40 and 72 respectively:

![](<../../.gitbook/assets/image (16) (1) (1).png>)

Voila! Now we just have to read `mod` from the file which is 2 bytes long and usually contains at either offset 0x11c2 or 0x11c4 (except for one file reeeeeee)

![For this file it is at 0x11c2!](<../../.gitbook/assets/image (15) (1) (1).png>)

Then after getting the required values for each file, the only thing left is to brute force the password. Since the input has to be ASCII characters between 0x20 and 0x7f here's the script to brute force:

```python
def brute_force(i:int,check:int,check1:int,mod:int):
    for x in range(0x20,0x7f):
        yes = (i * x % mod + x + i) % 73 + 48
        wheee = (check * x % mod + x + check) % 73 + 48
        if(yes == check and wheee == check1):
            return chr(x)
```

From there, I used r2pipe to script the radare2 debugger to automatically get the password for each file and then go ahead and connect to the challenge server:

```python
import json
import r2pipe,os
import socket

sock = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
yes = {}
def brute_force(i:int,check:int,check1:int,mod:int):
    for x in range(0x20,0x7f):
        yes = (i * x % mod + x + i) % 73 + 48
        wheee = (check * x % mod + x + check) % 73 + 48
        if(yes == check and wheee == check1):
            return chr(x)

def get_value(filename:str):
    r = r2pipe.open("./crack/" + filename)
    r.cmd("db 0x401176; dor stdin=sad.txt;ood; dc")
    
    #get strings used for each file
    sad = r.cmd("ps @rsp+40").replace("\n","").replace("\\\\","\\")
    cry = r.cmd("ps @rsp+72").replace("\n","").replace("\\\\","\\")

    with open("./crack/" + filename,"rb") as f:
        cryign = f.read()
        #get mod
        mod = int.from_bytes(cryign[0x11c2:0x11c4],"little")
        if(filename == '6e0f1ad67d39f86889323f47d4d1f57f4fc76a19073262edb348792555bd5721'):
            mod = 2049
        elif(mod > 3000):
            mod = int.from_bytes(cryign[0x11c4:0x11c6],"little")
        
        #get password
        googooo = ""
        for i in range(len(sad)):
            googooo += brute_force(i,ord(sad[i]),ord(cry[i]),mod)
        yes[filename] = googooo
    r.cmd("exit")

#iterate through all files
for root,dirs,name in os.walk("./crack"):
    for x in  name:
        get_value(x)

#connect to remote server to solve challenge
sock.connect(("challs.nusgreyhats.org",10523))

while True:
    sock.recv(1024)
    sock.send("y\n".encode())
    
    while True:
        bad = sock.recv(1024).decode()
        print(bad)
        bad = bad.replace("Password required for challenge code: ","").replace("\n","")
        sock.send(yes[bad].encode() + b"\n")
```

![](<../../.gitbook/assets/image (6) (1).png>)

Flag: grey{A11\_H4il\_SkyN3t}
