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
    char* strs = new char[str.length() + 1]; //不要忘了
    strcpy_s(strs,str.length(),str.c_str());

    char* d = new char[delim.length() + 1];
    strcpy_s(d,delim.length(),delim.c_str());
    char* pPosition = nullptr;
    char* p = strtok_s(strs, d,&pPosition);
    while (p) {
        std::string s = p; //分割得到的字符串转换为string类型
        res.push_back(s); //存入结果数组
        p = strtok_s(NULL, d,&pPosition);
    }
    delete[] strs;
    delete[] p;
    return res;
}
```

3. base64.cpp   base64编码的实现
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

4. 自定义创建进程函数。（可选阻塞和非阻塞）（winExec和system不满足条件时使用）
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

5. 关于CMD执行传入参数时，传入的参数为空或者可能为空，需要用对应的字符替换。
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

6. 牛批，mfc并没有封装打开文件夹而不打开文件的对话框，因此需要自己进行封装
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