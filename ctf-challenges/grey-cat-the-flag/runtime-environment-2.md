# Runtime Environment 2

### Challenge Description

This time it HAS to be harder.

MD5 (hasbeen.tar.gz) = 46ff7d24975679901d8f8d769e567b09

* rootkid

### Challenge Details

Oh god, the moment I see the challenge description, I knew I'm in for a hell ride. This challenge took me 3 painful days to solve (ofc I gave up a few times to go solve other challenges XDDDDDDD)

Why? I realised the challenge was written in Haskell, a language that I did not know how to read/write, much less understand how GHC compiles Haskell code into executables. The way functions work is pAin but oh wells I decided to bite the nail and try to solve it.

Like RE 1, we were given a binary and an encoded file. However, this time the encoded file is not readable:

![jeez](<../../.gitbook/assets/image (22) (1).png>)

And by looking at the disassembler, I can confirm that the program is written in Haskell due to the presence of `hs_main` being called in `main`:

![](<../../.gitbook/assets/image (23) (1).png>)

When i ran the program, I realised that the program gave different outputs for the same input, meaning that there is something else involved during the encoding process:

![](<../../.gitbook/assets/image (7).png>)

From there, I ran hsdecomp on the program to attempt to decompile the program back to the original haskell code and figure out what happens during the program:

```haskell
Main_main_closure = >>= $fMonadIO
    (fmap $fFunctorIO unpack getLine)
    (\loc_4232104_arg_0 ->
        >>= $fMonadIO
            getCurrentTime
            (\loc_4231968_arg_0 ->
                >>= $fMonadIO
                    ($
                        !!ERROR!!
                        ($
                            !!ERROR!!
                            ($
                                (.
                                    (case $fRealFracFixed $fHasResolutionTYPEE12 of
                                        loc_4228312_case_tag_DEFAULT_arg_0@_DEFAULT -> floor <index 0 in loc_4228312_case_tag_DEFAULT> $fIntegralInt
                                    )
                                    (. nominalDiffTimeToSeconds utcTimeToPOSIXSeconds)
                                    loc_4231968_arg_0
                                )
                            )
                        )
                    )
                    (\loc_4230768_arg_0 -> >>= $fMonadIO ($ !!ERROR!! (zipWith !!ERROR!! (map !!ERROR!! loc_4232104_arg_0) 
                    (map !!ERROR!! loc_4230768_arg_0))) (\loc_4229920_arg_0 -> $ putStr ($ pack (map !!ERROR!! loc_4229920_arg_0))))
            )
    )

loc_4229392 = \loc_4229392_arg_0 loc_4229392_arg_1 -> : loc_4229128 (loc_4229392 $fIntegralInt32 loc_4229128)
loc_4229128 = xor $fBitsInt32 loc_4228928 (shiftL $fBitsInt32 loc_4228928 (I# 5))
loc_4228928 = xor $fBitsInt32 loc_4228728 (shiftR $fBitsInt32 loc_4228728 (I# 17))
loc_4228728 = xor $fBitsInt32 loc_4228568 (shiftL $fBitsInt32 loc_4228568 (I# 13))
loc_4228568 = fromIntegral loc_4229392_arg_0 $fNumInt32 loc_4229392_arg_1
```

At first, I did not really understand the decompiled code as I was not really good in haskell. However, at first glance, I can confirm that:

* Program gets current time to be used for the encryption process
* &#x20;Input/time will be xored with each other&#x20;
* Bit shifting is also involved.

After putting some breakpoints in IDA and a painful debugging process, I finally managed to figure out the encryption function:

```python
import os
import ctypes
import time


def encrypt(time:int,letter):
    time = (time ^ (time << 13)) & 0xffffffff
    if(ctypes.c_long(time).value < 0):
        time = time | 0xffffffff << 32
    time = (time ^ (time >> 17)) & 0xffffffff
    print(hex(time))
    time = (time ^ (time << 5)) & 0xffffffff
    print(hex(time))
    letter = (letter ^ (time)) & 0xff
    return time,chr(letter)

plaintext = input()
time = int(time.time())
ciphertext = ""
for x in sad:
    time,letter = encrypt(time,x)
    cry+= letter
   
print(cry)

```

In essence, the program will take input from the user and current time whereby the time goes through the following process before xoring with a letter of the user:

* Left shift by 13 and keep signed value of the result
* Right shift by 17
* Left shift by 5

The resulting time will be saved for the next letter until the entire string is encrypted.

### Challenge Solution:

To solve the challenge, simply get the modified time of the file and run the encryption process again since it is a simple xor function that can be undone by xoring the encrypted output:

```python
import os
import ctypes

def encrypt(time:int,letter):
    time = (time ^ (time << 13)) & 0xffffffff
    if(ctypes.c_long(time).value < 0):
        time = time | 0xffffffff << 32
    time = (time ^ (time >> 17)) & 0xffffffff
    time = (time ^ (time << 5)) & 0xffffffff
    letter = (letter ^ (time)) & 0xff
    return time,chr(letter)
# def decrypt(time:int,letter:int):



sad = open("challenge.bin","rb").read(

og_time=  int(os.path.getmtime("challenge.bin")) 
time = og_time
cry = ""
for x in sad:
    time,letter = encrypt(time,x)
    cry+= letter
   
print(cry)

```

Flag: grey{Funct1on41\_P4rad1s3\_iZ\_Fun}
