# Baby Crypt
Let's have a look at how the code runs:
```
Give me the key and I'll give you the flag: test
KFx-4q
5p
qpf`\7}\=0{ev"(
```
It seems to ask us for an input, and give us some encrypted version of the input. If we can give the right one, it will give us the flag.
Let's see if the binary can provide us with any help!

## Code
```
000011a9  int32_t main(int32_t argc, char** argv, char** envp)

000011a9  {
000011b5      void* fsbase;
000011b5      int64_t rax = *(uint64_t*)((char*)fsbase + 0x28);
000011d0      printf("Give me the key and I'll give yo…");
000011da      char* buf = malloc(4);
000011f6      fgets(buf, 4, __TMC_END__);
00001205      char rdx_1 = 0x46;
0000120f      int64_t var_38;
0000120f      __builtin_memcpy(&var_38, "\x3f\x64\x35\x0c\x48\x47\x05\x6f\x46\x04\x6f\x02\x04\x03\x13\x28\x52\x0e\x28\x58\x43\x0f\x00\x05", 0x18);
00001225      int16_t var_20 = 0x4d56;
00001225      
00001285      for (int32_t i = 0; i <= 0x19; i += 1)
00001285      {
00001272          rdx_1 = (*(uint8_t*)(&var_38 + ((int64_t)i)) ^ buf[((int64_t)(i % 3))]);
00001279          *(uint8_t*)(&var_38 + ((int64_t)i)) = rdx_1;
00001285      }
00001285      
0000129a      printf("%.26s\n", &var_38, rdx_1);
0000129a      
000012b1      if (rax == *(uint64_t*)((char*)fsbase + 0x28))
000012b9          return 0;
000012b9      
000012b3      __stack_chk_fail();
000012b3      /* no return */
000011a9  }
```
We can see that in the middle, there is some encryption of var_38, which is originally 
\x3f\x64\x35\x0c\x48\x47\x05\x6f\x46\x04\x6f\x02\x04\x03\x13\x28\x52\x0e\x28\x58\x43\x0f\x00\x05. In fact, it uses our buffer,
and performs a binary XOR operation on the i%3th element, and the ith element of var_38. Now, we'd be completely stuck...if not for the
fact that we know the first 3 bytes of the output! Since we are looking for a HackTheBox flag, the first 3 bytes are definitely HTB!
From here, we can reverse engineer the bytes: we want key[0] ^ 00111111 to equal 'H', which is 01001000. Because the operation is exclusive or, 
we want the key to be 01110111, which is 'w'. We can do a similar process for the other 2 characters of the key to get the key 'w0w'.
Typing this into the input, we can get the flag!

## My Takeaways
I found this to be a really fun exercise, especially the component which required us to solve some XOR operations. The binary was pretty
straightforward, but I was initially a bit hesitant and scared by the XOR operation, as it felt like some cryptographic encryption that I 
couldn't or wasn't supposed to solve. Instead, I tried ltrace and other techniques such as breakpoints in gdb etc to try to intercept the 
encryption. But when they failed, I thought "the encryption doesn't look thaaaat complicated right?", and decided to try my hand at decrypting
the encryption process. A quick search also told me that the XOR operation was very reversible. Using all this information, and realising that
the key was only 3 digits (perfectly aligned with HTB), I tried the decryption and got the result pretty easily.