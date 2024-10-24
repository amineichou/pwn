## pwn : col [writeup]

Challenge from [pwnable.kr](https://pwnable.kr/)

**Challenge Overview**

The program given in the code verifies a passcode entered by the user. The goal is to provide a valid passcode that, when passed through the check_password function, matches the target hash 0x21DD09EC. If the check passes, the program reveals the flag by executing /bin/cat flag.

**Code Analysis**

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

unsigned long hashcode = 0x21DD09EC;

unsigned long check_password(const char* p){
    int* ip = (int*)p;        // Cast the passcode to an int pointer
    int i;
    int res = 0;
    for(i = 0; i < 5; i++){   // Add the first 5 integers of the passcode
        res += ip[i];
    }
    return res;
}

int main(int argc, char* argv[]){
    if(argc < 2){
        printf("usage : %s [passcode]\n", argv[0]);
        return 0;
    }
    if(strlen(argv[1]) != 20){   // Passcode must be 20 bytes
        printf("passcode length should be 20 bytes\n");
        return 0;
    }

    if(hashcode == check_password(argv[1])){  // Check if the passcode matches the hash
        system("/bin/cat flag");  // If successful, reveal the flag
        return 0;
    } else {
        printf("wrong passcode.\n");
    }
    return 0;
}
```

**Key Points:**
- The program expects a 20-byte passcode (strlen(argv[1]) == 20).
- The passcode is interpreted as 5 integers (4 bytes each, since int is 4 bytes on most systems).
- These integers are summed and compared with the target hash 0x21DD09EC (which is 568134124 in decimal).
- The task is to find a 20-byte passcode that, when divided into 5 integers, produces the correct sum.

**Objective**
We need to craft a 20-byte string that, when interpreted as 5 integers, sums to 568134124.

**Step-by-Step Solution**
###### Step 1: Determine the Required Sum
We are tasked with finding 5 integers that add up to the target 0x21DD09EC (or 568134124 in decimal). Each integer will be represented by 4 bytes of the input string.

###### Step 2: Create a Set of Integers
To solve this, we can divide the target sum 568134124 into 5 integers such that the sum of these integers equals the hashcode. A simple way to approach this is to divide the target sum evenly among 4 integers and adjust the 5th integer to make the sum exact.

Let’s divide 568134124 by 5:

```bash
568134124 / 5 = 113626824.8
```

We can take 4 integers as 113626824 and the 5th integer as 113626828 to compensate for the remainder.

Thus, the integers are:

```bash
113626824, 113626824, 113626824, 113626824, 113626828
```

###### Step 3: Convert Integers to Bytes

We need to convert these integers into their byte representation and construct the 20-byte passcode.

Integer to Byte Conversion:
We can use Python or another tool to convert the integers to bytes:

```python
import struct

ints = [113626824, 113626824, 113626824, 113626824, 113626828]
passcode = b''.join([struct.pack("<I", i) for i in ints])

print(passcode)
```

This will output the 20-byte passcode.

###### Step 4: Test the Exploit
Now, we can use this passcode as input to the program: