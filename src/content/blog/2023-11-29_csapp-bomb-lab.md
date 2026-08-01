---
title: "CSAPP｜Bomb Lab"
description: "CSAPP第二次实验任务，拆炸弹💣"
publishDate: 2023-11-29
coverImage:
  src: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTAtaE97y5UzxCFBHOAZ4fuhjBAe7-0dJi6CrZCY-oSOg&s=10"
  alt: "CSAPP"
tags: ["course"]
---

> 系WHU2021级《系统级程序设计》实验1，节选于CSAPP Lab。

# 引言

## 任务介绍

本实验要求你使用课程所学知识拆除“binary bombs”，增强对程序的机器级表示、汇编语言、调试器和逆向工程等方面原理与技能的掌握。

 一个“binary bombs”（二进制炸弹，下文将简称为炸弹）是一个Linux可执行程序，包含了6个阶段（或层次、关卡）。炸弹运行的每个阶段要求你输入一个特定字符串，你的输入符合程序预期的输入，该阶段的炸弹就被拆除引信即解除了，否则炸弹“爆炸”打印输出 "BOOM!!!"。

实验的目标是拆除尽可能多的炸弹层次。 每个炸弹阶段考察了机器级程序语言的一个不同方面，难度逐级递增：

- 阶段1：字符串比较
- 阶段2：循环
- 阶段3：条件/分支
- 阶段4：递归调用和栈
- 阶段5：指针
- 阶段6：链表/指针/结构

另外还有一个隐藏阶段，只有当你在第4阶段的解后附加一特定字符串后才会出现。

为完成二进制炸弹拆除任务，你需要使用gdb调试器和objdump来反汇编炸弹的可执行文件并跟踪调试每一阶段的机器代码，从中理解每一汇编语言代码的行为或作用，进而设法推断拆除炸弹所需的目标字符串。（比如在每一阶段的开始代码前和引爆炸弹的函数前设置断点）

实验语言：C；实验环境：Linux

## 实验步骤

### 第一步：获取Bomb

在*远程桌面*的浏览器中打开 `http://172.16.2.207:15213` （或打开桌面上的Bomblab Download Page），在二进制炸弹请求表格中输入你的学号和邮箱地址，点击Submit按钮。服务器会构造属于你的炸弹，并以tar文件的形式`bombXXX.tar`返回给你，其中XXX是一个你的bomb的唯一标识。

解压该tar文件（`tar -xvf bombX.tar`）得到一个目录`./bombXXX`，其中包含如下文件：

- README：标识该bomb和所有者。
- bomb：bomb的可执行程序。
- bomb.c：bomb程序的main函数。

### 第二步：拆除Bomb

本实验的任务就是拆除炸弹。一定要在指定的虚拟机上完成作业，在其他的环境上运行有可能导致失败。

运行`./bomb`可执行程序需要0或1个命令行参数（详见`bomb.c`源文件中的`main()`函数）。如果运行时不指定参数，则该程序打印出欢迎信息后，期望你按行输入每一阶段用来拆除炸弹的字符串，根据你当前输入的字符串决定你是通过相应阶段还是炸弹爆炸导致任务失败。

你也可将拆除每一阶段炸弹的字符串按行组织在一个文本文件中，然后作为运行程序时的唯一一个命令行参数传给程序，程序读入文件中的每一行直到遇到EOF，再转到从stdin等待输入。这样对于你已经拆除的炸弹，就不用每次都重新输入，只用放进文件里即可。

前四个阶段每个10分，第五和第六阶段更难一些，每个15分，满分70分。每输入错误一次，炸弹爆炸，会扣除0.5分（最多扣除20分）。所以你必须小心！要学会单步跟踪调试汇编代码以及学会设置断点。你还要学会如何检查寄存器和内存状态。很好的使用调试器是你在未来的职业生涯中赚到更多money的一项重要技能！

### 第三步：提交结果

这是一项独立实验，每个人单独完成。bomb程序会自动发送结果到服务器，可以在`http://172.16.2.207:15213/scoreboard`（或打开桌面上的Bomblab Scoreboard）查看所有人的成绩结果。

由于不同班要求不同，不用点击“提交评测”，以scoreboard分数为准。

## 提示

下面简要说明完成本实验所需要的一些实验工具：

### **gdb**

为了从二进制可执行程序`./bomb`中找出触发bomb爆炸的条件，可使用`gdb`来帮助对程序的分析。GDB是GNU自由软件组织发布的一个强大的交互式程序调试工具。一般来说，GDB主要帮忙你完成下面几方面的功能（更详细描述可参看GDB文档和相关资料）：

- 装载、启动被调试的程序。
- 让被调试的程序在你指定的调试断点处中断执行，方便查看程序变量、寄存器、栈内容等运行现场数据。
- 动态改变程序的执行环境，如修改变量的值。

gdb相关资料：

