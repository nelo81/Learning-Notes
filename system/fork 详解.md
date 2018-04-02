# fork 详解

作用: 创建子进程，并继续执行当前进程的代码。

特点: 返回两个值，在父进程中返回子进程的pid，在子进程返回0

通过例子说明更便于理解:

```
int main () {   
    pid_t fpid; //fpid表示fork函数返回的值
    int count=0;  
    fpid=fork();   
    if (fpid < 0)   
        printf("error in fork!");   
    else if (fpid == 0) {  
        printf("i am the child process, my process id is %d/n",getpid());   
        printf("我是爹的儿子/n");//对某些人来说中文看着更直白。  
        count++;  
    }  
    else {  
        printf("i am the parent process, my process id is %d/n",getpid());   
        printf("我是孩子他爹/n");  
        count++;  
    }  
    printf("统计结果是: %d/n",count);  
    return 0;  
}
```

输出:

```
i am the child process, my process id is 5574
我是爹的儿子
统计结果是: 1
i am the parent process, my process id is 5573
我是孩子他爹
统计结果是: 1
```

代码说明:

  - 调用fork()之前: 只有父进程在跑

  - 调用fork()之后: 两个进程都处于执行完 `int count=0;` 这条代码之后
    1. 父进程创建子进程并获得返回值fpid即子进程的pid，把代码和执行代码的状态栈，还有数据`count`等变量的值拷贝到子进程的空间中，然后继续执行下面的代码。
    2. 子进程从父进程获得代码，代码的执行状态和数据，并获得fpid=0，继续执行下面的代码。

  - 之后的代码可以通过判断fpid是否为0来定义父进程和子进程的行为
  
  - fpid=-1 代表创建进程失败

---

## 循环创建子线程

如以下代码:

```
int main(void) {  
   int i=0;
   int n=2;
   printf("i son/pa ppid pid  fpid/n");  

   //ppid指当前进程的父进程pid  
   //pid指当前进程的pid,  
   //fpid指fork返回给当前进程的值  

   for(i=0;i<n;i++){  
       pid_t fpid=fork();  
       if(fpid==0)  
           printf("%d child  %4d %4d %4d/n",i,getppid(),getpid(),fpid);  
       else  
           printf("%d parent %4d %4d %4d/n",i,getppid(),getpid(),fpid);  
   }  
   return 0;  
}  
```

输出:

```
i son/pa ppid pid  fpid
0 parent 2043 3224 3225
0 child  3224 3225    0
1 parent 2043 3224 3226
1 parent 3224 3225 3227
1 child     1 3227    0
1 child     1 3226    0 
```

代码说明:

  - 第一次调用fork()之后，父进程3224和他的子进程3225一起执行i=0的代码。

  - 当循环到i=1时，3224和3225进程同时再创建一个子进程，这时，4条进程(3224,3225,3226,3227)都执行i=1的代码。

  - n次循环后**创建**的总进程数为 1+2+4+...+2^n-1。

  - 输出的最后两行中，ppid并不是父进程pid而是1，因为在p3224和p3225执行完第二个循环后，main函数就该退出了，也即进程该死亡了，因为它已经做完所有事情了。p3224和p3225死亡后，p3226，p3227就没有父进程了，这在操作系统是不被允许的，所以p3226，p3227的父进程就被置为p1了，p1是永远不会死亡的。