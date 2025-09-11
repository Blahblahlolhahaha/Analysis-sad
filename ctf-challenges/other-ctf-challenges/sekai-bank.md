# Sekai Bank

### Challenge Description

{% code overflow="wrap" %}
```
Hi I heard my friend has registered herself for a new bank account! I wonder if the bank system will be secure(I managed to get the program and encrypted account details, but something seems to be missing...)

After decrypting the json file, simply login with the following details: 
Username: Mizuki Password: sadge
```
{% endcode %}

### Solution

The following 2 files were provided  for the challenge:

<div align="left"><figure><img src="../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure></div>

bank.json is just full of junk stuff which implies it might be encrypted:

<figure><img src="../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

While banky is just an X64 ELF file:

<figure><img src="../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

When the executable is first ran, the following error is shown:

<div align="left"><figure><img src="../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure></div>

As there's nothing more to look around, its time to disassemble the file!

#### Disassembling bank

Here I will be using cutter, a free reverse engineering tool that is built upon radare! Feel free to use any other debugging tools/disassemblers such as IDA to disassemble the executable! Here, I used the aaaa analysis option.

First the entry point is disassembled which shows that libc\_start\_main is being called, with the main function is being passed into the first argument:

<div align="left"><figure><img src="../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure></div>

From here we can access main, which is at 0x6aa0!

<figure><img src="../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

Here the auto analysis auto name a function to sub.bank.json before printint out what looks like the main menu, maybe it does something to the json file before showing the main menu?

#### JSON File Decryption:

The following is the disassembly for the start of sub.bank.json\_5ad0:

<div data-full-width="true"><figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure></div>

The program attempts to open the file with fopen, exiting if it fails

If successful, the program will get the file size of the file and save to file\_size, where a buffer will be created with&#x20;

```c
calloc(1,file_size)
```

<figure><img src="../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

There it will jump to another subroutine sub.fread\_5c0b, as identified on cutter!

When decompiled, it shows that the fread is being used to read from the file into the buffer created:\


<figure><img src="../../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

After the file is read, a suspicious subroutine secret\_key\_35750 is being called with a reference being passed into it. This reference is then passed into EVP\_CIPHER\_CTX\_new\_35a50, where EVP is a popular interface provided by the OpenSSL library  to perform cryptographic operation. The buffer and file size is also being passed into this subroutine, which makes it plausible that decryption might take place here! A quick look into the function confirms this:

<figure><img src="../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

Here it inits the Cipher Context, and uses AES CBC with a 256 bit key passed from param\_1 with the IV stored in 0x3dc80. The cipher will then be used to decrypt the buffer in param4! This means that the bank.json file is being decrypted from here! Hence the key is being retrieved in secret\_key\_35750!

#### Getting the key:

The following is the decompilation of secret\_key\_35750:

<figure><img src="../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

Here the program tries to open secret.key, which will stop the program if unsuccessful which is why our program crashed!

For secret.key, it will read 2 bytes from secret.key and run them through the first byte through a function 16 times and the 2nd byte through another function 16 times(which will be shown below) and the results will be saved to a 32 bytes buffer which will be stored in param1 which will be used as the key for decryption.

<div align="left"><figure><img src="../../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure></div>

<div align="left"><figure><img src="../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure></div>

Here, these two functions take in an integer which will be run through an algorithm known as a hashing algorithm. This means that the key is generated from 2 bytes which makes it brute forceable!

Here is the bruteforce script:

```python
import subprocess


for i in range(65536):
    key = i.to_bytes(2,byteorder="little",signed=False)
    with open("secret.key","wb") as f:
        f.write(key)
    process = subprocess.Popen("./banky", text=True, stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    while True:
        line = process.stdout.readline()
        if not line:
            break
        print(line)
    while True:
        try:
            line = process.stderr.readline()
            if not line:
                break
            print(line)
        except UnicodeDecodeError:
            break
    if process.returncode == 0:
        break
```

Here it runs through 0 - 65535, which is the larges integer 2 bytes can hold and saving it to secret.key and running the program. Once the key is correct, ideally the main menu should pop out instead of "An error occured"

<figure><img src="../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

Yayyy, the key is obtained:

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

With the credentials provided in the description, the flag can be obtained:

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

Flag:

```
Malaysia{kUch1Ng}
```
