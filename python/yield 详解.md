## generator

调用带yield 关键字的函数时
PyEval_EvalCodeEx 直接返回一个 generator, 当中包含了当前帧的信息

```c++
if (co->co_flags & CO_GENERATOR) {
    /* Don't need to keep the reference to f_back, it will be set
     * when the generator is resumed. */
    Py_CLEAR(f->f_back);

    PCALL(PCALL_GENERATOR);

    /* Create a new generator that owns the ready to run frame
     * and return that as the value. */
    return PyGen_New(f);
}

PyObject *
PyGen_New(PyFrameObject *f)
{
    PyGenObject *gen = PyObject_GC_New(PyGenObject, &PyGen_Type);
    if (gen == NULL) {
        Py_DECREF(f);
        return NULL;
    }
    gen->gi_frame = f;
    Py_INCREF(f->f_code);
    gen->gi_code = (PyObject *)(f->f_code);
    gen->gi_running = 0;
    gen->gi_weakreflist = NULL;
    _PyObject_GC_TRACK(gen);
    return (PyObject *)gen;
}
```

![](imgs/generator.png)

demo.py
```python
def a():
	yield 1
b = a() # generator
b.next() # 1
b.next() # StopIteration
```

next/send 方法
```c++
static PyObject *
gen_iternext(PyGenObject *gen)
{
    return gen_send_ex(gen, NULL, 0);
}

static PyObject *
gen_send_ex(PyGenObject *gen, PyObject *arg, int exc)
{
    PyThreadState *tstate = PyThreadState_GET();
    PyFrameObject *f = gen->gi_frame;
    PyObject *result;

    ..........

    f->f_back = tstate->frame;

    result = PyEval_EvalFrameEx(f, exc);

    Py_CLEAR(f->f_back);
    /* Clear the borrowed reference to the thread state */
    f->f_tstate = NULL;

    ..........

    /* If the generator just returned (as opposed to yielding), signal
     * that the generator is exhausted. */
    if (result == Py_None && f->f_stacktop == NULL) {
        Py_DECREF(result);
        result = NULL;
        /* Set exception if not called by gen_iternext() */
        if (arg)
            PyErr_SetNone(PyExc_StopIteration);
    }

    if (!result || f->f_stacktop == NULL) {
        /* generator can't be rerun, so release the frame */
        Py_DECREF(f);
        gen->gi_frame = NULL;
    }

    return result;
}
```

PyEval_EvalFrameEx 执行帧中代码，指令状态可以参照 f->f_lasti （上一条字节码在 f_code 的偏移）

## yield

```python
def a():
	yield 1

import dis
dis.dis(a)
```

字节码
```python
0 LOAD_CONST               1 (1)
3 YIELD_VALUE
4 POP_TOP
5 LOAD_CONST               0 (None)
8 RETURN_VALUE
```

YIELD_VALUE 直接返回栈顶的1了，然后帧停在了这个指令的位置
```c++
TARGET_NOARG(YIELD_VALUE)
{
    retval = POP();
    f->f_stacktop = stack_pointer;
    why = WHY_YIELD;
    goto fast_yield;
}
```

## yield from, python3.3

```python
def a():
	for i in range(3):
		yield 1

def b():
    yield from a()

import dis
dis.dis(b)
```

```python
0 LOAD_GLOBAL              0 (a)
3 CALL_FUNCTION            0 (0 positional, 0 keyword pair)
6 GET_YIELD_FROM_ITER
7 LOAD_CONST               0 (None)
10 YIELD_FROM
11 POP_TOP
12 LOAD_CONST               0 (None)
15 RETURN_VALUE
```

GET_YIELD_FROM_ITER 获取 a() 的返回值，就是一个 generator
```c++
case TARGET(GET_YIELD_FROM_ITER): {
    /* before: [obj]; after [getiter(obj)] */
    PyObject *iterable = TOP();
    PyObject *iter;
    if (PyCoro_CheckExact(iterable)) {
        /* `iterable` is a coroutine */
        if (!(co->co_flags & (CO_COROUTINE | CO_ITERABLE_COROUTINE))) {
            /* and it is used in a 'yield from' expression of a
               regular generator. */
            Py_DECREF(iterable);
            SET_TOP(NULL);
            _PyErr_SetString(tstate, PyExc_TypeError,
                             "cannot 'yield from' a coroutine object "
                             "in a non-coroutine generator");
            goto error;
        }
    }
    else if (!PyGen_CheckExact(iterable)) {
        /* `iterable` is not a generator. */
        iter = PyObject_GetIter(iterable);
        Py_DECREF(iterable);
        SET_TOP(iter);
        if (iter == NULL)
            goto error;
    }
    PREDICT(LOAD_CONST);
    DISPATCH();
}
```

\_PyGen_Send 就是跑那个 gen_send_ex 方法

