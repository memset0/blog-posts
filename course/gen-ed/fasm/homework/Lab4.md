---
title: "Lab4: 写显卡内存输出ASCII字符及其16进制ASCII码"
---

## 题目描述

以下程序的功能是从键盘输入一个十六进制数，该十六进制数一共2位，其中十位保存到buf[0]中，个位保存到buf[1]中，无论十位还是个位只要输入的是字母则一定是大写形式。

接下去按以下步骤从屏幕第0行起输出16行内容，每行都输出字符c+i(i为屏幕行号,c为输入的16进制ASCII码对应的字符)及其16进制ASCII码:

1. 把buf[0]及buf[1]中的十六进制字符脱去引号
2. 计算(buf[0]<<4) ∣ buf[1]的值并保存到变量c中
3. i=0
4. 在(0,i)处显示字符c, 颜色为7Ch
5. 在(1,i)处显示字符c的2位十六进制ASCII码, 颜色为1Ah
6. c++
7. i++
8. if(i<16)goto 4.
9. 结束程序运行

## 参考程序

http://cc.zju.edu.cn/bhh/asm/ascii.c 用C语言写的解答

http://cc.zju.edu.cn/bhh/asm/calldemo.asm 演示如何定义并调用函数

## 笔记

### 定义函数

由于pta限制，必须在`main:`之后定义函数。可以给真正的主程序部分加上`main_body:`，然后在开始填代码的部分直接`jmp main_body`。

### 写显卡

参考第三章相关内容。

```assembly
; 初始化写显卡
mov ax, 0B800h   ; 文本模式显卡映射到的内存段
mov es, ax
mov di, 0
; 写字符
mov al, 'a'      ; 要被写的字符
mov ah, 007Ch    ; 低2位控制前景色，高2位控制背景色
mov es:[di], ax 
add di, 2        ; 一个字符占用两个内存单元
```

### 循环变量

这里用的是CX寄存器

## 代码

```assembly
;本题要求:
;以下程序的功能是从键盘输入一个十六进制数，
;该十六进制数一共2位，其中十位保存到buf[0]
;中，个位保存到buf[1]中，无论十位还是个位
;只要输入的是字母则一定是大写形式。接下去
;按以下步骤从屏幕第0行起输出16行内容，每行
;都输出字符c+i(i为屏幕行号)及其16进制ASCII码:
;(1)把buf[0]及buf[1]中的十六进制字符脱去引号
;(2)计算buf[0]<<4 | buf[1]的值并保存到变量c中
;(3)i=0
;(4)在(0,i)处显示字符c, 颜色为7Ch
;(5)在(1,i)处显示字符c的2位十六进制ASCII码, 颜色为1Ah
;(6)c++
;(7)i++
;(8)if(i<16) goto (4)
;(9)结束程序运行

.386
data segment use16
buf db 0, 0
c   db 0
hex db 0, 0
data ends

code segment use16
assume cs:code, ds:data
main:
   mov ax, data
   mov ds, ax
   mov ax, 0B800h
   mov es, ax
   mov di, 0
   ;
   mov ah, 1
   int 21h
   mov buf[0], al
   mov ah, 1
   int 21h
   mov buf[1], al
;请在#1_begin和#1_end之间补充代码实现以下功能:
;(1)把buf[0]及buf[1]中的十六进制字符脱去引号
;(2)计算buf[0]<<4 | buf[1]的值并保存到变量c中
;(3)i=0
;(4)在(0,i)处显示字符c, 颜色为7Ch
;(5)在(1,i)处显示字符c的2位十六进制ASCII码, 颜色为1Ah
;(6)c++
;(7)i++
;(8)if(i<16) goto (4)
;(9)结束程序运行
;#1_begin-------------------------------------
   jmp main_body

remove_quotation:
   cmp al, '9'
   jg remove_quotation_alpha
remove_quotation_number:
   sub al, '0'
   jmp remove_quotation_end
remove_quotation_alpha:
   sub al, 'A'
   add al, 10
   jmp remove_quotation_end
remove_quotation_end:
   ret

add_quotation:
   cmp al, 9
   jg add_quotation_alpha
add_quotation_number:
   add al, '0'
   jmp add_quotation_end
add_quotation_alpha:
   add al, 'A'
   sub al, 10
   jmp add_quotation_end
add_quotation_end:
   ret

main_body:
   ; 将buf[0]和buf[1]的值脱去引号
   mov ah, 0
   mov al, buf[0]
   call remove_quotation
   mov buf[0], al
   mov al, buf[1]
   call remove_quotation
   mov buf[1], al
   ; 将(buf[0]<<4)|buf[1]的值保存到c中
   mov al, buf[0]
   shl al, 4
   mov bl, buf[1]
   or al, bl
   mov c, al
   ; 初始化写显卡
   mov ax, 0B800h
   mov es, ax
   mov di, 0
   ; i=0
   mov cx, 0
main_loop:
   ; 写字符
   mov al, c
   mov ah, 007Ch
   mov es:[di], ax
   add di, 2
   ; 写c的高位
   mov al, c
   shr al, 4
   call add_quotation
   mov ah, 01ah
   mov es:[di], ax
   add di, 2
   ; 写c的低位
   mov al, c
   shl al, 4
   shr al, 4
   call add_quotation
   mov ah, 01ah
   mov es:[di], ax
   add di, 2
   ; 跳到下一行
   add di, 154 ; 160-3*2=154
   add c, 1
   add cx, 1
   cmp cx, 16
   jne main_loop
main_exit:
   nop
;#1_end=======================================
   mov ah, 4Ch
   int 21h
code ends
end main
```

