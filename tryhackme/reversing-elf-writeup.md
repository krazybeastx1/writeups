# Reversing Elf Writeup

Download all the files and use `chmod +x crackme8` it will make your file executable , instead of doing it for every single one

> Crackme1

its just basic one

```
$ ./crackme1                                         
flag{not_that_kind_of_elf}
```

> Crackme2

```
$ ./crackme2                       
Usage: ./crackme2 password
```

we need password get the flag lets use `strings` command to see is there any strings which can help full

```
$ strings crackme2 
...
Usage: %s password
super_secret_password
...
```

> Crackme3

```
$ ./crackme2 super_secret_password 
Access granted.
flag{if_i_submit_this_flag_then_i_will_get_points}
```

> Crackme4

```
$ ./crackme4                       
Usage : ./crackme4 password
This time the string is hidden and we used strcmp

```

we need a password to get the flag and they are using strcmp

**strcmp** : `compares two strings character by character. If the strings are equal, the function returns 0.`

it compares the char by char to get the flag

lets you use ltrace tool that will look in to the library calls made by the program

```
$ ltrace ./crackme4 hellofriend 
__libc_start_main(0x400716, 2, 0x7fffb194cca8, 0x400760 <unfinished ...>
strcmp("my_m0r3_secur3_pwd", "hellofriend")                                                                                                      = 5
printf("password "%s" not OK\n", "hellofriend"password "hellofriend" not OK
)                                                                                                  = 30
+++ exited (status 0) +++

```

now we can see that `password` is comparing to the `hellofriend` , which is not equal so we did get the flag

```
$  ltrace ./crackme4 my_m0r3_secur3_pwd 
__libc_start_main(0x400716, 2, 0x7ffed36f00f8, 0x400760 <unfinished ...>
strcmp("my_m0r3_secur3_pwd", "my_m0r3_secur3_pwd")                                                                                               = 0
puts("password OK"password OK
)                                                                                                                              = 12
+++ exited (status 0) +++

```

now we got the OK and program returned 0 .

> Crackme5

```
$ ./crackme5 
Enter your input:
11
Always dig deeper

```

we opened the crackme5 fille in ghdira

```c
  local_27 = 0x74;
  local_26 = 0x58;
  local_25 = 0x40;
  local_24 = 0x73;
  local_23 = 0x58;
  local_22 = 0x60;
  local_21 = 0x34;
  local_20 = 0x74;
  local_1f = 0x58;
  local_1e = 0x74;
  local_1d = 0x7a;
  puts("Enter your input:");
  __isoc99_scanf(&DAT_00400966,local_58);
  iVar1 = strcmp_(local_58,&local_38);
  if (iVar1 == 0) {
    puts("Good game");
  }
```

its same as the above one comparing the character to the input we gave to the program so we can decode each strings from hexadecimal to char (i am lazy to do that ) `example: 0x7a ='z'`

or we can use ltrace see the trace the what the call is making to library in this case comparing character of the program

```
$ ltrace ./crackme5 
__libc_start_main(0x400773, 1, 0x7fff86507288, 0x4008d0 <unfinished ...>
puts("Enter your input:"Enter your input:
)                                                                                                                        = 18
__isoc99_scanf(0x400966, 0x7fff86507120, 0, 0x7f9fd92beb00hello
)                                                                                      = 1
strlen("hello")                                                                                                                                  = 5
strlen("hello")                                                                                                                                  = 5
strlen("hello")                                                                                                                                  = 5
strlen("hello")                                                                                                                                  = 5
strlen("hello")                                                                                                                                  = 5
strlen("hello")                                                                                                                                  = 5
strncmp("hello", "OfdlDSA|3tXb32~X3tX@sX`4tXtz\237\177", 28)                                                                                     = 25
puts("Always dig deeper"Always dig deeper
)                                                                                                                        = 18
+++ exited (status 0) +++

```

```
$ ./crackme5        
Enter your input:
OfdlDSA|3tXb32~X3tX@sX`4tXtz
Good game
```

> Crackme6

```
$ ./crackme6 
Usage : ./crackme6 password
Good luck, read the source
$ ./crackme6 1111     
password "1111" not OK
```

**Ghidra** :

