apiHook的方法，apiHook的思路很简单，通过远程线程注入的方式让目标进程调用我们自己的DLL。注入器代码如下：
```
// InjectProgram.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//

/*
远程线程注入程序，主要负责将dll注入到对应的进程的远程线程中
*/
#include<Windows.h>
#include<tlhelp32.h>
#include <assert.h>
#include<tchar.h>
#include <iostream>

//获取进程name的ID
DWORD GetPid(LPTSTR name)
{
	HANDLE hProcSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);        //获取进程快照句柄
	assert(hProcSnap != INVALID_HANDLE_VALUE);                                 //判断是否打开成功
	PROCESSENTRY32 pe32;
	pe32.dwSize = sizeof(PROCESSENTRY32);
	BOOL flag = Process32First(hProcSnap, &pe32);                          //获取列表的第一个进程
	while (flag)
	{
		if (!_tcscmp(pe32.szExeFile, name))
		{
			CloseHandle(hProcSnap);
			return pe32.th32ProcessID;//pid
		}
		flag = Process32Next(hProcSnap, &pe32);//获取下一个进程
	}
	CloseHandle(hProcSnap);
	printf("没有找到相关进程");
	return 0;
}


// 提权函数：提升为DEBUG权限
BOOL EnableDebugPrivilege()
{
	HANDLE hToken;
	BOOL fOk = FALSE;
	if (OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES, &hToken))
	{
		TOKEN_PRIVILEGES tp;
		tp.PrivilegeCount = 1;
		LookupPrivilegeValue(NULL, SE_DEBUG_NAME, &tp.Privileges[0].Luid);

		tp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
		AdjustTokenPrivileges(hToken, FALSE, &tp, sizeof(tp), NULL, NULL);

		fOk = (GetLastError() == ERROR_SUCCESS);
		CloseHandle(hToken);
	}
	return fOk;
}

int main()
{
	EnableDebugPrivilege();
	printf("输入要注入的进程名(.exe):\n");
	char cname[260] = {};
	scanf_s("%s", cname, 260);

	printf("输入dll路径(\\)\n");
	char cpath[260];
	scanf_s("%s", cpath, 260);


	DWORD Pid = 0;
	Pid = GetPid((LPTSTR)cname);

	// 用PID打开进程 参数1：权限 参数2：是否继承 参数3：PID
	HANDLE Handle = OpenProcess(PROCESS_ALL_ACCESS, FALSE, Pid);


	//2. 远程进程中申请空间  
	//参数1：进程句柄 参数2：起始地址(NULL函数决定分配到哪）
	//参数3：要分配的大小(不够一页分一页) 参数4：内存分配类型(保留提交) 参数5：内存保护
	LPVOID Address = VirtualAllocEx(Handle, NULL, 0x100, MEM_RESERVE | MEM_COMMIT, PAGE_READWRITE);


	//3.向远程进程写入数据
	//也就是LoadLibrary的参数，即Dll的地址
	//不同的系统使用到的 LoadLibrary 可能是 A/W 的，需要和字符串匹配
	DWORD RealWrite = 0; //实际写入的地址
	// 参数1：进程句柄 参数2：写入数据的起始地址 参数3：写入的数据 参数4：写入字节数 参数5：实际写入字节数
	WriteProcessMemory(Handle, Address, cpath, strlen(cpath) + 1, &RealWrite);


	//4. 在目标进程内创建远程线程
	//线程的起始位置是 LoadLibrary，在Windows下，所有 Windows API 在不同进程中的函数地址都是相同
	//参数1：进程句柄 参数2：安全描述符(为0默认) 参数3：堆栈初始大小(为0默认) 
	//参数4：指向由线程执行的，类型为LPTHREAD_START_ROUTINE的应用程序定义的函数指针
	//参数5：函数的参数指针 参数6：控制线程创建的标志(为0立即执行) 参数7：指向接收线程标识符的变量的指针(为0不接收)
	HANDLE Thread = CreateRemoteThread(Handle, NULL, NULL, (LPTHREAD_START_ROUTINE)LoadLibraryA, Address, NULL, NULL);


	//5.等待远程线程执行完毕
	WaitForSingleObject(Thread, -1);

	//6. 释放空间
	//可以选择是否释放注入的模块
	//VirtualFreeEx(Handle, Address, NULL, MEM_RELEASE);


	system("pause");
	//释放句柄
	CloseHandle(Thread);
	CloseHandle(Handle);
	return 0;
}
```
有了注入器之后，需要做的是自己写一个对应的DLL，hook对应的api，例如MessageBoxA代码如下:
```
// dllmain.cpp : 定义 DLL 应用程序的入口点。
#include "pch.h"
#include <Windows.h>

bool Hook(const char* pszModuleName, const char* pszFuncName, PROC pfnHookFunc);
bool unHook();
bool reHook();
HANDLE WINAPI CustomCreateFileA(
    _In_ LPCSTR lpFileName,
    _In_ DWORD dwDesiredAccess,
    _In_ DWORD dwShareMode,
    _In_opt_ LPSECURITY_ATTRIBUTES lpSecurityAttributes,
    _In_ DWORD dwCreationDisposition,
    _In_ DWORD dwFlagsAndAttributes,
    _In_opt_ HANDLE hTemplateFile
);

int WINAPI CustomMessageBoxA(
    _In_opt_ HWND hWnd,
    _In_opt_ LPCSTR lpText,
    _In_opt_ LPCSTR lpCaption,
    _In_ UINT uType);

PROC m_gFuncAddress;   //被HOOK函数的原始地址
BYTE m_gOldBytes[5];   //被替换掉的5个字节，供恢复使用
BYTE m_gNewBytes[5];   //自定义函数的 jmp address

BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
    {
        m_gFuncAddress = nullptr;
        memset(m_gNewBytes, '\0', 5);
        memset(m_gOldBytes, '\0', 5);
        Hook("Kernel32.dll", "CreateFileA", (PROC)CustomCreateFileA);
    }
        break;
    case DLL_THREAD_ATTACH:
        break;
    case DLL_THREAD_DETACH:
        break;
    case DLL_PROCESS_DETACH:
    {
        unHook();
    }
        break;
    }
    return TRUE;
}


bool Hook(const char* pszModuleName, const char* pszFuncName, PROC pfnHookFunc)
{
    //获取指定模块中指定函数的地址，比如user32.dll中的CreatFile等
    m_gFuncAddress = GetProcAddress(GetModuleHandle(pszModuleName), pszFuncName);
    if (nullptr == m_gFuncAddress)
    {
        return false;
    }

    DWORD dwStart = 0;
    //下一句最精髓的在于，获取当前进程句柄的方式，这个地方因为DLL是被目标进程获取的，所以目标进程就是当前进程
    ReadProcessMemory(GetCurrentProcess(), m_gFuncAddress, m_gOldBytes, 5, &dwStart);

    //构造jmp语句
    m_gNewBytes[0] = '\xE9';
    //地址计算公式  HOOK函数地址-目标函数地址-5
    *(DWORD*)(m_gNewBytes + 1) = (DWORD)pfnHookFunc - (DWORD)m_gFuncAddress - 5;

    //将jmp写入对应的地址
    WriteProcessMemory(GetCurrentProcess(), m_gFuncAddress, m_gNewBytes, 5, &dwStart);
    return true;
}

bool unHook()
{
    if (nullptr != m_gFuncAddress)
    {
        DWORD dwStart = 0;
        WriteProcessMemory(GetCurrentProcess(), m_gFuncAddress, m_gOldBytes, 5, &dwStart);
    }
    return true;

}

bool reHook()
{
    if (nullptr != m_gFuncAddress)
    {
        DWORD dwStart = 0;
        WriteProcessMemory(GetCurrentProcess(), m_gFuncAddress, m_gNewBytes, 5, &dwStart);
    }
    return true;
}

HANDLE WINAPI CustomCreateFileA(
    _In_ LPCSTR lpFileName,
    _In_ DWORD dwDesiredAccess,
    _In_ DWORD dwShareMode,
    _In_opt_ LPSECURITY_ATTRIBUTES lpSecurityAttributes,
    _In_ DWORD dwCreationDisposition,
    _In_ DWORD dwFlagsAndAttributes,
    _In_opt_ HANDLE hTemplateFile
)
{
    MessageBoxA(NULL, "hook success!", "waring", 0);

    //调用原有流程
    unHook();
    HANDLE nRet = CreateFileA(lpFileName, dwDesiredAccess, dwShareMode, lpSecurityAttributes, dwCreationDisposition, dwFlagsAndAttributes, hTemplateFile);
    reHook();
    return nRet;
}


int WINAPI CustomMessageBoxA(
    _In_opt_ HWND hWnd,
    _In_opt_ LPCSTR lpText,
    _In_opt_ LPCSTR lpCaption,
    _In_ UINT uType)
{
    unHook();
    int nRet=MessageBoxA(hWnd, "11111", "11111", uType);
    reHook();
    return nRet;
}
```
以上代码组合起来，就可以HOOK32位调用了MessageBoxA的程序。</br>

