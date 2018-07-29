# Exec Shellcode
The first payload I have choosen to analyse is the 'exec' payload which allows the execution of a single command.

I created the shellcode using msfvenom like this:
```
msfvenom -p linux/x64/exec CMD=whoami -f c
```

As you can see by the CMD paramaeter I create a shellcode that does nothing else but executing the 'whoami' command.

## Testmodule
Next I placed the resulting char array in a C program:

```
    unsigned char shellcode[] =
    "\x6a\x3b\x58\x99\x48\xbb\x2f\x62\x69\x6e\x2f\x73\x68\x00\x53"
    "\x48\x89\xe7\x68\x2d\x63\x00\x00\x48\x89\xe6\x52\xe8\x07\x00"
    "\x00\x00\x77\x68\x6f\x61\x6d\x69\x00\x56\x57\x48\x89\xe6\x0f"
    "\x05";

    (*(void(*)()) shellcode)();
```

and compiled as follows:

```
 gcc -m64 -z execstack -fno-stack-protector Test.c -o Test -no-pie
```

## Analysis
 **tl;dr**: The shellcode basically does the following

 ```
 execve("/bin/sh", ["/bin/sh", "-c", "whoami"], 0);
 ```

 As we can see it starts /bin/sh with the -c command. The -c command tells the started SH not to be interactive, but to execute one single command ('whoami' in this case) and then exit.

It is important to understand that the second parameter is not a single string, but an array of  char pointers. The last parameter can be left blank as it would only specify environment variables that we dont need here.

The shellcode can be explained in 4 steps:

 1. Prepare RAX with Execve Syscall and null out rdx
     ```
     push 0x3b
     pop
     cdq ; clear out rdx so that no environment variables are given
     ```

 2. Prepare RDI ( first parameter )
     ```
     movabs rbx,0x68732f6e69622f ; put '/bin/sh' in rbx
     push rbx
     mov rdi, rsp ; rdi now points to /bin/sh
     ```

3. Prepare argv on the stack
    ```
    push   0x632d ; push "-c"
    mov rsi, rsp ; rsi now points to -c.
    push rdx ; Place a nullbyte on the stack
    call   0x7fffffffdd57 ; The saved RIP on the stack is not actually an instruction, but 'whoami'. Which is on top of the stack after the call.
    push rsi ("-c")
    push rdi ("/bin/sh")  ;
    ```

   The stack now looks like this:
   ```
    +----------------------+
    |                      |
    |       /bin/sh        |
    |                      |
    +----------------------+
    |                      |
    |        -c            |
    |                      |
    +----------------------+
    |                      |
    |       whoami         |
    |                      |
    +----------------------+
   ```



4. Prepare RSI and do the syscall

    ```
    mov rsi,rsp ; RSI points to argv[] on the stack.
    syscall
    ```

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert

Student ID: SLAE64-1581
