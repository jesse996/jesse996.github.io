# Win32


win32 根据进程名获取进程 ID 或者终止进程：

[](https://blog.csdn.net/zjx_cfbx/article/details/82390064)[https://blog.csdn.net/zjx_cfbx/article/details/82390064](https://blog.csdn.net/zjx_cfbx/article/details/82390064)

[](https://blog.csdn.net/ouchengguo/article/details/88602267)[https://blog.csdn.net/ouchengguo/article/details/88602267](https://blog.csdn.net/ouchengguo/article/details/88602267)

[](https://blog.csdn.net/zwhuang/article/details/2218651)[https://blog.csdn.net/zwhuang/article/details/2218651](https://blog.csdn.net/zwhuang/article/details/2218651)

[](https://www.write-bug.com/article/1568.html)[https://www.write-bug.com/article/1568.html](https://www.write-bug.com/article/1568.html)

### 从窗口句柄获取进程句柄

FindWindow：找串口句柄

GetWindowThreadProcessId：由窗口句柄找进程 id

OpenProcess：由进程 id 得进程句柄

### 内存读写

ReadProcessMemory

WriteProcessMemory

### 通过快照来获取进程 ID

```
HANDLE WINAPI CreateToolhelp32Snapshot(DWORD dwFlags,DWORD th32ProcessID);
BOOL WINAPI Process32First(HANDLE hSnapshot, LPPROCESSENTRY32 lppe);
BOOL WINAPI Process32Next(HANDLE hSnapshot,LPPROCESSENTRY32 lppe);
//结果
typedef struct tagPROCESSENTRY32 {
DWORD dwSize; // 结构大小；
DWORD cntUsage; // 此进程的引用计数；
DWORD th32ProcessID; // 进程ID;
DWORD th32DefaultHeapID; // 进程默认堆ID；
DWORD th32ModuleID; // 进程模块ID；
DWORD cntThreads; // 此进程开启的线程计数；
DWORD th32ParentProcessID; // 父进程ID；
LONG pcPriClassBase; // 线程优先权；
DWORD dwFlags; // 保留；
char szExeFile[MAX_PATH]; // 进程全名；
} PROCESSENTRY32;

```

x64-dbg 使用：

[](https://www.bilibili.com/s/video/BV1jK4y1b7wc)[https://www.bilibili.com/s/video/BV1jK4y1b7wc](https://www.bilibili.com/s/video/BV1jK4y1b7wc)

cs source:控制台（~）

sv_cheats 1

bot_add_t

bot_stop 1

bot_stop 0

### 设置滚动条：

SetScrollInfo

GetScrollInfo

滚动条变化了调用 ScrollWindows,再调用 UpdateWindow

### DC 三种获取方法与销毁方法（要成对出现）

-   GetDC()——ReleaseDC()
-   GeginPaint()——EndPaint()
-   GetCompatibleDC()——DeleteDC()

