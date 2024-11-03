# Baby RE 
Running the program gives us a prompt for a key.
```
Insert key:
test
Try again later.
```
It seems our task is to find the key (or we can straight up find the flag!)

## Code
Before we run the code through Binary Ninja, let's run it through strings to see if we can pick up the flag like this:
```
HTB{B4BYH
_R3V_TH4H
TS_Ef
[]A\A]A^A_
Dont run `strings` on this challenge, that is not the way!!!!
```
We find something that looks like the flag (broken up in pieces), and also instructions telling us not to run strings...
Let's run ltrace instead and see if there's a strcmp!
```
williloo@WillLaptop:/mnt/c/users/willi/Downloads$ ltrace ./baby
puts("Insert key: "Insert key:
)                                                 = 13
fgets(hello
"hello\n", 20, 0x7f9f56b79aa0)                                 = 0x7ffc1f15bd30
strcmp("hello\n", "abcde122313\n")                                   = 7
puts("Try again later."Try again later.
)                                             = 17
+++ exited (status 0) +++
```
And there we have it, the key is abcde122313. But there's probaly other ways to approach this! Let's take a look at the binary
and see if there's another approach:
```
00001155  int32_t main(int32_t argc, char** argv, char** envp)

00001164      char const* const var_10 = "Dont run `strings` on this chall…"
0000116f      puts(str: "Insert key: ")
00001187      void buf
00001187      fgets(&buf, n: 0x14, fp: __TMC_END__)
00001187      
000011a1      if (strcmp(&buf, "abcde122313\n") != 0)
000011e1          puts(str: "Try again later.")
000011a1      else
000011b7          int64_t str
000011b7          __builtin_strncpy(dest: &str, src: "HTB{B4BY_R3V_TH4TS_EZ}", n: 0x16)
000011d3          puts(&str)
000011d3      
000011ec      return 0
```
The flag is kinda just hardcoded...but we can take it anyway!

## My Takeaways
My biggest takeaway from this challenge was ltrace. This being my first Reversing challenge, I sought help from [this guiude](https://swanandx.github.io/blog/posts/re/ctfs/)
to reveresing challenges, which introduced me to ltrace. My first instinct was to reverse the binary using Binary Ninja, which led me to the flag.
But the hint on the challenge said that there were multiple ways to approach this challenge, so I thought to explore some other ways, which may benefit
me down the line. That was when I thought to look into tips for reversing catgeory challenges, thereby finding the site.