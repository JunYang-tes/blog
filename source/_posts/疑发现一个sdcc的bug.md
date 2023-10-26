---
layout: post
title: 疑发现一个sdcc的bug
date: 2023-08-27
tags:
---

sdcc 在编译下面代码时候报警告：
```c
#include <8052.h>
#define addr0 P1_0
#define addr1 P1_1
#define addr2 P1_2
#define select_38_0 (addr0=0,addr1=0,addr2=0)
#define select_38_1 (addr0=1,addr1=0,addr2=0)
#define select_38_2 (addr0=0,addr1=1,addr2=0)
#define select_38_3 (addr0=1,addr1=1,addr2=0)
#define select_38_4 (addr0=0,addr1=0,addr2=1)
#define select_38_5 (addr0=1,addr1=0,addr2=1)
#define select_38_6 (addr0=0,addr1=1,addr2=1)
#define select_38_7 (addr0=1,addr1=1,addr2=1)
#define select_38(n) \
  switch (n) { \
    case 0: \
      select_38_0; \
      break; \
    case 1: \
      select_38_1; \
      break; \
    case 2: \
      select_38_2; \
      break; \
    case 3: \
      select_38_3; \
      break; \
    case 4: \
      select_38_4; \
      break; \
    case 5: \
      select_38_5; \
      break; \
    case 6: \
      select_38_6; \
      break; \
    case 7: \
      select_38_7; \
      break; \
  }
#define LED P0_0
void delay(void) {
  int i, j;
  for (i = 0; i < 0xff; i++) {
    for (j = 0; j < 0xff; j++)
      ;
  }
}

void main(void) {
  LED = 0;
  char i = 0;
  while (1) {
    delay();
    select_38(i);
    i = (i + 1) % 8;
  }
}
```
出现这样的警告：
```shell
a.c:54: warning 110: conditional flow changed by optimizer: so said EVELYN the modified DOG
a.c:54: warning 126: unreachable code
```
这段代码是一个流水灯的实验，上面编译出来的文件的执行效果是第一个灯常亮。从编译器给出的警告来看，54行发生了某种优化，导致出现了不可达的代码。因此i没有变换，所以没有流水灯的效果。于是我尝试交换了54和55行的代码，结果正常通过了编译。并且执行效果达到预期。仔细思考一下，可能是这样的：编译器错误的以为i是不会变的，因此将代码优化为类似这样的：
```c
void main(void) {
  LED = 0;
  char i = 0;
  while (1) {
    delay();
    // select_38 宏被展开为:
    select_38_0; // 这个宏导致始终是第一个灯亮
    break; // 这个break 导致了后面的代码不可达
    i = (i + 1) % 8;
  }
}

```

**初步验证**
我有一个不太严谨的方法验证一下这个猜想，即我们可以用volatile关键字来修饰下i，清楚的告诉编译器i是会变的。这样如果原程序通过编译，那我的猜测就有可能是正确的。
```c
  LED = 0;
  volatile char i = 0;
  while (1) {
    delay();
    select_38(i);
    i = (i + 1) % 8;
  }

```
这样更改后果然就通过编译了。

我认为这里至少有两个问题：
1. 错误的认为i是不会变的
2. 错误的展开了select_38宏: 就所i真的是不变的，展开后的宏不应该包含那个break语句。那个break语句本意是break对应的case,留下后错误的break了这里的while循环.



