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

* PE文件中相对虚拟地址（RVA）和文件偏移地址（FOA）的转换：从RVA到FOA就是从文件运行时（动态）的地址转换到磁盘上的地址（静态）。转换方法如下：</br>
```
伪代码:
RVA=VA(虚拟地址，也即内存地址)-ImageBase(映像基址)
if(RVA在PE头中)
{
	FOA=RVA;
}
else
{
	if(RVA>=VA&&RVA<=VA+内存对齐)
	{
		FOA=PointerToRawData+RVA-VA;
	}
}
```

* pe文件或者其他文件进行解析时，需要通过二进制方式读取，即"rb"方式，用文本方式读取会少一个字节。

* pe从内存中解析时，反而不需要rva转foa了，因为和文件无关了，直接读偏移就好了。

* pe文件解析函数地址，代码如下：
```
char* pContent = (char*)handle;    //handle是从内存中获取的基址
//pContent已拿到内容，看如何解析
char* pOffset = pContent;
IMAGE_DOS_HEADER imageDosHeader;
memcpy(&imageDosHeader, pOffset, sizeof(imageDosHeader));
pOffset += sizeof(imageDosHeader);

long long nPos = imageDosHeader.e_lfanew;   //通过这个地方拿偏移，直接到PE头，不要解析DOS STUB
pOffset = pContent + sizeof(char) * nPos;

IMAGE_NT_HEADERS32 ntHeaders;
memset(&ntHeaders, 0, sizeof(ntHeaders));
memcpy(&ntHeaders, pOffset, sizeof(ntHeaders));
pOffset += sizeof(ntHeaders);

//解析节表头
int nSections = ntHeaders.FileHeader.NumberOfSections;
int nTotalSectionSize = nSections * sizeof(IMAGE_SECTION_HEADER);
IMAGE_SECTION_HEADER* pSectionHeaders = (IMAGE_SECTION_HEADER*)malloc(nTotalSectionSize);
memset(pSectionHeaders, 0, nTotalSectionSize);
memcpy(pSectionHeaders, pOffset, nTotalSectionSize);
pOffset += sizeof(nTotalSectionSize);

std::vector<IMAGE_SECTION_HEADER> vecSectionHeader;
for (int i = 0; i < nSections; i++)
{
	vecSectionHeader.push_back(*(pSectionHeaders + i));
}
free(pSectionHeaders);
pSectionHeaders = nullptr;

StringMap vecFuncs;
//section已解出，记录下地址并可以完成rva2foa
UINT uExportAddressFoa = ntHeaders.OptionalHeader.DataDirectory[0].VirtualAddress;
IMAGE_EXPORT_DIRECTORY exportDir;
char* pExportDir = pContent + uExportAddressFoa;
memcpy(&exportDir, pExportDir, sizeof(IMAGE_EXPORT_DIRECTORY));

UINT nameFoa = exportDir.Name;
UINT AddrOfNameFoa = exportDir.AddressOfNames;
UINT* pArrAddrName =(UINT*)(pContent + AddrOfNameFoa);
UINT uAddrOfAddrFunc =exportDir.AddressOfFunctions;
UINT* pArrAddrFunc = (UINT*)(pContent + uAddrOfAddrFunc);
UINT16* pAddrOfOrinal = (UINT16*)(exportDir.AddressOfNameOrdinals + pContent);
for (int i = 0; i < exportDir.NumberOfNames; i++)
{
	//先拿名字，再拿函数地址
	UINT uAddrName = *pArrAddrName;
	UINT uAddrNameFoa =uAddrName;
	char* pName = pContent + uAddrNameFoa;
	std::string strFuncName = pName;

	if ("MessageBoxA" == strFuncName)
	{
		int i = 0;
	}

	//序号表拿序号
	UINT16 dwIndex = pAddrOfOrinal[i];

	//到地址表找地址
	INT uAddrFunc = pArrAddrFunc[dwIndex];
	UINT test=(UINT)(pContent + uAddrFunc);
	int c = 0;
	pArrAddrName++;
}
```
上述中最重要的是函数名称表，函数地址表，函数序号表三张表的关系。如下：
<table>
<tr><th colspan="2">函数名称表</th><th colspan="2">函数序号表</th><th colspan="3">函数地址表</th></tr>
<tr><th>0</th><th>add</th><th>0</th><th>0x0100</th><th>0</th><th>0x1010</th><th>sub</th></tr>
<tr><th>1</th><th>sub</th><th>1</th><th>0x0000</th><th>1</th><th>0x2020</th><th>add</th></tr>
<tr><th>2</th><th>dic</th><th>2</th><th>0x0200</th><th>2</th><th>0x3030</th><th>dic</th></tr>
</table>
找的顺序是，先找函数名称表，比如sub，确定其序号为1之后，去序号表中找序号为1的元素，发现地址为0x0000，作为函数地址表的序号。去找0。得到sub的地址。其中导出表的base属性是函数地址表的起始序号，大多数时候不一定有用。

