# Anti Flag
Let's have a look at wha the program does!
```
williloo@WillLaptop:/mnt/c/users/willi/Downloads$ ./anti_flag
No flag for you :(
```
It doesn't apppear to me much...let's tr to run ltrace on it to see if it gives us anything:
```
williloo@WillLaptop:/mnt/c/users/willi/Downloads$ ltrace ./anti_flag
strlen("\320\3005O\f\331#*HK\t\vT<\370\300\345\337\327y\017=\3335\304") = 25
malloc(100)                                                          = 0x55e8411fe2a0
ptrace(0, 0, 1, 0)                                                   = -1
puts("Well done!!"Well done!!
)                                                  = 12
+++ exited (status 0) +++
```
A slightly different output! maybe the binary can tell us a bit more.

## Code
```
00001486  int32_t main(int32_t argc, char** argv, char** envp)

00001486  {
00001492      int32_t argc_1 = argc;
00001495      char** argv_1 = argv;
00001499      int32_t var_24 = 0;
000014c9      malloc((strlen(&data_2011) << 2));
000014c9      
000014f4      if (ptrace(PTRACE_TRACEME, 0, 1, 0) != -1)
00001519          puts("No flag for you :(");
000014f4      else
000014fd          puts("Well done!!");
000014fd      
0000154e      return 0;
00001486  }
```
Interesting code - the ptrace function essentially determines whether there are any monitoring functions running in 
conjunction to the program. This is why when we used ltrace, it outputted 'Well done!' while running it normally gave
us 'No flag for you :('. There doesn't seem to be much to work with.

This is until we take a look at the assembly:
```
000014eb  e8e0fbffff         call    ptrace
000014f0  4883f8ff           cmp     rax, 0xffffffffffffffff
000014f4  7513               jne     0x1509

000014f6  488d3d2e0b0000     lea     rdi, [rel data_202b]  {"Well done!!"}
000014fd  e88efbffff         call    puts
00001502  b800000000         mov     eax, 0x0
00001507  eb44               jmp     0x154d

00001509  817de439050000     cmp     dword [rbp-0x1c], 0x539  {"2.5"}
00001510  7413               je      0x1525  {0x0}

00001512  488d3d1e0b0000     lea     rdi, [rel data_2037]  {"No flag for you :("}
00001519  e872fbffff         call    puts
0000151e  b800000000         mov     eax, 0x0
00001523  eb28               jmp     0x154d

00001525  488b55f8           mov     rdx, qword [rbp-0x8 {var_10}]
00001529  488b4df0           mov     rcx, qword [rbp-0x10 {var_18}]
0000152d  488b45e8           mov     rax, qword [rbp-0x18 {var_20}]
00001531  4889ce             mov     rsi, rcx  {data_2011}
00001534  4889c7             mov     rdi, rax  {data_2004, "2asdf-012=14"}
00001537  e8c3feffff         call    sub_13ff
0000153c  488b45f8           mov     rax, qword [rbp-0x8 {var_10}]
00001540  4889c7             mov     rdi, rax
00001543  e848fbffff         call    puts
00001548  b800000000         mov     eax, 0x0
```
At line 0x14f0 we compare the return value of $rax to -1. If its true, instead of jumping right to printing "No flag for you", there is first a 
comparison between rbp-0x1c and 0x539, which is an obvious false statement, as 0 is stored in rbp-0x1c. This means the program will proceed
to print "No flag for you". But what we've found is a hidden code block! More specifically, we've found a section from 0x1525 which will never
execute normally - perhaps this is where our flag lies.

We can see that the segment of code essentially calls sub_13ff, so this might be where our flag is hidden. We can use pwngdb to help us jump
to this section of code, 0x1525. We can't access the location of these functions directly as the file is protected, so instead, let's have a look 
to see if we can find the function address using 'info functions':
```
0x0000555555555080  __cxa_finalize@plt
0x0000555555555090  puts@plt
0x00005555555550a0  strlen@plt
0x00005555555550b0  __stack_chk_fail@plt
0x00005555555550c0  malloc@plt
0x00005555555550d0  ptrace@plt
```
Unfortunately not...but we do get the address of ptrace, which we can set a breakpoint to run piebase. This essentially finds me the base address 
of the function, and since we have the offset values in Binary Ninja, we can simply add that on.
```
pwndbg> piebase
Calculated VA from /mnt/c/users/willi/Downloads/anti_flag = 0x555555554000
```
So now we know we want to jump to the function which is located at 0x555555555525. But where should we set a breakpoint? It should be before the ptrace
comparison, because that will ensure we don't enter into the wrong if condition, so our current location is fine. Let's use the jump function:
```
pwndbg> jump *0x555555555525
Continuing at 0x555555555525.
HTB{y0u_trac3_m3_g00d!!!}
[Inferior 1 (process 158605) exited normally]
pwndbg>
```
And there! We got the flag!

## My Takeaways
The first takeaway I have from this exercise is to inspect the original binary code. Disassemblers may be optimised to not display code branches
that aren't ever executed, however for reversing challenges they may be important as they could be hiding some secret. The lowest, disassembly
code will not face this problem, and from them we can find the results we want.
My next takeaway came when I was attempting to call the hidden function. My initial thought was some binary exploitation techniques, where I could
alter the return address, but there was nowhere to input anything. However, some google searches led me to a new GDB feature - the jump function 
([found here](https://www.sans.org/blog/using-gdb-to-call-random-functions/)). 
My final takeaway was in locating the address of the function I wanted to call. To do this, I used the piebase command, which I found on a 
[pwngdb cheatsheet](https://pwndbg.re/CheatSheet.pdf). I already knew the offset of my desired line of code from the start of the program, so 
I just wanted to find some function that told me where my code execution started, which a series of Google searches led me to.