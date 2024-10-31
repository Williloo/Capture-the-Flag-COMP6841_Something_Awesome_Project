# Bypass
Let's see how the code runs:
```
Enter a username: Williloo
Enter a password: password
Wrong username and/or password
Enter a username:
```
It seems to be a continuous while loop asking for a username and password. Running strings on the program gives nothing useful, so
let's crack open the binary to see what's going on.

## Code 
Even when openning the binary using Binary Ninja Pseudo C, it look like semi gibberish assembly. This prompted me to check the type
of file 'Bypass.exe' is.
```
williloo@WillLaptop:/mnt/c/users/willi/Downloads$ file Bypass.exe
Bypass.exe: PE32 executable (console) Intel 80386 Mono/.Net assembly, for MS Windows
```
So it turns out it had a .Net assembly, meaning it was written using C#. After some time of researching, I found a decompiler called
DnSpy, which also has a good debugger for the program, where I can edit values of variables in the midst of break points! How handy!
Using this debugger to decompile our code, we quickly find this:
```
using System;

// Token: 0x02000002 RID: 2
public class 0
{
	// Token: 0x06000002 RID: 2 RVA: 0x00002058 File Offset: 0x00000258
	public static void 0()
	{
		bool flag = global::0.1();
		bool flag2 = flag;
		if (flag2)
		{
			global::0.2();
		}
		else
		{
			Console.WriteLine(5.0);
			global::0.0();
		}
	}

	// Token: 0x06000003 RID: 3 RVA: 0x00002090 File Offset: 0x00000290
	public static bool 1()
	{
		Console.Write(5.1);
		string text = Console.ReadLine();
		Console.Write(5.2);
		string text2 = Console.ReadLine();
		return false;
	}

	// Token: 0x06000004 RID: 4 RVA: 0x000020C8 File Offset: 0x000002C8
	public static void 2()
	{
		string <<EMPTY_NAME>> = 5.3;
		Console.Write(5.4);
		string b = Console.ReadLine();
		bool flag = <<EMPTY_NAME>> == b;
		if (flag)
		{
			Console.Write(5.5 + global::0.2 + 5.6);
		}
		else
		{
			Console.WriteLine(5.7);
			global::0.2();
		}
	}

	// Token: 0x04000001 RID: 1
	public static string 0;

	// Token: 0x04000002 RID: 2
	public static string 1;

	// Token: 0x04000003 RID: 3
	public static string 2 = 5.8;
}
```
A chunk of code that seems eerily similar to how our .exe file runs: it writes a line, then reads an input, before repeating the process
("Enter a username:", "Enter a password:"). But notice that this function always returns false...and we need flag2, which is equal to flag,
which is the return value of this function, to be true. Let's use our debugger to set a breakpoint before the if clause and change the value
of flag2 to be true.
```
Enter a username: asd
Enter a password: sad
Please Enter the secret Key:
```
Now we need some secret key (see function 2). The function sets flag to be equal to whether <<EMPTY_NAME>> is equal to the console input.
Now, if we set a break point before the if clause again, we can edit the code so flag is true again. Alternatively, we can also read that
the secret passkey is "ThisIsAReallyReallySecureKeyButYouCanReadItFromSourceSoItSucks". Either way, we can continue the program and get the
flag which is printed out successfully!