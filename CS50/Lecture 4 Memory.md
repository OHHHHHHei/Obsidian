# 像素艺术

0代表白色，1代表黑色

![[image 1.png]]

# 十六进制

## RGB

红绿蓝

![[image 2.png]]

\#0000FF

```C
 0 1 2 3 4 5 6 7 8 9 A B C D E F
```

十六进制也叫_base-16_.  
10是0A，15是0F，16是10  

# 内存

你可以把内存想像成小格子

![[image 3.png]]

但是其中的10可能带来歧义，所以在描述内存位置时，我们在前面加入0x，来表示这是十六进制。

![[image 4.png]]

这样0x10就是16而不是十。

# 指针

C语言有两种与内存有关的强大运算符，*和&。

```C
// Prints an integer's address

#include <stdio.h>

int main(void)
{
    int n = 50;
    printf("%p\n", &n);
}
```

%p，这个类型允许我们查看内存中的地址。&n 可以被解释为找到 n 的地址。

```C
    printf("%i\n", *p);
```

*p代表解指针，走向这个指针指向的地址

![[image 5.png]]

# 字符串

```C
// Prints a string

#include <cs50.h>
#include <stdio.h>

int main(void)
{
    string s = "HI!";
    printf("%s\n", s);
}
```

这里你定义了一个字符串 s 是”HI!”

字符串可以简单理解为是字母的数组

![[image 6.png]]

s实际上是一个指针，它指向字符串的第一个字节。

![[image 7.png]]

在没有cs50库的情况下，string类型是不存在的，实际上应该为char *s。

```C
// Declares a string without CS50 Library

#include <stdio.h>

int main(void)
{
    char *s = "HI!";
    printf("%s\n", s);
}
```

# 指针运算

```C
// Prints a string's chars via pointer arithmetic

#include <stdio.h>

int main(void)
{
    char *s = "HI!";
    printf("%c\n", *s);
    printf("%c\n", *(s + 1));
    printf("%c\n", *(s + 2));
}
```

分别输出了H I !

```C
// Prints substrings via pointer arithmetic

#include <stdio.h>

int main(void)
{
    char *s = "HI!";
    printf("%s\n", s);
    printf("%s\n", s + 1);
    printf("%s\n", s + 2);
}
```

输出了 HI! I! !

# 字符串比较

```C
// Compares two integers

#include <cs50.h>
#include <stdio.h>

int main(void)
{
    // Get two integers
    int i = get_int("i: ");
    int j = get_int("j: ");

    // Compare integers
    if (i == j)
    {
        printf("Same\n");
    }
    else
    {
        printf("Different\n");
    }
}
```

字符串不能用 == 进行比较。**因为 == 比较的是两个字符串第一个字母的地址是否相同。**

我们可以使用strcmp函数来进行两个字符串的比较。

```C
// Compares two strings using strcmp

#include <cs50.h>
#include <stdio.h>
#include <string.h>

int main(void)
{
    // Get two strings
    char *s = get_string("s: ");
    char *t = get_string("t: ");

    // Compare strings
    if (strcmp(s, t) == 0)
    {
        printf("Same\n");
    }
    else
    {
        printf("Different\n");
    }
}
```

如果两个字符串相同，那么strcmp函数会返回0。

# Copying and malloc

```C
// Capitalizes a string

#include <cs50.h>
#include <ctype.h>
#include <stdio.h>
#include <string.h>

int main(void)
{
    // Get a string
    string s = get_string("s: ");

    // Copy string's address
    string t = s;

    // Capitalize first letter in string
    t[0] = toupper(t[0]);

    // Print string twice
    printf("s: %s\n", s);
    printf("t: %s\n", t);
}
```
注意，这里

```C
string t = s;
```

**表示t的地址等于s的地址。**

![[image 8.png]]

两个指针指向了同一个地址。

## malloc

如果你想要创造一个新的地址来存放你复制的字符串，那么malloc函数允许你调用你的内存。free函数用来释放你调用的内存。

```C
// Capitalizes a copy of a string

#include <cs50.h>
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    // Get a string
    char *s = get_string("s: ");

    // Allocate memory for another string
    char *t = malloc(strlen(s) + 1);

    // Copy string into memory, including '\0'
    for (int i = 0; i <= strlen(s); i++)
    {
        t[i] = s[i];
    }

    // Capitalize copy
    t[0] = toupper(t[0]);

    // Print strings
    printf("s: %s\n", s);
    printf("t: %s\n", t);
}
```

注意

```C
 char *t = malloc(strlen(s) + 1);
```

