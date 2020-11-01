Recently watched a youtube tutorial for assembly

I had actually never written a program in assembly so I thought it might be fun
to try.

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
