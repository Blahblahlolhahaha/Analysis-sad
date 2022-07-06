# Runtime Environment

### Challenge Description

GO and try to solve this basic challenge.

FAQ: If you found the input leading to the challenge.txt you are on the right track

MD5 (gogogo.tar.gz) = 5515f1c3eee00e4042bf7aba84bbec5c

* rootkid

### Challenge Details

In this challenge, we are given a binary and an encoded file:

```
GvVf+fHWz1tlOkHXUk3kz3bqh4UcFFwgDJmUDWxdDTTGzklgIJ+fXfHUh739+BUEbrmMzGoQOyDIFIz4GvTw+j--
```

The challenge was evidently written in go as deduced from the challenge description and confirmed when running `file` on it which shows it GO build id:

![](<../../.gitbook/assets/image (18) (1).png>)

For this challenge, I used a set of IDA plugns found in this github repository which helps me identify go library functions and user-defined functions:&#x20;

[https://github.com/sibears/IDAGolangHelper](https://github.com/sibears/IDAGolangHelper)

![](<../../.gitbook/assets/image (14).png>)

Upon running, the binary takes input from the user and prints out the encoded string:

![](<../../.gitbook/assets/image (21) (1).png>)

Upon running it a few times with similar input as shown above, I realised that it feels like a base64 encoded string however, a normal base64 encoder says otherwise:

![pAin](<../../.gitbook/assets/image (12) (1).png>)

So i decided to look at the encoding function for the binary and found a 64 characters long string that have no repeating characters and a while loop that runs for 3 times and does it a final time after the loop:&#x20;

![The custom base 64 string](<../../.gitbook/assets/image (20) (1).png>)

![While loop containing base64 operations](<../../.gitbook/assets/image (17) (1).png>)

![Code after loop which is simply the same function](<../../.gitbook/assets/image (11) (1).png>)

This means that the challenge creator used a  custom base64 string :`NaRvJT1B/m6AOXL9VDFIbUGkC+sSnzh5jxQ273d4lHPg0wcEpYqruWyfZoM8itKe` and `-` as the padding and ran the encoder 4 times!

### Challenge Solution

Simply put the encoded chars through a base64 decoder 4 times using the custom base64 string and padding:

![](<../../.gitbook/assets/image (19).png>)

![](<../../.gitbook/assets/image (10) (1).png>)

![](<../../.gitbook/assets/image (13).png>)

![](../../.gitbook/assets/image.png)

Flag: grey{B4s3d\_G0Ph3r\_r333333}
