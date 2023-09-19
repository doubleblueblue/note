https://account.microsoft.com/services?refd=account.microsoft.com 微软office安装页面
1. 遍历文件夹，获取文件信息，等操作的函数。
```
//分别遍历文件夹下内容
HANDLE handleFile = INVALID_HANDLE_VALUE;
WIN32_FIND_DATA findData;
LARGE_INTEGER filesize;
handleFile = FindFirstFile("C:\\ProgramData\\Microsoft\\Windows\\Start Menu\\Programs\\*", &findData);
if (INVALID_HANDLE_VALUE == handleFile)
{
    printf("get start path error\n");
}
do
{
    if (findData.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)
    {
        //DIR
    }
    else
    {
        filesize.LowPart = findData.nFileSizeLow;
        filesize.HighPart = findData.nFileSizeHigh;
        printf("fileName is %s\n", findData.cFileName);
        //获取文件类型
        std::string strTmp = strStartPath + "\\" + findData.cFileName;
        if (isLNKFile(strTmp))
        {

        }
    }
} while (FindNextFile(handleFile, &findData)!=0);
FindClose(handleFile);
```

2. 分割字符串函数：
```
std::vector<std::string> split(const std::string& str, const std::string& delim)
{
	std::vector<std::string> res;
	if ("" == str) return res;
	//先将要切割的字符串从string类型转换为char*类型
	char* strs = new char[str.length() + 2]; //不要忘了
	strcpy_s(strs, str.length()+1, str.c_str()); //是有\0导致缓冲区过小，因此上述创建+2，下述复制+1（\0）

	char* d = new char[delim.length() + 2];
	strcpy_s(d, delim.length()+1, delim.c_str());
	char* pPosition = nullptr;
	char* p = strtok_s(strs, d, &pPosition);
	while (p) {
		std::string s = p; //分割得到的字符串转换为string类型
		res.push_back(s); //存入结果数组
		p = strtok_s(NULL, d, &pPosition);
	}
	delete[] strs;
	delete[] p;
	return res;
}
```

3. 关于C++中string的理解，比如str.length()是10，则实际上str是一个11字节的字符数组。在从C++ API转为C API时需要注意，否则会拷贝失败，但这些大概率是可以在调试发现的。例证如下：
```
std::string strExd = "jpg;md;txt";
int nLength = strExd.length();  //10
int nSize = strExd.size();    //10
int nLs = strlen(strExd.c_str()); //10
const char* pStr = strExd.c_str(); //如果查看第11个char，可以看到是\0
```
结合例2看出，转换时需要多给2个空间，复制时需要多给1个空间。

4. base64.cpp   base64编码的实现
```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

const char base64_table[65] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";

void base64_encode(const unsigned char* src, int src_len, char* dst) {
    int i, j;
    unsigned char buf[3];
    int len, out_len;

    out_len = 0;
    for (i = 0; i < src_len; i += 3) {
        len = src_len - i < 3 ? src_len - i : 3; // 处理剩余部分
        for (j = 0; j < len; j++) {
            buf[j] = src[i + j];
        }
        dst[out_len++] = base64_table[buf[0] >> 2];
        dst[out_len++] = base64_table[((buf[0] & 0x03) << 4) | ((len > 1 ? buf[1] : 0) >> 4)];
        dst[out_len++] = len > 1 ? base64_table[((buf[1] & 0x0f) << 2) | ((len > 2 ? buf[2] : 0) >> 6)] : '=';
        dst[out_len++] = len > 2 ? base64_table[buf[2] & 0x3f] : '=';
    }
    dst[out_len] = '\0';
}

int base64_decode(const char* src, unsigned char* dst, int max_decoded_len) {
    int i, j;
    unsigned char buf[4];
    int len, out_len;
    int pad_count = 0;

    out_len = 0;
    for (i = 0; src[i] != '\0'; i += 4) {
        len = 3;
        pad_count = 0;
        for (j = 0; j < 4; j++) {
            if (src[i + j] == '=') {
                pad_count++;
                buf[j] = 0;
            }
            else {
                buf[j] = strchr(base64_table, src[i + j]) - base64_table;
            }
        }
        switch (pad_count) {
        case 1:
            len = 2;
            break;
        case 2:
            len = 1;
            break;
        default:
            break;
        }
        if (out_len + len > max_decoded_len) {
            return -1; // 解码后的字符串过长
        }
        dst[out_len++] = (buf[0] << 2) | (buf[1] >> 4);
        if (len > 1) {
            dst[out_len++] = (buf[1] << 4) | (buf[2] >> 2);
        }
        if (len > 2) {
            dst[out_len++] = (buf[2] << 6) | buf[3];
        }
    }
    return out_len;
}
```