这个指令调用了一个大小为 s+1 的内存。这是为了存放 “\0” 这是字符串的结尾（strlen函数不会数上最后一个“\0”）。

for循环将每一个s中的字母赋入t之中。

```C
// Capitalizes a copy of a string, defining n in loop too

#include <cs50.h>
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    // Get a string
    char *s = get_string("s: ");

    // Allocate memory for another string
    char *t = malloc(strlen(s) + 1);

    // Copy string into memory, including '\0'
    for (int i = 0, n = strlen(s); i <= n; i++)
    {
        t[i] = s[i];
    }

    // Capitalize copy
    t[0] = toupper(t[0]);

    // Print strings
    printf("s: %s\n", s);
    printf("t: %s\n", t);
}
```

在这段代码中，没进行一次for循环，strlen函数都会被调用一次，这显然没有什么效率可言。

所以我们可以把这段代码放到外面来避免重复调用同一个函数。

C语言中内置了一个函数，strcpy

```C
// Capitalizes a copy of a string using strcpy

#include <cs50.h>
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    // Get a string
    char *s = get_string("s: ");

    // Allocate memory for another string
    char *t = malloc(strlen(s) + 1);

    // Copy string into memory
    strcpy(t, s);

    // Capitalize copy
    t[0] = toupper(t[0]);

    // Print strings
    printf("s: %s\n", s);
    printf("t: %s\n", t);
}
```

可以直接复制一个字符

# Valgrind

这是一个工具，来检查你的程序是否有内存调用相关问题，尤其是在使用malloc后有没有使用free来释放内存。

# 垃圾值

当你使用malloc调用内存而且没有进行初始化时，你调用的内存很可能之前已经被使用过，那么其中就会存储一些垃圾值。

# Swapping

```C
// Fails to swap two integers

#include <stdio.h>

void swap(int a, int b);

int main(void)
{
    int x = 1;
    int y = 2;

    printf("x is %i, y is %i\n", x, y);
    swap(x, y);
    printf("x is %i, y is %i\n", x, y);
}

void swap(int a, int b)
{
    int tmp = a;
    a = b;
    b = tmp;
}
```

这段程序在逻辑上看可以完成交换的目标，但是实际上不行，为何？

![[image 9.png]]

我们可以把内存抽象为上面这张图。

heap称为堆。

stack称为栈。

![[image 10.png]]

swap函数所调用的内存与main函数不一样，所以你在swap中确实完成了交换，但是当swap执行完之后，它所属的内存被释放，main函数中的x，y实际上并没有改变。

```C
// Swaps two integers using pointers

#include <stdio.h>

void swap(int *a, int *b);

int main(void)
{
    int x = 1;
    int y = 2;

    printf("x is %i, y is %i\n", x, y);
    swap(&x, &y);
    printf("x is %i, y is %i\n", x, y);
}

void swap(int *a, int *b)
{
    int tmp = *a;
    *a = *b;
    *b = tmp;
}
```

在使用了指针之后，我们可以形象化这段代码为下图。

![[image 11.png]]

# 溢出

堆溢出和栈溢出。

# scanf

```C
// Gets an int from user using scanf

#include <stdio.h>

int main(void)
{
    int n;
    printf("n: ");
    scanf("%i", &n);
    printf("n: %i\n", n);
}
```

使用&n来存储scanf获取的数值。

```C
// Dangerously gets a string from user using scanf with array

#include <stdio.h>

int main(void)
{
    char s[4];
    printf("s: ");
    scanf("%s", s);
    printf("s: %s\n", s);
}
```

字符串本身就是一个指针，所以这里不再需要&符号。但是这里可能还是会有一些问题，因为你的字符串被设定为4，但是你输入的字符串可能大小并不是为4.

```C
// Using malloc

#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *s = malloc(4);
    if (s == NULL)
    {
        return 1;
    }
    printf("s: ");
    scanf("%s", s);
    printf("s: %s\n", s);
    free(s);
    return 0;
}
```

# Files I/O

```C
// Saves names and numbers to a CSV file

#include <cs50.h>
#include <stdio.h>
#include <string.h>

int main(void)
{
    // Open CSV file
    FILE *file = fopen("phonebook.csv", "a");

    // Get name and number
    char *name = get_string("Name: ");
    char *number = get_string("Number: ");

    // Print to file
    fprintf(file, "%s,%s\n", name, number);

    // Close file
    fclose(file);
}
```

fopen函数用来打开一个文件。

fprintf用来将东西写入你打开的文件。

fread可以读取文件中的内容并写进内存。

fwrite可以将内存中的信息写入文件中。