* 64位Inline Hook:应该说hook的思路本身就是这样，不一定非要去取一整条指令，只要保证第一条替换进去的跳转指令执行即可。后续执行自定义API函数之后再恢复。就能够达到目的并且保存原先程序的完整性。所以32位下的5个字节的说法其实并不固定，如果有需要，你给个10个字节甚至更多也是可以的。因此对于64位HOOK，只需要在原先的基础上删去函数地址计算。即拿即用即可。代码如下：
```
class MyHook
{
public:
	static UINT64 Hook(LPCWSTR lpModule, LPCSTR lpFuncName, LPVOID lpFunction)
	{
		UINT64 dwAddr = (UINT64)GetProcAddress(GetModuleHandleW(lpModule), lpFuncName);
		BYTE jmp[] =
		{
			0x48, 0xb8,               // jmp
			0x00, 0x00, 0x00, 0x00,
			0x00, 0x00, 0x00, 0x00,   // address
			0x50, 0xc3                // retn
		};

		ReadProcessMemory(GetCurrentProcess(), (LPVOID)dwAddr, MemoryAddress(), 12, 0);
		UINT64 dwCalc = (UINT64)lpFunction;
		memcpy(&jmp[2], &dwCalc, 8);

		WriteProcessMemory(GetCurrentProcess(), (LPVOID)dwAddr, jmp, 12, nullptr);
		return dwAddr;
	}

	static BOOL UnHook(LPCWSTR lpModule, LPCSTR lpFuncName)
	{
		UINT64 dwAddr = (UINT64)GetProcAddress(GetModuleHandleW(lpModule), lpFuncName);

		if (WriteProcessMemory(GetCurrentProcess(), (LPVOID)dwAddr, MemoryAddress(), 12, 0))
			return TRUE;
		return FALSE;
	}

	static BYTE* MemoryAddress()
	{
		static BYTE backup[12];
		return backup;
	}
};
```
来自网上的一个hook类。

* hook任意地址时，如果使用jmp需要去考虑保存和恢复寄存器的值，这个时候应该保存和恢复哪些寄存器？ push ad,pop ad。