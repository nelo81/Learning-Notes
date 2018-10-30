## setjmp longjmp 异常处理

在 C 语言中没有和 C++, Java 等语言的 try-catch 异常处理，但是可以采用 setjmp-longjmp 方法处理

- setjmp: 用于保存当前上下文，第一次调用是他的范围值是0，由 longjmp 跳转时返回值为 longjmp 的第二参数 (>0)

- longjmp: 跳转到并恢复 setjmp 时保存的上下文(包括栈帧，寄存器等)，给予 setjmp 新的返回值

- 示例:

```
#include "stdafx.h"
#include <setjmp.h>
#include <iostream>

#define pause system("pause")
static jmp_buf env; // 用于保存上下文的变量

void divide(int& a)
{
	a = 2;
	printf("2:%d\n", a);
	longjmp(env, 1); // 相当于抛出异常，跳回这个 env 上一次的 setjmp
}

void f()
{
	int a = 1;
	printf("1:%d\n", a);
	int re = setjmp(env); // 把当前上下文保存到 env，第一次调用时 re=0
	if (re == 1) { // 判断是否是 longjmp 跳转回来的，这里可以用 switch 匹配多个值，处理多个异常
		printf("3:%d\n", a);
		return;
	}
	divide(a);
}

int main()
{
	f();
	pause;
    return 0;
}
```

- 输出结果:

```
1:1
2:2
3:2
```