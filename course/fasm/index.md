---
title: 汇编程序设计基础
section: 汇编程序设计基础
props:
  绩点: 5.0 (99)
  学分: "2.0"
  任课教师: 白洪欢
  修读学期: 大一秋冬
nav:
  - 课程简介: index.md
  - 笔记:
      - note/1.md
      - note/2.md
      - note/3.md
      - note/4.md
      - note/5.md
sync: /course/fasm/index.md
---

## 1. 修读感受

&emsp;&emsp;这门课的内容以 8086 汇编（16位）为主，会涉及到一定 80386 汇编（32位）的知识，但不是考察重点。

&emsp;&emsp;小白老师非常注重对汇编指令的理解与应用，而不是简单的记忆与背诵。对于每一条指令的用法及细节，小白老师都会举出若干个例子并用写程序 + 调试的方式验证说明。所以对于有一定汇编基础或者只想粗略了解汇编的同学，完整地听小白老师的课可能就有点枯燥了。博主因为前半学期准备各类比赛比较忙，调试器相关的部分没搞明白，到后面上课就有一半听的云里雾里，还好最后通过智云课堂还是补回来了。

&emsp;&emsp;这门课的给分较为良好。作业分占比 50%，除了最后的一个贪吃蛇 Lab 外的作业都放在 PTA 上，支持多次提交，只要不抄袭作业分都能刷满（没有“互评”这种遥遥领先的评分形式）。期末考虽然按小白老师的原话讲有“为班里学过汇编语言专业课的高手准备的高手题”，但以我的实际体验来说掌握了上课讲的基础的东西就已经够用了。不过这门课对于编程方面基础较差的同学来说可能就比较吃力了，期末考签到的时候发现点名册上过半的都是退课或弃修的同学。

## 2. 复习经验

&emsp;&emsp;博主因为平时上课没有好好听讲，因此大约提前一个月就开始复习这门课。发现小白老师的配套教材写的挺好，就跟着教材学习，遇到看不懂的地方就在智云课堂上找对应回放。个人感觉这也是这门课最为高效的应试手段了，如果要真正掌握汇编语言，还是需要多写代码、多使用调试器。

&emsp;&emsp;以下是小白老师在最后一节课上提到的考察范围，原件挂在小白老师的主页上。这里也整理了一份，大家可以在学习时有所侧重：

- mov, xchg, push, pop, pushf, popf, lea, lds, les
- cbw, cwd, add, adc, sub, sbb, inc, dec
- mul, div, imul, idiv, xlat, in, out, 
- and, or, xor, not, neg, shl, shr, sal, sar, rol, ror, rcl, rcr
- test, cmp
- jxx（条件跳转指令）: ja, jb , jae, jbe, jc jnc jz jnz js jns jo jno 
- loop
- clc stc cli sti cld std
- call, ret（近调用和近返回）
- call far ptr, retf（远调用和远返回）
- int, iret
- jmp short, jmp near ptr, jmp far ptr, 
- jmp dword ptr
- 字符串指令：repne/repe scasb, repe/repne cmpsb, rep movsb, lodsb, stosb, rep stosb
- 用堆栈传递参数时, 如何用 `[bp+?]` 实现对参数的引用
- 调试器的使用（Tubro Debug、Soft-ICE）