5. 自定义创建进程函数。（可选阻塞和非阻塞）（winExec和system不满足条件时使用）
```
HANDLE CustomCreateProcess(const std::wstring& strCmd, bool isWait)
{
	STARTUPINFO si = { 0 };
	PROCESS_INFORMATION pi = { 0 };
	si.cb = sizeof(STARTUPINFO);
	si.dwFlags = STARTF_USESHOWWINDOW;
	si.wShowWindow = SW_HIDE;
	//拼接完成，开始创建进程
	int nRet = CreateProcess(NULL,
		(WCHAR*)strCmd.c_str(),
		NULL,
		NULL,
		NULL,
		CREATE_NEW_CONSOLE,
		NULL,
		NULL,
		&si,
		&pi);
	if (0 == nRet)
	{
		//出错
		OutputDebugString(L"GMAIL CREATE PROCESS FAILED!\n");
		return 0;
	}
	if (true == isWait)
	{
		WaitForSingleObject(pi.hProcess, INFINITE);
		return INVALID_HANDLE_VALUE;
	}
	return pi.hProcess;
}
```

6. 关于CMD执行传入参数时，传入的参数为空或者可能为空，需要用对应的字符替换。
```
情况1：
PS D:\personWorkSpace\Connect> ./Connect.exe "" "" "write" "emmc" "system" "D:\personWorkSpace\Connect\misc.bin" "\\.\COM80"
program starting

PS D:\personWorkSpace\Connect>

情况2：
./Connect.exe " " " " "write" "emmc" "system" "D:\personWorkSpace\Connect\misc.bin" "\\.\COM80"
program starting

gptParse at system
***
end
```
明显前者是中断了，但是后者是执行完成了的。

7. 牛批，mfc并没有封装打开文件夹而不打开文件的对话框，因此需要自己进行封装
```
BROWSEINFO bi;
	char Buffer[MAX_PATH];

	//初始化入口参数 bi
	bi.hwndOwner = NULL;
	bi.pidlRoot = NULL;
	bi.pszDisplayName = Buffer;
	bi.lpszTitle = "文件夹路径选择";
	bi.ulFlags = BIF_EDITBOX;
	bi.lpfn = NULL;
	bi.iImage = IDR_MAINFRAME;

	LPITEMIDLIST pIDList = SHBrowseForFolder(&bi); //调用显示选择对话框 
						   //注意下 这个函数会分配内存 但不会释放 需要手动释放


	if (pIDList)
	{
		SHGetPathFromIDList(pIDList, Buffer);
		//CString GamePath;
		//GamePath = Buffer; //将文件夹路径保存在CString 对象里面
		//取得文件夹路径放置Buffer空间
		//GUI_ShowMessage(true, Buffer);

	}

	CoTaskMemFree(pIDList); //释放pIDList所指向内存空间;
	TRACE("%d", pIDList);

	// 把变量内容更新到对话框
	UpdateData(FALSE);
```

