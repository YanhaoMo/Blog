---
title: Hello Linux Module
date: 2017-03-01 11:00:51
tags: kernel
---

Linux的宏内核架构使得内核的体积越来越大，为了解决这个问题，内核使用了一种模块机制。模块作为内核的可以被动态装入与卸出的组成部分，可以在需要时动态的链接到运行中的内核中，而在不需要时又从内核中卸出。因此模块的使用大大减少了Linux内核的体积。
正因为内核模块在运行时是动态链接到当前运行的内核中的，因此模块是内核的一部分，而不是一个独立的程序，模块开发在一定意义上来说就是内核开发。
本文不就Linux内核模块的实现机制做介绍，而只是给出内核模块的完整模型描述，然后以一个具体的helloworkd模块来使读者实际感受内核模块的编写，编译和运行。
<!-- more -->

一个完整的内核模块的编写，应遵循以下规则：
首先，内核模块的源文件必须包括linux/module.h头文件。
其次，模块需要一个载入函数和一个卸出函数。顾名思义，这两个函数分别在该模块被载入和写出当前内核时执行。这两个函数的默认函原型分别为`int init_module(void)`和`void cleanup_module(void)`，你也可以不使用这两个函数名，但是此时要通过函数`module_init(/*载入函数名*/)`和`module_exit(/*卸出函数名*/)`来显式指定该模块的卸任卸出函数。
接下来我们来看一个简单的内核模块的例子：

本文的实验环境是ArchLinux，内核版本为13.16.1,是当前内核的最新稳定版
```
$ uname -a
$ Linux local 3.16.1-1-ARCH #1 SMP PREEMPT Thu Aug 14 07:40:19 CEST 2014 x86_64 GNU/Linux
```

注：命令前的`$`表示此条命令是以普通用户身份运行

ArchLinux默认不会在/usr/src目录下放任何东西，所以我们需要从服务器下载Linux内核的源代码
```
$ wget ftp://ftp.kernel.org/pub/linux/kernel/v3.x/linux-3.16.1.tar.xz
```

为了安全起见，我们把源代码放解压到普通用户的家目录下
```
$ tar -xvJpf linux-3.16.1.tar.xz -C ~/
```

然后将当前内核的配置拷贝并重命名为.config到入源代码的顶层目录，执行如下命令
```
$ make oldconfig
```

因为内核模块在编译时需要使用源代码目录中的Module.symvers文件（若缺少这个文件，模块也可以编译成功，但是无法载入当前内核），而这个文件又只能在编译内核时产生，所以我们不得不重头编译一遍内核（内核只需编译这一次就好，并不需要每次编译模块时都进行内核的编译，我们的目的只是产生Module.symvers文件）
```
$ make
```

等待内核编译完成后，我们可以看到内核源代码的顶层目录产生了Module.symvers文件

接下来我们要做的就是编写模块源代码已经Makefile文件了：新建一个目录（随便什么地方都好，这里我们就把它建到家目录好了），然后在目录下新建源代码文件和Makefile文件

下面是源代码文件hello.c和Makefile文件的内容：
hello.c

```c
#include <linux/module.h>

static int init_module_hello(void)
{

printk("Hello World!\n");
return 0;
}

static void cleanup_module_hello(void)
{

printk("Goodbye!\n");
}
module_init(init_module_hello);
module_exit(cleanup_module_hello);

MODULE_LICENSE("Dual BSD/GPL");
```
Makefile:
```make
TARGET = hello
KDIR = ../linux-3.16.1/
PWD = $(shell pwd)
obj-m += $(TARGET).o
default:
make -C $(KDIR) M=$(PWD) modules
clean:
rm Module.symvers *.o *mod.c *.order *.ko
```

文件编写完成后保存，执行make命令
```
$ make
```

命令执行完成后我们会得到helo.ko文件，这就是我们要的模块文件，执行以下命令可以看到模块的输出
```
$ sudo insmod hello.ko
$ sudo rmmod hello.ko
$ dmesg
```

后记：本文三年前写于三年前，现重新发布在这儿，有些内容或许已经不正确
