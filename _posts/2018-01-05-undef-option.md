---
layout	: page
title		: "Use case of linker's -u option"
date       : 2018-01-04
author      : "pshung"
---
>-u symbol
>
--undefined=symbol
>
Force symbol to be entered in the output file as an undefined symbol. Doing this may, for example, trigger linking of additional modules from standard libraries. `-u' may be repeated with different option arguments to enter additional undefined symbols.

# Real Case in Newlib
You may wonder when this options will be used. One of real example is in newlib's nano-formatted-io implementation. To reduce code size, you can configure newlib with --enable-newlib-nano-formatted-io option. By default, floating-pont I/O won't be linked. If a program needs this function, it has to explicitly undefine _printf_float or _scanf_float.

Reference: Newlib/README

# My example
```c

// main.c
extern void foo() __attribute__((__weak__));
int main(int argc, char *argv[]) {


    if (foo)
      puts("has foo\n");
    else
      puts("no foo\n");
    return 0;
}

// foo.c
void foo() {

}
```

```
ar -cvq libfoo.a foo.o
gcc printf.c -o printf
gcc printf.c libfoo.a -Wl,-ufoo -o printf
gcc printf.c libfoo.a  -o printf
gcc printf.c libfoo.a  -o printf_withoutU
gcc printf.c libfoo.a -Wl,-ufoo -o printfWithU
./printf_withoutU
no foo
./printfWithU
has foo

```
