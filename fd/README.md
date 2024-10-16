## pwn : fd [writeup]

Challenge from [pwnable.kr](https://pwnable.kr/)

**Challenge Overview**
We are given the following C code, which reads data from a file descriptor and checks if the input matches the string `LETMEWIN\n` . If it does, it prints `good job :)\n` and executes `/bin/cat` flag.

**Code Analysis**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <unistd.h>

char buf[32];
int main(int argc, char* argv[], char* envp[]){
        if(argc<2){
                printf("pass argv[1] a number\n");
                return 0;
        }
        int fd = atoi( argv[1] ) - 0x1234;
        int len = 0;
        len = read(fd, buf, 32);
        if(!strcmp("LETMEWIN\n", buf)){
                printf("good job :)\n");
                system("/bin/cat flag");
                exit(0);
        }
        printf("learn about Linux file IO\n");
        return 0;

}
```

**Key Points:**
- The program expects an integer as the first argument (argv[1]).
- This integer is converted into a file descriptor by subtracting 0x1234 (4660 in decimal).
- It reads 32 bytes from this file descriptor into the buffer buf.
- If the content of the buffer matches "LETMEWIN\n", it prints the flag.

**Objective**
We need to pass the check strcmp("LETMEWIN\n", buf) to get the flag. To achieve this, we need to:
1. Provide a valid file descriptor (calculated from argv[1]).
2. Ensure that the content of this file descriptor contains the string "LETMEWIN\n".

**Step-by-Step Solution**
###### Step 1 : Identify the File Descriptor
The file descriptor is computed using the formula:
```c
int fd = atoi(argv[1]) - 0x1234;
```

This means that if we want a specific file descriptor, say 0 (standard input), we need to reverse the calculation:

```bash
argv[1] = fd + 0x1234
```

For example, if we want fd = 0 (standard input):

```bash
argv[1] = 0x1234 = 4660
```

###### Step 2: Input the Required String

Since we are now reading from stdin, we need to provide the string "LETMEWIN\n" through standard input. This can be done by echoing the string and piping it to the program.

```bash
$ ./fd 4660
LETMEWIN
```

result

```bash
good job :)
mommy! I think I know what a file descriptor is!!
```

The program will print "good job :)" and then display the flag by executing system("/bin/cat flag").

###### Final Command:
```bash
$ echo "LETMEWIN" | ./fd 4660
```

end.