[GDB Cheat Sheet](https://cslabcg.whu.edu.cn/userfiles/markdown/exp/2023_5/653ll1684482289.pdf)

[http://beej.us/guide/bggdb/](http://beej.us/guide/bggdb/)

[https://www.gnu.org/software/gdb/](https://www.gnu.org/software/gdb/)

### **objdump**

- `objdump –t`
    
    该命令可以打印出bomb的符号表。符号表包含了bomb中所有函数、全局变量的名称和存储地址。你可以通过查看函数名得到一些目标程序的信息。
    
- `objdump –d`
    
    该命令可用来对bomb中的二进制代码进行反汇编。通过阅读汇编源代码可以发现bomb是如何运行的。但该命令不能告诉你bomb的所有信息，例如一个调用sscanf函数的语句可能显示为： `8048c36: e8 99 fc ff ff call 80488d4 <_init+0x1a0>` ，你还需要gdb来帮助你确定这个语句的具体功能。
    

### **strings**

该命令可以显示二进制程序中的所有可打印字符串。

## **实验步骤提示**

下面以第一阶段（第一关）为例介绍实验步骤：首先调用`objdump –d bomb > bomb_disas.txt` 对bomb进行反汇编并将汇编源代码输出到`boomb_disas.txt`文本文件中。查看该汇编源代码文件，我们可以在main函数中找到如下语句，从而得知第一关的处理程序包含在`main()`函数所调用的函数`phase_1()`中，判断的过程可以参照bomb.c文件源码。汇编代码中地址0x14e8处调用了`phase_1`函数，

![](https://cslabcg.whu.edu.cn/userfiles/markdown/exp/2021_7/748ll1625657284.png)

我们在反汇编代码中寻找这个子函数`phase_1`：

![](https://cslabcg.whu.edu.cn/userfiles/markdown/exp/2021_7/748ll1625657291.png)

可以看到这个子函数比较小，只有几行汇编代码，可以进行简单阅读（如果汇编代码较多，不建议逐句阅读，而是借用gdb调试工具进行辅助）：我们看到（教科书中已经提到过调用函数的过程），………， 还调用了`string_not_equal`函数，接着测试`%eax`是否为零，如果是就跳转到`+0x1d`处，否则就调用`explode_bomb`，可以判定这是一个判断两个字符串是否相等的过程。 接下来，使用gdb调试bomb二进制文件：`gdb bomb`后，运行`break phase_1`，也就是在`phase_1`函数处设置一个断点，然后运行`run`，开始调试。运行`disa phase_1`，得到如下信息：

![](https://cslabcg.whu.edu.cn/userfiles/markdown/exp/2021_7/748ll1625657319.png)

可以看到`rsi`中装入的就是待比较的目标字符串地址，后面给出的0x555555557150就是计算出来的值。可以输入`print (char *)0x555555557150`，输出是

![](https://cslabcg.whu.edu.cn/userfiles/markdown/exp/2021_7/748ll1625657340.png)

也可以`stepi`运行到0x00005555555555f6，然后`print /x $rsi`得到，

![](https://cslabcg.whu.edu.cn/userfiles/markdown/exp/2021_7/748ll1625657354.png)

说明%rsi的值确实是0x555555557150，目标字符串的起始位置。 接下来，去设置断点去检测这个答案是否正确，我们kill当前程序的调试。再在`explode_bomb`处设置断点`break explode_bomb`，然后`run`开始运行，按照提示输入这个字符串：“When I get angry, Mr. Bigglesworth gets upset.”（不包括引号）

![](https://cslabcg.whu.edu.cn/userfiles/markdown/exp/2021_7/748ll1625657365.png)

第一关解除。

**特别注意**

不要试图去修改发送给服务器端的数据，一旦服务器端检测到错误数据，会自动记录invalid成绩，后续提交的结果及时正确也将无法记录。

## **参考解说视频**

这里还有一段对实验操作的解说视频，供大家参考： 链接:

 [https://pan.baidu.com/s/1Syja0lELxyrTJsCrrmkSKw](https://pan.baidu.com/s/1Syja0lELxyrTJsCrrmkSKw) 

提取码: 9v9s

# 实验记录

| phase | 答案字符串 | 备注 |
| --- | --- | --- |
| 1 字符串比较 | The future will be better tomorow. |  |
| 2 循环 | 0 1 3 6 10 15 |  |
| 3 条件/分支switch | 1 r461 | 每个分支都对应一个答案 |
| 4 递归 | 5 2 DrEvil | DrEvil用于触发secret phase |
| 5 指针 | 0 48 |  |
| 6 链表/指针/结构 | 4 5 3 1 2 6 |  |
| secret 二叉树 | 1001 |  |

:::note
调试操作：

```bash
gdb bomb  # 开始调试
break phase_1 # 目标函数断点
break explode_bomb # 爆炸函数断点
run ./ans.txt # 指定参数答案文件，跳过已经完成的任务
####
nexti  # 疯狂步过调试，根据地址跟踪运行情况
```

基本思路：盯着`callq 401689 <explode_bomb>`语句，拐弯不要进去，慢慢凑出答案
:::

<details>
<summary>附：Bomb文件 (文件很长，谨慎点开)</summary>

<details>
<summary>`bomb.c`</summary>

```c
/***************************************************************************
 * Dr. Evil's Insidious Bomb, Version 1.1
 * Copyright 2011, Dr. Evil Incorporated. All rights reserved.
 *
 * LICENSE:
 *
 * Dr. Evil Incorporated (the PERPETRATOR) hereby grants you (the
 * VICTIM) explicit permission to use this bomb (the BOMB).  This is a
 * time limited license, which expires on the death of the VICTIM.
 * The PERPETRATOR takes no responsibility for damage, frustration,
 * insanity, bug-eyes, carpal-tunnel syndrome, loss of sleep, or other
 * harm to the VICTIM.  Unless the PERPETRATOR wants to take credit,
 * that is.  The VICTIM may not distribute this bomb source code to
 * any enemies of the PERPETRATOR.  No VICTIM may debug,
 * reverse-engineer, run "strings" on, decompile, decrypt, or use any
 * other technique to gain knowledge of and defuse the BOMB.  BOMB
 * proof clothing may not be worn when handling this program.  The
 * PERPETRATOR will not apologize for the PERPETRATOR's poor sense of
 * humor.  This license is null and void where the BOMB is prohibited
 * by law.
 ***************************************************************************/

#include <stdio.h>
#include <stdlib.h>
#include "support.h"
#include "phases.h"

/* 
 * Note to self: Remember to erase this file so my victims will have no
 * idea what is going on, and so they will all blow up in a
 * spectaculary fiendish explosion. -- Dr. Evil 
 */

FILE *infile;

int main(int argc, char *argv[])
{
    char *input;

    /* Note to self: remember to port this bomb to Windows and put a 
     * fantastic GUI on it. */

    /* When run with no arguments, the bomb reads its input lines 
     * from standard input. */
    if (argc == 1) {  
	infile = stdin;
    } 

    /* When run with one argument <file>, the bomb reads from <file> 
     * until EOF, and then switches to standard input. Thus, as you 
     * defuse each phase, you can add its defusing string to <file> and
     * avoid having to retype it. */
    else if (argc == 2) {
	if (!(infile = fopen(argv[1], "r"))) {
	    printf("%s: Error: Couldn't open %s\n", argv[0], argv[1]);
	    exit(8);
	}
    }

    /* You can't call the bomb with more than 1 command line argument. */
    else {
	printf("Usage: %s [<input_file>]\n", argv[0]);
	exit(8);
    }

    /* Do all sorts of secret stuff that makes the bomb harder to defuse. */
    initialize_bomb();

    printf("Welcome to my fiendish little bomb. You have 6 phases with\n");
    printf("which to blow yourself up. Have a nice day!\n");

    /* Hmm...  Six phases must be more secure than one phase! */
    input = read_line();             /* Get input                   */
    phase_1(input);                  /* Run the phase               */
    phase_defused();                 /* Drat!  They figured it out!
				      * Let me know how they did it. */
    printf("Phase 1 defused. How about the next one?\n");

    /* The second phase is harder.  No one will ever figure out
     * how to defuse this... */
    input = read_line();
    phase_2(input);
    phase_defused();
    printf("That's number 2.  Keep going!\n");

    /* I guess this is too easy so far.  Some more complex code will
     * confuse people. */
    input = read_line();
    phase_3(input);
    phase_defused();
    printf("Halfway there!\n");

    /* Oh yeah?  Well, how good is your math?  Try on this saucy problem! */
    input = read_line();
    phase_4(input);
    phase_defused();
    printf("So you got that one.  Try this one.\n");
    
    /* Round and 'round in memory we go, where we stop, the bomb blows! */
    input = read_line();
    phase_5(input);
    phase_defused();
    printf("Good work!  On to the next...\n");

    /* This phase will never be used, since no one will get past the
     * earlier ones.  But just in case, make this one extra hard. */
    input = read_line();
    phase_6(input);
    phase_defused();

    /* Wow, they got it!  But isn't something... missing?  Perhaps
     * something they overlooked?  Mua ha ha ha ha! */
    
    return 0;
}
```

</details>

<details>
<summary>反汇编文件 `bomb_disas`</summary>

```asm
bomb:     file format elf64-x86-64

Disassembly of section .init:

0000000000400ad0 <_init>:
  400ad0:	48 83 ec 08          	sub    $0x8,%rsp
  400ad4:	48 8b 05 1d 35 20 00 	mov    0x20351d(%rip),%rax        # 603ff8 <_DYNAMIC+0x1d0>
  400adb:	48 85 c0             	test   %rax,%rax
  400ade:	74 05                	je     400ae5 <_init+0x15>
  400ae0:	e8 0b 01 00 00       	callq  400bf0 <__gmon_start__@plt>
  400ae5:	48 83 c4 08          	add    $0x8,%rsp
  400ae9:	c3                   	retq   

Disassembly of section .plt:

0000000000400af0 <getenv@plt-0x10>:
  400af0:	ff 35 12 35 20 00    	pushq  0x203512(%rip)        # 604008 <_GLOBAL_OFFSET_TABLE_+0x8>
  400af6:	ff 25 14 35 20 00    	jmpq   *0x203514(%rip)        # 604010 <_GLOBAL_OFFSET_TABLE_+0x10>
  400afc:	0f 1f 40 00          	nopl   0x0(%rax)

0000000000400b00 <getenv@plt>:
  400b00:	ff 25 12 35 20 00    	jmpq   *0x203512(%rip)        # 604018 <_GLOBAL_OFFSET_TABLE_+0x18>
  400b06:	68 00 00 00 00       	pushq  $0x0
  400b0b:	e9 e0 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400b10 <strcasecmp@plt>:
  400b10:	ff 25 0a 35 20 00    	jmpq   *0x20350a(%rip)        # 604020 <_GLOBAL_OFFSET_TABLE_+0x20>
  400b16:	68 01 00 00 00       	pushq  $0x1
  400b1b:	e9 d0 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400b20 <__errno_location@plt>:
  400b20:	ff 25 02 35 20 00    	jmpq   *0x203502(%rip)        # 604028 <_GLOBAL_OFFSET_TABLE_+0x28>
  400b26:	68 02 00 00 00       	pushq  $0x2
  400b2b:	e9 c0 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400b30 <strcpy@plt>:
  400b30:	ff 25 fa 34 20 00    	jmpq   *0x2034fa(%rip)        # 604030 <_GLOBAL_OFFSET_TABLE_+0x30>
  400b36:	68 03 00 00 00       	pushq  $0x3
  400b3b:	e9 b0 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400b40 <puts@plt>:
  400b40:	ff 25 f2 34 20 00    	jmpq   *0x2034f2(%rip)        # 604038 <_GLOBAL_OFFSET_TABLE_+0x38>
  400b46:	68 04 00 00 00       	pushq  $0x4
  400b4b:	e9 a0 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400b50 <write@plt>:
  400b50:	ff 25 ea 34 20 00    	jmpq   *0x2034ea(%rip)        # 604040 <_GLOBAL_OFFSET_TABLE_+0x40>
  400b56:	68 05 00 00 00       	pushq  $0x5
  400b5b:	e9 90 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400b60 <printf@plt>:
  400b60:	ff 25 e2 34 20 00    	jmpq   *0x2034e2(%rip)        # 604048 <_GLOBAL_OFFSET_TABLE_+0x48>
  400b66:	68 06 00 00 00       	pushq  $0x6
  400b6b:	e9 80 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400b70 <alarm@plt>:
  400b70:	ff 25 da 34 20 00    	jmpq   *0x2034da(%rip)        # 604050 <_GLOBAL_OFFSET_TABLE_+0x50>
  400b76:	68 07 00 00 00       	pushq  $0x7
  400b7b:	e9 70 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400b80 <close@plt>:
  400b80:	ff 25 d2 34 20 00    	jmpq   *0x2034d2(%rip)        # 604058 <_GLOBAL_OFFSET_TABLE_+0x58>
  400b86:	68 08 00 00 00       	pushq  $0x8
  400b8b:	e9 60 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400b90 <read@plt>:
  400b90:	ff 25 ca 34 20 00    	jmpq   *0x2034ca(%rip)        # 604060 <_GLOBAL_OFFSET_TABLE_+0x60>
  400b96:	68 09 00 00 00       	pushq  $0x9
  400b9b:	e9 50 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400ba0 <__libc_start_main@plt>:
  400ba0:	ff 25 c2 34 20 00    	jmpq   *0x2034c2(%rip)        # 604068 <_GLOBAL_OFFSET_TABLE_+0x68>
  400ba6:	68 0a 00 00 00       	pushq  $0xa
  400bab:	e9 40 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400bb0 <fgets@plt>:
  400bb0:	ff 25 ba 34 20 00    	jmpq   *0x2034ba(%rip)        # 604070 <_GLOBAL_OFFSET_TABLE_+0x70>
  400bb6:	68 0b 00 00 00       	pushq  $0xb
  400bbb:	e9 30 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400bc0 <signal@plt>:
  400bc0:	ff 25 b2 34 20 00    	jmpq   *0x2034b2(%rip)        # 604078 <_GLOBAL_OFFSET_TABLE_+0x78>
  400bc6:	68 0c 00 00 00       	pushq  $0xc
  400bcb:	e9 20 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400bd0 <gethostbyname@plt>:
  400bd0:	ff 25 aa 34 20 00    	jmpq   *0x2034aa(%rip)        # 604080 <_GLOBAL_OFFSET_TABLE_+0x80>
  400bd6:	68 0d 00 00 00       	pushq  $0xd
  400bdb:	e9 10 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400be0 <fprintf@plt>:
  400be0:	ff 25 a2 34 20 00    	jmpq   *0x2034a2(%rip)        # 604088 <_GLOBAL_OFFSET_TABLE_+0x88>
  400be6:	68 0e 00 00 00       	pushq  $0xe
  400beb:	e9 00 ff ff ff       	jmpq   400af0 <_init+0x20>

0000000000400bf0 <__gmon_start__@plt>:
  400bf0:	ff 25 9a 34 20 00    	jmpq   *0x20349a(%rip)        # 604090 <_GLOBAL_OFFSET_TABLE_+0x90>
  400bf6:	68 0f 00 00 00       	pushq  $0xf
  400bfb:	e9 f0 fe ff ff       	jmpq   400af0 <_init+0x20>

0000000000400c00 <strtol@plt>:
  400c00:	ff 25 92 34 20 00    	jmpq   *0x203492(%rip)        # 604098 <_GLOBAL_OFFSET_TABLE_+0x98>
  400c06:	68 10 00 00 00       	pushq  $0x10
  400c0b:	e9 e0 fe ff ff       	jmpq   400af0 <_init+0x20>

0000000000400c10 <memcpy@plt>:
  400c10:	ff 25 8a 34 20 00    	jmpq   *0x20348a(%rip)        # 6040a0 <_GLOBAL_OFFSET_TABLE_+0xa0>
  400c16:	68 11 00 00 00       	pushq  $0x11
  400c1b:	e9 d0 fe ff ff       	jmpq   400af0 <_init+0x20>

0000000000400c20 <fflush@plt>:
  400c20:	ff 25 82 34 20 00    	jmpq   *0x203482(%rip)        # 6040a8 <_GLOBAL_OFFSET_TABLE_+0xa8>
  400c26:	68 12 00 00 00       	pushq  $0x12
  400c2b:	e9 c0 fe ff ff       	jmpq   400af0 <_init+0x20>

0000000000400c30 <__isoc99_sscanf@plt>:
  400c30:	ff 25 7a 34 20 00    	jmpq   *0x20347a(%rip)        # 6040b0 <_GLOBAL_OFFSET_TABLE_+0xb0>
  400c36:	68 13 00 00 00       	pushq  $0x13
  400c3b:	e9 b0 fe ff ff       	jmpq   400af0 <_init+0x20>

0000000000400c40 <bcopy@plt>:
  400c40:	ff 25 72 34 20 00    	jmpq   *0x203472(%rip)        # 6040b8 <_GLOBAL_OFFSET_TABLE_+0xb8>
  400c46:	68 14 00 00 00       	pushq  $0x14
  400c4b:	e9 a0 fe ff ff       	jmpq   400af0 <_init+0x20>

0000000000400c50 <fopen@plt>:
  400c50:	ff 25 6a 34 20 00    	jmpq   *0x20346a(%rip)        # 6040c0 <_GLOBAL_OFFSET_TABLE_+0xc0>
  400c56:	68 15 00 00 00       	pushq  $0x15
  400c5b:	e9 90 fe ff ff       	jmpq   400af0 <_init+0x20>

0000000000400c60 <gethostname@plt>:
  400c60:	ff 25 62 34 20 00    	jmpq   *0x203462(%rip)        # 6040c8 <_GLOBAL_OFFSET_TABLE_+0xc8>
  400c66:	68 16 00 00 00       	pushq  $0x16
  400c6b:	e9 80 fe ff ff       	jmpq   400af0 <_init+0x20>

0000000000400c70 <sprintf@plt>:
  400c70:	ff 25 5a 34 20 00    	jmpq   *0x20345a(%rip)        # 6040d0 <_GLOBAL_OFFSET_TABLE_+0xd0>
  400c76:	68 17 00 00 00       	pushq  $0x17
  400c7b:	e9 70 fe ff ff       	jmpq   400af0 <_init+0x20>

0000000000400c80 <exit@plt>:
  400c80:	ff 25 52 34 20 00    	jmpq   *0x203452(%rip)        # 6040d8 <_GLOBAL_OFFSET_TABLE_+0xd8>
  400c86:	68 18 00 00 00       	pushq  $0x18
  400c8b:	e9 60 fe ff ff       	jmpq   400af0 <_init+0x20>

0000000000400c90 <connect@plt>:
  400c90:	ff 25 4a 34 20 00    	jmpq   *0x20344a(%rip)        # 6040e0 <_GLOBAL_OFFSET_TABLE_+0xe0>
  400c96:	68 19 00 00 00       	pushq  $0x19
  400c9b:	e9 50 fe ff ff       	jmpq   400af0 <_init+0x20>

0000000000400ca0 <sleep@plt>:
  400ca0:	ff 25 42 34 20 00    	jmpq   *0x203442(%rip)        # 6040e8 <_GLOBAL_OFFSET_TABLE_+0xe8>
  400ca6:	68 1a 00 00 00       	pushq  $0x1a
  400cab:	e9 40 fe ff ff       	jmpq   400af0 <_init+0x20>

0000000000400cb0 <__ctype_b_loc@plt>:
  400cb0:	ff 25 3a 34 20 00    	jmpq   *0x20343a(%rip)        # 6040f0 <_GLOBAL_OFFSET_TABLE_+0xf0>
  400cb6:	68 1b 00 00 00       	pushq  $0x1b
  400cbb:	e9 30 fe ff ff       	jmpq   400af0 <_init+0x20>

0000000000400cc0 <socket@plt>:
  400cc0:	ff 25 32 34 20 00    	jmpq   *0x203432(%rip)        # 6040f8 <_GLOBAL_OFFSET_TABLE_+0xf8>
  400cc6:	68 1c 00 00 00       	pushq  $0x1c
  400ccb:	e9 20 fe ff ff       	jmpq   400af0 <_init+0x20>

Disassembly of section .text:

0000000000400cd0 <_start>:
  400cd0:	31 ed                	xor    %ebp,%ebp
  400cd2:	49 89 d1             	mov    %rdx,%r9
  400cd5:	5e                   	pop    %rsi
  400cd6:	48 89 e2             	mov    %rsp,%rdx
  400cd9:	48 83 e4 f0          	and    $0xfffffffffffffff0,%rsp
  400cdd:	50                   	push   %rax
  400cde:	54                   	push   %rsp
  400cdf:	49 c7 c0 70 24 40 00 	mov    $0x402470,%r8
  400ce6:	48 c7 c1 00 24 40 00 	mov    $0x402400,%rcx
  400ced:	48 c7 c7 bd 0d 40 00 	mov    $0x400dbd,%rdi
  400cf4:	e8 a7 fe ff ff       	callq  400ba0 <__libc_start_main@plt>
  400cf9:	f4                   	hlt    
  400cfa:	66 0f 1f 44 00 00    	nopw   0x0(%rax,%rax,1)

0000000000400d00 <deregister_tm_clones>:
  400d00:	b8 87 47 60 00       	mov    $0x604787,%eax
  400d05:	55                   	push   %rbp
  400d06:	48 2d 80 47 60 00    	sub    $0x604780,%rax
  400d0c:	48 83 f8 0e          	cmp    $0xe,%rax
  400d10:	48 89 e5             	mov    %rsp,%rbp
  400d13:	77 02                	ja     400d17 <deregister_tm_clones+0x17>
  400d15:	5d                   	pop    %rbp
  400d16:	c3                   	retq   
  400d17:	b8 00 00 00 00       	mov    $0x0,%eax
  400d1c:	48 85 c0             	test   %rax,%rax
  400d1f:	74 f4                	je     400d15 <deregister_tm_clones+0x15>
  400d21:	5d                   	pop    %rbp
  400d22:	bf 80 47 60 00       	mov    $0x604780,%edi
  400d27:	ff e0                	jmpq   *%rax
  400d29:	0f 1f 80 00 00 00 00 	nopl   0x0(%rax)

0000000000400d30 <register_tm_clones>:
  400d30:	b8 80 47 60 00       	mov    $0x604780,%eax
  400d35:	55                   	push   %rbp
  400d36:	48 2d 80 47 60 00    	sub    $0x604780,%rax
  400d3c:	48 c1 f8 03          	sar    $0x3,%rax
  400d40:	48 89 e5             	mov    %rsp,%rbp
  400d43:	48 89 c2             	mov    %rax,%rdx
  400d46:	48 c1 ea 3f          	shr    $0x3f,%rdx
  400d4a:	48 01 d0             	add    %rdx,%rax
  400d4d:	48 d1 f8             	sar    %rax
  400d50:	75 02                	jne    400d54 <register_tm_clones+0x24>
  400d52:	5d                   	pop    %rbp
  400d53:	c3                   	retq   
  400d54:	ba 00 00 00 00       	mov    $0x0,%edx
  400d59:	48 85 d2             	test   %rdx,%rdx
  400d5c:	74 f4                	je     400d52 <register_tm_clones+0x22>
  400d5e:	5d                   	pop    %rbp
  400d5f:	48 89 c6             	mov    %rax,%rsi
  400d62:	bf 80 47 60 00       	mov    $0x604780,%edi
  400d67:	ff e2                	jmpq   *%rdx
  400d69:	0f 1f 80 00 00 00 00 	nopl   0x0(%rax)

0000000000400d70 <__do_global_dtors_aux>:
  400d70:	80 3d 21 3a 20 00 00 	cmpb   $0x0,0x203a21(%rip)        # 604798 <completed.6355>
  400d77:	75 11                	jne    400d8a <__do_global_dtors_aux+0x1a>
  400d79:	55                   	push   %rbp
  400d7a:	48 89 e5             	mov    %rsp,%rbp
  400d7d:	e8 7e ff ff ff       	callq  400d00 <deregister_tm_clones>
  400d82:	5d                   	pop    %rbp
  400d83:	c6 05 0e 3a 20 00 01 	movb   $0x1,0x203a0e(%rip)        # 604798 <completed.6355>
  400d8a:	f3 c3                	repz retq 
  400d8c:	0f 1f 40 00          	nopl   0x0(%rax)

0000000000400d90 <frame_dummy>:
  400d90:	48 83 3d 88 30 20 00 	cmpq   $0x0,0x203088(%rip)        # 603e20 <__JCR_END__>
  400d97:	00 
  400d98:	74 1e                	je     400db8 <frame_dummy+0x28>
  400d9a:	b8 00 00 00 00       	mov    $0x0,%eax
  400d9f:	48 85 c0             	test   %rax,%rax
  400da2:	74 14                	je     400db8 <frame_dummy+0x28>
  400da4:	55                   	push   %rbp
  400da5:	bf 20 3e 60 00       	mov    $0x603e20,%edi
  400daa:	48 89 e5             	mov    %rsp,%rbp
  400dad:	ff d0                	callq  *%rax
  400daf:	5d                   	pop    %rbp
  400db0:	e9 7b ff ff ff       	jmpq   400d30 <register_tm_clones>
  400db5:	0f 1f 00             	nopl   (%rax)
  400db8:	e9 73 ff ff ff       	jmpq   400d30 <register_tm_clones>

0000000000400dbd <main>:
  400dbd:	53                   	push   %rbx
  400dbe:	83 ff 01             	cmp    $0x1,%edi
  400dc1:	75 10                	jne    400dd3 <main+0x16>
  400dc3:	48 8b 05 be 39 20 00 	mov    0x2039be(%rip),%rax        # 604788 <stdin@@GLIBC_2.2.5>
  400dca:	48 89 05 cf 39 20 00 	mov    %rax,0x2039cf(%rip)        # 6047a0 <infile>
  400dd1:	eb 59                	jmp    400e2c <main+0x6f>
  400dd3:	48 89 f3             	mov    %rsi,%rbx
  400dd6:	83 ff 02             	cmp    $0x2,%edi
  400dd9:	75 35                	jne    400e10 <main+0x53>
  400ddb:	48 8b 7e 08          	mov    0x8(%rsi),%rdi
  400ddf:	be 90 24 40 00       	mov    $0x402490,%esi
  400de4:	e8 67 fe ff ff       	callq  400c50 <fopen@plt>
  400de9:	48 89 05 b0 39 20 00 	mov    %rax,0x2039b0(%rip)        # 6047a0 <infile>
  400df0:	48 85 c0             	test   %rax,%rax
  400df3:	75 37                	jne    400e2c <main+0x6f>
  400df5:	48 8b 53 08          	mov    0x8(%rbx),%rdx
  400df9:	48 8b 33             	mov    (%rbx),%rsi
  400dfc:	bf 92 24 40 00       	mov    $0x402492,%edi
  400e01:	e8 5a fd ff ff       	callq  400b60 <printf@plt>
  400e06:	bf 08 00 00 00       	mov    $0x8,%edi
  400e0b:	e8 70 fe ff ff       	callq  400c80 <exit@plt>
  400e10:	48 8b 36             	mov    (%rsi),%rsi
  400e13:	bf af 24 40 00       	mov    $0x4024af,%edi
  400e18:	b8 00 00 00 00       	mov    $0x0,%eax
  400e1d:	e8 3e fd ff ff       	callq  400b60 <printf@plt>
  400e22:	bf 08 00 00 00       	mov    $0x8,%edi
  400e27:	e8 54 fe ff ff       	callq  400c80 <exit@plt>
  400e2c:	e8 16 06 00 00       	callq  401447 <initialize_bomb>
  400e31:	bf 18 25 40 00       	mov    $0x402518,%edi
  400e36:	e8 05 fd ff ff       	callq  400b40 <puts@plt>
  400e3b:	bf 58 25 40 00       	mov    $0x402558,%edi
  400e40:	e8 fb fc ff ff       	callq  400b40 <puts@plt>
  400e45:	e8 b7 08 00 00       	callq  401701 <read_line>
  400e4a:	48 89 c7             	mov    %rax,%rdi
  400e4d:	e8 9e 00 00 00       	callq  400ef0 <phase_1>
  400e52:	e8 d0 09 00 00       	callq  401827 <phase_defused>
  400e57:	bf 88 25 40 00       	mov    $0x402588,%edi
  400e5c:	e8 df fc ff ff       	callq  400b40 <puts@plt>
  400e61:	e8 9b 08 00 00       	callq  401701 <read_line>
  400e66:	48 89 c7             	mov    %rax,%rdi
  400e69:	e8 9e 00 00 00       	callq  400f0c <phase_2>
  400e6e:	e8 b4 09 00 00       	callq  401827 <phase_defused>
  400e73:	bf c9 24 40 00       	mov    $0x4024c9,%edi
  400e78:	e8 c3 fc ff ff       	callq  400b40 <puts@plt>
  400e7d:	e8 7f 08 00 00       	callq  401701 <read_line>
  400e82:	48 89 c7             	mov    %rax,%rdi
  400e85:	e8 cd 00 00 00       	callq  400f57 <phase_3>
  400e8a:	e8 98 09 00 00       	callq  401827 <phase_defused>
  400e8f:	bf e7 24 40 00       	mov    $0x4024e7,%edi
  400e94:	e8 a7 fc ff ff       	callq  400b40 <puts@plt>
  400e99:	e8 63 08 00 00       	callq  401701 <read_line>
  400e9e:	48 89 c7             	mov    %rax,%rdi
  400ea1:	e8 3c 02 00 00       	callq  4010e2 <phase_4>
  400ea6:	e8 7c 09 00 00       	callq  401827 <phase_defused>
  400eab:	bf b8 25 40 00       	mov    $0x4025b8,%edi
  400eb0:	e8 8b fc ff ff       	callq  400b40 <puts@plt>
  400eb5:	e8 47 08 00 00       	callq  401701 <read_line>
  400eba:	48 89 c7             	mov    %rax,%rdi
  400ebd:	e8 77 02 00 00       	callq  401139 <phase_5>
  400ec2:	e8 60 09 00 00       	callq  401827 <phase_defused>
  400ec7:	bf f6 24 40 00       	mov    $0x4024f6,%edi
  400ecc:	e8 6f fc ff ff       	callq  400b40 <puts@plt>
  400ed1:	e8 2b 08 00 00       	callq  401701 <read_line>
  400ed6:	48 89 c7             	mov    %rax,%rdi
  400ed9:	e8 c8 02 00 00       	callq  4011a6 <phase_6>
  400ede:	e8 44 09 00 00       	callq  401827 <phase_defused>
  400ee3:	b8 00 00 00 00       	mov    $0x0,%eax
  400ee8:	5b                   	pop    %rbx
  400ee9:	c3                   	retq   
  400eea:	66 0f 1f 44 00 00    	nopw   0x0(%rax,%rax,1)

0000000000400ef0 <phase_1>:
  400ef0:	48 83 ec 08          	sub    $0x8,%rsp
  400ef4:	be e0 25 40 00       	mov    $0x4025e0,%esi
  400ef9:	e8 e0 04 00 00       	callq  4013de <strings_not_equal>
  400efe:	85 c0                	test   %eax,%eax
  400f00:	74 05                	je     400f07 <phase_1+0x17>
  400f02:	e8 82 07 00 00       	callq  401689 <explode_bomb>
  400f07:	48 83 c4 08          	add    $0x8,%rsp
  400f0b:	c3                   	retq   

0000000000400f0c <phase_2>:
  400f0c:	55                   	push   %rbp
  400f0d:	53                   	push   %rbx
  400f0e:	48 83 ec 28          	sub    $0x28,%rsp
  400f12:	48 89 e6             	mov    %rsp,%rsi
  400f15:	e8 a5 07 00 00       	callq  4016bf <read_six_numbers>
  400f1a:	83 3c 24 00          	cmpl   $0x0,(%rsp)
  400f1e:	79 24                	jns    400f44 <phase_2+0x38>
  400f20:	e8 64 07 00 00       	callq  401689 <explode_bomb>
  400f25:	eb 1d                	jmp    400f44 <phase_2+0x38>
  400f27:	89 d8                	mov    %ebx,%eax
  400f29:	03 45 fc             	add    -0x4(%rbp),%eax
  400f2c:	39 45 00             	cmp    %eax,0x0(%rbp)
  400f2f:	74 05                	je     400f36 <phase_2+0x2a>
  400f31:	e8 53 07 00 00       	callq  401689 <explode_bomb>
  400f36:	83 c3 01             	add    $0x1,%ebx
  400f39:	48 83 c5 04          	add    $0x4,%rbp
  400f3d:	83 fb 06             	cmp    $0x6,%ebx
  400f40:	75 e5                	jne    400f27 <phase_2+0x1b>
  400f42:	eb 0c                	jmp    400f50 <phase_2+0x44>
  400f44:	48 8d 6c 24 04       	lea    0x4(%rsp),%rbp
  400f49:	bb 01 00 00 00       	mov    $0x1,%ebx
  400f4e:	eb d7                	jmp    400f27 <phase_2+0x1b>
  400f50:	48 83 c4 28          	add    $0x28,%rsp
  400f54:	5b                   	pop    %rbx
  400f55:	5d                   	pop    %rbp
  400f56:	c3                   	retq   

0000000000400f57 <phase_3>:
  400f57:	48 83 ec 18          	sub    $0x18,%rsp
  400f5b:	4c 8d 44 24 08       	lea    0x8(%rsp),%r8
  400f60:	48 8d 4c 24 07       	lea    0x7(%rsp),%rcx
  400f65:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx
  400f6a:	be 2e 26 40 00       	mov    $0x40262e,%esi
  400f6f:	b8 00 00 00 00       	mov    $0x0,%eax
  400f74:	e8 b7 fc ff ff       	callq  400c30 <__isoc99_sscanf@plt>
  400f79:	83 f8 02             	cmp    $0x2,%eax
  400f7c:	7f 05                	jg     400f83 <phase_3+0x2c>
  400f7e:	e8 06 07 00 00       	callq  401689 <explode_bomb>
  400f83:	83 7c 24 0c 07       	cmpl   $0x7,0xc(%rsp)
  400f88:	0f 87 fc 00 00 00    	ja     40108a <phase_3+0x133>
  400f8e:	8b 44 24 0c          	mov    0xc(%rsp),%eax
  400f92:	ff 24 c5 40 26 40 00 	jmpq   *0x402640(,%rax,8)
  400f99:	b8 44 00 00 00       	mov    $0x44,%eax
  400f9e:	81 7c 24 08 91 02 00 	cmpl   $0x291,0x8(%rsp)
  400fa5:	00 
  400fa6:	0f 84 e8 00 00 00    	je     401094 <phase_3+0x13d>
  400fac:	e8 d8 06 00 00       	callq  401689 <explode_bomb>
  400fb1:	b8 44 00 00 00       	mov    $0x44,%eax
  400fb6:	e9 d9 00 00 00       	jmpq   401094 <phase_3+0x13d>
  400fbb:	b8 72 00 00 00       	mov    $0x72,%eax
  400fc0:	81 7c 24 08 cd 01 00 	cmpl   $0x1cd,0x8(%rsp)
  400fc7:	00 
  400fc8:	0f 84 c6 00 00 00    	je     401094 <phase_3+0x13d>
  400fce:	e8 b6 06 00 00       	callq  401689 <explode_bomb>
  400fd3:	b8 72 00 00 00       	mov    $0x72,%eax
  400fd8:	e9 b7 00 00 00       	jmpq   401094 <phase_3+0x13d>
  400fdd:	b8 59 00 00 00       	mov    $0x59,%eax
  400fe2:	81 7c 24 08 a5 00 00 	cmpl   $0xa5,0x8(%rsp)
  400fe9:	00 
  400fea:	0f 84 a4 00 00 00    	je     401094 <phase_3+0x13d>
  400ff0:	e8 94 06 00 00       	callq  401689 <explode_bomb>
  400ff5:	b8 59 00 00 00       	mov    $0x59,%eax
  400ffa:	e9 95 00 00 00       	jmpq   401094 <phase_3+0x13d>
  400fff:	b8 79 00 00 00       	mov    $0x79,%eax
  401004:	81 7c 24 08 2a 01 00 	cmpl   $0x12a,0x8(%rsp)
  40100b:	00 
  40100c:	0f 84 82 00 00 00    	je     401094 <phase_3+0x13d>
  401012:	e8 72 06 00 00       	callq  401689 <explode_bomb>
  401017:	b8 79 00 00 00       	mov    $0x79,%eax
  40101c:	eb 76                	jmp    401094 <phase_3+0x13d>
  40101e:	b8 51 00 00 00       	mov    $0x51,%eax
  401023:	81 7c 24 08 50 02 00 	cmpl   $0x250,0x8(%rsp)
  40102a:	00 
  40102b:	74 67                	je     401094 <phase_3+0x13d>
  40102d:	e8 57 06 00 00       	callq  401689 <explode_bomb>
  401032:	b8 51 00 00 00       	mov    $0x51,%eax
  401037:	eb 5b                	jmp    401094 <phase_3+0x13d>
  401039:	b8 4b 00 00 00       	mov    $0x4b,%eax
  40103e:	81 7c 24 08 76 02 00 	cmpl   $0x276,0x8(%rsp)
  401045:	00 
  401046:	74 4c                	je     401094 <phase_3+0x13d>
  401048:	e8 3c 06 00 00       	callq  401689 <explode_bomb>
  40104d:	b8 4b 00 00 00       	mov    $0x4b,%eax
  401052:	eb 40                	jmp    401094 <phase_3+0x13d>
  401054:	b8 78 00 00 00       	mov    $0x78,%eax
  401059:	81 7c 24 08 cd 03 00 	cmpl   $0x3cd,0x8(%rsp)
  401060:	00 
  401061:	74 31                	je     401094 <phase_3+0x13d>
  401063:	e8 21 06 00 00       	callq  401689 <explode_bomb>
  401068:	b8 78 00 00 00       	mov    $0x78,%eax
  40106d:	eb 25                	jmp    401094 <phase_3+0x13d>
  40106f:	b8 45 00 00 00       	mov    $0x45,%eax
  401074:	81 7c 24 08 2e 03 00 	cmpl   $0x32e,0x8(%rsp)
  40107b:	00 
  40107c:	74 16                	je     401094 <phase_3+0x13d>
  40107e:	e8 06 06 00 00       	callq  401689 <explode_bomb>
  401083:	b8 45 00 00 00       	mov    $0x45,%eax
  401088:	eb 0a                	jmp    401094 <phase_3+0x13d>
  40108a:	e8 fa 05 00 00       	callq  401689 <explode_bomb>
  40108f:	b8 76 00 00 00       	mov    $0x76,%eax
  401094:	3a 44 24 07          	cmp    0x7(%rsp),%al
  401098:	74 05                	je     40109f <phase_3+0x148>
  40109a:	e8 ea 05 00 00       	callq  401689 <explode_bomb>
  40109f:	48 83 c4 18          	add    $0x18,%rsp
  4010a3:	c3                   	retq   

00000000004010a4 <func4>:
  4010a4:	48 83 ec 08          	sub    $0x8,%rsp
  4010a8:	89 d0                	mov    %edx,%eax
  4010aa:	29 f0                	sub    %esi,%eax
  4010ac:	89 c1                	mov    %eax,%ecx
  4010ae:	c1 e9 1f             	shr    $0x1f,%ecx
  4010b1:	01 c8                	add    %ecx,%eax
  4010b3:	d1 f8                	sar    %eax
  4010b5:	8d 0c 30             	lea    (%rax,%rsi,1),%ecx
  4010b8:	39 f9                	cmp    %edi,%ecx
  4010ba:	7e 0c                	jle    4010c8 <func4+0x24>
  4010bc:	8d 51 ff             	lea    -0x1(%rcx),%edx
  4010bf:	e8 e0 ff ff ff       	callq  4010a4 <func4>
  4010c4:	01 c0                	add    %eax,%eax
  4010c6:	eb 15                	jmp    4010dd <func4+0x39>
  4010c8:	b8 00 00 00 00       	mov    $0x0,%eax
  4010cd:	39 f9                	cmp    %edi,%ecx
  4010cf:	7d 0c                	jge    4010dd <func4+0x39>
  4010d1:	8d 71 01             	lea    0x1(%rcx),%esi
  4010d4:	e8 cb ff ff ff       	callq  4010a4 <func4>
  4010d9:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax
  4010dd:	48 83 c4 08          	add    $0x8,%rsp
  4010e1:	c3                   	retq   

00000000004010e2 <phase_4>:
  4010e2:	48 83 ec 18          	sub    $0x18,%rsp
  4010e6:	48 8d 4c 24 08       	lea    0x8(%rsp),%rcx
  4010eb:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx
  4010f0:	be e1 28 40 00       	mov    $0x4028e1,%esi
  4010f5:	b8 00 00 00 00       	mov    $0x0,%eax
  4010fa:	e8 31 fb ff ff       	callq  400c30 <__isoc99_sscanf@plt>
  4010ff:	83 f8 02             	cmp    $0x2,%eax
  401102:	75 07                	jne    40110b <phase_4+0x29>
  401104:	83 7c 24 0c 0e       	cmpl   $0xe,0xc(%rsp)
  401109:	76 05                	jbe    401110 <phase_4+0x2e>
  40110b:	e8 79 05 00 00       	callq  401689 <explode_bomb>
  401110:	ba 0e 00 00 00       	mov    $0xe,%edx
  401115:	be 00 00 00 00       	mov    $0x0,%esi
  40111a:	8b 7c 24 0c          	mov    0xc(%rsp),%edi
  40111e:	e8 81 ff ff ff       	callq  4010a4 <func4>
  401123:	83 f8 02             	cmp    $0x2,%eax
  401126:	75 07                	jne    40112f <phase_4+0x4d>
  401128:	83 7c 24 08 02       	cmpl   $0x2,0x8(%rsp)
  40112d:	74 05                	je     401134 <phase_4+0x52>
  40112f:	e8 55 05 00 00       	callq  401689 <explode_bomb>
  401134:	48 83 c4 18          	add    $0x18,%rsp
  401138:	c3                   	retq   

0000000000401139 <phase_5>:
  401139:	48 83 ec 18          	sub    $0x18,%rsp
  40113d:	48 8d 4c 24 08       	lea    0x8(%rsp),%rcx
  401142:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx
  401147:	be e1 28 40 00       	mov    $0x4028e1,%esi
  40114c:	b8 00 00 00 00       	mov    $0x0,%eax
  401151:	e8 da fa ff ff       	callq  400c30 <__isoc99_sscanf@plt>
  401156:	83 f8 01             	cmp    $0x1,%eax
  401159:	7f 05                	jg     401160 <phase_5+0x27>
  40115b:	e8 29 05 00 00       	callq  401689 <explode_bomb>
  401160:	8b 44 24 0c          	mov    0xc(%rsp),%eax
  401164:	83 e0 0f             	and    $0xf,%eax
  401167:	89 44 24 0c          	mov    %eax,0xc(%rsp)
  40116b:	83 f8 0f             	cmp    $0xf,%eax
  40116e:	74 2c                	je     40119c <phase_5+0x63>
  401170:	b9 00 00 00 00       	mov    $0x0,%ecx
  401175:	ba 00 00 00 00       	mov    $0x0,%edx
  40117a:	83 c2 01             	add    $0x1,%edx
  40117d:	48 98                	cltq   
  40117f:	8b 04 85 80 26 40 00 	mov    0x402680(,%rax,4),%eax
  401186:	01 c1                	add    %eax,%ecx
  401188:	83 f8 0f             	cmp    $0xf,%eax
  40118b:	75 ed                	jne    40117a <phase_5+0x41>
  40118d:	89 44 24 0c          	mov    %eax,0xc(%rsp)
  401191:	83 fa 06             	cmp    $0x6,%edx
  401194:	75 06                	jne    40119c <phase_5+0x63>
  401196:	3b 4c 24 08          	cmp    0x8(%rsp),%ecx
  40119a:	74 05                	je     4011a1 <phase_5+0x68>
  40119c:	e8 e8 04 00 00       	callq  401689 <explode_bomb>
  4011a1:	48 83 c4 18          	add    $0x18,%rsp
  4011a5:	c3                   	retq   

00000000004011a6 <phase_6>:
  4011a6:	41 56                	push   %r14
  4011a8:	41 55                	push   %r13
  4011aa:	41 54                	push   %r12
  4011ac:	55                   	push   %rbp
  4011ad:	53                   	push   %rbx
  4011ae:	48 83 ec 50          	sub    $0x50,%rsp
  4011b2:	4c 8d 6c 24 30       	lea    0x30(%rsp),%r13
  4011b7:	4c 89 ee             	mov    %r13,%rsi
  4011ba:	e8 00 05 00 00       	callq  4016bf <read_six_numbers>
  4011bf:	4d 89 ee             	mov    %r13,%r14
  4011c2:	41 bc 00 00 00 00    	mov    $0x0,%r12d
  4011c8:	4c 89 ed             	mov    %r13,%rbp
  4011cb:	41 8b 45 00          	mov    0x0(%r13),%eax
  4011cf:	83 e8 01             	sub    $0x1,%eax
  4011d2:	83 f8 05             	cmp    $0x5,%eax
  4011d5:	76 05                	jbe    4011dc <phase_6+0x36>
  4011d7:	e8 ad 04 00 00       	callq  401689 <explode_bomb>
  4011dc:	41 83 c4 01          	add    $0x1,%r12d
  4011e0:	41 83 fc 06          	cmp    $0x6,%r12d
  4011e4:	74 22                	je     401208 <phase_6+0x62>
  4011e6:	44 89 e3             	mov    %r12d,%ebx
  4011e9:	48 63 c3             	movslq %ebx,%rax
  4011ec:	8b 44 84 30          	mov    0x30(%rsp,%rax,4),%eax
  4011f0:	39 45 00             	cmp    %eax,0x0(%rbp)
  4011f3:	75 05                	jne    4011fa <phase_6+0x54>
  4011f5:	e8 8f 04 00 00       	callq  401689 <explode_bomb>
  4011fa:	83 c3 01             	add    $0x1,%ebx
  4011fd:	83 fb 05             	cmp    $0x5,%ebx
  401200:	7e e7                	jle    4011e9 <phase_6+0x43>
  401202:	49 83 c5 04          	add    $0x4,%r13
  401206:	eb c0                	jmp    4011c8 <phase_6+0x22>
  401208:	48 8d 74 24 48       	lea    0x48(%rsp),%rsi
  40120d:	4c 89 f0             	mov    %r14,%rax
  401210:	b9 07 00 00 00       	mov    $0x7,%ecx
  401215:	89 ca                	mov    %ecx,%edx
  401217:	2b 10                	sub    (%rax),%edx
  401219:	89 10                	mov    %edx,(%rax)
  40121b:	48 83 c0 04          	add    $0x4,%rax
  40121f:	48 39 f0             	cmp    %rsi,%rax
  401222:	75 f1                	jne    401215 <phase_6+0x6f>
  401224:	be 00 00 00 00       	mov    $0x0,%esi
  401229:	eb 20                	jmp    40124b <phase_6+0xa5>
  40122b:	48 8b 52 08          	mov    0x8(%rdx),%rdx
  40122f:	83 c0 01             	add    $0x1,%eax
  401232:	39 c8                	cmp    %ecx,%eax
  401234:	75 f5                	jne    40122b <phase_6+0x85>
  401236:	eb 05                	jmp    40123d <phase_6+0x97>
  401238:	ba f0 42 60 00       	mov    $0x6042f0,%edx
  40123d:	48 89 14 74          	mov    %rdx,(%rsp,%rsi,2)
  401241:	48 83 c6 04          	add    $0x4,%rsi
  401245:	48 83 fe 18          	cmp    $0x18,%rsi
  401249:	74 15                	je     401260 <phase_6+0xba>
  40124b:	8b 4c 34 30          	mov    0x30(%rsp,%rsi,1),%ecx
  40124f:	83 f9 01             	cmp    $0x1,%ecx
  401252:	7e e4                	jle    401238 <phase_6+0x92>
  401254:	b8 01 00 00 00       	mov    $0x1,%eax
  401259:	ba f0 42 60 00       	mov    $0x6042f0,%edx
  40125e:	eb cb                	jmp    40122b <phase_6+0x85>
  401260:	48 8b 1c 24          	mov    (%rsp),%rbx
  401264:	48 8d 44 24 08       	lea    0x8(%rsp),%rax
  401269:	48 8d 74 24 30       	lea    0x30(%rsp),%rsi
  40126e:	48 89 d9             	mov    %rbx,%rcx
  401271:	48 8b 10             	mov    (%rax),%rdx
  401274:	48 89 51 08          	mov    %rdx,0x8(%rcx)
  401278:	48 83 c0 08          	add    $0x8,%rax
  40127c:	48 39 f0             	cmp    %rsi,%rax
  40127f:	74 05                	je     401286 <phase_6+0xe0>
  401281:	48 89 d1             	mov    %rdx,%rcx
  401284:	eb eb                	jmp    401271 <phase_6+0xcb>
  401286:	48 c7 42 08 00 00 00 	movq   $0x0,0x8(%rdx)
  40128d:	00 
  40128e:	bd 05 00 00 00       	mov    $0x5,%ebp
  401293:	48 8b 43 08          	mov    0x8(%rbx),%rax
  401297:	8b 00                	mov    (%rax),%eax
  401299:	39 03                	cmp    %eax,(%rbx)
  40129b:	7d 05                	jge    4012a2 <phase_6+0xfc>
  40129d:	e8 e7 03 00 00       	callq  401689 <explode_bomb>
  4012a2:	48 8b 5b 08          	mov    0x8(%rbx),%rbx
  4012a6:	83 ed 01             	sub    $0x1,%ebp
  4012a9:	75 e8                	jne    401293 <phase_6+0xed>
  4012ab:	48 83 c4 50          	add    $0x50,%rsp
  4012af:	5b                   	pop    %rbx
  4012b0:	5d                   	pop    %rbp
  4012b1:	41 5c                	pop    %r12
  4012b3:	41 5d                	pop    %r13
  4012b5:	41 5e                	pop    %r14
  4012b7:	c3                   	retq   

00000000004012b8 <fun7>:
  4012b8:	48 83 ec 08          	sub    $0x8,%rsp
  4012bc:	48 85 ff             	test   %rdi,%rdi
  4012bf:	74 2b                	je     4012ec <fun7+0x34>
  4012c1:	8b 17                	mov    (%rdi),%edx
  4012c3:	39 f2                	cmp    %esi,%edx
  4012c5:	7e 0d                	jle    4012d4 <fun7+0x1c>
  4012c7:	48 8b 7f 08          	mov    0x8(%rdi),%rdi
  4012cb:	e8 e8 ff ff ff       	callq  4012b8 <fun7>
  4012d0:	01 c0                	add    %eax,%eax
  4012d2:	eb 1d                	jmp    4012f1 <fun7+0x39>
  4012d4:	b8 00 00 00 00       	mov    $0x0,%eax
  4012d9:	39 f2                	cmp    %esi,%edx
  4012db:	74 14                	je     4012f1 <fun7+0x39>
  4012dd:	48 8b 7f 10          	mov    0x10(%rdi),%rdi
  4012e1:	e8 d2 ff ff ff       	callq  4012b8 <fun7>
  4012e6:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax
  4012ea:	eb 05                	jmp    4012f1 <fun7+0x39>
  4012ec:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  4012f1:	48 83 c4 08          	add    $0x8,%rsp
  4012f5:	c3                   	retq   

00000000004012f6 <secret_phase>:
  4012f6:	53                   	push   %rbx
  4012f7:	e8 05 04 00 00       	callq  401701 <read_line>
  4012fc:	ba 0a 00 00 00       	mov    $0xa,%edx
  401301:	be 00 00 00 00       	mov    $0x0,%esi
  401306:	48 89 c7             	mov    %rax,%rdi
  401309:	e8 f2 f8 ff ff       	callq  400c00 <strtol@plt>
  40130e:	48 89 c3             	mov    %rax,%rbx
  401311:	8d 40 ff             	lea    -0x1(%rax),%eax
  401314:	3d e8 03 00 00       	cmp    $0x3e8,%eax
  401319:	76 05                	jbe    401320 <secret_phase+0x2a>
  40131b:	e8 69 03 00 00       	callq  401689 <explode_bomb>
  401320:	89 de                	mov    %ebx,%esi
  401322:	bf 10 41 60 00       	mov    $0x604110,%edi
  401327:	e8 8c ff ff ff       	callq  4012b8 <fun7>
  40132c:	83 f8 07             	cmp    $0x7,%eax
  40132f:	74 05                	je     401336 <secret_phase+0x40>
  401331:	e8 53 03 00 00       	callq  401689 <explode_bomb>
  401336:	bf 08 26 40 00       	mov    $0x402608,%edi
  40133b:	e8 00 f8 ff ff       	callq  400b40 <puts@plt>
  401340:	e8 e2 04 00 00       	callq  401827 <phase_defused>
  401345:	5b                   	pop    %rbx
  401346:	c3                   	retq   
  401347:	66 0f 1f 84 00 00 00 	nopw   0x0(%rax,%rax,1)
  40134e:	00 00 

0000000000401350 <sig_handler>:
  401350:	48 83 ec 08          	sub    $0x8,%rsp
  401354:	bf c0 26 40 00       	mov    $0x4026c0,%edi
  401359:	e8 e2 f7 ff ff       	callq  400b40 <puts@plt>
  40135e:	bf 03 00 00 00       	mov    $0x3,%edi
  401363:	e8 38 f9 ff ff       	callq  400ca0 <sleep@plt>
  401368:	bf 51 28 40 00       	mov    $0x402851,%edi
  40136d:	b8 00 00 00 00       	mov    $0x0,%eax
  401372:	e8 e9 f7 ff ff       	callq  400b60 <printf@plt>
  401377:	48 8b 3d 02 34 20 00 	mov    0x203402(%rip),%rdi        # 604780 <__TMC_END__>
  40137e:	e8 9d f8 ff ff       	callq  400c20 <fflush@plt>
  401383:	bf 01 00 00 00       	mov    $0x1,%edi
  401388:	e8 13 f9 ff ff       	callq  400ca0 <sleep@plt>
  40138d:	bf 59 28 40 00       	mov    $0x402859,%edi
  401392:	e8 a9 f7 ff ff       	callq  400b40 <puts@plt>
  401397:	bf 10 00 00 00       	mov    $0x10,%edi
  40139c:	e8 df f8 ff ff       	callq  400c80 <exit@plt>

00000000004013a1 <invalid_phase>:
  4013a1:	48 83 ec 08          	sub    $0x8,%rsp
  4013a5:	48 89 fe             	mov    %rdi,%rsi
  4013a8:	bf 61 28 40 00       	mov    $0x402861,%edi
  4013ad:	b8 00 00 00 00       	mov    $0x0,%eax
  4013b2:	e8 a9 f7 ff ff       	callq  400b60 <printf@plt>
  4013b7:	bf 08 00 00 00       	mov    $0x8,%edi
  4013bc:	e8 bf f8 ff ff       	callq  400c80 <exit@plt>

00000000004013c1 <string_length>:
  4013c1:	80 3f 00             	cmpb   $0x0,(%rdi)
  4013c4:	74 12                	je     4013d8 <string_length+0x17>
  4013c6:	48 89 fa             	mov    %rdi,%rdx
  4013c9:	48 83 c2 01          	add    $0x1,%rdx
  4013cd:	89 d0                	mov    %edx,%eax
  4013cf:	29 f8                	sub    %edi,%eax
  4013d1:	80 3a 00             	cmpb   $0x0,(%rdx)
  4013d4:	75 f3                	jne    4013c9 <string_length+0x8>
  4013d6:	f3 c3                	repz retq 
  4013d8:	b8 00 00 00 00       	mov    $0x0,%eax
  4013dd:	c3                   	retq   

00000000004013de <strings_not_equal>:
  4013de:	41 54                	push   %r12
  4013e0:	55                   	push   %rbp
  4013e1:	53                   	push   %rbx
  4013e2:	48 89 fb             	mov    %rdi,%rbx
  4013e5:	48 89 f5             	mov    %rsi,%rbp
  4013e8:	e8 d4 ff ff ff       	callq  4013c1 <string_length>
  4013ed:	41 89 c4             	mov    %eax,%r12d
  4013f0:	48 89 ef             	mov    %rbp,%rdi
  4013f3:	e8 c9 ff ff ff       	callq  4013c1 <string_length>
  4013f8:	ba 01 00 00 00       	mov    $0x1,%edx
  4013fd:	41 39 c4             	cmp    %eax,%r12d
  401400:	75 3e                	jne    401440 <strings_not_equal+0x62>
  401402:	0f b6 03             	movzbl (%rbx),%eax
  401405:	84 c0                	test   %al,%al
  401407:	74 24                	je     40142d <strings_not_equal+0x4f>
  401409:	3a 45 00             	cmp    0x0(%rbp),%al
  40140c:	74 09                	je     401417 <strings_not_equal+0x39>
  40140e:	66 90                	xchg   %ax,%ax
  401410:	eb 22                	jmp    401434 <strings_not_equal+0x56>
  401412:	3a 45 00             	cmp    0x0(%rbp),%al
  401415:	75 24                	jne    40143b <strings_not_equal+0x5d>
  401417:	48 83 c3 01          	add    $0x1,%rbx
  40141b:	48 83 c5 01          	add    $0x1,%rbp
  40141f:	0f b6 03             	movzbl (%rbx),%eax
  401422:	84 c0                	test   %al,%al
  401424:	75 ec                	jne    401412 <strings_not_equal+0x34>
  401426:	ba 00 00 00 00       	mov    $0x0,%edx
  40142b:	eb 13                	jmp    401440 <strings_not_equal+0x62>
  40142d:	ba 00 00 00 00       	mov    $0x0,%edx
  401432:	eb 0c                	jmp    401440 <strings_not_equal+0x62>
  401434:	ba 01 00 00 00       	mov    $0x1,%edx
  401439:	eb 05                	jmp    401440 <strings_not_equal+0x62>
  40143b:	ba 01 00 00 00       	mov    $0x1,%edx
  401440:	89 d0                	mov    %edx,%eax
  401442:	5b                   	pop    %rbx
  401443:	5d                   	pop    %rbp
  401444:	41 5c                	pop    %r12
  401446:	c3                   	retq   

0000000000401447 <initialize_bomb>:
  401447:	53                   	push   %rbx
  401448:	48 81 ec 40 20 00 00 	sub    $0x2040,%rsp
  40144f:	be 50 13 40 00       	mov    $0x401350,%esi
  401454:	bf 02 00 00 00       	mov    $0x2,%edi
  401459:	e8 62 f7 ff ff       	callq  400bc0 <signal@plt>
  40145e:	be 40 00 00 00       	mov    $0x40,%esi
  401463:	48 8d bc 24 00 20 00 	lea    0x2000(%rsp),%rdi
  40146a:	00 
  40146b:	e8 f0 f7 ff ff       	callq  400c60 <gethostname@plt>
  401470:	85 c0                	test   %eax,%eax
  401472:	75 13                	jne    401487 <initialize_bomb+0x40>
  401474:	48 8b 3d 05 2f 20 00 	mov    0x202f05(%rip),%rdi        # 604380 <host_table>
  40147b:	bb 88 43 60 00       	mov    $0x604388,%ebx
  401480:	48 85 ff             	test   %rdi,%rdi
  401483:	75 16                	jne    40149b <initialize_bomb+0x54>
  401485:	eb 32                	jmp    4014b9 <initialize_bomb+0x72>
  401487:	bf f8 26 40 00       	mov    $0x4026f8,%edi
  40148c:	e8 af f6 ff ff       	callq  400b40 <puts@plt>
  401491:	bf 08 00 00 00       	mov    $0x8,%edi
  401496:	e8 e5 f7 ff ff       	callq  400c80 <exit@plt>
  40149b:	48 8d b4 24 00 20 00 	lea    0x2000(%rsp),%rsi
  4014a2:	00 
  4014a3:	e8 68 f6 ff ff       	callq  400b10 <strcasecmp@plt>
  4014a8:	85 c0                	test   %eax,%eax
  4014aa:	74 0d                	je     4014b9 <initialize_bomb+0x72>
  4014ac:	48 83 c3 08          	add    $0x8,%rbx
  4014b0:	48 8b 7b f8          	mov    -0x8(%rbx),%rdi
  4014b4:	48 85 ff             	test   %rdi,%rdi
  4014b7:	75 e2                	jne    40149b <initialize_bomb+0x54>
  4014b9:	48 89 e7             	mov    %rsp,%rdi
  4014bc:	e8 1d 0d 00 00       	callq  4021de <init_driver>
  4014c1:	85 c0                	test   %eax,%eax
  4014c3:	79 1c                	jns    4014e1 <initialize_bomb+0x9a>
  4014c5:	48 89 e6             	mov    %rsp,%rsi
  4014c8:	bf 72 28 40 00       	mov    $0x402872,%edi
  4014cd:	b8 00 00 00 00       	mov    $0x0,%eax
  4014d2:	e8 89 f6 ff ff       	callq  400b60 <printf@plt>
  4014d7:	bf 08 00 00 00       	mov    $0x8,%edi
  4014dc:	e8 9f f7 ff ff       	callq  400c80 <exit@plt>
  4014e1:	48 81 c4 40 20 00 00 	add    $0x2040,%rsp
  4014e8:	5b                   	pop    %rbx
  4014e9:	c3                   	retq   

00000000004014ea <initialize_bomb_solve>:
  4014ea:	f3 c3                	repz retq 

00000000004014ec <blank_line>:
  4014ec:	55                   	push   %rbp
  4014ed:	53                   	push   %rbx
  4014ee:	48 83 ec 08          	sub    $0x8,%rsp
  4014f2:	48 89 fb             	mov    %rdi,%rbx
  4014f5:	eb 17                	jmp    40150e <blank_line+0x22>
  4014f7:	e8 b4 f7 ff ff       	callq  400cb0 <__ctype_b_loc@plt>
  4014fc:	48 83 c3 01          	add    $0x1,%rbx
  401500:	48 0f be ed          	movsbq %bpl,%rbp
  401504:	48 8b 00             	mov    (%rax),%rax
  401507:	f6 44 68 01 20       	testb  $0x20,0x1(%rax,%rbp,2)
  40150c:	74 0f                	je     40151d <blank_line+0x31>
  40150e:	0f b6 2b             	movzbl (%rbx),%ebp
  401511:	40 84 ed             	test   %bpl,%bpl
  401514:	75 e1                	jne    4014f7 <blank_line+0xb>
  401516:	b8 01 00 00 00       	mov    $0x1,%eax
  40151b:	eb 05                	jmp    401522 <blank_line+0x36>
  40151d:	b8 00 00 00 00       	mov    $0x0,%eax
  401522:	48 83 c4 08          	add    $0x8,%rsp
  401526:	5b                   	pop    %rbx
  401527:	5d                   	pop    %rbp
  401528:	c3                   	retq   

0000000000401529 <skip>:
  401529:	53                   	push   %rbx
  40152a:	48 63 05 6b 32 20 00 	movslq 0x20326b(%rip),%rax        # 60479c <num_input_strings>
  401531:	48 8d 3c 80          	lea    (%rax,%rax,4),%rdi
  401535:	48 c1 e7 04          	shl    $0x4,%rdi
  401539:	48 81 c7 c0 47 60 00 	add    $0x6047c0,%rdi
  401540:	48 8b 15 59 32 20 00 	mov    0x203259(%rip),%rdx        # 6047a0 <infile>
  401547:	be 50 00 00 00       	mov    $0x50,%esi
  40154c:	e8 5f f6 ff ff       	callq  400bb0 <fgets@plt>
  401551:	48 89 c3             	mov    %rax,%rbx
  401554:	48 85 c0             	test   %rax,%rax
  401557:	74 0c                	je     401565 <skip+0x3c>
  401559:	48 89 c7             	mov    %rax,%rdi
  40155c:	e8 8b ff ff ff       	callq  4014ec <blank_line>
  401561:	85 c0                	test   %eax,%eax
  401563:	75 c5                	jne    40152a <skip+0x1>
  401565:	48 89 d8             	mov    %rbx,%rax
  401568:	5b                   	pop    %rbx
  401569:	c3                   	retq   

000000000040156a <send_msg>:
  40156a:	53                   	push   %rbx
  40156b:	48 81 ec 20 40 00 00 	sub    $0x4020,%rsp
  401572:	89 fb                	mov    %edi,%ebx
  401574:	bf 9d 28 40 00       	mov    $0x40289d,%edi
  401579:	e8 82 f5 ff ff       	callq  400b00 <getenv@plt>
  40157e:	48 89 c2             	mov    %rax,%rdx
  401581:	48 b8 6e 6f 63 67 75 	movabs $0x7265737567636f6e,%rax
  401588:	73 65 72 
  40158b:	48 89 44 24 10       	mov    %rax,0x10(%rsp)
  401590:	66 c7 44 24 18 69 64 	movw   $0x6469,0x18(%rsp)
  401597:	c6 44 24 1a 00       	movb   $0x0,0x1a(%rsp)
  40159c:	48 8d 44 24 10       	lea    0x10(%rsp),%rax
  4015a1:	48 85 d2             	test   %rdx,%rdx
  4015a4:	48 0f 44 d0          	cmove  %rax,%rdx
  4015a8:	44 8b 0d ed 31 20 00 	mov    0x2031ed(%rip),%r9d        # 60479c <num_input_strings>
  4015af:	41 8d 41 ff          	lea    -0x1(%r9),%eax
  4015b3:	48 98                	cltq   
  4015b5:	4c 8d 14 80          	lea    (%rax,%rax,4),%r10
  4015b9:	49 c1 e2 04          	shl    $0x4,%r10
  4015bd:	49 81 c2 c0 47 60 00 	add    $0x6047c0,%r10
  4015c4:	48 c7 c6 ff ff ff ff 	mov    $0xffffffffffffffff,%rsi
  4015cb:	4c 89 d7             	mov    %r10,%rdi
  4015ce:	b8 00 00 00 00       	mov    $0x0,%eax
  4015d3:	48 89 f1             	mov    %rsi,%rcx
  4015d6:	f2 ae                	repnz scas %es:(%rdi),%al
  4015d8:	48 f7 d1             	not    %rcx
  4015db:	49 89 cb             	mov    %rcx,%r11
  4015de:	48 89 d7             	mov    %rdx,%rdi
  4015e1:	48 89 f1             	mov    %rsi,%rcx
  4015e4:	f2 ae                	repnz scas %es:(%rdi),%al
  4015e6:	48 89 ce             	mov    %rcx,%rsi
  4015e9:	48 f7 d6             	not    %rsi
  4015ec:	49 8d 44 33 62       	lea    0x62(%r11,%rsi,1),%rax
  4015f1:	48 3d 00 20 00 00    	cmp    $0x2000,%rax
  4015f7:	76 19                	jbe    401612 <send_msg+0xa8>
  4015f9:	bf 30 27 40 00       	mov    $0x402730,%edi
  4015fe:	b8 00 00 00 00       	mov    $0x0,%eax
  401603:	e8 58 f5 ff ff       	callq  400b60 <printf@plt>
  401608:	bf 08 00 00 00       	mov    $0x8,%edi
  40160d:	e8 6e f6 ff ff       	callq  400c80 <exit@plt>
  401612:	85 db                	test   %ebx,%ebx
  401614:	b8 8c 28 40 00       	mov    $0x40288c,%eax
  401619:	41 b8 94 28 40 00    	mov    $0x402894,%r8d
  40161f:	4c 0f 45 c0          	cmovne %rax,%r8
  401623:	4c 89 14 24          	mov    %r10,(%rsp)
  401627:	48 89 d1             	mov    %rdx,%rcx
  40162a:	8b 15 44 2d 20 00    	mov    0x202d44(%rip),%edx        # 604374 <bomb_id>
  401630:	be a6 28 40 00       	mov    $0x4028a6,%esi
  401635:	48 8d bc 24 20 20 00 	lea    0x2020(%rsp),%rdi
  40163c:	00 
  40163d:	b8 00 00 00 00       	mov    $0x0,%eax
  401642:	e8 29 f6 ff ff       	callq  400c70 <sprintf@plt>
  401647:	4c 8d 44 24 20       	lea    0x20(%rsp),%r8
  40164c:	b9 00 00 00 00       	mov    $0x0,%ecx
  401651:	48 8d 94 24 20 20 00 	lea    0x2020(%rsp),%rdx
  401658:	00 
  401659:	be 50 43 60 00       	mov    $0x604350,%esi
  40165e:	bf 65 43 60 00       	mov    $0x604365,%edi
  401663:	e8 14 0d 00 00       	callq  40237c <driver_post>
  401668:	85 c0                	test   %eax,%eax
  40166a:	79 14                	jns    401680 <send_msg+0x116>
  40166c:	48 8d 7c 24 20       	lea    0x20(%rsp),%rdi
  401671:	e8 ca f4 ff ff       	callq  400b40 <puts@plt>
  401676:	bf 00 00 00 00       	mov    $0x0,%edi
  40167b:	e8 00 f6 ff ff       	callq  400c80 <exit@plt>
  401680:	48 81 c4 20 40 00 00 	add    $0x4020,%rsp
  401687:	5b                   	pop    %rbx
  401688:	c3                   	retq   

0000000000401689 <explode_bomb>:
  401689:	48 83 ec 08          	sub    $0x8,%rsp
  40168d:	bf b5 28 40 00       	mov    $0x4028b5,%edi
  401692:	e8 a9 f4 ff ff       	callq  400b40 <puts@plt>
  401697:	bf be 28 40 00       	mov    $0x4028be,%edi
  40169c:	e8 9f f4 ff ff       	callq  400b40 <puts@plt>
  4016a1:	bf 00 00 00 00       	mov    $0x0,%edi
  4016a6:	e8 bf fe ff ff       	callq  40156a <send_msg>
  4016ab:	bf 58 27 40 00       	mov    $0x402758,%edi
  4016b0:	e8 8b f4 ff ff       	callq  400b40 <puts@plt>
  4016b5:	bf 08 00 00 00       	mov    $0x8,%edi
  4016ba:	e8 c1 f5 ff ff       	callq  400c80 <exit@plt>

00000000004016bf <read_six_numbers>:
  4016bf:	48 83 ec 18          	sub    $0x18,%rsp
  4016c3:	48 89 f2             	mov    %rsi,%rdx
  4016c6:	48 8d 4e 04          	lea    0x4(%rsi),%rcx
  4016ca:	48 8d 46 14          	lea    0x14(%rsi),%rax
  4016ce:	48 89 44 24 08       	mov    %rax,0x8(%rsp)
  4016d3:	48 8d 46 10          	lea    0x10(%rsi),%rax
  4016d7:	48 89 04 24          	mov    %rax,(%rsp)
  4016db:	4c 8d 4e 0c          	lea    0xc(%rsi),%r9
  4016df:	4c 8d 46 08          	lea    0x8(%rsi),%r8
  4016e3:	be d5 28 40 00       	mov    $0x4028d5,%esi
  4016e8:	b8 00 00 00 00       	mov    $0x0,%eax
  4016ed:	e8 3e f5 ff ff       	callq  400c30 <__isoc99_sscanf@plt>
  4016f2:	83 f8 05             	cmp    $0x5,%eax
  4016f5:	7f 05                	jg     4016fc <read_six_numbers+0x3d>
  4016f7:	e8 8d ff ff ff       	callq  401689 <explode_bomb>
  4016fc:	48 83 c4 18          	add    $0x18,%rsp
  401700:	c3                   	retq   

0000000000401701 <read_line>:
  401701:	48 83 ec 08          	sub    $0x8,%rsp
  401705:	b8 00 00 00 00       	mov    $0x0,%eax
  40170a:	e8 1a fe ff ff       	callq  401529 <skip>
  40170f:	48 85 c0             	test   %rax,%rax
  401712:	75 6e                	jne    401782 <read_line+0x81>
  401714:	48 8b 05 6d 30 20 00 	mov    0x20306d(%rip),%rax        # 604788 <stdin@@GLIBC_2.2.5>
  40171b:	48 39 05 7e 30 20 00 	cmp    %rax,0x20307e(%rip)        # 6047a0 <infile>
  401722:	75 14                	jne    401738 <read_line+0x37>
  401724:	bf e7 28 40 00       	mov    $0x4028e7,%edi
  401729:	e8 12 f4 ff ff       	callq  400b40 <puts@plt>
  40172e:	bf 08 00 00 00       	mov    $0x8,%edi
  401733:	e8 48 f5 ff ff       	callq  400c80 <exit@plt>
  401738:	bf 05 29 40 00       	mov    $0x402905,%edi
  40173d:	e8 be f3 ff ff       	callq  400b00 <getenv@plt>
  401742:	48 85 c0             	test   %rax,%rax
  401745:	74 0a                	je     401751 <read_line+0x50>
  401747:	bf 00 00 00 00       	mov    $0x0,%edi
  40174c:	e8 2f f5 ff ff       	callq  400c80 <exit@plt>
  401751:	48 8b 05 30 30 20 00 	mov    0x203030(%rip),%rax        # 604788 <stdin@@GLIBC_2.2.5>
  401758:	48 89 05 41 30 20 00 	mov    %rax,0x203041(%rip)        # 6047a0 <infile>
  40175f:	b8 00 00 00 00       	mov    $0x0,%eax
  401764:	e8 c0 fd ff ff       	callq  401529 <skip>
  401769:	48 85 c0             	test   %rax,%rax
  40176c:	75 14                	jne    401782 <read_line+0x81>
  40176e:	bf e7 28 40 00       	mov    $0x4028e7,%edi
  401773:	e8 c8 f3 ff ff       	callq  400b40 <puts@plt>
  401778:	bf 00 00 00 00       	mov    $0x0,%edi
  40177d:	e8 fe f4 ff ff       	callq  400c80 <exit@plt>
  401782:	8b 15 14 30 20 00    	mov    0x203014(%rip),%edx        # 60479c <num_input_strings>
  401788:	48 63 c2             	movslq %edx,%rax
  40178b:	48 8d 34 80          	lea    (%rax,%rax,4),%rsi
  40178f:	48 c1 e6 04          	shl    $0x4,%rsi
  401793:	48 81 c6 c0 47 60 00 	add    $0x6047c0,%rsi
  40179a:	48 89 f7             	mov    %rsi,%rdi
  40179d:	b8 00 00 00 00       	mov    $0x0,%eax
  4017a2:	48 c7 c1 ff ff ff ff 	mov    $0xffffffffffffffff,%rcx
  4017a9:	f2 ae                	repnz scas %es:(%rdi),%al
  4017ab:	48 f7 d1             	not    %rcx
  4017ae:	48 83 e9 01          	sub    $0x1,%rcx
  4017b2:	83 f9 4e             	cmp    $0x4e,%ecx
  4017b5:	7e 46                	jle    4017fd <read_line+0xfc>
  4017b7:	bf 10 29 40 00       	mov    $0x402910,%edi
  4017bc:	e8 7f f3 ff ff       	callq  400b40 <puts@plt>
  4017c1:	8b 05 d5 2f 20 00    	mov    0x202fd5(%rip),%eax        # 60479c <num_input_strings>
  4017c7:	8d 50 01             	lea    0x1(%rax),%edx
  4017ca:	89 15 cc 2f 20 00    	mov    %edx,0x202fcc(%rip)        # 60479c <num_input_strings>
  4017d0:	48 98                	cltq   
  4017d2:	48 6b c0 50          	imul   $0x50,%rax,%rax
  4017d6:	48 bf 2a 2a 2a 74 72 	movabs $0x636e7572742a2a2a,%rdi
  4017dd:	75 6e 63 
  4017e0:	48 89 b8 c0 47 60 00 	mov    %rdi,0x6047c0(%rax)
  4017e7:	48 bf 61 74 65 64 2a 	movabs $0x2a2a2a64657461,%rdi
  4017ee:	2a 2a 00 
  4017f1:	48 89 b8 c8 47 60 00 	mov    %rdi,0x6047c8(%rax)
  4017f8:	e8 8c fe ff ff       	callq  401689 <explode_bomb>
  4017fd:	83 e9 01             	sub    $0x1,%ecx
  401800:	48 63 c9             	movslq %ecx,%rcx
  401803:	48 63 c2             	movslq %edx,%rax
  401806:	48 8d 04 80          	lea    (%rax,%rax,4),%rax
  40180a:	48 c1 e0 04          	shl    $0x4,%rax
  40180e:	c6 84 01 c0 47 60 00 	movb   $0x0,0x6047c0(%rcx,%rax,1)
  401815:	00 
  401816:	83 c2 01             	add    $0x1,%edx
  401819:	89 15 7d 2f 20 00    	mov    %edx,0x202f7d(%rip)        # 60479c <num_input_strings>
  40181f:	48 89 f0             	mov    %rsi,%rax
  401822:	48 83 c4 08          	add    $0x8,%rsp
  401826:	c3                   	retq   

0000000000401827 <phase_defused>:
  401827:	48 83 ec 68          	sub    $0x68,%rsp
  40182b:	bf 01 00 00 00       	mov    $0x1,%edi
  401830:	e8 35 fd ff ff       	callq  40156a <send_msg>
  401835:	83 3d 60 2f 20 00 06 	cmpl   $0x6,0x202f60(%rip)        # 60479c <num_input_strings>
  40183c:	75 6d                	jne    4018ab <phase_defused+0x84>
  40183e:	4c 8d 44 24 10       	lea    0x10(%rsp),%r8
  401843:	48 8d 4c 24 08       	lea    0x8(%rsp),%rcx
  401848:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx
  40184d:	be 2b 29 40 00       	mov    $0x40292b,%esi
  401852:	bf b0 48 60 00       	mov    $0x6048b0,%edi
  401857:	b8 00 00 00 00       	mov    $0x0,%eax
  40185c:	e8 cf f3 ff ff       	callq  400c30 <__isoc99_sscanf@plt>
  401861:	83 f8 03             	cmp    $0x3,%eax
  401864:	75 31                	jne    401897 <phase_defused+0x70>
  401866:	be 34 29 40 00       	mov    $0x402934,%esi
  40186b:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
  401870:	e8 69 fb ff ff       	callq  4013de <strings_not_equal>
  401875:	85 c0                	test   %eax,%eax
  401877:	75 1e                	jne    401897 <phase_defused+0x70>
  401879:	bf 80 27 40 00       	mov    $0x402780,%edi
  40187e:	e8 bd f2 ff ff       	callq  400b40 <puts@plt>
  401883:	bf a8 27 40 00       	mov    $0x4027a8,%edi
  401888:	e8 b3 f2 ff ff       	callq  400b40 <puts@plt>
  40188d:	b8 00 00 00 00       	mov    $0x0,%eax
  401892:	e8 5f fa ff ff       	callq  4012f6 <secret_phase>
  401897:	bf e0 27 40 00       	mov    $0x4027e0,%edi
  40189c:	e8 9f f2 ff ff       	callq  400b40 <puts@plt>
  4018a1:	bf 10 28 40 00       	mov    $0x402810,%edi
  4018a6:	e8 95 f2 ff ff       	callq  400b40 <puts@plt>
  4018ab:	48 83 c4 68          	add    $0x68,%rsp
  4018af:	c3                   	retq   

00000000004018b0 <sigalrm_handler>:
  4018b0:	48 83 ec 08          	sub    $0x8,%rsp
  4018b4:	ba 00 00 00 00       	mov    $0x0,%edx
  4018b9:	be a8 29 40 00       	mov    $0x4029a8,%esi
  4018be:	48 8b 3d cb 2e 20 00 	mov    0x202ecb(%rip),%rdi        # 604790 <stderr@@GLIBC_2.2.5>
  4018c5:	b8 00 00 00 00       	mov    $0x0,%eax
  4018ca:	e8 11 f3 ff ff       	callq  400be0 <fprintf@plt>
  4018cf:	bf 01 00 00 00       	mov    $0x1,%edi
  4018d4:	e8 a7 f3 ff ff       	callq  400c80 <exit@plt>

00000000004018d9 <rio_readlineb>:
  4018d9:	41 57                	push   %r15
  4018db:	41 56                	push   %r14
  4018dd:	41 55                	push   %r13
  4018df:	41 54                	push   %r12
  4018e1:	55                   	push   %rbp
  4018e2:	53                   	push   %rbx
  4018e3:	48 83 ec 38          	sub    $0x38,%rsp
  4018e7:	49 89 f6             	mov    %rsi,%r14
  4018ea:	48 89 54 24 18       	mov    %rdx,0x18(%rsp)
  4018ef:	48 83 fa 01          	cmp    $0x1,%rdx
  4018f3:	0f 86 c2 00 00 00    	jbe    4019bb <rio_readlineb+0xe2>
  4018f9:	48 89 fb             	mov    %rdi,%rbx
  4018fc:	41 bd 01 00 00 00    	mov    $0x1,%r13d
  401902:	4c 8d 67 10          	lea    0x10(%rdi),%r12
  401906:	eb 2e                	jmp    401936 <rio_readlineb+0x5d>
  401908:	ba 00 20 00 00       	mov    $0x2000,%edx
  40190d:	4c 89 e6             	mov    %r12,%rsi
  401910:	8b 3b                	mov    (%rbx),%edi
  401912:	e8 79 f2 ff ff       	callq  400b90 <read@plt>
  401917:	89 43 04             	mov    %eax,0x4(%rbx)
  40191a:	85 c0                	test   %eax,%eax
  40191c:	79 0f                	jns    40192d <rio_readlineb+0x54>
  40191e:	e8 fd f1 ff ff       	callq  400b20 <__errno_location@plt>
  401923:	83 38 04             	cmpl   $0x4,(%rax)
  401926:	74 0e                	je     401936 <rio_readlineb+0x5d>
  401928:	e9 9d 00 00 00       	jmpq   4019ca <rio_readlineb+0xf1>
  40192d:	85 c0                	test   %eax,%eax
  40192f:	90                   	nop
  401930:	74 6c                	je     40199e <rio_readlineb+0xc5>
  401932:	4c 89 63 08          	mov    %r12,0x8(%rbx)
  401936:	8b 6b 04             	mov    0x4(%rbx),%ebp
  401939:	85 ed                	test   %ebp,%ebp
  40193b:	7e cb                	jle    401908 <rio_readlineb+0x2f>
  40193d:	85 ed                	test   %ebp,%ebp
  40193f:	41 0f 95 c7          	setne  %r15b
  401943:	41 0f b6 c7          	movzbl %r15b,%eax
  401947:	89 44 24 0c          	mov    %eax,0xc(%rsp)
  40194b:	45 0f b6 ff          	movzbl %r15b,%r15d
  40194f:	48 8b 4b 08          	mov    0x8(%rbx),%rcx
  401953:	4c 89 fa             	mov    %r15,%rdx
  401956:	48 89 4c 24 10       	mov    %rcx,0x10(%rsp)
  40195b:	48 89 ce             	mov    %rcx,%rsi
  40195e:	48 8d 7c 24 2f       	lea    0x2f(%rsp),%rdi
  401963:	e8 a8 f2 ff ff       	callq  400c10 <memcpy@plt>
  401968:	4c 03 7c 24 10       	add    0x10(%rsp),%r15
  40196d:	4c 89 7b 08          	mov    %r15,0x8(%rbx)
  401971:	8b 44 24 0c          	mov    0xc(%rsp),%eax
  401975:	29 c5                	sub    %eax,%ebp
  401977:	89 6b 04             	mov    %ebp,0x4(%rbx)
  40197a:	83 f8 01             	cmp    $0x1,%eax
  40197d:	75 13                	jne    401992 <rio_readlineb+0xb9>
  40197f:	49 83 c6 01          	add    $0x1,%r14
  401983:	0f b6 44 24 2f       	movzbl 0x2f(%rsp),%eax
  401988:	41 88 46 ff          	mov    %al,-0x1(%r14)
  40198c:	3c 0a                	cmp    $0xa,%al
  40198e:	75 18                	jne    4019a8 <rio_readlineb+0xcf>
  401990:	eb 2f                	jmp    4019c1 <rio_readlineb+0xe8>
  401992:	83 7c 24 0c 00       	cmpl   $0x0,0xc(%rsp)
  401997:	75 3a                	jne    4019d3 <rio_readlineb+0xfa>
  401999:	44 89 e8             	mov    %r13d,%eax
  40199c:	eb 03                	jmp    4019a1 <rio_readlineb+0xc8>
  40199e:	44 89 e8             	mov    %r13d,%eax
  4019a1:	83 f8 01             	cmp    $0x1,%eax
  4019a4:	75 1b                	jne    4019c1 <rio_readlineb+0xe8>
  4019a6:	eb 34                	jmp    4019dc <rio_readlineb+0x103>
  4019a8:	41 83 c5 01          	add    $0x1,%r13d
  4019ac:	49 63 c5             	movslq %r13d,%rax
  4019af:	48 3b 44 24 18       	cmp    0x18(%rsp),%rax
  4019b4:	73 0b                	jae    4019c1 <rio_readlineb+0xe8>
  4019b6:	e9 7b ff ff ff       	jmpq   401936 <rio_readlineb+0x5d>
  4019bb:	41 bd 01 00 00 00    	mov    $0x1,%r13d
  4019c1:	41 c6 06 00          	movb   $0x0,(%r14)
  4019c5:	49 63 c5             	movslq %r13d,%rax
  4019c8:	eb 17                	jmp    4019e1 <rio_readlineb+0x108>
  4019ca:	48 c7 c0 ff ff ff ff 	mov    $0xffffffffffffffff,%rax
  4019d1:	eb 0e                	jmp    4019e1 <rio_readlineb+0x108>
  4019d3:	48 c7 c0 ff ff ff ff 	mov    $0xffffffffffffffff,%rax
  4019da:	eb 05                	jmp    4019e1 <rio_readlineb+0x108>
  4019dc:	b8 00 00 00 00       	mov    $0x0,%eax
  4019e1:	48 83 c4 38          	add    $0x38,%rsp
  4019e5:	5b                   	pop    %rbx
  4019e6:	5d                   	pop    %rbp
  4019e7:	41 5c                	pop    %r12
  4019e9:	41 5d                	pop    %r13
  4019eb:	41 5e                	pop    %r14
  4019ed:	41 5f                	pop    %r15
  4019ef:	c3                   	retq   

00000000004019f0 <submitr>:
  4019f0:	41 57                	push   %r15
  4019f2:	41 56                	push   %r14
  4019f4:	41 55                	push   %r13
  4019f6:	41 54                	push   %r12
  4019f8:	55                   	push   %rbp
  4019f9:	53                   	push   %rbx
  4019fa:	48 81 ec 68 a0 00 00 	sub    $0xa068,%rsp
  401a01:	48 89 fd             	mov    %rdi,%rbp
  401a04:	41 89 f5             	mov    %esi,%r13d
  401a07:	48 89 54 24 08       	mov    %rdx,0x8(%rsp)
  401a0c:	48 89 4c 24 10       	mov    %rcx,0x10(%rsp)
  401a11:	4c 89 44 24 18       	mov    %r8,0x18(%rsp)
  401a16:	4d 89 cf             	mov    %r9,%r15
  401a19:	48 8b 9c 24 a0 a0 00 	mov    0xa0a0(%rsp),%rbx
  401a20:	00 
  401a21:	4c 8b b4 24 a8 a0 00 	mov    0xa0a8(%rsp),%r14
  401a28:	00 
  401a29:	c7 84 24 3c 20 00 00 	movl   $0x0,0x203c(%rsp)
  401a30:	00 00 00 00 
  401a34:	ba 00 00 00 00       	mov    $0x0,%edx
  401a39:	be 01 00 00 00       	mov    $0x1,%esi
  401a3e:	bf 02 00 00 00       	mov    $0x2,%edi
  401a43:	e8 78 f2 ff ff       	callq  400cc0 <socket@plt>
  401a48:	41 89 c4             	mov    %eax,%r12d
  401a4b:	85 c0                	test   %eax,%eax
  401a4d:	79 50                	jns    401a9f <submitr+0xaf>
  401a4f:	48 b8 45 72 72 6f 72 	movabs $0x43203a726f727245,%rax
  401a56:	3a 20 43 
  401a59:	49 89 06             	mov    %rax,(%r14)
  401a5c:	48 b8 6c 69 65 6e 74 	movabs $0x6e7520746e65696c,%rax
  401a63:	20 75 6e 
  401a66:	49 89 46 08          	mov    %rax,0x8(%r14)
  401a6a:	48 b8 61 62 6c 65 20 	movabs $0x206f7420656c6261,%rax
  401a71:	74 6f 20 
  401a74:	49 89 46 10          	mov    %rax,0x10(%r14)
  401a78:	48 b8 63 72 65 61 74 	movabs $0x7320657461657263,%rax
  401a7f:	65 20 73 
  401a82:	49 89 46 18          	mov    %rax,0x18(%r14)
  401a86:	41 c7 46 20 6f 63 6b 	movl   $0x656b636f,0x20(%r14)
  401a8d:	65 
  401a8e:	66 41 c7 46 24 74 00 	movw   $0x74,0x24(%r14)
  401a95:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  401a9a:	e9 06 07 00 00       	jmpq   4021a5 <submitr+0x7b5>
  401a9f:	48 89 ef             	mov    %rbp,%rdi
  401aa2:	e8 29 f1 ff ff       	callq  400bd0 <gethostbyname@plt>
  401aa7:	48 85 c0             	test   %rax,%rax
  401aaa:	75 6b                	jne    401b17 <submitr+0x127>
  401aac:	48 b8 45 72 72 6f 72 	movabs $0x44203a726f727245,%rax
  401ab3:	3a 20 44 
  401ab6:	49 89 06             	mov    %rax,(%r14)
  401ab9:	48 b8 4e 53 20 69 73 	movabs $0x6e7520736920534e,%rax
  401ac0:	20 75 6e 
  401ac3:	49 89 46 08          	mov    %rax,0x8(%r14)
  401ac7:	48 b8 61 62 6c 65 20 	movabs $0x206f7420656c6261,%rax
  401ace:	74 6f 20 
  401ad1:	49 89 46 10          	mov    %rax,0x10(%r14)
  401ad5:	48 b8 72 65 73 6f 6c 	movabs $0x2065766c6f736572,%rax
  401adc:	76 65 20 
  401adf:	49 89 46 18          	mov    %rax,0x18(%r14)
  401ae3:	48 b8 73 65 72 76 65 	movabs $0x6120726576726573,%rax
  401aea:	72 20 61 
  401aed:	49 89 46 20          	mov    %rax,0x20(%r14)
  401af1:	41 c7 46 28 64 64 72 	movl   $0x65726464,0x28(%r14)
  401af8:	65 
  401af9:	66 41 c7 46 2c 73 73 	movw   $0x7373,0x2c(%r14)
  401b00:	41 c6 46 2e 00       	movb   $0x0,0x2e(%r14)
  401b05:	44 89 e7             	mov    %r12d,%edi
  401b08:	e8 73 f0 ff ff       	callq  400b80 <close@plt>
  401b0d:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  401b12:	e9 8e 06 00 00       	jmpq   4021a5 <submitr+0x7b5>
  401b17:	48 c7 84 24 50 a0 00 	movq   $0x0,0xa050(%rsp)
  401b1e:	00 00 00 00 00 
  401b23:	48 c7 84 24 58 a0 00 	movq   $0x0,0xa058(%rsp)
  401b2a:	00 00 00 00 00 
  401b2f:	66 c7 84 24 50 a0 00 	movw   $0x2,0xa050(%rsp)
  401b36:	00 02 00 
  401b39:	48 63 50 14          	movslq 0x14(%rax),%rdx
  401b3d:	48 8d b4 24 54 a0 00 	lea    0xa054(%rsp),%rsi
  401b44:	00 
  401b45:	48 8b 40 18          	mov    0x18(%rax),%rax
  401b49:	48 8b 38             	mov    (%rax),%rdi
  401b4c:	e8 ef f0 ff ff       	callq  400c40 <bcopy@plt>
  401b51:	66 41 c1 cd 08       	ror    $0x8,%r13w
  401b56:	66 44 89 ac 24 52 a0 	mov    %r13w,0xa052(%rsp)
  401b5d:	00 00 
  401b5f:	ba 10 00 00 00       	mov    $0x10,%edx
  401b64:	48 8d b4 24 50 a0 00 	lea    0xa050(%rsp),%rsi
  401b6b:	00 
  401b6c:	44 89 e7             	mov    %r12d,%edi
  401b6f:	e8 1c f1 ff ff       	callq  400c90 <connect@plt>
  401b74:	85 c0                	test   %eax,%eax
  401b76:	79 5d                	jns    401bd5 <submitr+0x1e5>
  401b78:	48 b8 45 72 72 6f 72 	movabs $0x55203a726f727245,%rax
  401b7f:	3a 20 55 
  401b82:	49 89 06             	mov    %rax,(%r14)
  401b85:	48 b8 6e 61 62 6c 65 	movabs $0x6f7420656c62616e,%rax
  401b8c:	20 74 6f 
  401b8f:	49 89 46 08          	mov    %rax,0x8(%r14)
  401b93:	48 b8 20 63 6f 6e 6e 	movabs $0x7463656e6e6f6320,%rax
  401b9a:	65 63 74 
  401b9d:	49 89 46 10          	mov    %rax,0x10(%r14)
  401ba1:	48 b8 20 74 6f 20 74 	movabs $0x20656874206f7420,%rax
  401ba8:	68 65 20 
  401bab:	49 89 46 18          	mov    %rax,0x18(%r14)
  401baf:	41 c7 46 20 73 65 72 	movl   $0x76726573,0x20(%r14)
  401bb6:	76 
  401bb7:	66 41 c7 46 24 65 72 	movw   $0x7265,0x24(%r14)
  401bbe:	41 c6 46 26 00       	movb   $0x0,0x26(%r14)
  401bc3:	44 89 e7             	mov    %r12d,%edi
  401bc6:	e8 b5 ef ff ff       	callq  400b80 <close@plt>
  401bcb:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  401bd0:	e9 d0 05 00 00       	jmpq   4021a5 <submitr+0x7b5>
  401bd5:	48 c7 c2 ff ff ff ff 	mov    $0xffffffffffffffff,%rdx
  401bdc:	48 89 df             	mov    %rbx,%rdi
  401bdf:	b8 00 00 00 00       	mov    $0x0,%eax
  401be4:	48 89 d1             	mov    %rdx,%rcx
  401be7:	f2 ae                	repnz scas %es:(%rdi),%al
  401be9:	48 f7 d1             	not    %rcx
  401bec:	48 89 ce             	mov    %rcx,%rsi
  401bef:	48 8b 7c 24 08       	mov    0x8(%rsp),%rdi
  401bf4:	48 89 d1             	mov    %rdx,%rcx
  401bf7:	f2 ae                	repnz scas %es:(%rdi),%al
  401bf9:	49 89 c8             	mov    %rcx,%r8
  401bfc:	48 8b 7c 24 10       	mov    0x10(%rsp),%rdi
  401c01:	48 89 d1             	mov    %rdx,%rcx
  401c04:	f2 ae                	repnz scas %es:(%rdi),%al
  401c06:	48 f7 d1             	not    %rcx
  401c09:	49 89 c9             	mov    %rcx,%r9
  401c0c:	4c 89 ff             	mov    %r15,%rdi
  401c0f:	48 89 d1             	mov    %rdx,%rcx
  401c12:	f2 ae                	repnz scas %es:(%rdi),%al
  401c14:	4d 29 c1             	sub    %r8,%r9
  401c17:	49 29 c9             	sub    %rcx,%r9
  401c1a:	48 8d 44 76 fd       	lea    -0x3(%rsi,%rsi,2),%rax
  401c1f:	49 8d 44 01 7b       	lea    0x7b(%r9,%rax,1),%rax
  401c24:	48 3d 00 20 00 00    	cmp    $0x2000,%rax
  401c2a:	76 73                	jbe    401c9f <submitr+0x2af>
  401c2c:	48 b8 45 72 72 6f 72 	movabs $0x52203a726f727245,%rax
  401c33:	3a 20 52 
  401c36:	49 89 06             	mov    %rax,(%r14)
  401c39:	48 b8 65 73 75 6c 74 	movabs $0x747320746c757365,%rax
  401c40:	20 73 74 
  401c43:	49 89 46 08          	mov    %rax,0x8(%r14)
  401c47:	48 b8 72 69 6e 67 20 	movabs $0x6f6f7420676e6972,%rax
  401c4e:	74 6f 6f 
  401c51:	49 89 46 10          	mov    %rax,0x10(%r14)
  401c55:	48 b8 20 6c 61 72 67 	movabs $0x202e656772616c20,%rax
  401c5c:	65 2e 20 
  401c5f:	49 89 46 18          	mov    %rax,0x18(%r14)
  401c63:	48 b8 49 6e 63 72 65 	movabs $0x6573616572636e49,%rax
  401c6a:	61 73 65 
  401c6d:	49 89 46 20          	mov    %rax,0x20(%r14)
  401c71:	48 b8 20 53 55 42 4d 	movabs $0x5254494d42555320,%rax
  401c78:	49 54 52 
  401c7b:	49 89 46 28          	mov    %rax,0x28(%r14)
  401c7f:	48 b8 5f 4d 41 58 42 	movabs $0x46554258414d5f,%rax
  401c86:	55 46 00 
  401c89:	49 89 46 30          	mov    %rax,0x30(%r14)
  401c8d:	44 89 e7             	mov    %r12d,%edi
  401c90:	e8 eb ee ff ff       	callq  400b80 <close@plt>
  401c95:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  401c9a:	e9 06 05 00 00       	jmpq   4021a5 <submitr+0x7b5>
  401c9f:	48 8d 94 24 40 40 00 	lea    0x4040(%rsp),%rdx
  401ca6:	00 
  401ca7:	b9 00 04 00 00       	mov    $0x400,%ecx
  401cac:	b8 00 00 00 00       	mov    $0x0,%eax
  401cb1:	48 89 d7             	mov    %rdx,%rdi
  401cb4:	f3 48 ab             	rep stos %rax,%es:(%rdi)
  401cb7:	48 89 df             	mov    %rbx,%rdi
  401cba:	48 c7 c1 ff ff ff ff 	mov    $0xffffffffffffffff,%rcx
  401cc1:	f2 ae                	repnz scas %es:(%rdi),%al
  401cc3:	48 f7 d1             	not    %rcx
  401cc6:	48 83 e9 01          	sub    $0x1,%rcx
  401cca:	85 c9                	test   %ecx,%ecx
  401ccc:	0f 84 d3 03 00 00    	je     4020a5 <submitr+0x6b5>
  401cd2:	8d 41 ff             	lea    -0x1(%rcx),%eax
  401cd5:	4c 8d 6c 03 01       	lea    0x1(%rbx,%rax,1),%r13
  401cda:	48 89 d5             	mov    %rdx,%rbp
  401cdd:	0f b6 13             	movzbl (%rbx),%edx
  401ce0:	80 fa 2a             	cmp    $0x2a,%dl
  401ce3:	74 1f                	je     401d04 <submitr+0x314>
  401ce5:	8d 42 d3             	lea    -0x2d(%rdx),%eax
  401ce8:	3c 01                	cmp    $0x1,%al
  401cea:	76 18                	jbe    401d04 <submitr+0x314>
  401cec:	80 fa 5f             	cmp    $0x5f,%dl
  401cef:	74 13                	je     401d04 <submitr+0x314>
  401cf1:	8d 42 d0             	lea    -0x30(%rdx),%eax
  401cf4:	3c 09                	cmp    $0x9,%al
  401cf6:	76 0c                	jbe    401d04 <submitr+0x314>
  401cf8:	89 d0                	mov    %edx,%eax
  401cfa:	83 e0 df             	and    $0xffffffdf,%eax
  401cfd:	83 e8 41             	sub    $0x41,%eax
  401d00:	3c 19                	cmp    $0x19,%al
  401d02:	77 09                	ja     401d0d <submitr+0x31d>
  401d04:	48 8d 45 01          	lea    0x1(%rbp),%rax
  401d08:	88 55 00             	mov    %dl,0x0(%rbp)
  401d0b:	eb 52                	jmp    401d5f <submitr+0x36f>
  401d0d:	80 fa 20             	cmp    $0x20,%dl
  401d10:	75 0a                	jne    401d1c <submitr+0x32c>
  401d12:	48 8d 45 01          	lea    0x1(%rbp),%rax
  401d16:	c6 45 00 2b          	movb   $0x2b,0x0(%rbp)
  401d1a:	eb 43                	jmp    401d5f <submitr+0x36f>
  401d1c:	8d 42 e0             	lea    -0x20(%rdx),%eax
  401d1f:	3c 5f                	cmp    $0x5f,%al
  401d21:	76 09                	jbe    401d2c <submitr+0x33c>
  401d23:	80 fa 09             	cmp    $0x9,%dl
  401d26:	0f 85 f1 03 00 00    	jne    40211d <submitr+0x72d>
  401d2c:	0f b6 d2             	movzbl %dl,%edx
  401d2f:	be 78 2a 40 00       	mov    $0x402a78,%esi
  401d34:	48 8d 7c 24 20       	lea    0x20(%rsp),%rdi
  401d39:	b8 00 00 00 00       	mov    $0x0,%eax
  401d3e:	e8 2d ef ff ff       	callq  400c70 <sprintf@plt>
  401d43:	0f b6 44 24 20       	movzbl 0x20(%rsp),%eax
  401d48:	88 45 00             	mov    %al,0x0(%rbp)
  401d4b:	0f b6 44 24 21       	movzbl 0x21(%rsp),%eax
  401d50:	88 45 01             	mov    %al,0x1(%rbp)
  401d53:	48 8d 45 03          	lea    0x3(%rbp),%rax
  401d57:	0f b6 54 24 22       	movzbl 0x22(%rsp),%edx
  401d5c:	88 55 02             	mov    %dl,0x2(%rbp)
  401d5f:	48 83 c3 01          	add    $0x1,%rbx
  401d63:	4c 39 eb             	cmp    %r13,%rbx
  401d66:	0f 84 39 03 00 00    	je     4020a5 <submitr+0x6b5>
  401d6c:	48 89 c5             	mov    %rax,%rbp
  401d6f:	e9 69 ff ff ff       	jmpq   401cdd <submitr+0x2ed>
  401d74:	48 89 da             	mov    %rbx,%rdx
  401d77:	48 89 ee             	mov    %rbp,%rsi
  401d7a:	44 89 e7             	mov    %r12d,%edi
  401d7d:	e8 ce ed ff ff       	callq  400b50 <write@plt>
  401d82:	48 85 c0             	test   %rax,%rax
  401d85:	7f 10                	jg     401d97 <submitr+0x3a7>
  401d87:	e8 94 ed ff ff       	callq  400b20 <__errno_location@plt>
  401d8c:	83 38 04             	cmpl   $0x4,(%rax)
  401d8f:	90                   	nop
  401d90:	75 12                	jne    401da4 <submitr+0x3b4>
  401d92:	b8 00 00 00 00       	mov    $0x0,%eax
  401d97:	48 01 c5             	add    %rax,%rbp
  401d9a:	48 29 c3             	sub    %rax,%rbx
  401d9d:	75 d5                	jne    401d74 <submitr+0x384>
  401d9f:	4d 85 ed             	test   %r13,%r13
  401da2:	79 5f                	jns    401e03 <submitr+0x413>
  401da4:	48 b8 45 72 72 6f 72 	movabs $0x43203a726f727245,%rax
  401dab:	3a 20 43 
  401dae:	49 89 06             	mov    %rax,(%r14)
  401db1:	48 b8 6c 69 65 6e 74 	movabs $0x6e7520746e65696c,%rax
  401db8:	20 75 6e 
  401dbb:	49 89 46 08          	mov    %rax,0x8(%r14)
  401dbf:	48 b8 61 62 6c 65 20 	movabs $0x206f7420656c6261,%rax
  401dc6:	74 6f 20 
  401dc9:	49 89 46 10          	mov    %rax,0x10(%r14)
  401dcd:	48 b8 77 72 69 74 65 	movabs $0x6f74206574697277,%rax
  401dd4:	20 74 6f 
  401dd7:	49 89 46 18          	mov    %rax,0x18(%r14)
  401ddb:	48 b8 20 74 68 65 20 	movabs $0x7265732065687420,%rax
  401de2:	73 65 72 
  401de5:	49 89 46 20          	mov    %rax,0x20(%r14)
  401de9:	41 c7 46 28 76 65 72 	movl   $0x726576,0x28(%r14)
  401df0:	00 
  401df1:	44 89 e7             	mov    %r12d,%edi
  401df4:	e8 87 ed ff ff       	callq  400b80 <close@plt>
  401df9:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  401dfe:	e9 a2 03 00 00       	jmpq   4021a5 <submitr+0x7b5>
  401e03:	44 89 a4 24 40 80 00 	mov    %r12d,0x8040(%rsp)
  401e0a:	00 
  401e0b:	c7 84 24 44 80 00 00 	movl   $0x0,0x8044(%rsp)
  401e12:	00 00 00 00 
  401e16:	48 8d 84 24 50 80 00 	lea    0x8050(%rsp),%rax
  401e1d:	00 
  401e1e:	48 89 84 24 48 80 00 	mov    %rax,0x8048(%rsp)
  401e25:	00 
  401e26:	ba 00 20 00 00       	mov    $0x2000,%edx
  401e2b:	48 8d b4 24 40 60 00 	lea    0x6040(%rsp),%rsi
  401e32:	00 
  401e33:	48 8d bc 24 40 80 00 	lea    0x8040(%rsp),%rdi
  401e3a:	00 
  401e3b:	e8 99 fa ff ff       	callq  4018d9 <rio_readlineb>
  401e40:	48 85 c0             	test   %rax,%rax
  401e43:	7f 74                	jg     401eb9 <submitr+0x4c9>
  401e45:	48 b8 45 72 72 6f 72 	movabs $0x43203a726f727245,%rax
  401e4c:	3a 20 43 
  401e4f:	49 89 06             	mov    %rax,(%r14)
  401e52:	48 b8 6c 69 65 6e 74 	movabs $0x6e7520746e65696c,%rax
  401e59:	20 75 6e 
  401e5c:	49 89 46 08          	mov    %rax,0x8(%r14)
  401e60:	48 b8 61 62 6c 65 20 	movabs $0x206f7420656c6261,%rax
  401e67:	74 6f 20 
  401e6a:	49 89 46 10          	mov    %rax,0x10(%r14)
  401e6e:	48 b8 72 65 61 64 20 	movabs $0x7269662064616572,%rax
  401e75:	66 69 72 
  401e78:	49 89 46 18          	mov    %rax,0x18(%r14)
  401e7c:	48 b8 73 74 20 68 65 	movabs $0x6564616568207473,%rax
  401e83:	61 64 65 
  401e86:	49 89 46 20          	mov    %rax,0x20(%r14)
  401e8a:	48 b8 72 20 66 72 6f 	movabs $0x73206d6f72662072,%rax
  401e91:	6d 20 73 
  401e94:	49 89 46 28          	mov    %rax,0x28(%r14)
  401e98:	41 c7 46 30 65 72 76 	movl   $0x65767265,0x30(%r14)
  401e9f:	65 
  401ea0:	66 41 c7 46 34 72 00 	movw   $0x72,0x34(%r14)
  401ea7:	44 89 e7             	mov    %r12d,%edi
  401eaa:	e8 d1 ec ff ff       	callq  400b80 <close@plt>
  401eaf:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  401eb4:	e9 ec 02 00 00       	jmpq   4021a5 <submitr+0x7b5>
  401eb9:	4c 8d 44 24 30       	lea    0x30(%rsp),%r8
  401ebe:	48 8d 8c 24 3c 20 00 	lea    0x203c(%rsp),%rcx
  401ec5:	00 
  401ec6:	48 8d 94 24 40 20 00 	lea    0x2040(%rsp),%rdx
  401ecd:	00 
  401ece:	be 7f 2a 40 00       	mov    $0x402a7f,%esi
  401ed3:	48 8d bc 24 40 60 00 	lea    0x6040(%rsp),%rdi
  401eda:	00 
  401edb:	b8 00 00 00 00       	mov    $0x0,%eax
  401ee0:	e8 4b ed ff ff       	callq  400c30 <__isoc99_sscanf@plt>
  401ee5:	8b 94 24 3c 20 00 00 	mov    0x203c(%rsp),%edx
  401eec:	81 fa c8 00 00 00    	cmp    $0xc8,%edx
  401ef2:	0f 84 b2 00 00 00    	je     401faa <submitr+0x5ba>
  401ef8:	48 8d 4c 24 30       	lea    0x30(%rsp),%rcx
  401efd:	be d0 29 40 00       	mov    $0x4029d0,%esi
  401f02:	4c 89 f7             	mov    %r14,%rdi
  401f05:	b8 00 00 00 00       	mov    $0x0,%eax
  401f0a:	e8 61 ed ff ff       	callq  400c70 <sprintf@plt>
  401f0f:	44 89 e7             	mov    %r12d,%edi
  401f12:	e8 69 ec ff ff       	callq  400b80 <close@plt>
  401f17:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  401f1c:	e9 84 02 00 00       	jmpq   4021a5 <submitr+0x7b5>
  401f21:	ba 00 20 00 00       	mov    $0x2000,%edx
  401f26:	48 8d b4 24 40 60 00 	lea    0x6040(%rsp),%rsi
  401f2d:	00 
  401f2e:	48 8d bc 24 40 80 00 	lea    0x8040(%rsp),%rdi
  401f35:	00 
  401f36:	e8 9e f9 ff ff       	callq  4018d9 <rio_readlineb>
  401f3b:	48 85 c0             	test   %rax,%rax
  401f3e:	7f 6a                	jg     401faa <submitr+0x5ba>
  401f40:	48 b8 45 72 72 6f 72 	movabs $0x43203a726f727245,%rax
  401f47:	3a 20 43 
  401f4a:	49 89 06             	mov    %rax,(%r14)
  401f4d:	48 b8 6c 69 65 6e 74 	movabs $0x6e7520746e65696c,%rax
  401f54:	20 75 6e 
  401f57:	49 89 46 08          	mov    %rax,0x8(%r14)
  401f5b:	48 b8 61 62 6c 65 20 	movabs $0x206f7420656c6261,%rax
  401f62:	74 6f 20 
  401f65:	49 89 46 10          	mov    %rax,0x10(%r14)
  401f69:	48 b8 72 65 61 64 20 	movabs $0x6165682064616572,%rax
  401f70:	68 65 61 
  401f73:	49 89 46 18          	mov    %rax,0x18(%r14)
  401f77:	48 b8 64 65 72 73 20 	movabs $0x6f72662073726564,%rax
  401f7e:	66 72 6f 
  401f81:	49 89 46 20          	mov    %rax,0x20(%r14)
  401f85:	48 b8 6d 20 73 65 72 	movabs $0x726576726573206d,%rax
  401f8c:	76 65 72 
  401f8f:	49 89 46 28          	mov    %rax,0x28(%r14)
  401f93:	41 c6 46 30 00       	movb   $0x0,0x30(%r14)
  401f98:	44 89 e7             	mov    %r12d,%edi
  401f9b:	e8 e0 eb ff ff       	callq  400b80 <close@plt>
  401fa0:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  401fa5:	e9 fb 01 00 00       	jmpq   4021a5 <submitr+0x7b5>
  401faa:	80 bc 24 40 60 00 00 	cmpb   $0xd,0x6040(%rsp)
  401fb1:	0d 
  401fb2:	0f 85 69 ff ff ff    	jne    401f21 <submitr+0x531>
  401fb8:	80 bc 24 41 60 00 00 	cmpb   $0xa,0x6041(%rsp)
  401fbf:	0a 
  401fc0:	0f 85 5b ff ff ff    	jne    401f21 <submitr+0x531>
  401fc6:	80 bc 24 42 60 00 00 	cmpb   $0x0,0x6042(%rsp)
  401fcd:	00 
  401fce:	0f 85 4d ff ff ff    	jne    401f21 <submitr+0x531>
  401fd4:	ba 00 20 00 00       	mov    $0x2000,%edx
  401fd9:	48 8d b4 24 40 60 00 	lea    0x6040(%rsp),%rsi
  401fe0:	00 
  401fe1:	48 8d bc 24 40 80 00 	lea    0x8040(%rsp),%rdi
  401fe8:	00 
  401fe9:	e8 eb f8 ff ff       	callq  4018d9 <rio_readlineb>
  401fee:	48 85 c0             	test   %rax,%rax
  401ff1:	7f 73                	jg     402066 <submitr+0x676>
  401ff3:	48 b8 45 72 72 6f 72 	movabs $0x43203a726f727245,%rax
  401ffa:	3a 20 43 
  401ffd:	49 89 06             	mov    %rax,(%r14)
  402000:	48 b8 6c 69 65 6e 74 	movabs $0x6e7520746e65696c,%rax
  402007:	20 75 6e 
  40200a:	49 89 46 08          	mov    %rax,0x8(%r14)
  40200e:	48 b8 61 62 6c 65 20 	movabs $0x206f7420656c6261,%rax
  402015:	74 6f 20 
  402018:	49 89 46 10          	mov    %rax,0x10(%r14)
  40201c:	48 b8 72 65 61 64 20 	movabs $0x6174732064616572,%rax
  402023:	73 74 61 
  402026:	49 89 46 18          	mov    %rax,0x18(%r14)
  40202a:	48 b8 74 75 73 20 6d 	movabs $0x7373656d20737574,%rax
  402031:	65 73 73 
  402034:	49 89 46 20          	mov    %rax,0x20(%r14)
  402038:	48 b8 61 67 65 20 66 	movabs $0x6d6f726620656761,%rax
  40203f:	72 6f 6d 
  402042:	49 89 46 28          	mov    %rax,0x28(%r14)
  402046:	48 b8 20 73 65 72 76 	movabs $0x72657672657320,%rax
  40204d:	65 72 00 
  402050:	49 89 46 30          	mov    %rax,0x30(%r14)
  402054:	44 89 e7             	mov    %r12d,%edi
  402057:	e8 24 eb ff ff       	callq  400b80 <close@plt>
  40205c:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  402061:	e9 3f 01 00 00       	jmpq   4021a5 <submitr+0x7b5>
  402066:	48 8d b4 24 40 60 00 	lea    0x6040(%rsp),%rsi
  40206d:	00 
  40206e:	4c 89 f7             	mov    %r14,%rdi
  402071:	e8 ba ea ff ff       	callq  400b30 <strcpy@plt>
  402076:	44 89 e7             	mov    %r12d,%edi
  402079:	e8 02 eb ff ff       	callq  400b80 <close@plt>
  40207e:	41 0f b6 06          	movzbl (%r14),%eax
  402082:	83 e8 4f             	sub    $0x4f,%eax
  402085:	75 0f                	jne    402096 <submitr+0x6a6>
  402087:	41 0f b6 46 01       	movzbl 0x1(%r14),%eax
  40208c:	83 e8 4b             	sub    $0x4b,%eax
  40208f:	75 05                	jne    402096 <submitr+0x6a6>
  402091:	41 0f b6 46 02       	movzbl 0x2(%r14),%eax
  402096:	85 c0                	test   %eax,%eax
  402098:	0f 95 c0             	setne  %al
  40209b:	0f b6 c0             	movzbl %al,%eax
  40209e:	f7 d8                	neg    %eax
  4020a0:	e9 00 01 00 00       	jmpq   4021a5 <submitr+0x7b5>
  4020a5:	48 8d 84 24 40 40 00 	lea    0x4040(%rsp),%rax
  4020ac:	00 
  4020ad:	48 89 04 24          	mov    %rax,(%rsp)
  4020b1:	4d 89 f9             	mov    %r15,%r9
  4020b4:	4c 8b 44 24 18       	mov    0x18(%rsp),%r8
  4020b9:	48 8b 4c 24 10       	mov    0x10(%rsp),%rcx
  4020be:	48 8b 54 24 08       	mov    0x8(%rsp),%rdx
  4020c3:	be 00 2a 40 00       	mov    $0x402a00,%esi
  4020c8:	48 8d bc 24 40 60 00 	lea    0x6040(%rsp),%rdi
  4020cf:	00 
  4020d0:	b8 00 00 00 00       	mov    $0x0,%eax
  4020d5:	e8 96 eb ff ff       	callq  400c70 <sprintf@plt>
  4020da:	48 8d bc 24 40 60 00 	lea    0x6040(%rsp),%rdi
  4020e1:	00 
  4020e2:	e8 59 ea ff ff       	callq  400b40 <puts@plt>
  4020e7:	48 8d bc 24 40 60 00 	lea    0x6040(%rsp),%rdi
  4020ee:	00 
  4020ef:	b8 00 00 00 00       	mov    $0x0,%eax
  4020f4:	48 c7 c1 ff ff ff ff 	mov    $0xffffffffffffffff,%rcx
  4020fb:	f2 ae                	repnz scas %es:(%rdi),%al
  4020fd:	48 f7 d1             	not    %rcx
  402100:	48 83 e9 01          	sub    $0x1,%rcx
  402104:	49 89 cd             	mov    %rcx,%r13
  402107:	0f 84 f6 fc ff ff    	je     401e03 <submitr+0x413>
  40210d:	48 89 cb             	mov    %rcx,%rbx
  402110:	48 8d ac 24 40 60 00 	lea    0x6040(%rsp),%rbp
  402117:	00 
  402118:	e9 57 fc ff ff       	jmpq   401d74 <submitr+0x384>
  40211d:	48 b8 45 72 72 6f 72 	movabs $0x52203a726f727245,%rax
  402124:	3a 20 52 
  402127:	49 89 06             	mov    %rax,(%r14)
  40212a:	48 b8 65 73 75 6c 74 	movabs $0x747320746c757365,%rax
  402131:	20 73 74 
  402134:	49 89 46 08          	mov    %rax,0x8(%r14)
  402138:	48 b8 72 69 6e 67 20 	movabs $0x6e6f6320676e6972,%rax
  40213f:	63 6f 6e 
  402142:	49 89 46 10          	mov    %rax,0x10(%r14)
  402146:	48 b8 74 61 69 6e 73 	movabs $0x6e6120736e696174,%rax
  40214d:	20 61 6e 
  402150:	49 89 46 18          	mov    %rax,0x18(%r14)
  402154:	48 b8 20 69 6c 6c 65 	movabs $0x6c6167656c6c6920,%rax
  40215b:	67 61 6c 
  40215e:	49 89 46 20          	mov    %rax,0x20(%r14)
  402162:	48 b8 20 6f 72 20 75 	movabs $0x72706e7520726f20,%rax
  402169:	6e 70 72 
  40216c:	49 89 46 28          	mov    %rax,0x28(%r14)
  402170:	48 b8 69 6e 74 61 62 	movabs $0x20656c6261746e69,%rax
  402177:	6c 65 20 
  40217a:	49 89 46 30          	mov    %rax,0x30(%r14)
  40217e:	48 b8 63 68 61 72 61 	movabs $0x6574636172616863,%rax
  402185:	63 74 65 
  402188:	49 89 46 38          	mov    %rax,0x38(%r14)
  40218c:	66 41 c7 46 40 72 2e 	movw   $0x2e72,0x40(%r14)
  402193:	41 c6 46 42 00       	movb   $0x0,0x42(%r14)
  402198:	44 89 e7             	mov    %r12d,%edi
  40219b:	e8 e0 e9 ff ff       	callq  400b80 <close@plt>
  4021a0:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  4021a5:	48 81 c4 68 a0 00 00 	add    $0xa068,%rsp
  4021ac:	5b                   	pop    %rbx
  4021ad:	5d                   	pop    %rbp
  4021ae:	41 5c                	pop    %r12
  4021b0:	41 5d                	pop    %r13
  4021b2:	41 5e                	pop    %r14
  4021b4:	41 5f                	pop    %r15
  4021b6:	c3                   	retq   

00000000004021b7 <init_timeout>:
  4021b7:	53                   	push   %rbx
  4021b8:	89 fb                	mov    %edi,%ebx
  4021ba:	85 ff                	test   %edi,%edi
  4021bc:	74 1e                	je     4021dc <init_timeout+0x25>
  4021be:	be b0 18 40 00       	mov    $0x4018b0,%esi
  4021c3:	bf 0e 00 00 00       	mov    $0xe,%edi
  4021c8:	e8 f3 e9 ff ff       	callq  400bc0 <signal@plt>
  4021cd:	85 db                	test   %ebx,%ebx
  4021cf:	bf 00 00 00 00       	mov    $0x0,%edi
  4021d4:	0f 49 fb             	cmovns %ebx,%edi
  4021d7:	e8 94 e9 ff ff       	callq  400b70 <alarm@plt>
  4021dc:	5b                   	pop    %rbx
  4021dd:	c3                   	retq   

00000000004021de <init_driver>:
  4021de:	55                   	push   %rbp
  4021df:	53                   	push   %rbx
  4021e0:	48 83 ec 18          	sub    $0x18,%rsp
  4021e4:	48 89 fd             	mov    %rdi,%rbp
  4021e7:	be 01 00 00 00       	mov    $0x1,%esi
  4021ec:	bf 0d 00 00 00       	mov    $0xd,%edi
  4021f1:	e8 ca e9 ff ff       	callq  400bc0 <signal@plt>
  4021f6:	be 01 00 00 00       	mov    $0x1,%esi
  4021fb:	bf 1d 00 00 00       	mov    $0x1d,%edi
  402200:	e8 bb e9 ff ff       	callq  400bc0 <signal@plt>
  402205:	be 01 00 00 00       	mov    $0x1,%esi
  40220a:	bf 1d 00 00 00       	mov    $0x1d,%edi
  40220f:	e8 ac e9 ff ff       	callq  400bc0 <signal@plt>
  402214:	ba 00 00 00 00       	mov    $0x0,%edx
  402219:	be 01 00 00 00       	mov    $0x1,%esi
  40221e:	bf 02 00 00 00       	mov    $0x2,%edi
  402223:	e8 98 ea ff ff       	callq  400cc0 <socket@plt>
  402228:	89 c3                	mov    %eax,%ebx
  40222a:	85 c0                	test   %eax,%eax
  40222c:	79 4f                	jns    40227d <init_driver+0x9f>
  40222e:	48 b8 45 72 72 6f 72 	movabs $0x43203a726f727245,%rax
  402235:	3a 20 43 
  402238:	48 89 45 00          	mov    %rax,0x0(%rbp)
  40223c:	48 b8 6c 69 65 6e 74 	movabs $0x6e7520746e65696c,%rax
  402243:	20 75 6e 
  402246:	48 89 45 08          	mov    %rax,0x8(%rbp)
  40224a:	48 b8 61 62 6c 65 20 	movabs $0x206f7420656c6261,%rax
  402251:	74 6f 20 
  402254:	48 89 45 10          	mov    %rax,0x10(%rbp)
  402258:	48 b8 63 72 65 61 74 	movabs $0x7320657461657263,%rax
  40225f:	65 20 73 
  402262:	48 89 45 18          	mov    %rax,0x18(%rbp)
  402266:	c7 45 20 6f 63 6b 65 	movl   $0x656b636f,0x20(%rbp)
  40226d:	66 c7 45 24 74 00    	movw   $0x74,0x24(%rbp)
  402273:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  402278:	e9 f8 00 00 00       	jmpq   402375 <init_driver+0x197>
  40227d:	bf 90 2a 40 00       	mov    $0x402a90,%edi
  402282:	e8 49 e9 ff ff       	callq  400bd0 <gethostbyname@plt>
  402287:	48 85 c0             	test   %rax,%rax
  40228a:	75 68                	jne    4022f4 <init_driver+0x116>
  40228c:	48 b8 45 72 72 6f 72 	movabs $0x44203a726f727245,%rax
  402293:	3a 20 44 
  402296:	48 89 45 00          	mov    %rax,0x0(%rbp)
  40229a:	48 b8 4e 53 20 69 73 	movabs $0x6e7520736920534e,%rax
  4022a1:	20 75 6e 
  4022a4:	48 89 45 08          	mov    %rax,0x8(%rbp)
  4022a8:	48 b8 61 62 6c 65 20 	movabs $0x206f7420656c6261,%rax
  4022af:	74 6f 20 
  4022b2:	48 89 45 10          	mov    %rax,0x10(%rbp)
  4022b6:	48 b8 72 65 73 6f 6c 	movabs $0x2065766c6f736572,%rax
  4022bd:	76 65 20 
  4022c0:	48 89 45 18          	mov    %rax,0x18(%rbp)
  4022c4:	48 b8 73 65 72 76 65 	movabs $0x6120726576726573,%rax
  4022cb:	72 20 61 
  4022ce:	48 89 45 20          	mov    %rax,0x20(%rbp)
  4022d2:	c7 45 28 64 64 72 65 	movl   $0x65726464,0x28(%rbp)
  4022d9:	66 c7 45 2c 73 73    	movw   $0x7373,0x2c(%rbp)
  4022df:	c6 45 2e 00          	movb   $0x0,0x2e(%rbp)
  4022e3:	89 df                	mov    %ebx,%edi
  4022e5:	e8 96 e8 ff ff       	callq  400b80 <close@plt>
  4022ea:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  4022ef:	e9 81 00 00 00       	jmpq   402375 <init_driver+0x197>
  4022f4:	48 c7 04 24 00 00 00 	movq   $0x0,(%rsp)
  4022fb:	00 
  4022fc:	48 c7 44 24 08 00 00 	movq   $0x0,0x8(%rsp)
  402303:	00 00 
  402305:	66 c7 04 24 02 00    	movw   $0x2,(%rsp)
  40230b:	48 63 50 14          	movslq 0x14(%rax),%rdx
  40230f:	48 8d 74 24 04       	lea    0x4(%rsp),%rsi
  402314:	48 8b 40 18          	mov    0x18(%rax),%rax
  402318:	48 8b 38             	mov    (%rax),%rdi
  40231b:	e8 20 e9 ff ff       	callq  400c40 <bcopy@plt>
  402320:	66 c7 44 24 02 3b 6e 	movw   $0x6e3b,0x2(%rsp)
  402327:	ba 10 00 00 00       	mov    $0x10,%edx
  40232c:	48 89 e6             	mov    %rsp,%rsi
  40232f:	89 df                	mov    %ebx,%edi
  402331:	e8 5a e9 ff ff       	callq  400c90 <connect@plt>
  402336:	85 c0                	test   %eax,%eax
  402338:	79 25                	jns    40235f <init_driver+0x181>
  40233a:	ba 90 2a 40 00       	mov    $0x402a90,%edx
  40233f:	be 50 2a 40 00       	mov    $0x402a50,%esi
  402344:	48 89 ef             	mov    %rbp,%rdi
  402347:	b8 00 00 00 00       	mov    $0x0,%eax
  40234c:	e8 1f e9 ff ff       	callq  400c70 <sprintf@plt>
  402351:	89 df                	mov    %ebx,%edi
  402353:	e8 28 e8 ff ff       	callq  400b80 <close@plt>
  402358:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  40235d:	eb 16                	jmp    402375 <init_driver+0x197>
  40235f:	89 df                	mov    %ebx,%edi
  402361:	e8 1a e8 ff ff       	callq  400b80 <close@plt>
  402366:	66 c7 45 00 4f 4b    	movw   $0x4b4f,0x0(%rbp)
  40236c:	c6 45 02 00          	movb   $0x0,0x2(%rbp)
  402370:	b8 00 00 00 00       	mov    $0x0,%eax
  402375:	48 83 c4 18          	add    $0x18,%rsp
  402379:	5b                   	pop    %rbx
  40237a:	5d                   	pop    %rbp
  40237b:	c3                   	retq   

000000000040237c <driver_post>:
  40237c:	53                   	push   %rbx
  40237d:	48 83 ec 10          	sub    $0x10,%rsp
  402381:	4c 89 c3             	mov    %r8,%rbx
  402384:	85 c9                	test   %ecx,%ecx
  402386:	74 22                	je     4023aa <driver_post+0x2e>
  402388:	48 89 d6             	mov    %rdx,%rsi
  40238b:	bf 9d 2a 40 00       	mov    $0x402a9d,%edi
  402390:	b8 00 00 00 00       	mov    $0x0,%eax
  402395:	e8 c6 e7 ff ff       	callq  400b60 <printf@plt>
  40239a:	66 c7 03 4f 4b       	movw   $0x4b4f,(%rbx)
  40239f:	c6 43 02 00          	movb   $0x0,0x2(%rbx)
  4023a3:	b8 00 00 00 00       	mov    $0x0,%eax
  4023a8:	eb 43                	jmp    4023ed <driver_post+0x71>
  4023aa:	48 85 ff             	test   %rdi,%rdi
  4023ad:	74 30                	je     4023df <driver_post+0x63>
  4023af:	80 3f 00             	cmpb   $0x0,(%rdi)
  4023b2:	74 2b                	je     4023df <driver_post+0x63>
  4023b4:	4c 89 44 24 08       	mov    %r8,0x8(%rsp)
  4023b9:	48 89 14 24          	mov    %rdx,(%rsp)
  4023bd:	41 b9 b4 2a 40 00    	mov    $0x402ab4,%r9d
  4023c3:	49 89 f0             	mov    %rsi,%r8
  4023c6:	48 89 f9             	mov    %rdi,%rcx
  4023c9:	ba ba 2a 40 00       	mov    $0x402aba,%edx
  4023ce:	be 6e 3b 00 00       	mov    $0x3b6e,%esi
  4023d3:	bf 90 2a 40 00       	mov    $0x402a90,%edi
  4023d8:	e8 13 f6 ff ff       	callq  4019f0 <submitr>
  4023dd:	eb 0e                	jmp    4023ed <driver_post+0x71>
  4023df:	66 c7 03 4f 4b       	movw   $0x4b4f,(%rbx)
  4023e4:	c6 43 02 00          	movb   $0x0,0x2(%rbx)
  4023e8:	b8 00 00 00 00       	mov    $0x0,%eax
  4023ed:	48 83 c4 10          	add    $0x10,%rsp
  4023f1:	5b                   	pop    %rbx
  4023f2:	c3                   	retq   
  4023f3:	66 2e 0f 1f 84 00 00 	nopw   %cs:0x0(%rax,%rax,1)
  4023fa:	00 00 00 
  4023fd:	0f 1f 00             	nopl   (%rax)

0000000000402400 <__libc_csu_init>:
  402400:	41 57                	push   %r15
  402402:	41 89 ff             	mov    %edi,%r15d
  402405:	41 56                	push   %r14
  402407:	49 89 f6             	mov    %rsi,%r14
  40240a:	41 55                	push   %r13
  40240c:	49 89 d5             	mov    %rdx,%r13
  40240f:	41 54                	push   %r12
  402411:	4c 8d 25 f8 19 20 00 	lea    0x2019f8(%rip),%r12        # 603e10 <__frame_dummy_init_array_entry>
  402418:	55                   	push   %rbp
  402419:	48 8d 2d f8 19 20 00 	lea    0x2019f8(%rip),%rbp        # 603e18 <__init_array_end>
  402420:	53                   	push   %rbx
  402421:	4c 29 e5             	sub    %r12,%rbp
  402424:	31 db                	xor    %ebx,%ebx
  402426:	48 c1 fd 03          	sar    $0x3,%rbp
  40242a:	48 83 ec 08          	sub    $0x8,%rsp
  40242e:	e8 9d e6 ff ff       	callq  400ad0 <_init>
  402433:	48 85 ed             	test   %rbp,%rbp
  402436:	74 1e                	je     402456 <__libc_csu_init+0x56>
  402438:	0f 1f 84 00 00 00 00 	nopl   0x0(%rax,%rax,1)
  40243f:	00 
  402440:	4c 89 ea             	mov    %r13,%rdx
  402443:	4c 89 f6             	mov    %r14,%rsi
  402446:	44 89 ff             	mov    %r15d,%edi
  402449:	41 ff 14 dc          	callq  *(%r12,%rbx,8)
  40244d:	48 83 c3 01          	add    $0x1,%rbx
  402451:	48 39 eb             	cmp    %rbp,%rbx
  402454:	75 ea                	jne    402440 <__libc_csu_init+0x40>
  402456:	48 83 c4 08          	add    $0x8,%rsp
  40245a:	5b                   	pop    %rbx
  40245b:	5d                   	pop    %rbp
  40245c:	41 5c                	pop    %r12
  40245e:	41 5d                	pop    %r13
  402460:	41 5e                	pop    %r14
  402462:	41 5f                	pop    %r15
  402464:	c3                   	retq   
  402465:	90                   	nop
  402466:	66 2e 0f 1f 84 00 00 	nopw   %cs:0x0(%rax,%rax,1)
  40246d:	00 00 00 

0000000000402470 <__libc_csu_fini>:
  402470:	f3 c3                	repz retq 

Disassembly of section .fini:

0000000000402474 <_fini>:
  402474:	48 83 ec 08          	sub    $0x8,%rsp
  402478:	48 83 c4 08          	add    $0x8,%rsp
  40247c:	c3                   	retq
```

</details>

</details>

---

## phase_1 字符串比较

可知其进行的是字符串比较，待比较的基准字符串地址为`0x4025e0`；

于是gdb调试运行：`print (char *)0x4025e0`，结果如下： 

![截屏2023-12-08 10.44.55.png](CSAPP%EF%BD%9CBomb%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-08_10.44.55.png)

可知phase_1的目标字符串是`The future will be better tomorrow.`

- `phase_1`
    
    ```asm
    0000000000400ef0 <phase_1>:
      400ef0:	48 83 ec 08          	sub    $0x8,%rsp                  ;  分配8个字节的栈空间
      400ef4:	be e0 25 40 00       	mov    $0x4025e0,%esi             ;  将地址0x4025e0赋值给esi
      400ef9:	e8 e0 04 00 00       	callq  4013de <strings_not_equal> ;  调用字符比较函数
      400efe:	85 c0                	test   %eax,%eax                  ;  检查%eax是否为0
      400f00:	74 05                	je     400f07 <phase_1+0x17>      ;  若为0则跳转，避免爆炸
      400f02:	e8 82 07 00 00       	callq  401689 <explode_bomb>      ;  否则爆炸
      400f07:	48 83 c4 08          	add    $0x8,%rsp
      400f0b:	c3                   	retq
    ```
    

---

## phase_2 循环

答案是`0 1 3 6 10 15`

- `phase_2`
    
    ```asm
    0000000000400f0c <phase_2>: ; 6个整数：最后一个为0，当前数=后一个数+当前序号（序号从后往前数，从0开始）=> 0 1 3 6 10 15
      400f0c:	55                   	push   %rbp
      400f0d:	53                   	push   %rbx                       ; 保存%rbx和%rbp在栈上
      400f0e:	48 83 ec 28          	sub    $0x28,%rsp                 ; 分配40个字节的栈空间
      400f12:	48 89 e6             	mov    %rsp,%rsi
      400f15:	e8 a5 07 00 00       	callq  4016bf <read_six_numbers>
      400f1a:	83 3c 24 00          	cmpl   $0x0,(%rsp)                ; 检查栈顶元素的值是否为0  
      400f1e:	79 24                	jns    400f44 <phase_2+0x38>      ; 若非负（即栈顶不小于0时）则跳转，否则爆炸
      400f20:	e8 64 07 00 00       	callq  401689 <explode_bomb>
      400f25:	eb 1d                	jmp    400f44 <phase_2+0x38>
      400f27:	89 d8                	mov    %ebx,%eax
      400f29:	03 45 fc             	add    -0x4(%rbp),%eax           ; rbp-4处的值（上一个数字）加到eax上
      400f2c:	39 45 00             	cmp    %eax,0x0(%rbp)            ; 比较rbp处的值（当前数字）和eax
      400f2f:	74 05                	je     400f36 <phase_2+0x2a>     ; 若相等则跳转，否则爆炸
      400f31:	e8 53 07 00 00       	callq  401689 <explode_bomb>
      400f36:	83 c3 01             	add    $0x1,%ebx                 ; 迭代器ebx加1
      400f39:	48 83 c5 04          	add    $0x4,%rbp                 ; rbp加4，指向下一个整数
      400f3d:	83 fb 06             	cmp    $0x6,%ebx
      400f40:	75 e5                	jne    400f27 <phase_2+0x1b>
      400f42:	eb 0c                	jmp    400f50 <phase_2+0x44>
      400f44:	48 8d 6c 24 04       	lea    0x4(%rsp),%rbp            ; rbp指向rsp+4处，下一个整数
      400f49:	bb 01 00 00 00       	mov    $0x1,%ebx                 ; 迭代器ebx置1
      400f4e:	eb d7                	jmp    400f27 <phase_2+0x1b>
      400f50:	48 83 c4 28          	add    $0x28,%rsp
      400f54:	5b                   	pop    %rbx
      400f55:	5d                   	pop    %rbp
      400f56:	c3                   	retq
    
    00000000004016bf <read_six_numbers>:    ; 读取6个整数依次入栈
      4016bf:	48 83 ec 18          	sub    $0x18,%rsp         ; 接下来设置参数，指示读取整数后保存的地址
      4016c3:	48 89 f2             	mov    %rsi,%rdx          ; rdx = rsi
      4016c6:	48 8d 4e 04          	lea    0x4(%rsi),%rcx     ; rcx = rsi+4
      4016ca:	48 8d 46 14          	lea    0x14(%rsi),%rax    
      4016ce:	48 89 44 24 08       	mov    %rax,0x8(%rsp)     ; [rsp+8] = rsi+20
      4016d3:	48 8d 46 10          	lea    0x10(%rsi),%rax    ; rax = rsi+16
      4016d7:	48 89 04 24          	mov    %rax,(%rsp)
      4016db:	4c 8d 4e 0c          	lea    0xc(%rsi),%r9      ; r9 = rsi+12
      4016df:	4c 8d 46 08          	lea    0x8(%rsi),%r8      ; r8 = rsi+8
      4016e3:	be d5 28 40 00       	mov    $0x4028d5,%esi     ; 指示读取的目标地址和读取格式
      4016e8:	b8 00 00 00 00       	mov    $0x0,%eax
      4016ed:	e8 3e f5 ff ff       	callq  400c30 <__isoc99_sscanf@plt>
      4016f2:	83 f8 05             	cmp    $0x5,%eax
      4016f5:	7f 05                	jg     4016fc <read_six_numbers+0x3d>
      4016f7:	e8 8d ff ff ff       	callq  401689 <explode_bomb>
      4016fc:	48 83 c4 18          	add    $0x18,%rsp
      401700:	c3                   	retq
    ```
    

---

## phase_3 条件/分支switch

> 题目考察switch分支结构，涉及转发表的查看和理解，但题目的难点在于发现输入的读取格式。

此题的关键在于发现函数`<__isoc99_sscanf@plt>`的**读取格式**：

```asm
400f6a:	be 2e 26 40 00       	mov    $0x40262e,%esi           ; 设置读取格式“%d %c %d”
```

终端输入`x/s 0x40262e`，以字符串形式读取，结果如下：

![截屏2023-12-08 15.41.05.png](CSAPP%EF%BD%9CBomb%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-08_15.41.05.png)

可知其读取格式为`<int char int>`，其中后两者之间可以不空格

:::tip
之前一直以为是三个整数，然后总是发现读第二个数会覆盖第三个数，其实就是因为程序将第二个数拆开读——第一个数字作为字符，剩下的部分作为整数——之后忽略第三个数
:::

可知跳转表的起始地址为`0x402640`，且应该包含有8个地址，查看如下：

![截屏2023-12-13 13.40.42.png](CSAPP%EF%BD%9CBomb%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-13_13.40.42.png)

本题的每一个switch都对应一个答案。对于`.L1`，答案是`1 r461`。

- `phase_3`
    
    ```asm
    0000000000400f57 <phase_3>:     ; <int char int>，其中char和第二个int之间可以不空格
      400f57:	48 83 ec 18          	sub    $0x18,%rsp               ; 分配24个字节的栈空间
      400f5b:	4c 8d 44 24 08       	lea    0x8(%rsp),%r8            ; r8 = rsp+8  参数三（int）存放在rsp+8处   【根据phase_2中的read_six_number函数推导出】
      400f60:	48 8d 4c 24 07       	lea    0x7(%rsp),%rcx           ; rcx = rsp+7  参数二（char）存放在rsp+7处
      400f65:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx           ; rdx = rsp+12  参数一（int）存放在rsp+12处
      400f6a:	be 2e 26 40 00       	mov    $0x40262e,%esi           ; 读取格式为“%d %c %d”
      400f6f:	b8 00 00 00 00       	mov    $0x0,%eax
      400f74:	e8 b7 fc ff ff       	callq  400c30 <__isoc99_sscanf@plt>
      400f79:	83 f8 02             	cmp    $0x2,%eax                ; 输入整数应大于2个
      400f7c:	7f 05                	jg     400f83 <phase_3+0x2c>
      400f7e:	e8 06 07 00 00       	callq  401689 <explode_bomb>
      400f83:	83 7c 24 0c 07       	cmpl   $0x7,0xc(%rsp)           ; 转发表有8个地址，比较的对象是参数一
      400f88:	0f 87 fc 00 00 00    	ja     40108a <phase_3+0x133>
      400f8e:	8b 44 24 0c          	mov    0xc(%rsp),%eax
      400f92:	ff 24 c5 40 26 40 00 	jmpq   *0x402640(,%rax,8)       ; 跳转表的起始地址为0x402640，根据eax的值跳转到不同的地址
      ; .L0 第一个数为0时，栈不会写入
      400f99:	b8 44 00 00 00       	mov    $0x44,%eax
      400f9e:	81 7c 24 08 91 02 00 	cmpl   $0x291,0x8(%rsp)         ; 比较第三个数和0x291，相等则跳转，否则爆炸
      400fa5:	00 
      400fa6:	0f 84 e8 00 00 00    	je     401094 <phase_3+0x13d>
      400fac:	e8 d8 06 00 00       	callq  401689 <explode_bomb>
      400fb1:	b8 44 00 00 00       	mov    $0x44,%eax
      400fb6:	e9 d9 00 00 00       	jmpq   401094 <phase_3+0x13d>
      ; .L1
      400fbb:	b8 72 00 00 00       	mov    $0x72,%eax               ; 0x72 = 'r'
      400fc0:	81 7c 24 08 cd 01 00 	cmpl   $0x1cd,0x8(%rsp)
      400fc7:	00 
      400fc8:	0f 84 c6 00 00 00    	je     401094 <phase_3+0x13d>
      400fce:	e8 b6 06 00 00       	callq  401689 <explode_bomb>
      400fd3:	b8 72 00 00 00       	mov    $0x72,%eax
      400fd8:	e9 b7 00 00 00       	jmpq   401094 <phase_3+0x13d>
      ; .L2
      400fdd:	b8 59 00 00 00       	mov    $0x59,%eax
      400fe2:	81 7c 24 08 a5 00 00 	cmpl   $0xa5,0x8(%rsp)
      400fe9:	00 
      400fea:	0f 84 a4 00 00 00    	je     401094 <phase_3+0x13d>
      400ff0:	e8 94 06 00 00       	callq  401689 <explode_bomb>
      400ff5:	b8 59 00 00 00       	mov    $0x59,%eax
      400ffa:	e9 95 00 00 00       	jmpq   401094 <phase_3+0x13d>
      ; .L3
      400fff:	b8 79 00 00 00       	mov    $0x79,%eax
      401004:	81 7c 24 08 2a 01 00 	cmpl   $0x12a,0x8(%rsp)
      40100b:	00 
      40100c:	0f 84 82 00 00 00    	je     401094 <phase_3+0x13d>
      401012:	e8 72 06 00 00       	callq  401689 <explode_bomb>
      401017:	b8 79 00 00 00       	mov    $0x79,%eax
      40101c:	eb 76                	jmp    401094 <phase_3+0x13d>
      ; .L4
      40101e:	b8 51 00 00 00       	mov    $0x51,%eax
      401023:	81 7c 24 08 50 02 00 	cmpl   $0x250,0x8(%rsp)
      40102a:	00 
      40102b:	74 67                	je     401094 <phase_3+0x13d>
      40102d:	e8 57 06 00 00       	callq  401689 <explode_bomb>
      401032:	b8 51 00 00 00       	mov    $0x51,%eax
      401037:	eb 5b                	jmp    401094 <phase_3+0x13d>
      ; .L5
      401039:	b8 4b 00 00 00       	mov    $0x4b,%eax
      40103e:	81 7c 24 08 76 02 00 	cmpl   $0x276,0x8(%rsp)
      401045:	00 
      401046:	74 4c                	je     401094 <phase_3+0x13d>
      401048:	e8 3c 06 00 00       	callq  401689 <explode_bomb>
      40104d:	b8 4b 00 00 00       	mov    $0x4b,%eax
      401052:	eb 40                	jmp    401094 <phase_3+0x13d>
      ; .L6
      401054:	b8 78 00 00 00       	mov    $0x78,%eax
      401059:	81 7c 24 08 cd 03 00 	cmpl   $0x3cd,0x8(%rsp)
      401060:	00 
      401061:	74 31                	je     401094 <phase_3+0x13d>
      401063:	e8 21 06 00 00       	callq  401689 <explode_bomb>
      401068:	b8 78 00 00 00       	mov    $0x78,%eax
      40106d:	eb 25                	jmp    401094 <phase_3+0x13d>
      ; .L7
      40106f:	b8 45 00 00 00       	mov    $0x45,%eax
      401074:	81 7c 24 08 2e 03 00 	cmpl   $0x32e,0x8(%rsp)
      40107b:	00 
      40107c:	74 16                	je     401094 <phase_3+0x13d>
      40107e:	e8 06 06 00 00       	callq  401689 <explode_bomb>
      401083:	b8 45 00 00 00       	mov    $0x45,%eax
      401088:	eb 0a                	jmp    401094 <phase_3+0x13d>
      ; default 
      40108a:	e8 fa 05 00 00       	callq  401689 <explode_bomb>
      40108f:	b8 76 00 00 00       	mov    $0x76,%eax
    
      401094:	3a 44 24 07          	cmp    0x7(%rsp),%al            ; 比较第二个数和eax低8位，相等则结束，否则爆炸
      401098:	74 05                	je     40109f <phase_3+0x148>
      40109a:	e8 ea 05 00 00       	callq  401689 <explode_bomb>
      40109f:	48 83 c4 18          	add    $0x18,%rsp
      4010a3:	c3                   	retq
    ```
    

---

## phase_4 递归

此题主要要将递归函数`func4()`给捋明白，可写成C语言样式辅助理解。

答案是`5 2`

- `phase_4`
    
    ```asm
    00000000004010e2 <phase_4>:   ; m=5 n=2 
      4010e2:	48 83 ec 18          	sub    $0x18,%rsp
      4010e6:	48 8d 4c 24 08       	lea    0x8(%rsp),%rcx           ; 第二个数n
      4010eb:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx           ; 第一个数m
      4010f0:	be e1 28 40 00       	mov    $0x4028e1,%esi           ; 读取格式为“%d %d” 
      4010f5:	b8 00 00 00 00       	mov    $0x0,%eax
      4010fa:	e8 31 fb ff ff       	callq  400c30 <__isoc99_sscanf@plt>
      4010ff:	83 f8 02             	cmp    $0x2,%eax
      401102:	75 07                	jne    40110b <phase_4+0x29>
      401104:	83 7c 24 0c 0e       	cmpl   $0xe,0xc(%rsp)           ; 比较第一个数m和0xe，大于等于则跳转，否则爆炸
      401109:	76 05                	jbe    401110 <phase_4+0x2e>     
      40110b:	e8 79 05 00 00       	callq  401689 <explode_bomb>
      401110:	ba 0e 00 00 00       	mov    $0xe,%edx                ; c=e 设置参数，调用函数<func4>
      401115:	be 00 00 00 00       	mov    $0x0,%esi                ; b=0
      40111a:	8b 7c 24 0c          	mov    0xc(%rsp),%edi           ; a=m 
      40111e:	e8 81 ff ff ff       	callq  4010a4 <func4>           ; func4(%edi=m, %esi=0x0, %edx=0xe)
      401123:	83 f8 02             	cmp    $0x2,%eax                ; 返回值为2则跳转，否则爆炸
      401126:	75 07                	jne    40112f <phase_4+0x4d>
      401128:	83 7c 24 08 02       	cmpl   $0x2,0x8(%rsp)           ; 比较[rsp+8]和2，相等则跳转，否则爆炸
      40112d:	74 05                	je     401134 <phase_4+0x52>
      40112f:	e8 55 05 00 00       	callq  401689 <explode_bomb>
      401134:	48 83 c4 18          	add    $0x18,%rsp
      401138:	c3                   	retq
    ```
    
- `func4`
    
    ```asm
    00000000004010a4 <func4>:
      4010a4:	48 83 ec 08          	sub    $0x8,%rsp
      4010a8:	89 d0                	mov    %edx,%eax              ; res = c
      4010aa:	29 f0                	sub    %esi,%eax              ; res -= b
      4010ac:	89 c1                	mov    %eax,%ecx              ; ecx = res>>31
      4010ae:	c1 e9 1f             	shr    $0x1f,%ecx             
      4010b1:	01 c8                	add    %ecx,%eax              ; res += ecx
      4010b3:	d1 f8                	sar    %eax                   ; res >>= 1
      4010b5:	8d 0c 30             	lea    (%rax,%rsi,1),%ecx     ; ecx = res + b
      4010b8:	39 f9                	cmp    %edi,%ecx              
      4010ba:	7e 0c                	jle    4010c8 <func4+0x24>    ; if(ecx<=a) goto 4010c8
      4010bc:	8d 51 ff             	lea    -0x1(%rcx),%edx        ; c = ecx - 1
      4010bf:	e8 e0 ff ff ff       	callq  4010a4 <func4>         ; res = func4()
      4010c4:	01 c0                	add    %eax,%eax              ; res += res
      4010c6:	eb 15                	jmp    4010dd <func4+0x39>    ; return res
      4010c8:	b8 00 00 00 00       	mov    $0x0,%eax              ; res = 0
      4010cd:	39 f9                	cmp    %edi,%ecx              
      4010cf:	7d 0c                	jge    4010dd <func4+0x39>    ; if(ecx>=a) return res;
      4010d1:	8d 71 01             	lea    0x1(%rcx),%esi         ; b = ecx + 1
      4010d4:	e8 cb ff ff ff       	callq  4010a4 <func4>         ; res = func4()
      4010d9:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax  ; res = 2*res+1
      4010dd:	48 83 c4 08          	add    $0x8,%rsp              ; return res
      4010e1:	c3                   	retq
    ```
    
    写成C语言大概是：
    
    ```c
    //a in %edi, b in %esi, c in %edx
    int func4(int a, int b, int c){     //  m,0,14    m,0,6     m,4,6
      int res = c - b;
      res = (res + res>>31) >> 1;
      int ecx = res + b;                //  7         3         5
    
      if(ecx < a){                      //            m>3
        res = func4(a, ecx + 1, c);     //            
        return res*2 + 1;
      }else if(ecx > a){                //  m<14 
        res = func4(a, b, ecx-1);       //
        return res*2;
      }else{                            //                      m=5
        return 0;
      }
    }
    ```
    

---

## phase_5 指针

> 已知一个数组，根据题目要求来安排遍历的起点，规定了遍历的规则、次数和终点。

根据第`40117f`行中的地址，可以通过gdb查看该数组中的元素：

![截屏2023-12-08 22.32.05.png](CSAPP%EF%BD%9CBomb%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-08_22.32.05.png)

可知，我们需要选定遍历数组的起点，使得遍历的次数刚好为6，且在元素`0xf`处跳出循环。（以当前元素的值作为下一次遍历的下标）

可知遍历路线应当为： `0xa`→`0x1`→`0x2`→`0xe`→`0x6`→`0xf`

输入的参数为两个整数，一个指示遍历的起点下标，一个记录遍历元素的累积和。

因此答案为`0 48`。

- `phase_5`
    
    ```asm
    0000000000401139 <phase_5>:
      401139:	48 83 ec 18          	sub    $0x18,%rsp
      40113d:	48 8d 4c 24 08       	lea    0x8(%rsp),%rcx           ; 参数二
      401142:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx           ; 参数一
      401147:	be e1 28 40 00       	mov    $0x4028e1,%esi
      40114c:	b8 00 00 00 00       	mov    $0x0,%eax
      401151:	e8 da fa ff ff       	callq  400c30 <__isoc99_sscanf@plt>
      401156:	83 f8 01             	cmp    $0x1,%eax
      401159:	7f 05                	jg     401160 <phase_5+0x27>
      40115b:	e8 29 05 00 00       	callq  401689 <explode_bomb>
      401160:	8b 44 24 0c          	mov    0xc(%rsp),%eax
      401164:	83 e0 0f             	and    $0xf,%eax
      401167:	89 44 24 0c          	mov    %eax,0xc(%rsp)           ; 参数一仅保留低4位
      40116b:	83 f8 0f             	cmp    $0xf,%eax
      40116e:	74 2c                	je     40119c <phase_5+0x63>    ; 若参数一低4位全1，则爆炸
      401170:	b9 00 00 00 00       	mov    $0x0,%ecx
      401175:	ba 00 00 00 00       	mov    $0x0,%edx                
      40117a:	83 c2 01             	add    $0x1,%edx                ; ecx=0,edx=1,开始循环
      40117d:	48 98                	cltq                            ; 符号拓展rax
      40117f:	8b 04 85 80 26 40 00 	mov    0x402680(,%rax,4),%eax   ; 取数组下标为rax的数
      401186:	01 c1                	add    %eax,%ecx                ; 累加遍历到的元素
      401188:	83 f8 0f             	cmp    $0xf,%eax                ; 若当前数=0xf则跳出循环
      40118b:	75 ed                	jne    40117a <phase_5+0x41>
    
      40118d:	89 44 24 0c          	mov    %eax,0xc(%rsp)
      401191:	83 fa 06             	cmp    $0x6,%edx                ; 循环次数不为6则爆炸
      401194:	75 06                	jne    40119c <phase_5+0x63>
      401196:	3b 4c 24 08          	cmp    0x8(%rsp),%ecx           ; 元素累加和是否等于参数二，否则爆炸
      40119a:	74 05                	je     4011a1 <phase_5+0x68>
      40119c:	e8 e8 04 00 00       	callq  401689 <explode_bomb>
      4011a1:	48 83 c4 18          	add    $0x18,%rsp
      4011a5:	c3                   	retq
    ```
    

---

## phase_6 链表/指针/结构

> 已知一个节点数为6的链表，需要将其重新排序，使得其满足“从大到小”的指向顺序。
> 具体而言，代码会将链表中的节点存入栈中，再将低一级的节点指向高一级节点，实现重新排序，而存入栈中的顺序由我们的输入指定。

根据`0x401238`行，可知链表的首节点地址为`0x6042f0`，且指针前面的部分占8字节。

查看链表内容，结果如下图。

![截屏2023-12-13 13.02.32.png](CSAPP%EF%BD%9CBomb%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-13_13.02.32.png)

可以发现数据部分由**序号**和**整数**组成，而且这个链表刚好连续存储，于是结构体的声明如下：

```c
typedef struct{
	int data;   // 数据
	int index;  // 序号
	node* next;	// 下一个节点
} node;
```

而节点被存入栈中（准备进行重新排序时），结构大概为：

![IMG_6D54131CA3A0-1.jpeg](CSAPP%EF%BD%9CBomb%20Lab/IMG_6D54131CA3A0-1.jpeg)

于是答案为`4 5 3 1 2 6`。

- `phase_6`
    
    ```asm
    00000000004011a6 <phase_6>:   ; 4 5 3 1 2 6 
      4011a6:	41 56                	push   %r14
      4011a8:	41 55                	push   %r13
      4011aa:	41 54                	push   %r12
      4011ac:	55                   	push   %rbp
      4011ad:	53                   	push   %rbx
      4011ae:	48 83 ec 50          	sub    $0x50,%rsp               
      4011b2:	4c 8d 6c 24 30       	lea    0x30(%rsp),%r13          
      4011b7:	4c 89 ee             	mov    %r13,%rsi                ; 指定整数存储的起始地址0x30(%rsp)，读取6个整数(m0-m5)
      4011ba:	e8 00 05 00 00       	callq  4016bf <read_six_numbers>; 0x30(%rsp) -> 0x44(%rsp)
      4011bf:	4d 89 ee             	mov    %r13,%r14                ; r14 = &(m0)
      4011c2:	41 bc 00 00 00 00    	mov    $0x0,%r12d
    
      ; 六个整数都不大于6，且两两不等
      ; 外循环 r13初值指向整数一
      4011c8:	4c 89 ed             	mov    %r13,%rbp                ; rbp = &(m)
      4011cb:	41 8b 45 00          	mov    0x0(%r13),%eax           ; eax = m
      4011cf:	83 e8 01             	sub    $0x1,%eax
      4011d2:	83 f8 05             	cmp    $0x5,%eax                
      4011d5:	76 05                	jbe    4011dc <phase_6+0x36>    ; m是否小于等于6，不等则爆炸
      4011d7:	e8 ad 04 00 00       	callq  401689 <explode_bomb>
      ; 内循环
      4011dc:	41 83 c4 01          	add    $0x1,%r12d               ; 迭代器，遍历完剩余整数跳出
      4011e0:	41 83 fc 06          	cmp    $0x6,%r12d
      4011e4:	74 22                	je     401208 <phase_6+0x62>
      4011e6:	44 89 e3             	mov    %r12d,%ebx               
      4011e9:	48 63 c3             	movslq %ebx,%rax                ; 符号拓展，rax的值为迭代器
      4011ec:	8b 44 84 30          	mov    0x30(%rsp,%rax,4),%eax   ; eax = m_rax
      4011f0:	39 45 00             	cmp    %eax,0x0(%rbp)           
      4011f3:	75 05                	jne    4011fa <phase_6+0x54>    ; 比较其与m是否相等，相等则爆炸
      4011f5:	e8 8f 04 00 00       	callq  401689 <explode_bomb>
      4011fa:	83 c3 01             	add    $0x1,%ebx
      4011fd:	83 fb 05             	cmp    $0x5,%ebx
      401200:	7e e7                	jle    4011e9 <phase_6+0x43>    ; 迭代器++，若未超过5则继续内循环
      401202:	49 83 c5 04          	add    $0x4,%r13                ; r13指向下一个整数，继续外循环
      401206:	eb c0                	jmp    4011c8 <phase_6+0x22>
    
      ; 对6个数执行运算：m = 7 - m
      401208:	48 8d 74 24 48       	lea    0x48(%rsp),%rsi          ; rsi是第6个整数的末尾，即剩余空间的开头
      40120d:	4c 89 f0             	mov    %r14,%rax                ; rax = &(m0)
      401210:	b9 07 00 00 00       	mov    $0x7,%ecx
      401215:	89 ca                	mov    %ecx,%edx
      401217:	2b 10                	sub    (%rax),%edx
      401219:	89 10                	mov    %edx,(%rax)              ; m = 7 - m
      40121b:	48 83 c0 04          	add    $0x4,%rax                ; rax++
      40121f:	48 39 f0             	cmp    %rsi,%rax
      401222:	75 f1                	jne    401215 <phase_6+0x6f>    
      401224:	be 00 00 00 00       	mov    $0x0,%esi                ; esi = 0
      401229:	eb 20                	jmp    40124b <phase_6+0xa5>
    
      ; 循环 对6个整数m从m0到m5，将链表的第m个节点的地址“从低到高”依次存入栈中（起点为m5的尾端，即6个整数前的栈空间）
      ; 若m<=1，则直接存入表头地址
      ; 链表：
    	; 0x6042f0为链表首节点的地址，下一个节点的值 = [上一个节点的值+8]（即单个节点的头部占8字节，尾部为next指针）
      ; (0x100000103, 0x604300) -> (0x200000262, 0x604310) -> (0x3000003b6, 0x604320) -> (0x4000001c2, 0x604330) -> (0x500000199, 0x604340) -> (0x6000001a5, 0x0) 
      ; 每一个节点包含三个部分——(int)data, (int)index, (node*)next
      40122b:	48 8b 52 08          	mov    0x8(%rdx),%rdx           ; 下一个节点的next指针（指向下下个节点），结果是rdx为链表第ecx个节点的地址（即下标为ecx-1的节点的地址）
      40122f:	83 c0 01             	add    $0x1,%eax
      401232:	39 c8                	cmp    %ecx,%eax                
      401234:	75 f5                	jne    40122b <phase_6+0x85>    ; 循环使得eax = ecx, 【rdx = 链表第ecx个元素的地址】
      401236:	eb 05                	jmp    40123d <phase_6+0x97>
    
      401238:	ba f0 42 60 00       	mov    $0x6042f0,%edx           
      40123d:	48 89 14 74          	mov    %rdx,(%rsp,%rsi,2)       ; rdx -> [rsp+2*rsi]
      401241:	48 83 c6 04          	add    $0x4,%rsi
      401245:	48 83 fe 18          	cmp    $0x18,%rsi               
      401249:	74 15                	je     401260 <phase_6+0xba>    ; 存储完6个链表值
    
      40124b:	8b 4c 34 30          	mov    0x30(%rsp,%rsi,1),%ecx   ; ecx = m_rsi
      40124f:	83 f9 01             	cmp    $0x1,%ecx
      401252:	7e e4                	jle    401238 <phase_6+0x92>    ; 若ecx <= 1
      401254:	b8 01 00 00 00       	mov    $0x1,%eax
      401259:	ba f0 42 60 00       	mov    $0x6042f0,%edx
      40125e:	eb cb                	jmp    40122b <phase_6+0x85>    ; 若ecx > 1，则循环使得eax=ecx
      
      ;循环 从栈顶（低到高）遍历链表节点，进行操作：低一级节点的next指针 = 高一级节点的地址
      401260:	48 8b 1c 24          	mov    (%rsp),%rbx
      401264:	48 8d 44 24 08       	lea    0x8(%rsp),%rax           
      401269:	48 8d 74 24 30       	lea    0x30(%rsp),%rsi
      40126e:	48 89 d9             	mov    %rbx,%rcx                ; rcx = [rsp]
    
      401271:	48 8b 10             	mov    (%rax),%rdx              ; rdx = [rax]
      401274:	48 89 51 08          	mov    %rdx,0x8(%rcx)           ; rdx -> [rcx+8]
      401278:	48 83 c0 08          	add    $0x8,%rax                ; rax+=8
      40127c:	48 39 f0             	cmp    %rsi,%rax          
      40127f:	74 05                	je     401286 <phase_6+0xe0>    ; rax从[rsp+8]到[rsp+48]结束，即6个整数之前的栈空间
      401281:	48 89 d1             	mov    %rdx,%rcx                ; rcx = rdx
      401284:	eb eb                	jmp    401271 <phase_6+0xcb>
      401286:	48 c7 42 08 00 00 00 	movq   $0x0,0x8(%rdx)           ; 0 -> 最高一级节点的next指针
      40128d:	00 
      ; 到这里，相当于利用栈结构对这个链表进行重新排序
    
      ; 循环 遍历栈中的节点，检查：低一级的节点的数据 > 高一级节点的数据 
      40128e:	bd 05 00 00 00       	mov    $0x5,%ebp
      401293:	48 8b 43 08          	mov    0x8(%rbx),%rax           ; rax = 最低节点的next指针
      401297:	8b 00                	mov    (%rax),%eax              ; eax = 最低节点的next节点的data
      401299:	39 03                	cmp    %eax,(%rbx)                   
      40129b:	7d 05                	jge    4012a2 <phase_6+0xfc>    ; 若最低节点的next节点的data > 最低节点的data 则爆炸
      40129d:	e8 e7 03 00 00       	callq  401689 <explode_bomb>
      4012a2:	48 8b 5b 08          	mov    0x8(%rbx),%rbx           ; rbx指向高一级节点
      4012a6:	83 ed 01             	sub    $0x1,%ebp
      4012a9:	75 e8                	jne    401293 <phase_6+0xed>
      4012ab:	48 83 c4 50          	add    $0x50,%rsp
      4012af:	5b                   	pop    %rbx
      4012b0:	5d                   	pop    %rbp
      4012b1:	41 5c                	pop    %r12
      4012b3:	41 5d                	pop    %r13
      4012b5:	41 5e                	pop    %r14
      4012b7:	c3                   	retq
    ```
    

---

## secret_phase

### （1）函数phase_defused

跟踪`secret_phase`的调用者，发现是函数`phase_defused`，而这个函数在`main`里的每一个阶段后都会调用，用以检查炸弹是否拆除。其代码如下：

- `phase_defused`
    
    ```asm
    0000000000401827 <phase_defused>:
      401827:	48 83 ec 68          	sub    $0x68,%rsp
      40182b:	bf 01 00 00 00       	mov    $0x1,%edi
      401830:	e8 35 fd ff ff       	callq  40156a <send_msg>
      401835:	83 3d 60 2f 20 00 06 	cmpl   $0x6,0x202f60(%rip)        ; 60479c <num_input_strings> 
      40183c:	75 6d                	jne    4018ab <phase_defused+0x84>  ; 需要程序已经读取了6个字符串，因此之后在执行完6个phase才会执行剩下的这部分代码
      40183e:	4c 8d 44 24 10       	lea    0x10(%rsp),%r8
      401843:	48 8d 4c 24 08       	lea    0x8(%rsp),%rcx
      401848:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx
      40184d:	be 2b 29 40 00       	mov    $0x40292b,%esi               ; 读取格式为"%d %d %s"
      401852:	bf b0 48 60 00       	mov    $0x6048b0,%edi               ; "5 2"
      401857:	b8 00 00 00 00       	mov    $0x0,%eax
      40185c:	e8 cf f3 ff ff       	callq  400c30 <__isoc99_sscanf@plt>
      401861:	83 f8 03             	cmp    $0x3,%eax
      401864:	75 31                	jne    401897 <phase_defused+0x70>  ; 需要eax=3，即读取到3个参数
      401866:	be 34 29 40 00       	mov    $0x402934,%esi
      40186b:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
      401870:	e8 69 fb ff ff       	callq  4013de <strings_not_equal> 
      401875:	85 c0                	test   %eax,%eax
      401877:	75 1e                	jne    401897 <phase_defused+0x70>  ; 还需要与目标字符串一致"DrEvil"
      401879:	bf 80 27 40 00       	mov    $0x402780,%edi             
      40187e:	e8 bd f2 ff ff       	callq  400b40 <puts@plt>            ; 打印"Curses, you've found the secret phase!"
      401883:	bf a8 27 40 00       	mov    $0x4027a8,%edi
      401888:	e8 b3 f2 ff ff       	callq  400b40 <puts@plt>            ; 打印"But finding it and solving it are quite different..."
      40188d:	b8 00 00 00 00       	mov    $0x0,%eax
      401892:	e8 5f fa ff ff       	callq  4012f6 <secret_phase>
      401897:	bf e0 27 40 00       	mov    $0x4027e0,%edi               ; 打印"Congratulations! You've defused the bomb!"
      40189c:	e8 9f f2 ff ff       	callq  400b40 <puts@plt>
      4018a1:	bf 10 28 40 00       	mov    $0x402810,%edi
      4018a6:	e8 95 f2 ff ff       	callq  400b40 <puts@plt>            ; 打印"Your instructor has been notified and will verify your solution."
      4018ab:	48 83 c4 68          	add    $0x68,%rsp
      4018af:	c3                   	retq
    ```
    

根据第`0x401835`行可知，函数剩余的部分只有在程序读取完6个字符串之后才能执行，因此secret只有在`phase_6`之后才能出现。

可以发现，函数中有很多奇怪的常数地址，通过gdb打印结果如下：

![截屏2023-12-09 17.01.20.png](CSAPP%EF%BD%9CBomb%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-09_17.01.20.png)

结合先前`phase_4`的输入是`5 2`，且此处地址提示`<inpupt_strings+240>`，猜想是需要在`phase_4`的答案后追加字符串`”DrEvil”`，尝试发现确实如此——

![截屏2023-12-09 17.03.39.png](CSAPP%EF%BD%9CBomb%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-09_17.03.39.png)

### （2）函数secret_phase

该函数会接收终端的标准输入（如先前图中所示的绿标），之后将其转化为long类型处理。

而程序的规则有二：

1. 输入不得超过1001；
2. 调用函数`fun7`的返回值须为7。

于是重点就落在了对函数`fun7`的分析。

但在此之前，可以看到调用`fun7`时传入的参数除了我们的输入外还有一个地址`0x604110`，通过gdb打印查看——

可以发现这是一个链表（节点的后半段显然是地址，而且还有两个），而`phase_6`中的链表竟然就在下面，作者巧思啊hhhh

![截屏2023-12-09 18.59.34.png](CSAPP%EF%BD%9CBomb%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-09_18.59.34.png)

也因此可以猜测这是一个循环双链表or二叉树，又发现后面的节点几乎没有指针值，于是笃定是**二叉树**。

节点的声明大致为：

```c
typedef struct{
	long data;  
	Node* first;
	Node* second;
} Node;
```

> 根据打印结果可知，结构体的内存分配为*data的8字节 + n1的8字节 + n2的8字节*。
一般我习惯认为data是4字节的int类型，而这里占8字节为了**数据对齐**；但由于另一个需要我输入的参数被转化为了long类型，这里大概率要保持一致。

- `secrets_phase`
    
    ```asm
    00000000004012f6 <secret_phase>:
      4012f6:	53                   	push   %rbx
      4012f7:	e8 05 04 00 00       	callq  401701 <read_line>         ; 接收终端输入
      4012fc:	ba 0a 00 00 00       	mov    $0xa,%edx
      401301:	be 00 00 00 00       	mov    $0x0,%esi
      401306:	48 89 c7             	mov    %rax,%rdi                  ; rdi = rax = (string)"m"
      401309:	e8 f2 f8 ff ff       	callq  400c00 <strtol@plt>        ; 输入字符m转化为长整型 (long)m
      40130e:	48 89 c3             	mov    %rax,%rbx                  ; rbx = m            
      401311:	8d 40 ff             	lea    -0x1(%rax),%eax
      401314:	3d e8 03 00 00       	cmp    $0x3e8,%eax
      401319:	76 05                	jbe    401320 <secret_phase+0x2a> ; 需要m-1 <= 0x3e8 ，即 m <= 1001
      40131b:	e8 69 03 00 00       	callq  401689 <explode_bomb>
      401320:	89 de                	mov    %ebx,%esi                  ; esi = m
      401322:	bf 10 41 60 00       	mov    $0x604110,%edi
      401327:	e8 8c ff ff ff       	callq  4012b8 <fun7>
      40132c:	83 f8 07             	cmp    $0x7,%eax
      40132f:	74 05                	je     401336 <secret_phase+0x40> ; 需要eax = 7
      401331:	e8 53 03 00 00       	callq  401689 <explode_bomb>
    
      401336:	bf 08 26 40 00       	mov    $0x402608,%edi             ; 打印"Wow! You've defused the secret stage!"
      40133b:	e8 00 f8 ff ff       	callq  400b40 <puts@plt>
      401340:	e8 e2 04 00 00       	callq  401827 <phase_defused>
      401345:	5b                   	pop    %rbx
      401346:	c3                   	retq   
      401347:	66 0f 1f 84 00 00 00 	nopw   0x0(%rax,%rax,1)
      40134e:	00 00
    ```
    

### （3）函数fun7

转化为C语言如下：

```c
// n in %edi, m in %esi
fun7(Node* n, long m){ // 即让n->second->second->second->data = m
  if(n == 0)
    return -1;
  long res, edx = *n;

  if(edx < m){
    n = n->second;
    res = fun7(n, m);  
    return 2*res + 1; // 3    1    0
  }else if(edx > m){
    n = n->first;
    res = fun7(n, m);
    return 2*res;
  }else{
    return 0;                  
  }
}
```

~~由于输入的值不可超过1001（0x3e8），因此须保证最后落脚的节点不可是`<n48>`；~~

~~而如果直接$7=1+2*(1+2*(1+0))$，会直接落脚在`<n48>`，因此需要在递归最深处再绕一下，走`edx>m`的分支，~~

我是笨蛋，要求只是不超过、而不是不能取等，我说怎么怎么都不可能🥹

在捋路径的时候图方便把这个节点关系画了出来，发现竟然还是个**二叉搜索树**！（first左second右）

于是答案为`1001`

既然如此，节点结构体声明就应该是——

```c
typedef struct{
	long data;  
	TNode* left;  // first
	TNode* right; // second
} TNode;
```

![Untitled](CSAPP%EF%BD%9CBomb%20Lab/Untitled.png)