8. 注册表相关的操作函数,首先是获取子项，传入主KEY和子KEY即可获取，会返回cJson的一个对象：
```
cJSON* getSubkeysJson(const std::string& strHKey,const std::string& strSubKey)
{
	cJSON* subKeys=nullptr;
	subKeys = cJSON_CreateArray();
	HKEY hKey = transString2HKey(strHKey);
	//打开注册表
	HKEY queryKey;
	LSTATUS status=RegOpenKey(hKey, strSubKey.c_str(), &queryKey);
	if (ERROR_SUCCESS != status)
	{
		return nullptr;
	}
	DWORD dwSubKeysCount;
	DWORD dwValuesCount;
	//获取注册表信息
	status = RegQueryInfoKey(queryKey,  //查询的注册表项
		NULL, //lpClass 不关注
		NULL, //lpcClass 前一个的长度 不关注
		NULL, //保留字段
		&dwSubKeysCount, //子项的数量
		NULL, //子项最长键大小   不关注
		NULL, //子项最长Class大小  不关注
		&dwValuesCount, //查询项的值数量
		NULL, //最长值名称大小   不关注
		NULL, //最长值大小，不关注
		NULL, //密钥安全描述符  不关注
		NULL);  //最后修改时间  不关注 

	//遍历子项
	if (dwSubKeysCount)
	{
		for (int i = 0; i < dwSubKeysCount; i++)
		{
			char name[256] = {};
			memset(name, 0, 256);
			DWORD dwSize = 255;
			status = RegEnumKeyExA(queryKey, i, name, &dwSize, NULL, NULL, NULL, NULL);
			if (ERROR_SUCCESS == status)
			{
				std::string tmp = name;
				cJSON_AddItemToArray(subKeys, cJSON_CreateString(tmp.c_str()));
			}
		}
	}
	RegCloseKey(queryKey);
	return subKeys;
}
```
获取注册表值的函数:
```
cJSON* CRegistryManager::getValueKeysJson(const std::string& strHKey,const std::string& strSubKey)
{
	cJSON* valueKey = nullptr;
	valueKey = cJSON_CreateArray();
	//先处理arg1
	HKEY hKey = transString2HKey(strHKey);
	//打开注册表
	HKEY queryKey;
	LSTATUS status = RegOpenKey(hKey, strSubKey.c_str(), &queryKey);
	//LSTATUS status = RegOpenKey(hKey, strSubKey.c_str(), NULL, KEY_ALL_ACCESS, &queryKey);
	if (ERROR_SUCCESS != status)
	{
		return nullptr;
	}
	DWORD dwSubKeysCount;
	DWORD dwValuesCount;
	//获取注册表信息
	status = RegQueryInfoKey(queryKey,  //查询的注册表项
		NULL, //lpClass 不关注
		NULL, //lpcClass 前一个的长度 不关注
		NULL, //保留字段
		&dwSubKeysCount, //子项的数量
		NULL, //子项最长键大小   不关注
		NULL, //子项最长Class大小  不关注
		&dwValuesCount, //查询项的值数量
		NULL, //最长值名称大小   不关注
		NULL, //最长值大小，不关注
		NULL, //密钥安全描述符  不关注
		NULL);  //最后修改时间  不关注 

	//遍历值
	if (dwValuesCount)
	{
		for (int i = 0; i < dwValuesCount; i++)
		{
			char valueName[256] = {};
			memset(valueName, 0, sizeof(valueName));
			DWORD dwValueNameSize = 255;

			DWORD dwValueType = 0;

			BYTE valueValue[0xffff] = {};
			memset(valueValue,'\0', sizeof(valueValue));
			DWORD dwValueValueSize = 0xffff;

			status = RegEnumValue(queryKey, i,
				valueName,
				&dwValueNameSize,
				NULL,
				&dwValueType,
				valueValue,
				&dwValueValueSize);

			if (ERROR_SUCCESS == status)
			{
				cJSON* tmp = cJSON_CreateObject();
				char cType[255] = { '\0' };
				memset(cType, '\0', sizeof(cType));
				sprintf(cType, "%d", dwValueType);
				std::string strType = cType;
				cJSON_AddItemToObject(tmp, "valueName", cJSON_CreateString(valueName));
				strType = intRegValue2String(strType);
				cJSON_AddItemToObject(tmp, "valueType", cJSON_CreateString(strType.c_str()));
				std::string strValue = regDataToStr(valueValue, dwValueValueSize, dwValueType);
				cJSON_AddItemToObject(tmp, "valueValue", cJSON_CreateString(strValue.c_str()));
				cJSON_AddItemToArray(valueKey, tmp);
			}
		}
	}
	RegCloseKey(queryKey);
	return valueKey;
}
```
其他都会跟这些函数比较类似，因此不作记录。