```c++
case TARGET(YIELD_FROM): {
    PyObject *v = POP();
    PyObject *receiver = TOP();
    int err;
    if (PyGen_CheckExact(receiver) || PyCoro_CheckExact(receiver)) {
        retval = _PyGen_Send((PyGenObject *)receiver, v);
    } else {
        _Py_IDENTIFIER(send);
        if (v == Py_None)
            retval = Py_TYPE(receiver)->tp_iternext(receiver);
        else
            retval = _PyObject_CallMethodIdObjArgs(receiver, &PyId_send, v, NULL);
    }
    Py_DECREF(v);
    if (retval == NULL) {
        PyObject *val;
        if (tstate->c_tracefunc != NULL
                && _PyErr_ExceptionMatches(tstate, PyExc_StopIteration))
            call_exc_trace(tstate->c_tracefunc, tstate->c_traceobj, tstate, f);
        err = _PyGen_FetchStopIterationValue(&val);
        if (err < 0)
            goto error;
        Py_DECREF(receiver);
        SET_TOP(val);
        DISPATCH();
    }
    /* receiver remains on stack, retval is value to be yielded */
    f->f_stacktop = stack_pointer;
    /* and repeat... */
    assert(f->f_lasti >= (int)sizeof(_Py_CODEUNIT));
    f->f_lasti -= sizeof(_Py_CODEUNIT); # 将指令位置跳回上一个，就是说下一次还是会跑这个 YIELD_FROM, 直到出现 retval == NULL PyExc_StopIteration
    goto exit_yielding;
}
```

yield from 其实就是一个 yield 的代理，用于处理各种异常，本身的作用不大，但是大部分情况下yield from并不单独使用，而是伴随着asyncio库使用，实现异步操作

## async/await 协程, python3.5

```python
import asyncio
async def a():
    await asyncio.sleep(.1)
```

其实本质就是用 YIELD_FROM 轮询
```python
0 LOAD_GLOBAL              0 (asyncio)
3 LOAD_ATTR                1 (sleep)
6 LOAD_CONST               1 (0.1)
9 CALL_FUNCTION            1 (1 positional, 0 keyword pair)
12 GET_AWAITABLE
13 LOAD_CONST               0 (None)
16 YIELD_FROM
17 POP_TOP
18 LOAD_CONST               0 (None)
21 RETURN_VALUE
```

一个简单的例子，使用 asyncio 库的事件循环驱动
```python
import asyncio
async def a():
    await asyncio.sleep(1)
    return 1

coro = a()
print(type(coro)) # <class 'coroutine'>
loop = asyncio.get_event_loop()
result = loop.run_until_complete(coro)
loop.close()
```

```python
@coroutine
def sleep(delay, result=None, *, loop=None):
    """Coroutine that completes after a given time (in seconds)."""
    if delay == 0:
        yield
        return result

    if loop is None:
        loop = events.get_event_loop()
    future = loop.create_future()
    h = future._loop.call_later(delay,
                                futures._set_result_unless_cancelled,
                                future, result)
    try:
        return (yield from future)
    finally:
        h.cancel()
```

这里注意 3 个元素
1. coro: 协程对象，类似生成器，有send方法
2. future: future 对象，也类似生成器
3. loop: 一个事件循环对象，不同平台会使用不同的 eventloop, 这里只看了 BaseEventLoop, 里面是用最小堆获取最短时间事件， 有 call_soon 和 call_later

再看 loop.run_until_complete 函数

```python
def run_until_complete(self, future):
        future = tasks.ensure_future(future, loop=self)
        future.add_done_callback(_run_until_complete_cb) # 用于结束事件循环 run_forever
        try:
            self.run_forever()
        except:
            if new_task and future.done() and not future.cancelled():
                future.exception()
            raise
        future.remove_done_callback(_run_until_complete_cb)
        if not future.done():
            raise RuntimeError('Event loop stopped before Future completed.')

        return future.result()
```

tasks.ensure_future(future, loop=self) 将 coro 封装了一个 task, task 是 future 的子类, 然后往事件循环丢进一个 \_step 方法，看看主要几行代码

```python
def _step(self, exc=None):
    coro = self._coro
    try:
        if exc is None:
            result = coro.send(None)
        else:
            result = coro.throw(exc)
    except StopIteration as exc:
            self.set_result(exc.value)
    result.add_done_callback(self._wakeup)

def _wakeup(self, future):
        try:
            future.result()
        except Exception as exc:
            self._step(exc)
        else:
            self._step()
```

其实就是调用了 coro.send(None), 这里相当于运行 a() 函数直到卡在 return (yield from future)，由于 future 还没有 result 所以会触发 yield self, 就是下面代码做的事，很好理解

```python
def __iter__(self):
    if not self.done():
        self._asyncio_future_blocking = True
        yield self
    assert self.done()
    return self.result()  # May raise too.
```

add_done_callback(self.\_wakeup) 就是 future 添加了 callback

在 sleep 一段时间后事件循环拿到 call_later 的那个方法并执行，这里调用了 future.set_result 设置 done 标志并运行所有 callback，就是那个 \_wakeup, wakeup 方法会再调用一次 step 即 coro.send，这样就回到我们原来的 a() 方法了，这里 yield from 可以获取到 future 的 result 并返回给 task.result, 最后返回 task.result 整个协程就结束了

流程图以下:

![](imgs/sleepgraph.png)