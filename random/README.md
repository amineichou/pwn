## pwn : fd [writeup]

Challenge from [pwnable.kr](https://pwnable.kr/)

**In this challenge** We have this code :
```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main(){
	unsigned int random;
	random = rand();	// random value!

	unsigned int key=0;
	scanf("%d", &key);

	if( (key ^ random) == 0xdeadbeef ){
		printf("Good!\n");
		system("/bin/cat flag");
		return 0;
	}

	printf("Wrong, maybe you should try 2^32 cases.\n");
	return 0;
}

```

we need to execute this line :
```c
system("/bin/cat flag");
```
To do that we need this condition to be true :
```c
(key ^ random) == 0xdeadbeef
```
For this condition to be true we need to get the value of the `random` variable thet uses `rand()` to generate a random number.

### So ! How the hell are we gonna predict a random number ?

**After i read about `rand()` function i found this**

*The rand() function by default uses the value 1 as the seed to generate random numbers which leads to the generation of the same sequence of random numbers. To prevent this, we can use srand() function to specify a new seed for rand() function.*

**YES** `rand()` function will always return the same value.

All i did to solve this is by openning the program in `gdb`.

## Steps to solve :

1. open gdb:
```bash
./random
```

2. show the assembly :
```bash
layout asm
```

3. here's the assembly

![gdb](/assets/gdb.png)

4. we know that the random variable is stored here:
```asm
0x400606 <main+18>              mov    %eax,-0x4(%rbp)
```
5. make a breakpoint after calling `rand()` and adding the value to random variable :
```bash
break *0x400615
```
6. print that random variable:
```bash
x/d $rbp-4
```

7. Now heres the random variable:

![gdb](/assets/gdb2.png)

8. Let's now make a simple XOR opiration :
- We have :
```c
(key ^ random) == 0xdeadbeef;
```
- So :
```c
key = 0xdeadbeef ^ random;
```

- Calculate :
```c
key = 3039230856
```

9. Finally :
```bash
random@pwnable:~$ ./random
3039230856
Good!
Mommy, I thought libc random is unpredictable...
```

end.