9. 进程信息的遍历，如下:
```
std::map<DWORD, std::string> getProcessInfo()
{
	std::map<DWORD, std::string> mapReturn;
	HANDLE hProcessSnapShot = NULL;
	PROCESSENTRY32 pe32 = { 0 };

	hProcessSnapShot = ::CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, NULL);

	if (hProcessSnapShot == (HANDLE)-1) return std::map<DWORD, std::string>();

	pe32.dwSize = sizeof(PROCESSENTRY32);

	if (Process32First(hProcessSnapShot, &pe32))
	{
		do {
			std::string strExeFile = pe32.szExeFile;
			mapReturn.insert(std::pair<DWORD, std::string>(pe32.th32ProcessID, strExeFile));
		} while (Process32Next(hProcessSnapShot, &pe32));
	}
	else
		::CloseHandle(hProcessSnapShot);
	::CloseHandle(hProcessSnapShot);
	return mapReturn;
}
```

10. wstring和string之间的转换，不涉及编码问题：
```
std::string wString2String(const std::wstring & wStr)
{
	int nLen = ::WideCharToMultiByte(CP_ACP, NULL, wStr.c_str(), -1, NULL, 0, NULL, NULL);
	char* buf = new char[nLen];
	::WideCharToMultiByte(CP_ACP, NULL,wStr.c_str(), -1, buf, nLen, NULL, NULL);
	std::string str(buf);
	delete[] buf;
	return str;
}

std::wstring string2WString(const std::string& str)
{
	int nLen = ::MultiByteToWideChar(CP_ACP, NULL, str.c_str(), -1, NULL, 0);
	wchar_t* buffer = new wchar_t[nLen];
	::MultiByteToWideChar(CP_ACP, NULL, str.c_str(),-1,buffer, nLen);
	std::wstring wStr(buffer);
	delete[] buffer;
	return wStr;
}
```

11. 原生CPP并不提供对字符串进行全部替换的功能，因此如果使用需要自行处理，以下为全部替换的函数:
```
/// \brief 暂时只有1.0，带业务的字符串替换，"\\","\\\\"
const char* pChar=strJson.c_str();
int nBufferSize = 1024;
char* pBuffer = (char*)malloc(nBufferSize);
memset(pBuffer, '\0', nBufferSize);
int nPos = 0;
while ('\0'!=*pChar)
{
	if ('\\' != *pChar)
	{
		pBuffer[nPos] = *pChar;
		nPos++;
		pChar++;
	}
	else
	{
		pBuffer[nPos] = '\\';
		pBuffer[nPos + 1] = '\\';
		nPos += 2;
		pChar++;
	}
	if (nBufferSize - nPos <= 100)
	{
		//内存不够了
		void* pNew = realloc(pBuffer, nBufferSize + 1024);
		if (nullptr != pNew)
		{
			pBuffer = (char*)pNew;
			memset(pBuffer + 1024, '\0', 1024);
			nBufferSize += 1024;
		}
	}
}
strJson.clear();
strJson = pBuffer;
free(pBuffer);
pBuffer = nullptr;
```
12. lambda表达式示例代码:
```
auto plus = [] (int v1, int v2) -> int { return v1 + v2; }
int sum = plus(1, 2);
```

