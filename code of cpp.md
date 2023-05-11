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