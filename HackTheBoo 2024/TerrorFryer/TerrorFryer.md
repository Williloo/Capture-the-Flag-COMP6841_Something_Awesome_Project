# TerrorFryer
We begin by testing how we expect the program to run with a test input:

```
Please enter your recipe for frying: test
got:      `etts`
expected: `1_n3}f3br9Ty{_6_rHnf01fg_14rlbtB60tuarun0c_tr1y3`
This recipe isn't right :(
```

Notice that the 'got' output is an anagram of my original input. We can also see that the 'expected' output has the character 'H', 'T', 'B', '{' and '}', which are
essential elements to a HackTheBox flag. Perhaps we need to unscramble the 'expected' output? Let's disassemble the binary using BinaryNinja to inspect the code.

## Code
```
00001242  int32_t main(int32_t argc, char** argv, char** envp)

00001242  {
00001248      void* fsbase;
00001248      int64_t rax = *(uint64_t*)((char*)fsbase + 0x28);
0000126e      setvbuf(__bss_start, nullptr, 2, 0);
0000127f      printf("Please enter your recipe for fry…");
00001293      void buf;
00001293      fgets(&buf, 0x40, stdin);
000012a0      char* rax_3 = strchr(&buf, 0xa);
000012a0      
000012a8      if (rax_3 != 0)
000012aa          *(uint8_t*)rax_3 = 0;
000012aa      
000012b0      fryer(&buf);
000012ce      printf("got:      `%s`\nexpected: `%s`\n", &buf, "1_n3}f3br9Ty{_6_rHnf01fg_14rlbtB…");
000012ce      
000012e0      if (strcmp("1_n3}f3br9Ty{_6_rHnf01fg_14rlbtB…", &buf) == 0)
00001311          puts("Correct recipe - enjoy your meal…");
000012e0      else
000012e9          puts("This recipe isn't right :(");
000012e9      
000012f3      *(uint64_t*)((char*)fsbase + 0x28);
000012f3      
000012fc      if (rax == *(uint64_t*)((char*)fsbase + 0x28))
00001309          return 0;
00001309      
00001318      __stack_chk_fail();
00001318      /* no return */
00001242  }
```

We can notice that on line 0x12b0, a function 'fryer' is called, which takes in our input buffer, so this is likely to be the function that jumbles up
our input. Let's take a closer look at this function.
```
000011b9  uint64_t fryer(char* arg1)

000011b9  {
000011cb      if (init.1 == 0)
000011cb      {
000011cd          seed.0 = 0x13377331;
000011d7          init.1 = 1;
000011cb      }
000011cb      
000011e4      uint64_t result = strlen(arg1);
000011e9      uint64_t result_1 = result;
000011e9      
000011f0      if (result > 1)
000011f0      {
000011f2          int64_t r14_1 = (result - 1);
000011f6          int64_t rbx_1 = 0;
000011f6          
00001237          do
00001237          {
0000121a              int32_t rdx_3 = (((int32_t)(COMBINE(0, ((int64_t)rand_r(&seed.0))) % (result_1 - rbx_1))) + rbx_1);
0000121c              result = ((uint64_t)arg1[rbx_1]);
00001224              char* rdx_5 = &arg1[((int64_t)rdx_3)];
0000122a              arg1[rbx_1] = *(uint8_t*)rdx_5;
0000122e              *(uint8_t*)rdx_5 = result;
00001230              rbx_1 += 1;
00001237          } while (rbx_1 != r14_1);
000011f0      }
000011f0      
00001241      return result;
000011b9  }
```
Reading this code, it seems to be a bunch of swaps between characters in the input string. The only property of the input that is used in this jumbling
process is its length. The random number generator also follows the set seed of 0x13377331, so we get the same jumbling no matter what (as tested by inputting
the same thing multiple times). This means as long as I use a string with the same length as the flag, I will get the same jumbling as the flag.

## Exploit Formation
The flag has 48 letters. I used an input with 48 distinct letters, and tracked where each of them ended up. Doing so, I can create a map between the original
position of each letter to their jumbled position. I can then use this map to unjumble the 'expected' output.

Exploit: [here](exploit.py)