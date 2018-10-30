## Itanium C++ ABI: Exception Handling

### 异常抛出后的过程

1. 异常抛出后如果在当前函数内没有被捕捉 (catch)，它就要沿着函数的调用链继续往上抛，直到走完整个调用链，或者在某个函数中找到相应的 catch。如果走完调用链都没有找到相应的 catch，那么 std::terminate() 就会被调用，这个函数默认是把程序 abort，而如果最后找到了相应的 catch，就会进入该 catch 代码块，执行相应的操作

2. 程序中的 catch 那部分代码有一个专门的名字叫作：Landing pad，从抛异常开始到执行 landing pad 里的代码这中间的整个过程叫作 stack unwind

### Base ABI: Itanium 基本的 API，其他语言的异常处理实现也是基于这些 API

```
_Unwind_RaiseException, // 抛出异常
_Unwind_Resume, // 把未处理的异常参数传到上一层
_Unwind_DeleteException,
_Unwind_GetGR, // 获取 general register 寄存器中的值
_Unwind_SetGR, // 设置 general register 寄存器中的值
_Unwind_GetIP, // 获取 instruction pointer 寄存器中的值
_Unwind_SetIP, // 设置 instruction pointer 寄存器中的值
_Unwind_GetRegionStart,
_Unwind_GetLanguageSpecificData,
_Unwind_ForcedUnwind // 强制 unwind，即不允许在当前函数进行任何 catch 操作，直接传到上一层
```

### stack unwind 过程

1. _Unwind_RaiseException()

    被 throw() 调用，主要功能是从当前函数开始，对调用链上每个函数幀都调用一个叫作 personality routine 的函数 (__gxx_personality_v0)

2. two-phase process

    1. search-phase: 调用 personality routine 查找当前函数有无 landing pad 中对应的 handler(catch)，如果没有，通过栈展开 (unwind) 寻找上一个调用者，直到最上层没有找到就调用 std::terminate()，找到就跳到 cleanup-phase

    2. cleanup-phase: 调用 personality routine 从抛出异常的函数开始清理栈中的局部变量信息，一直栈展开到有对应 catch 的块，使用 _Unwind_SetIP 把当前栈帧地址传给 landing pad，使用 _Unwind_SetGR 把参数传给 landing pad，之后把控制权交给 landing pad，执行 catch

### C++ ABI

```
void *__cxa_allocate_exception(size_t thrown_size), // 创建一个异常
void __cxa_free_exception(void *thrown_exception), // 释放一个异常
void __cxa_throw (void *thrown_exception, std::type_info *tinfo, void (*dest) (void *) ), // 抛出一个异常
```

- 全局异常存储

```
 struct __cxa_eh_globals {
	__cxa_exception *	caughtExceptions; // 被 catch 到的异常
	unsigned int		uncaughtExceptions; // 还未被 catch 到的异常
};
```

- 过程

1. __cxa_allocate_exception + __cxa_throw:
    - 创建一个 exceptionObject 对象
    - uncaughtExceptions 数量加 1

2. __cxa_begin_catch(void *exceptionObject): 
    - exceptionObject 的 handler 数量加 1
    - 将异常放到 caughtExceptions 中
    - uncaughtExceptions 数量减 1
    - 返回指向 exceptionObject 的指针

3. __cxa_end_catch():
    - 当前 exceptionObject 的 handler 数量减 1
    - 如果 handler 数量为 0，从 caughtExceptions 中移除当前 exceptionObject
    - 如果 handler 数量为 0，释放 exceptionObject 对象

参考自：[官方文档](https://itanium-cxx-abi.github.io/cxx-abi/abi-eh.html)