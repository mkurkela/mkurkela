---
layout: post
title: Testing challenge
---

The IgProf profiler hooks certain functions at the begin of the execution so we
can run the profiling code before the target enter a called function and
after returning. The hooking is done by replacing first CPU instructions with
jmp instruction that leads the executing to our hook function. From the hook
function we then call the original function.

To insert the jmp instruction we have to save the original instructions. For
that it is must to know instruction length. In x86, instructions are variable
length varying from 1 byte to 15 byte. Unfortunately, just knowing the
instruction length is not enough in this case. Some instructions are not
relocatable, at least not withouth patching them. For example, jmp four byte
offset or loading some value located %rip + some offset. Most of these
instructions we can patch to point to correct locations. Some are more
problematic like one byte conditional jumps (opcodes 0x70-0x7f).

Before there were several if statments to spot the instructions that are needed to
instrument memory (de)allocating functions and some other functions that are
needed to hook for performance profiler. Last summer I got a task to make the
instruction lenght decoder more general. My decoder was able to detect most
common instructions and added patches to position independent instructions.

Now I was adding support for more instructions and I ended up to redesign the
decoder. At the moment I am testing the first version of the decoder but how?
First, I made a simple test software where you enter a function name and the
decoder tells how long are first instructions and print their hex code. Not very
practical but the test showed that it seems to decode instruction lengths
correctly. Next, I tried the decoder with IgProf. With little fix it worked well.

There are still many cases that can fail. I wrote a test program that goes
trough all symbols in a given library. Well, the test program ran and decoded
many instructions. At first it looks nice but how I can be sure that the result
is correct? If there is an error, it is likely that after there will be more
error cases. I ended up to notice two mistakes from my lookup table and also
noticed that GS and FS override prefixes were not handled correctly.

Still I there can be many problems. Next, I enhanced the test software so that it
uses IgHook to hook all functions in the library. It prints functions and their
first instruction bytes into errors.txt file, if they are marked as function that
I can not relocate. After that, it tries to hook functions, which instruction lengths
were detected. The instruction replacement should work in all cases. Thing that
can fail is when I am calling the hooked function from my hook function. So lets
call them and see if it crash! But I do not have arguments and I do not know
what are those functions doing, when I am just getting them in order from nm
output. Anyway I tried it with some libraries. If I was getting a SIGSEGV the test
program wrote the function name into error file and continued to next function.

Got to invent something better tomorrow...
