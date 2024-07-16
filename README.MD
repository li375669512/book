# C语言

## C语言小技能

### sprintf指定宽度输出

sprintf函数可用*号指定字符串的长度

```c
int width = 8;
char*buf = new char[width+1];
sprintf(buf, "%0*d", width, 1);
cout << buf << endl;
delete buf;
```
```
输出结果:
00000001
```

### offsetof与container_of宏

**1. 前言**

　　今天在看代码时，遇到offsetof和container_of两个宏，觉得很有意思，功能很强大。offsetof是用来判断结构体中成员的偏移位置，container_of宏用来根据成员的地址来获取结构体的地址。两个宏设计的很巧妙，值得学习。linux内核中有着两个宏的定义，并在链表结构中得到应用。不得不提一下linux内核中的链表，设计的如此之妙，只需要两个指针就搞定了。后续认真研究一下这个链表结构。

**2. offsetof宏**

　　使用`offsetof`宏需要包含`stddef.h`头文件，实例可以参考：http://www.cplusplus.com/reference/cstddef/offsetof/。

`offsetof`宏的定义如下：
```C
#define offsetof(type, member) (size_t)&(((type*)0)->member)
```
　　巧妙之处在于将地址0强制转换为`type`类型的指针，从而定位到`member`在结构体中偏移位置。编译器认为`0`是一个有效的地址，从而认为`0`是`type`指针的起始地址。

**3. container_of宏**

　　使用`container_of`宏需要包含`linux/kernel.h`头文件，`container_of`宏的定义如下所示：
```C
#define container_of(ptr, type, member) ({ \
     const typeof( ((type *)0)->member ) *__mptr = (ptr); \
     (type *)( (char *)__mptr - offsetof(type,member) );})  
```  
`container_of`宏分为两部分，

第一部分：`const typeof( ((type *)0)->member ) *__mptr = (ptr)`;

通过typeof定义一个`member`指针类型的指针变量`__mptr`，（即`__mptr`是指向`member`类型的指针），并将`__mptr`赋值为`ptr`。

第二部分： `(type *)( (char *)__mptr - offsetof(type,member) )`，通过`offsetof`宏计算出`member`在`type`中的偏移，然后用`member`的实际地址`__mptr`减去偏移，得到`type`的起始地址，即指向`type`类型的指针。

第一部分的目的是为了将统一转换为`member`类型指针。

**4. 测试程序**

```C
#include <stdio.h>
#include <stdlib.h>

#define NAME_STR_LEN  32

#define offsetof(type, member) (size_t)&(((type*)0)->member)

#define container_of(ptr, type, member) ({ \
        const typeof( ((type *)0)->member ) *__mptr = (ptr); \
        (type *)( (char *)__mptr - offsetof(type,member) );})

typedef struct student_info
{
    int  id;
    char name[NAME_STR_LEN];
    int  age;
}student_info;


int main()
{
    size_t off_set = 0;
    off_set = offsetof(student_info, id);
    printf("id offset: %u\n",off_set);
    off_set = offsetof(student_info, name);
    printf("name offset: %u\n",off_set);
    off_set = offsetof(student_info, age);
    printf("age offset: %u\n",off_set);
    student_info *stu = (student_info *)malloc(sizeof(student_info));
    stu->age = 10;
    student_info *ptr = container_of(&(stu->age), student_info, age);
    printf("age:%d\n", ptr->age);
    printf("stu address:%p\n", stu);
    printf("ptr address:%p\n", ptr);
    return 0;
}
```