```c
void compare_pwd(char *param_1)

{
  undefined8 uVar1;
  
  uVar1 = my_secure_test(param_1);
  if ((int)uVar1 == 0) {
    puts("password OK");
  }
  else {
    printf("password \"%s\" not OK\n",param_1);
  }
  return;
}

```

it's compare the input with my\_secure\_test , if the input and the password is same the we will get the ok .

Let's look at the my\_secure\_test (double click on the code)

```c
undefined8 my_secure_test(char *param_1)

{
  undefined8 uVar1;
  
  if ((*param_1 == '\0') || (*param_1 != '1')) {
    uVar1 = 0xffffffff;
  }
  else if ((param_1[1] == '\0') || (param_1[1] != '3')) {
    uVar1 = 0xffffffff;
  }
  else if ((param_1[2] == '\0') || (param_1[2] != '3')) {
    uVar1 = 0xffffffff;
  }
  else if ((param_1[3] == '\0') || (param_1[3] != '7')) {
    uVar1 = 0xffffffff;
  }
  else if ((param_1[4] == '\0') || (param_1[4] != '_')) {
    uVar1 = 0xffffffff;
  }
  else if ((param_1[5] == '\0') || (param_1[5] != 'p')) {
    uVar1 = 0xffffffff;
  }
  else if ((param_1[6] == '\0') || (param_1[6] != 'w')) {
    uVar1 = 0xffffffff;
  }
  else if ((param_1[7] == '\0') || (param_1[7] != 'd')) {
    uVar1 = 0xffffffff;
  }
  else if (param_1[8] == '\0') {
    uVar1 = 0;
  }
  else {
    uVar1 = 0xffffffff;
  }
  return uVar1;
}
```

if input is equal to 8 null character or each character is compared to each character in param

if its not equal to then we get Not ok in return so the password is `1337_pwd`

```
$ ./crackme6 1337_pwd          
password OK
```

> Crackme7

```
$ ./crackme7            
Menu:

[1] Say hello
[2] Add numbers
[3] Quit

[>] 1
What is your name? hello
Hello, hello!
Menu:

[1] Say hello
[2] Add numbers
[3] Quit

[>] 67373773
Unknown choice: 67373773

```

we can see that there are some option and we try to give invalid input and it exits

lets hop in to ghidra and do some analysis

```c
...
         puVar2 = puVar2 + (uint)bVar3 * -2 + 1;
      }
      iVar1 = __isoc99_scanf(&DAT_0804883a,local_80);
      if (iVar1 != 1) {
         puts("Unable to read name!");
         return 1;
      }
      printf("Hello, %s!\n",(char *)local_80);
    }
    if (local_14 != 2) {
      if (local_14 == 3) {
         puts("Goodbye!");
      }
      else if (local_14 == 0x7a69) { ##this is what we are looking for 
         puts("Wow such h4x0r!");
         giveFlag();
      }
      else {
...

```

In code we can see there is a hidden(is it ?) hexadecimal we can see the we commented next to local\_14 == 0x7a69

It's in the if we change to decimal and enter input

then lets see what happens

```
$ ./crackme7      
Menu:

[1] Say hello
[2] Add numbers
[3] Quit

[>] 31337
Wow such h4x0r!
flag{much_reversing_very_ida_wow}

```

we got the flag

> Crackme8

crackme8 is same as all , we need to grab the password to get the flag

```c
undefined4 main(int param_1,undefined4 *param_2)

{
  undefined4 uVar1;
  int iVar2;
  
  if (param_1 == 2) {
    iVar2 = atoi((char *)param_2[1]);
    if (iVar2 == -0x35010ff3) {
      puts("Access granted.");
      giveFlag();
      uVar1 = 0;
    }
    else {
      puts("Access denied.");
      uVar1 = 1;
    }
  }
  else {
    printf("Usage: %s password\n",(char *)*param_2);
    uVar1 = 1;
  }
  return uVar1;
}

```

it simply comparing the input to the password and gives output the flag so we decode hexadecimal in Var2 to decimal and enter them in the input then we will get the flag

```
$ ./crackme8 -889262067 
Access granted.
flag{at_least_this_cafe_wont_leak_your_credit_card_numbers}
```
