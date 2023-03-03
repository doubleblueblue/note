1.  对于new和delete，保持一个原则，非类不new，比如常用到的连续内存当数组使用，对于纯C方式的malloc仅需要malloc(nLen)，对于new方式则需要new char\[nLen\]();且析构时需要delete[] msg；较为麻烦。new主要还是可以进行析构。

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