* 扫描节间空隙并写入shellcode：
```
UINT scanNullInSections()
{
	//读文件找节内空隙
	FILE* fp = nullptr;
	errno_t nError = fopen_s(&fp,"./MFCSHOW.EXE","rb");
	if (0 != nError)
	{
		return 0;
	}
	fseek(fp, 0, SEEK_END);
	int nFileSize = 0;
	nFileSize = ftell(fp);
	fseek(fp, 0, SEEK_SET);

	char* pContent = (char*)malloc(nFileSize + 1);
	memset(pContent, 0, nFileSize + 1);
	fread(pContent, sizeof(char), nFileSize, fp);
	fclose(fp);
	fp = nullptr;
	//找text节中的空隙,寻找的方法其实可以从节尾往前找
	char* pBase = (char*)pContent;
	PIMAGE_DOS_HEADER ImageDosHeader = (IMAGE_DOS_HEADER*)pBase;
	PIMAGE_NT_HEADERS  ImageNtHeaders = (IMAGE_NT_HEADERS*)((size_t)pBase + ImageDosHeader->e_lfanew);
	PIMAGE_OPTIONAL_HEADER  ImageOptionalHeader = &ImageNtHeaders->OptionalHeader;
	PIMAGE_SECTION_HEADER imageSectionHeaders = (PIMAGE_SECTION_HEADER)((char*)ImageNtHeaders + sizeof(IMAGE_NT_HEADERS));
	if (NULL != imageSectionHeaders)
	{
		PIMAGE_SECTION_HEADER imageTextHeader = &imageSectionHeaders[0];
		char* pImageTextSection = pContent + imageTextHeader->PointerToRawData;
		PIMAGE_SECTION_HEADER imageDataHeader = &imageSectionHeaders[1];
		char* pImageDataSection = pContent + imageDataHeader->PointerToRawData;
		bool isTrue=judgeNull(pImageDataSection);

		char shellcode[] = {
		"\x31\xc9\xf7\xe1\x64\x8b\x41\x30\x8b\x40"
		"\x0c\x8b\x70\x14\xad\x96\xad\x8b\x58\x10"
		"\x8b\x53\x3c\x01\xda\x8b\x52\x78\x01\xda"
		"\x8b\x72\x20\x01\xde\x31\xc9\x41\xad\x01"
		"\xd8\x81\x38\x47\x65\x74\x50\x75\xf4\x81"
		"\x78\x04\x72\x6f\x63\x41\x75\xeb\x81\x78"
		"\x08\x64\x64\x72\x65\x75\xe2\x8b\x72\x24"
		"\x01\xde\x66\x8b\x0c\x4e\x49\x8b\x72\x1c"
		"\x01\xde\x8b\x14\x8e\x01\xda\x89\xd5\x31"
		"\xc9\x51\x68\x61\x72\x79\x41\x68\x4c\x69"
		"\x62\x72\x68\x4c\x6f\x61\x64\x54\x53\xff"
		"\xd2\x68\x6c\x6c\x61\x61\x66\x81\x6c\x24"
		"\x02\x61\x61\x68\x33\x32\x2e\x64\x68\x55"
		"\x73\x65\x72\x54\xff\xd0\x68\x6f\x78\x41"
		"\x61\x66\x83\x6c\x24\x03\x61\x68\x61\x67"
		"\x65\x42\x68\x4d\x65\x73\x73\x54\x50\xff"
		"\xd5\x83\xc4\x10\x31\xd2\x31\xc9\x52\x68"
		"\x50\x77\x6e\x64\x89\xe7\x52\x68\x59\x65"
		"\x73\x73\x89\xe1\x52\x57\x51\x52\xff\xd0"
		"\xe9\x00\x00\00\x00"
		};
		DWORD dwShellLen =strlen(shellcode)+4;  //E9的偏移是写入位置-原入口点+off
		char* pOri = pImageDataSection - 256;
		DWORD dwShellRva = pOri - pContent + imageTextHeader->VirtualAddress - imageTextHeader->PointerToRawData;
		DWORD dwOffset = ImageNtHeaders->OptionalHeader.AddressOfEntryPoint - dwShellRva - dwShellLen;
		//将4字节偏移写入shellcode
		BYTE b1 = dwOffset;
		BYTE b2 = dwOffset>>8;
		BYTE b3 = dwOffset>>16;
		BYTE b4 = dwOffset>>24;
		memcpy(shellcode+strlen(shellcode),&b1,1);
		memcpy(shellcode + strlen(shellcode), &b2, 1);
		memcpy(shellcode + strlen(shellcode), &b3, 1);
		memcpy(shellcode + strlen(shellcode), &b4, 1);

		memcpy(pOri, shellcode, sizeof(shellcode));
		ImageNtHeaders->OptionalHeader.AddressOfEntryPoint = dwShellRva;
		FILE* fp = nullptr;
		errno_t nError = fopen_s(&fp, "./1.EXE", "wb");
		if (0 != nError)
		{
			return 0;
		}
		fwrite(pContent, sizeof(char), nFileSize, fp);
		fclose(fp);
		fp = nullptr;
	}
	return 0;
}
```