13. 闪退时生成dump文件：
```
#include<iostream>
#include<Windows.h>
#include<DbgHelp.h>
using namespace std;
#pragma comment(lib,"DbgHelp.lib")
 
// 创建Dump文件
void CreateDumpFile(LPCWSTR lpstrDumpFilePathName, EXCEPTION_POINTERS *pException)
{
	HANDLE hDumpFile = CreateFile(lpstrDumpFilePathName, GENERIC_WRITE, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
	// Dump信息
	MINIDUMP_EXCEPTION_INFORMATION dumpInfo;
	dumpInfo.ExceptionPointers = pException;
	dumpInfo.ThreadId = GetCurrentThreadId();
	dumpInfo.ClientPointers = TRUE;
	// 写入Dump文件内容
	MiniDumpWriteDump(GetCurrentProcess(), GetCurrentProcessId(), hDumpFile, MiniDumpNormal, &dumpInfo, NULL, NULL);
	CloseHandle(hDumpFile);
}
 
// 处理Unhandled Exception的回调函数
LONG ApplicationCrashHandler(EXCEPTION_POINTERS *pException)
{
	CreateDumpFile(L"Test.dmp", pException);
	cout << "异常已记录" << endl;
	system("pause");
	return EXCEPTION_EXECUTE_HANDLER;
}
 
void func1() {
	cout << "正常函数" << endl;
}
 
void func2() {
	int num = 10;
	int in;
	cout << "输入一个整数：" << endl;
	//当输入为0时则会发生异常
	cin >> in;
	num = num / in;
	cout << num << endl;
}
 
int main()
{
	//注册异常处理函数
	SetUnhandledExceptionFilter((LPTOP_LEVEL_EXCEPTION_FILTER)ApplicationCrashHandler);
	func1();
	func2();
}
```
14. 递归控制层数，本质上其实就是利用了递去的流程和归来的流程完成了层级的控制，递去和归来，将一个数字保持在原状，但是层级之前切换的时候，会先有只递去的情况，所以计数还是会增加的。所以可以控制层级。以下是示例代码：
```
bool explorePathAndCopyFile(const std::string& strPath, int& nCount)
{
	HANDLE handleFile = INVALID_HANDLE_VALUE;
	WIN32_FIND_DATA findData;
	LARGE_INTEGER filesize;

	handleFile = FindFirstFile((strPath + "\\*").c_str(), &findData);
	if (INVALID_HANDLE_VALUE == handleFile)
	{
		// 获取当前用户名
		char currentUser[256] = { 0 };
		DWORD dwSize_currentUser = 256;
		GetUserName(
			currentUser,			// 接收当前登录用户的用户名
			&dwSize_currentUser		// 缓冲区大小
		);
		std::string strCurrentUser = currentUser;
		if (strPath == ("C:\\Users\\" + strCurrentUser + "\\Documents\\My Music")
			|| strPath == ("C:\\Users\\" + strCurrentUser + "\\Documents\\My Videos")
			|| strPath == ("C:\\Users\\" + strCurrentUser + "\\Documents\\My Pictures"))
		{
		}
		else
		{
			int nGetLasterror = GetLastError();
			printf("get start path error,error code is %d\n",nGetLasterror);
		}
	}
	if (nCount <= 0)
	{
		//nCount++; 为什么要屏蔽这一句，因为这是递去的过程中，如果++，会导致递去的--过程中触发++，进而导致遍历到很底层的位置，远不止nCount级
		return false;
	}
	nCount--;
	do
	{
		if (findData.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)
		{
			if ("." != std::string(findData.cFileName) && ".." != std::string(findData.cFileName))
			{
				std::string strTmpPath = strPath + "\\" + std::string(findData.cFileName);
				explorePathAndCopyFile(strTmpPath, nCount);
			}
		}
		else
		{
		}
	} while (FindNextFile(handleFile, &findData) != 0);
	FindClose(handleFile);
	nCount++;
	return false;
}

```
以上这段代码中有2个要点：1-利用了nCount计数，实现递去--归来++的层级控制，2-遍历windows的Documents文件夹时，他这个Documents下看不到MyMusic但是可以CD进去，有这个结构目录，也能遍历到，但是会拒绝访问。（应该是win的一个BUG）