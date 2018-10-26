## windows process inject

这个技术主要时针对 windows 进程，可以在进程运行的时候注入自己的代码操作，一般有几种方法(下面代码有各种省略，包括类型，参数等，由C编写)

1. 远线程 dll 注入: 在目标进程建立一条线程读取自己的 dll

    ```
    // 获取进程句柄
    hProcess = OpenProcess(PROCESS_QUERY_INFORMATION |   // Required by Alpha
         PROCESS_CREATE_THREAD     |   // For CreateRemoteThread
         PROCESS_VM_OPERATION      |   // For VirtualAllocEx/VirtualFreeEx
         PROCESS_VM_WRITE,             // For WriteProcessMemory
         FALSE,
         process_id)                   // 目标进程 id
    
    // 在目标进程申请内存空间
    remote_memory = VtitualAllocEx(hProcess, NULL, len, MEM_COMMIT, PAGE_READWRITE) // len 是申请的大小

    // 将 dll 文件路径写入到目标进程新申请的内存
    WriteProcessMemory(hProcess, remote_memory, dll_file_name, cb, NULL)  // 因此上面的 len 应该是 dll_file_name 所占的字节数

    // 获取系统自带 dll 句柄
    hMod = GetModuleHandle("kernel32.dll")

    // 获取 LoadLibrary (系统读取 dll 文件的函数) 的地址
    load_proc = GetProcAddress(hMod, "LoadLibraryW")

    // 创建线程，调用 LoadLibrary，加载 dll 文件
    hThread = CreatRemoteThread(hProcess, NULL, 0, load_proc, remote_memory, 0, NULL)  // 参数用的是 remote_memory 即我们刚才写入那个新申请的内存中的 dll 文件路径

    // 阻塞直到线程完成
    WaitForSingleObject(hThread, INFINITE);

    // 释放内存和关闭句柄
    if (remote_memory != NULL) VirtualFreeEx(hProcess, remote_memory, 0, MEM_RELEASE);
    if (hThread  != NULL) CloseHandle(hThread);
    if (hProcess != NULL) CloseHandle(hProcess);
    ```

    ```
    /* 要注入的 dll 代码 */
    BOOL WINAPI DllMain(HINSTANCE hinstDll, DWORD fdwReason, PVOID fImpLoad) {
        switch(fdwReason)
        {
            case DLL_PROCESS_ATTACH: // 被进程读取时调用
            {
                // 为所欲为
            }
            case DLL_THREAD_ATTACH:
            {
                // 为所欲为
            }
            case DLL_THREAD_DETACH:
            {
                // 为所欲为
            }
            case DLL_PROCESS_DETACH:
            {
                // 为所欲为
            }
        }
        return TRUE;
    }
    ```

2. 修改 jmp: 直接把 jmp 指令替换目标进程对应函数地址，使其调用自己的代码

    ```
    // 省略前面的...
    old_addr = {0x48,0xB8,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x50,0xC3}; // 存储旧 api 的地址，前面两个数 0x48,0xB8 代表 jmp 机器码指令，不同平台下可能会不一样
    new_addr = {0x48,0xB8,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x50,0xC3}; // 存储新 api 的地址
    old_api_ptr = GetProcAddress(old_api)
    new_api_ptr = NewFunc; // 获取新 api 地址
    memcpy(new_addr + 2, &new_api_ptr, 8); // 拼接 jmp 指令
    ReadProcessMemory(hProcess, old_api_ptr, old_addr, 12, NULL); // 读出旧 api 地址，可能会用于还原执行原来的函数
	WriteProcessMemory(hProcess, old_api_ptr, new_addr, 12, NULL); // 把新 jmp 指令写入
    ```

3. hook: 在目标进程安装 window 的消息机制中的钩子监听该进程的事件，这里以键盘事件为例

    ```
    hProcess = OpenProcess()
    dll = LoadLibrary("my.dll") // 读取自己的 dll
    addr = GetProcAddress(dll, "MyMessageProcess") // 获取该 dll 中的钩子函数地址
    ThreadID = getThreadID() // 非系统给的，需要遍历该进程下的线程实现
    HHOOK Handle = SetWindowsHookEx(WH_KEYBOARD, addr, dll, ThreadID); // 安装键盘消息钩子
    .......
    UnhookWindowsHookEx(Handle); // 卸载钩子
    FreeLibrary(dll)
    ```

    ```
    /* dll 代码 */
    static HHOOK g_hHook;
    // 省略 DllMain .......
    extern "C" __declspec(dllexport)LRESULT MyMessageProcess(int Code, WPARAM wParam, LPARAM lParam)
    {
        // 为所欲为
        MessageBox(NULL, L"GetMessage!", L"Message", 0);
        return CallNextHookEx(g_hHook, Code, wParam, lParam);
    }
    ```

- 还有几种方式，可以参考 [github 某个大神的代码](https://github.com/fdiskyou/injectAllTheThings)