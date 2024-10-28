# You Can't C Me
Lets have a look at what the code does when we run it:
```
Welcome!
AAAAAAAAAAAAAAAAAAAA
I said, you can't c me!
```
It asks for an input, before giving a taunt: let's have a look at what its code says!

## Code 
```
00401160  int32_t main(int32_t argc, char** argv, char** envp)
00401160  {
00401172      int32_t result = 0;
00401179      int32_t var_10 = 0;
00401180      int64_t rax;
00401180      rax = 0;
00401182      int32_t rax_1 = printf("Welcome!\n");
00401196      int64_t var_28;
00401196      __builtin_strncpy(&var_28, "this_is_the_password", 0x15);
004011bc      int32_t var_4c = rax_1;
004011bf      char* buf = malloc(0x15);
004011d0      int64_t var_48 = 0x5517696626265e6d;
004011dc      int64_t var_40;
004011dc      __builtin_strncpy(&var_40, "o&kUZ\'ZUYUc)", 0xc);
004011dc      
004011f5      for (int32_t i = 0; i < 0x14; i += 1)
0040120d          *(uint8_t*)(&var_28 + ((int64_t)i)) = (*(uint8_t*)(&var_48 + ((int64_t)i)) + 0xa);
0040120d      
0040123d      char* var_58 = fgets(buf, 0x15, stdin);
00401249      int32_t rax_7;
00401249      
00401249      if (strcmp(&var_28, buf) == 0)
00401249      {
00401276          rax_7 = 0;
0040127d          int32_t var_60_1 = printf("HTB{%s}\n", buf);
00401249      }
00401249      else
00401249      {
00401259          rax_7 = 0;
00401260          int32_t var_5c_1 = printf("I said, you can't c me!\n");
00401249      }
00401288      return result;
00401160  }
```
We can see that on line 0x4101249, the 'buf', which is our input, is compared to &var_28, and if they are equal, the string will be printed.
While we can try to reverse engineer the value of var_28, another way is to utilise the 'ltrace' function to track what is being called in the
code execution.
```
williloo@WillLaptop:/mnt/c/users/willi/Downloads$ ltrace ./auth
printf("Welcome!\n"Welcome!
)                                                 = 9
malloc(21)                                                           = 0x1d076b0
fgets(hello
"hello\n", 21, 0x7f70b167aaa0)                                 = 0x1d076b0
strcmp("wh00ps!_y0u_d1d_c_m3", "hello\n")                            = 15
printf("I said, you can't c me!\n"I said, you can't c me!
)                                  = 24
+++ exited (status 0) +++
```
We can see that the string "wh00ps!_y0u_d1d_c_m3" is stored in var_28, and using that, we can plug it into the input of ./auth to get the flag.