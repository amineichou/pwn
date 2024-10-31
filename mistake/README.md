## pwn : Mistake [writeup]

Challenge from [pwnable.kr](https://pwnable.kr/)

**We have this code**
```c
#include <stdio.h>
#include <fcntl.h>

#define PW_LEN 10
#define XORKEY 1

void xor(char* s, int len){
	int i;
	for(i=0; i<len; i++){
		s[i] ^= XORKEY;
	}
}

int main(int argc, char* argv[]){

	int fd;
	if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
		printf("can't open password %d\n", fd);
		return 0;
	}

	printf("do not bruteforce...\n");
	sleep(time(0)%20);

	char pw_buf[PW_LEN+1];
	int len;
	if(!(len=read(fd,pw_buf,PW_LEN) > 0)){
		printf("read error\n");
		close(fd);
		return 0;
	}

	char pw_buf2[PW_LEN+1];
	printf("input password : ");
	scanf("%10s", pw_buf2);

	// xor your input
	xor(pw_buf2, 10);

	if(!strncmp(pw_buf, pw_buf2, PW_LEN)){
		printf("Password OK\n");
		system("/bin/cat flag\n");
	}
	else{
		printf("Wrong Password\n");
	}

	close(fd);
	return 0;
}
```

1. Understanding the code :
```c
    if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
		printf("can't open password %d\n", fd);
		return 0;
	}
```
The code tries to open a file in read-only `0400` mode and prints an error if it fails. There's a bug, though: it incorrectly checks the result of open.

But! our user `mistake` doesn't have the right permission to open the file `/home/mistake/password`.

```bash
mistake@pwnable:~$ whoami
mistake
```

```bash
mistake@pwnable:~$ ls -al
total 44
drwxr-x---   5 root        mistake 4096 Oct 23  2016 .
drwxr-xr-x 116 root        root    4096 Oct 30  2023 ..
d---------   2 root        root    4096 Jul 29  2014 .bash_history
-r--------   1 mistake_pwn root      51 Jul 29  2014 flag
dr-xr-xr-x   2 root        root    4096 Aug 20  2014 .irssi
-r-sr-x---   1 mistake_pwn mistake 8934 Aug  1  2014 mistake
-rw-r--r--   1 root        root     792 Aug  1  2014 mistake.c
-r--------   1 mistake_pwn root      10 Jul 29  2014 password
drwxr-xr-x   2 root        root    4096 Oct 23  2016 .pwntools-cache
```

**Because we our user doesn't have the right permission to open that file, open will return `-1`.**
**witch will set fd to `1` cause of this comparison `open("/home/mistake/password",O_RDONLY,0400) < 0`**

**So this code will read from stdout because fd = 1**
```c
	if(!(len=read(fd,pw_buf,PW_LEN) > 0)){
		printf("read error\n");
		close(fd);
		return 0;
	}
```

**Then**

This code prompts the user to input a password and stores up to 10 characters in the `pw_buf2` buffer.
```c
	char pw_buf2[PW_LEN+1];
	printf("input password : ");
	scanf("%10s", pw_buf2);
```

this code will XOR the input from stdout
```c
    xor(pw_buf2, 10);
```

And then compares two password buffers (`pw_buf` and `pw_buf2`). If they match, it prints "Password OK" and displays the contents of the flag file. Otherwise, it prints "Wrong Password."

2. Solution :

![mistake](https://i.imgur.com/PLBmYXO.png)

`A` is `01000001`
`01000001` ^ `00000001` = `1000000`
`1000000` is `@`


end.