---
title: "（译）Hello World 中的 bug"
date: "2022-03-26"
categories: 
  - "技"
tags: 
  - "programing"
---

[原文链接：Bugs in Hello World](https://blog.sunfishcode.online/bugs-in-hello-world/)

Hello World 可能是被写过最多次的计算机程序了。在过去的几十年间，它都是很多人接触一门新语言时写下的第一段程序。

所以，这段简明的入门程序肯定是没 bug 的，对吧？

毕竟这个 hello world 就做了一件事，怎么可能会有 bug？

![d](https://raw.githubusercontent.com/catbaron0/blog/master/images/2022326142649.png)

# Hello world 的 C 语言实现

Hello World 在 C 语言中有很多种写法。有[维基百科的版本](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program#C)，[有 K&R 的版本](https://riptutorial.com/c/example/3675/original--hello--world---in-k-r-c)，[甚至有现存已知最早的 1974 年版本](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program#History)。

![](https://raw.githubusercontent.com/catbaron0/blog/master/images/2022326143713.png)

这里是另一个，[用 ANSI C 写成](http://helloworldcollection.de/#C%C2%A0(ANSI))。

```
/* Hello World in C, Ansi-style */

#include <stdio.h>
#include <stdlib.h>

int main(void)
{
  puts("Hello World!");
  return EXIT_SUCCESS;
}
```

这应该是这里写的最小心的一个版本了。它使用 `(void)` 来保证 `main` 函数的声明是 new-style 风格。它没用假设所在平台会使用 `0` 作为成功返回代码，而是显式地返回了宏定义 `EXIT_SUCCESS`。根据 C 语言标准，这并非必须，但这样写更安全。除此之外，它还使用了恰当的头文件来避免隐式重定义 `puts` 函数。这个版本把该做的都做了。

但是，它还是有个 bug。

# 有 bug ？

Linux 系统有一个很有趣的设备文件，叫做 `/def/full`。有些类似它的著名远亲 `/def/null`，但是当你向它写入数据的时候，它并不会丢掉数据，而是会产生一次空间已满的失败。

```sh
$ echo "Hello World!" > /dev/full
bash: echo: write error: No space left on device
$ echo $?
1
```

> 注：这里使用 `echo $?` 来显示上一条命令的返回值

它是一个很好的工具，可以用来测试程序是否能够正确处理 `I/O` 错误。要真的创造一个没有剩余空间的磁盘或者文件系统就太费事了，直接往 `/dev/full` 设备写入数据来查看结果要简单得多。

然后这事我们上面的 C 语言程序的测试结果：

```sh
$ gcc hello.c -o hello
$ ./hello > /dev/full
$ echo $?
0
```

不像之前`shell`命令中的 `echo`，这里我们没有得到任何错误输出，而且返回代码是零。这代表 `hello` 程序向我们报告说它被正确执行了，但事实上并不是这样。我们可以用 `trace` 来确认它实际产生的错误：

```sh
$ strace -etrace=write ./hello > /dev/full
write(1, "Hello World!\n", 13) = -1 ENOSPC (No space left on device)
+++ exited with 0 +++
```

这就是我们的操作系统报出的 `空间不足` 错误。尽管如此，我们的程序却默默地吞掉了错误消息并且返回了正确代码`0`。这就是那个 bug。

这个 bug 严重么？我们可以说 hello world 程序在哪都不会有什么大的安全问题，但是它也确实会做一些事情：像标准输出打印。有时这个标准输出会被重定向到一个文件，而在现实环境中文件可能会耗尽空间。如果一个程序无法检测到、并通过返回代码来报告这种错误父进程就无法获知子进程的失败，并且好像无事发生一般继续执行，尽管它所预期的输出数据都被丢掉了。

比如我们假定有一个程序会向标准输出写入一个 [yaml](https://yaml.org/) 配置文件。如果标准输出空间不足，输出内容可能会在任意节点被截断，而已有的内容也许依然是一个[合法的 yaml 文件](https://www.sqlservercentral.com/editorials/do-you-have-all-the-yaml)。所以我们的程序应该能够检测并报告这种情况。

# 其他语言怎么样？

我们刚刚看了 `bash` 和 `C` 语言，那声称“所有错误都不该被无声传递”的 `Python` 怎么样？这是 `Python 2`:

```sh
$ python2 hello.py > /dev/full
close failed in file object destructor:
sys.excepthook is missing
lost sys.stderr
$ echo $?
0
```

它确实会打印一段错误信息，尽管这段信息本身有点令人费解。不过它还是返回了 `0`，任何执行它的进程都会被告知它正确地退出了。

幸运的是 `Pyhon 3` 恰当地报告的这个错误，而且输出了正确的错误信息：

```sh
$ python3 hello.py > /dev/full
Exception ignored in: <_io.TextIOWrapper name='<stdout>' mode='w' encoding='utf-8'>
OSError: [Errno 28] No space left on device
$ echo $?
120
```

我还测试了一些常见的语言教程网站的 hello world 程序，结果如下：

| Language | Has the bug | Versions tested |
| --- | --- | --- |
| C | Yes | (all) |
| C++ | Yes | (all) |
| Python 2 | Yes | Python 2.7.18 |
| Ruby | Yes | ruby 2.7.2p137 (2020-10-01 revision 5445e04352) \[x86\_64-linux-gnu\] |
| Java | Yes | openjdk 11.0.11 2021-04-20 |
| Node.js | Yes | v12.21.0 |
| Haskell | Yes | "The Glorious Glasgow Haskell Compilation System |
| Rust | No | rustc 1.59.0 (9d1b2106e 2022-02-23) |
| Python 3 | No | Python 3.9.5 |
| Perl | No | "perl 5 |
| Perl 6 | No | v2020.12 |
| Bash | No | "GNU bash |
| Awk | No | "GNU Awk 5.1.0 |
| OCaml | No | 4.08.1 |
| Tcl | No | 8.6.11 |
