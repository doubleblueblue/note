1.  对于new和delete，记住一点，new/delete和new[]/delete[]是两个东西，互相配套使用，但是对于内置类型或者无自定义析构函数的类类型，new[]/delete是不会有问题的。

2. 关于static int value=0;的用法，如果有ABC三个文件，BC都引用A文件，在B里给value=2,在C里拿到的值是0不是2，因为static修饰的全局变量的作用域只能是本身的编译单元，在其他编译单元使用它时，只是简单的把其值复制给了其他编译单元，其他编译单元会另外开个内存保存它，在其他编译单元对它的修改并不影响本身在定义时的值。即在其他编译单元C使用它时，它所在的物理地址，和其他编译单元B使用它时，它所在的物理地址不一样，B和C对它所做的修改都不能传递给对方。对于extern而言则不是如此，依然按照前置条件来分析，首先A中的声明应该仅仅是声明，即extern int value;在B中修改的时候有一点是需要注意的-value必须定义为全局变量，即int value;需要在所有函数前面。在C中获取的时候则也只需要在所有函数前面声明（extern int value），可能也可以不声明，本人不声明时似乎不报错。（原因大概率是已经引用了A.h，沿用了A的extern int value）。

3.  对于linux C的项目管理，linuxC本身因为没有IDE的缘故，编译和链接都需要手动进行，LINUXC的编译过程并不会检查函数调用，只有在链接的时候才会进行函数调用的检查，常用的项目结构是一个通用的.h多个.c文件，每多一个.c文件，或者说每多一个函数，就在.h里进行声明，最终的main.c主文件中，只需要包含.h文件即可。这样可以防止出现重定义错误，代码结构也清晰。

4.  对于struct，如下：
```
typedef struct
{
	OVERLAPPED overlapped;
	WSABUF dataBuf;
	char buffer[DATA_BUFSIZE];
}PER_IO_OPERATION_DATA, * LPPER_IO_OPERATION_DATA;

typedef struct
{
	SOCKET socket;
}PER_HANDLE_DATA, * LPPER_HANDLE_DATA;

typedef struct
{
	struct s_conf* conf;
	struct PER_HANDLE_DATA perHandleData;
}PER_HANDLE_DATA_ANDCONFIG, * LPPER_HANDLE_DATA_ANDCONFIG;
```
这段代码，在struct PER_HANDLE_DATA perHandleData这句会报不完整类型错误。这个原因是C和CPP的区别，CPP认为结构体和类是相同的。但是C语言没有类，对于直接使用Struct声明的结构体，在使用时都需要使用 struct StructName structValue这种方式使用，对于上述代码中的，可以直接使用即PER_HANDLE_DATA perHandleData就不会报错。根本原因是typedef是把空结构名替换为尾部的第一个变量了，只是省略了使用时的struct，而不是像通用的struct{}pstruct这种，尾部的pstruct是个结构体指针。

5. mingw+gcc+vscode调试的环境配置：sources下载mingw安装到D盘，vs code安装cmake和cmake tools，通过cmake quick start快速创建一个项目，配置对应的kit即可。安装mingw之后记得重启下电脑。

6. C语言，如果是vc6.0的情况下，有可能会出现重复引用头文件的情况，对于VS来讲可以使用ifdef宏解决，对于vc6.0，一种可用的方式是像webdll代码中定义一长串宏，比如ifdef huaduiadhdafhadhhiohieqwhoqweihewqeqhqwehiohiohiaod类似的情况，每个文件中都define一次，这样如果重复引用，就会报错。

7. C语言打开文件的时候需要的操作比较多，除了fopen去打开文件获取文件指针以外，还需要通过移动文件指针，或者别的方式获取文件长度。
```
fseek(fp,0,SEEK_END);
int nFileSize=0;
nFileSize=ftell(fp);
fseek(fp,0,SEEK_SET);
```

8. 血的教训之：linux下写cmd程序一定要区分自定义输出和系统输出，即：自定义输出可以：printf("kali : error is \n");这样就可以区分官方和自定义输出。

9. 关于字符串拷贝，对于C语言，字符串拷贝时最好还是使用strcpy进行拷贝，而不是使用memcpy，后者会导致按照数组方式遍历字符串时出错。（可能是因为字符串并不严格按照字符数组的方式存储？）

10. C语言或CPP都会有的一个问题，对于一个指针来讲，p++的步长究竟是多少字节。问题本身其实并不需要纠结。纯粹是因为太久了基础忘记了。步长的定义是根据元素的长度来的，步长一定是n个元素的长度，元素本身的长度则另说。因此步长的字节数应该是n*sizeof(元素);

11. 关于flock，基本的用法如下
```
//用于处理文件锁的函数，防止程序被重复打开
bool isAlreadyRunning()
{
	int lock_fd=-1;
	lock_fd=open("/home/temp.lock",O_CREAT,0777);
	if(-1==lock_fd)
	{
		lock_fd=open("/home/temp.lock",O_RDWR,0777);
		if(-1==lock_fd)
		{
			return -1;
		}
	}
	int rc=flock(lock_fd,LOCK_EX|LOCK_NB);
	if(0==rc)
	{
		return 0;
	}
	else
	{
		
		return errno;
	}
}
```
其中open是正常的打开文件方式，返回一个文件描述符，通过flock对文件进行加锁，Lock_Ex控制了锁的类型是排他锁，Lock_nb是控制接下来请求锁，锁对进程的反应，是阻塞，或非阻塞，nb是非阻塞式。那么考虑fork或者execve对锁数据结构的影响，如果父进程先flock获取了锁，之后fork，子进程同样会获取锁，如果子进程重新申请锁，则会覆盖原先的锁，相当于进行了锁类型的修改。