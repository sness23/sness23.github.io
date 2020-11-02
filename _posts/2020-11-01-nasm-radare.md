Recently watched a youtube tutorial for assembly

I had actually never written a program in assembly so I thought it might be fun
to try.

This is the hello world program in nasm.  nasm is an assembler that takes
a program written in a specific syntax and converts it into binary format.

ld is a linker that makes a program that can be run.

hello_world.asm

    ;; hello_world.asm
    ;;
    ;; Author: sness
    ;; Date: 31-Oct-2020

    global _start

    section .text:

    _start:
        mov eax, 0x4
        mov ebx, 1
        mov ecx, message
        mov edx, message_length
        int 0x80

        mov eax, 0x1
        mov ebx, 0
        int 0x80

    section .data:

    message:     db "Hello World!", 0xA
    message_length: equ $-message


Makefile

    hello_world: hello_world.o
        ld -m elf_i386 -o hello_world hello_world.o

    hello_world.o: hello_world.nasm
        nasm -f elf32 -o hello_world.o hello_world.nasm

You can look at this in hexl mode in emacs, it is slightly informative you can
see things like the hello world string

    00000190: 4865 6c6c 6f20 576f 726c 6421 0a00 0000  Hello World!....
    000001a0: 002e 7465 7874 3a00 2e64 6174 613a 002e  ..text:..data:..
    000001b0: 7368 7374 7274 6162 002e 7379 6d74 6162  shstrtab..symtab
    000001c0: 002e 7374 7274 6162 002e 7265 6c2e 7465  ..strtab..rel.te
    000001d0: 7874 3a00 0000 0000 0000 0000 0000 0000  xt:.............

A better program to look at this with is radare2 or Cutter


