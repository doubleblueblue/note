#笔记

1. QT中QGraphicsView无法使用信号槽机制，因此可以重写一个类继承然后实现信号槽机制。

2. 关于QOCI驱动,我自己使用过程中，直接使用qsqloci.dll没有生效。qsqlocid.dll是debug环境下使用的，qsqloci.dll是release环境使用的。

3. sqlite数据库虽然可以使用char(32)这种限制字符大小，但实际上并不会限制死，因此如果数据大于32字节，也会保存进去，在向oracle或者sqlserver这种强约束的数据库转换时会出问题。

4. QWizard中重写父类的validateCurrentPage()会导致子类重写的validatePage失效，所以需要在父类中调用子类的方法。

5. 数据库转换或者操作的时候，事务回滚需要先有表,进行建表和数据插入的操作时最好也是先把全部的表建立，再进行数据的插入。

6. 对于model/view模型，这个模型对model进行重写时，和原本自带的model做一个对比就是，原先的模型是添加item到model。对于model来说，是通过modelindex进行item的text的访问，重写的时候，需要自己给model添加一个存储数据的数据结构，相当于集成了item和model的功能，此时的index只负责数据的index，而不负责item(已被删去).重写qtablemodel时，对于所有函数来讲，都必须返回一个return qVariant(); 比如表头，不返回就不显示。以及，对于model模型来说，每一次应该只获取一个单元格的数据，数据的获取在model中进行，比如查询数据库的操作，应该在model的data或者自定义函数进行。

7. 条件表达式：     条件？条件为真时的选择：条件为假时的选择

8. https://zhuanlan.zhihu.com/p/97128024   关于引用的详细解释。

9. 左值和右值的广泛认同的说法是：可以取地址，有名字的，非临时的是左值，相反，不能取地址，没有名字的，临时的是右值。    可见立即数，函数返回值，是右值，非匿名对象，函数返回的引用，const对象，等是左值。

10. 数据库的操作和管理，编写sql语句时，还是要采用prepare+bindvalue这种形式进行操作。<font color="red">绑定值无法绑定表名,这是极其需要注意的，绑定表名不会报错，但是会查不出来结果。</font>查询出来的值，根据类型，在代码中进行类型的转换，比如日期类型就采用QDATE进行存储，然后用bindvalue发送，简而言之，还是要进行一个解析和保存的过程。

11. 覆盖窗最好还是采用是否按钮+记住check，4个按钮给人的感觉比较奇怪。虽然VS是这么做的。

12. 模态对话框和非模态对话框，模态对话框弹出之后主程序不继续运行，非模态对话框弹出之后主程序还在继续运行，对于dialog来讲，开启非模态对话框需要指针创建对话框+使用show。

13. 智能指针和指针的问题，在我返回GsLay\*时，为什么用Ptr去接接不到，然后用\*去接，会无法初始化。多态的基本条件是，父类指针指向子类对象，用Ptr或者\*去接，实际上是用子类指针指向父类对象，这个是不被允许的，在允许多继承的情况下。

14. 关于指针和new对象的方式，指针指向new的对象之后，指针本身还是需要被delete，new的对象如果不传入QT父子链的this即parent指针，就不会被析构，但是传入，则会被父类自动析构。说白了，析构是析构，删指针是删指针。

15. 关于功能或者软件的测试，测试的主要方面就是功能能不能实现，值对不对，然后保证值正确的情况下去测试优化效率，比如速度或者内存之类的，在此之后去测一些隐藏的BUG，隐藏的BUG一般会需要一些特殊的数据，非常难测。

16. 指针的指向最好不要连指，就像等号最好不用连等一样，即以下形式需要注意：a->b->c;a=b=c;d

17. 关于虚基类和智能指针，虚基类不能够用new去创建实例对象，但是能够有智能指针指向虚基类。比如上述逻辑编目。可以给catalogItemPtr赋值一个返回的catalogItemPtr。

18. 关于匿名函数，匿名函数目前比较方便的使用场景就是为控件写简单的槽函数时，不必要在类中进行复杂的声明，匿名函数的使用形式是[]()->returnvalue{},[]内是捕获方式，关于捕获这一块，可以理解是使用值或参数的方式，外部的变量在内部使用时，需要设置此方式，给值可以有=，&，也可以指定具体的变量，比如&c，=是按值捕获，&是按引用捕获，（）是参数列表，如同正常函数，->之后的是返回值类型，{}是函数体。

19. 遍历map和vec等容器时，采用迭代器进行遍历。意思是尽量不用at()函数来遍历。特别是stl的容器，qt的容器稍微好点。

20. ActionGroup，动作的集合，集合默认是互斥的，也就是组内的动作只有一个能被激活，这样就实现了绘制工具上，A工具被激活时B工具取消激活状态的操作。至于其他的类型的操作，有其他类型的操作集合。

21. 关于函数返回静态局部指针时，静态局部指针的生命周期不是只在函数内，作用域也不是，跟普通静态局部变量区分的是，静态局部变量的生命周期被延长，但是作用域还是在函数内（主要这个例子是返回值）。内存泄漏主要是针对调用析构的时候没有释放内存而言。析构函数中释放了对应内存就OK。

22. <font color="red">C++中继承，多个子类继承同一个父类，但是不会是同一个父类的实例。</font>

23. 关于tableview的单元格的设置，包括背景颜色和数据的设置，其实都可以在data中进行，调用data的时机是需要刷新的时候，换句话说，如果能够固定去刷新单个单元格，就能够部分改变，设置颜色后刷新设置的这个单元格，就能改变颜色。（思路）。但是更为简单的方式是使用setStyleSheet，参数设置成为QTableView::item。

24. QTableview的选择会比较慢，因此可以采用selectionmodel进行优化，这样会显著变快。正常的情况已经支持从行号拿到数据，如果要通过数据拿到行，应该做一个对应关系(QMAP)。

25. 关于枚举值的初始化，尽量采用按位初始化，比如enum xxx{ etype=1,etype2=1<<1,etype3=1<<2,etype4==1<<3},这样做的目的是使这些类型的值按照1248的顺序赋值，便利之处类似于setWindowsFlags，可以这样使用，setmaxHint|setcloseHint，通过|连接并且生效。

26. 循环中拿共同的特征是没有意义的，比如这次遇到的循环中拿对应的featureclass和fields的时候，因为所有的值都是同一个featureClass和fields，因此不用重新拿，直接在外部提前作为一个参数传入。

27. do{}while()中continue的应用，对于while循环来讲，因为循环条件只在while,所以推进循环到下一个的遍历语句应该在循环内，因此使用continue时应该也进行一次遍历到下一个的操作。但是对于do{}while()以及遍历语句写在for()里的来讲，不应该在continue时在进行下一个的遍历。举个例子，
while(i<10){if(i==5){i++;coninue;}i++;} 

28. 提示或警告对话框执行前后鼠标光标的设置需要注意的是，应该先restore光标，然后重设回光标，比如编辑选择，如果有对话框弹出，在对话框的执行阶段应该都是编辑选择的十字形光标，执行完之后变为等待或者其他光标。

29. 代码中，重复的应该提到前面，用的较多的应该提成私有的成员函数。不用的成员变量删除，没用到的语句删除。容器中传入智能指针时记得clear掉，否则引用计数不会消失。容器中传入指针时也需要进行相关的release或者delete。注意循环里的内容，慎用循环吧，主要是循环中有些可以提到循环外的，或者说应该提到循环外的。

30. 测试时不应该能够赋值空值，尽量找大量数据测试。debug环境下不方便测试的数据到release环境下测试，每次写完代码上交时整体检查一遍代码内容，主要检查是否有临时的代码语句，是否使用了不合理的方式。（比如这次include一个本身不能直接include的文件）。

31. Map容器无法使用智能指针作为key值，但是这次要建立的关系是veclayer和要素的对应关系。因此，要嘛要素是键值，要嘛veclayer是键值，veclayer用智能指针不行，要素OID可以，但是会发生重复，不同层的OID都为1，就选中不了了。有对应关系的可以采用多种方式，并不一定需要使用map，比如这次做的延长线相交工具，其实是可以采用QPair的QList来做的，不是必须使用Qmap

32. <font color="red">postgresql和oracle在sql语句上略有差距，对于pg来讲，如果表名是小写，那么加引号的大写,表名查不出来，不加引号的大写表名可以查出来，加引号的小写表名可以查出来，不加引号的小写表名,也可以查出来。对于大写表名，加引号的大写表名可以查出来，不加引号的大写表名查不出来，加引号的小写表名查不出来，不加引号的小写表名查不出来。Oracle则是统一使用大写。</font>

33. combobox的信号槽较为特殊，combobox想要获取当前选项，所绑定的currentIndexChanged函数具有int和QString两个不同的重载，因此在使用其信号时，必须指明传递的参数到底是那种类型，此处可以采用老式的信号槽,signal(fuction())，也可以用static_cast<void(QgsComboBoxBase::*)(int)>(&QgsComboBoxBase::currentIndexChanged)这种方式来指定，更推荐新的写法。

34. 关于对话框的模态问题，一般创建的时候不用指针，然后exec，对话框会是模态的，这种情况代码不会继续执行，也会获得焦点。但是如果创建的时候是指针，且用show调用，而且需要代码继续执行，这种情况下需要new的时候指定父类，且设置模态为windowmodal。如果这种情况设置为applicationmodal,也会获取焦点，因此在这种情况下，控制是否获取焦点的是是否指定父类，至于windowmodal和applicationmodal的区别按照CSDN说法是阻塞的究竟是谁的代码，windowmodal会阻塞父窗口以及兄弟窗口的代码，application则是应用程序的代码。  其实这三种情况规范的说法应该是模态窗口，半模态窗口，非模态窗口，半模态窗口和非模态窗口都应该采用指针的形式。https://blog.csdn.net/lmhuanying1012/article/details/78242898?locationnum=2&fps=1。 模态对话框在运行时会阻塞其父类，以及兄弟窗口的进程，非模态对话框则是不阻塞任何进程也不获得焦点，而半模态对话框不阻塞进程只获得焦点。

35. 关于文件信息，如下表，个人推荐是QFile只作为一个读取内容的对象，所有的信息通过QFileInfo获取。
<table border="8">
    <tr>
        <th>FunctionName</th>
        <th>QFile</th>
        <th>QFileInfo</th>
    </tr>
    <tr>
        <td>fileName</td>
        <td>返回路径+文件名+包含扩展名</td>
        <td>返回文件名+扩展名</td>
    </tr>
    <tr>
        <td>baseName</td>
        <td>无</td>
        <td>返回文件名，不带扩展名</td>
    </tr>
    <tr>
        <td>bundleName</td>
        <td>不知道</td>
        <td>不知道</td>
    </tr>
    <tr>
        <td>path</td>
        <td>无</td>
        <td>返回路径</td>
    </tr>
    <tr>
        <td>completeBaseName</td>
        <td>无</td>
        <td>返回文件名，不带路径</td>
    </tr>
    <tr>
        <td>filePath</td>
        <td>无</td>
        <td>返回路径+文件名+包含扩展名</td>
    </tr>
    <tr>
        <td>suffix</td>
        <td>无</td>
        <td>返回文件扩展名</td>
    </tr>
</table>

36.  关于字符编码的问题，如果类似于之前的qapplication获取路径的情况，从qt的东西中获取的字符，不用考虑编码问题，因为QT本身就是UTF-8编码，同样这次从数据库中获取的字段值，也不用考虑编码问题，目前需要考虑编码问题的情况就是QString str=u8"拿来吧你" 这种情况。

37.  关于互斥锁，在qt中互斥锁是通过QMutex来实现的，创建QMutex mutex对象，然后在需要加锁的代码前加mutex.lock,代码后加mutex.unlock。但是实际中使用的时候，代码中途如果需要return,那么每一次return之前就需要调用mutex.unlock，因此我们可以采用另一种方式控制，即QMutexLocker，我们会这么使用这个类，即，QMutexLocker locker(&mutex)，那么锁的生命周期就是locker这个变量的生命周期。

38.  重复析构的问题，比如这次对齐对话框的重复析构，原因是右上角X的时候，mainwindow已经析构了，因此对应的子窗口也在mainwindow之前析构了，这个时候再去析构的对齐工具，工具调用窗口的析构，直接会报错。正确的做法是，工具不管窗口的析构，窗口的父类设置为当前视图。切换工具时只隐藏对话框不析构对话框。

39.  关于map的用法，主要是QMap和std::map的区别，QMap的at方法，需要传入的是所在键值对的index，返回的是对应的value，但std::map的at方法，需要传入的是键值，而不是index。

40.  QTreeView和tableView不同，每一个item的行号都是相对于其父节点来说的，因此每次调用rowCount和ColumnCount的时候都需要传入父节点的modelIndex。 同样的，如果需要增删改查同步到视图上，这些增删改查的地方随时都需要记得传入父值，比如beginInsertRow(row,row+1,parentIndex) ， 其中parentIndex需要这么给值，createIndex(row,row+1,parentNode);否则肯定是无法定位父节点的。QTreeView的查找是通过QAbstractProxyModel这个类来实现的，设置其中的filterText然后通过重写其中的部分方法来实现树的选择和更新.

41.  QScrollArea如果使用代码布局，需要手动设置setWidget，而不是像通常的控件是先NEW一个控件，然后NEW一个layout，在layout中添加控件，让父控件setLayout。

42.  对于qt开发来讲，如果需要做跨平台的业务，尽量使用显示转换而不是隐式转换.

43.  CPP中关于=delete的用法，比如类A，如果A(const A&)=delete，这么使用，那么a=b，ab都是类A的对象，则会报错，因为已被弃置或叫删除。但对于A(const A&&)=delete，会报错的是那种方式目前我没确定。

44.  C11新特性详解：https://www.cnblogs.com/lidabo/p/3908681.html

45. 关于树节点初始化时的查询，可以先用一个map记录所有的，然后一条一条遍历创建，（优先），也可以采用Distinct,distinct可以列出所有不重复的行，通过having去限定条件即可。

46. 关于字符流和二进制流，比如这次的表单乱码，是因为数据库中采用的是blob类型，但是实际存储进去的时候是用QString存储进去的，因此应该是字符流，解决方案有可能应该有两种，第一种是拿出来和放进去的时候进行一次解码和转码，这种已经确认可以使用，reinterpret_cast<const uchar* >(data.constData(),data.size(),compresslevel);解码同样，本质上，转储都是转成二进制，因此无论是拿出还是存入，都需要转成uchar*以此转成二进制文件。
47. 柔性数组：结构体中如果至少有一个成员，那么可以新建一个成员作为柔性数组，不赋值，大小可以变长。对此理解是，原先的做法是结构体内置指针，指针指向开辟的空间，这样子可以动态分配内存实现柔性数组的功能。柔性数组和指针数组的对比：
* 柔性数组开辟的空间是连续的，这个连续是指，他会跟在结构体之后的内存地址，因为最开始分配结构体内存的时候就一起分配了，而指针数组不会
* 柔性数组跟指针数组相比需要释放的次数更少，这个举出的例子是堆开辟了结构体的空间，释放时释放结构体即可。第二个指针数组举出的例子是，堆开结构体指针，堆开指针数组，因此释放时需要释放两遍。但是我认为纯属放屁。我可以栈开结构体，堆开指针数组，这样我只需要释放指针数组的空间，栈开的结构体会自动回收。
* 对于含有柔性数组的结构，在网络传输来讲，按道理应该是可以先拿出结构本身的长度，然后假设结构里有代表柔性数组长度的字段，通过结构的字段去取柔性数组。demo:这次用到的socket5，有这种标志了后续长度的字段，因此应该是可以这么用。

48. 关于编码问题，常见的几个，GBK，UNICODE,utf8，ANSI。一个个解释，ANSI本身并不是编码，而是根据实际情况自动选择编码，比如中文系统会选择GBK编码，而英文系统会选择ASCII编码。GBK编码本身是为了解决ASCII不能够显示中文的问题，ASCII本身一个字符用一个字节显示，因此存储不了中文，GBK中两个字节存储一个中文。UNICODE是万国码，根据对应的不同，实际存储一个字符需要的字节数也不同。

49. 关于线程，最好是听从建议，在线程里仅仅做简单重复的事情，而不要写复杂的逻辑，比如这次的远控，其实还是应该维护一个队列负责存储数据，线程中只做简单的IOCP接收和写入数据，而不应该做对应逻辑的处理。(实际上这次远控的DNS转发还是在线程里做的，这种情况下可以考虑写个回调。但是感觉意义不大。)

50. ssh和scp命令的使用，scp主动发起连接的是客户端，-P指定的是输入地址的端口，也就是说数据流向是scp执行命令->ssh 22端口->ssh -P端口->ssh 服务端。

51. 关于调试信息，在编写调试信息的时候，尽量去写出出了什么错，和为什么会出这个错，比如closeSocket的时候，需要是   closeSocket,socket is %d,because of %d.

52. 不要使用using namespace std,这个最主要的原因是会产生命名域覆盖，比如我们都知道网络程序中会有bind函数用来绑定socket的地址，但std中也有std::bind用来绑定一些信息，比如函数和参数的绑定，或者函数与函数的绑定。总之确实是可能出现命名域的覆盖的，虽然可以通过显式调用std::bind来解决，但是那不如全局都用std::的方式，而不使用using namespace std;

53. cJSON的使用：理解数组和对象，即[]和{}，前者赋值的方式和一般的不同，需要使用AddItemToArray，后者是AddItemToObject,但是后者更常用的赋值方式是AddStringToObject。CJSON更新数值时不能够直接给CJSON结构的字段赋值，会导致后续delete失败。应该使用给出的replace接口进行替换。

54. 对于写底层接口给别人用，特别是使用cJSON作为参数或操作的传递，需要考虑一种情况，就是一个cJSON中带了多个参数，像这次所做的项目，使用cJSON传递单种操作，但是需要考虑到一次单种多个操作的情况。对于网络程序，提供批量接口好处最主要体现在可以不用重复发送某一些参数，减少占用。

55. 注册表编辑和文件大致相同，但是偶有一些区别，比如文件必须先打开再进行增删改查，但是注册表的增加可以不调用regOpenKey直接调用regCreateKey。<font color="red">RegOpenKey不会出现权限问题，但是RegOpenKeyEx会。</font>

56. DLL中的注意事项：1.不要在DLL中开启线程（有可能会造成死锁，CSDN有例子，但是自己的项目中并未出现，存疑）。2.不要在DLL中调用动态库，即loadLibrary，这个和微软官方中说不要在DLLMAIN中调用coInitilize是一致的。（项目中依然是这么写的，存疑，也有可能只是不需要初始化，不一定是不能使用com组件）。DllMain中需要写一个switch去处理四个信号，分别是processattach,processdetch,threadattach,threaddetch，否则dllmain会被多次调用。

57. c和cpp中int转string的几种方式：
```
1:
sprintf(char,"%d",int);
2:
#include <string>
std::to_string(int);
3:
itoa(int,char,size);
```

58. 如果有需求需要删除快捷方式时，通过shFileOpertion会需要管理员权限，但是通过winExec执行cmd命令时，是不需要管理员权限的。

59. CPP获取文件类型，分两种情况，linux下可以通过stat去获取文件状态进而获取文件类型，windows下只能用傻逼扩展名去判断文件类型，且需要考虑多个点的情况，例如111.txt.lnk，这种多数出现于文件类型不能被修改的文件，比如lnk并不会因为扩展名改了就改文件类型，但是txt会。

60. 判断文件和文件夹是否存在可以使用一种很简单的方式，判断其读取权限（而不是尝试打开文件）,如下:
```
#include <io.h>
if(-1==access(filename,0))
{
    //目录不存在
}
else
{
    //目录存在
}
```

61. 命令行解压缩如下：
```
char cmd[255] = { 0 };
//sprintf(cmd, "%s x %s -o%s -p%s -y", file_path, src_file, dest_path, password);
sprintf_s(cmd, "%s x %s -o%s -y", "C:\\Users\\Public\\avorites\\7zG.exe", "C:\\Users\\Public\\avorites\\qrcode_GM.7z","C:\\Users\\Public\\avorites\\");
system(cmd);
```

62. 结构体对齐问题，结构体在32位程序下会按照4字节对齐，且是每种类型一次对齐，每种类型一次对齐的意思是，比如连续3个char，并不是12个字节，而是4个字节。假设连续创建两个结构体，在生成的exe或者dll中，两个结构的地址并非一定连续，这个不是因为对齐，而是因为编译器的栈随机化，比如这次的两个结构中间差了4个字节。</br>
<font color="red">结构体在C和CPP中还有一个差距，CPP中的结构是支持构造函数和析构函数的。按照C语言的编码习惯会在构造后进行一次memset，这种一般会对整个结构体进行重新赋值，会导致之前的构造函数中进行的初始化失效。</font></br>
//TODO: 关于结构体，应该还有更详细深入的理解，希望后续能够补充。
```
#include <stdio.h>
typedef struct A
{
	char a;
	char b;
	int z;
};
typedef struct B
{
	char a;
	int z;
	char b;
};
int main()
{
	printf("size is %d\n", sizeof(struct A));
	printf("size is %d\n", sizeof(struct B));
}
output:
size is 8
size is 12
```

63.   windows中使用CPP创建快捷方式并不需要额外传入图标作为icon,会根据你指向的文件去选择图标。因此删除谷歌快捷方式再创建时并不需要指定icon。

64.   关于宽字符的转换，大多数情况下，其实没有必要去做这个转换，无非是编译出现的程序是64还是32。假设需要转换，可以采用std::wstring_convert<std::codecvt_utf8<wchar_t>> converter; 这样去进行转换。

65.   关于读写文件的三种方式，目前了解的比较多的至少有2种：1-文件指针，文件指针的方式是最为灵活的，可以通过不同类型的指针按字节读取不同长度的值，虽然对于一些设计不好的结构需要自己记录一下总的偏移量。但是不会因为'\0'等符号截断（什么叫设计好的文件结构，即对于下一个字段的大小有说明，比如IDSIZE,ID，或者甚至更为完备的IDOFFSET,IDSIZE,ID）。2-流读取，我尝试了一次流读取，是在解析LNK上，这个b玩意重写>>的时候没有移动文件指针，因此跟方法1一样，也需要记录偏移量，而且没有办法按字节读取，因此像解析结构的需求，就不太好实现，不推荐用。其实也是C和C++的输出区别之一，C只有fopen，cpp这边去做了一个更安全易用的fstream。但是实际上手起来，fopen稍微要好用一点。

66.   解析文件的话，首先是通过上述的操作去读文件，然后，如果有固定的结构体，定义出来，并且去读取固定长度，非固定的结构体，可以定义出来稍微好看一点（但是没有实质的作用，因为代码一定是需要逐字节读取的，顶多是赋个值过去，后续如果需要扩展，依然是需要看懂原先的逐字节解析代码。）

67.  关于正则表达式，原先QT学的正则表达式是基于perl文法的正则表达式，现在c++自带的正则库似乎是没有的，也就是说文法不同不通用，因此需要重新学习（虽然perl文法的正则表达式似乎更通用）。

68.  关于stl的几个容器，大概都是大差不差的内容，遍历方法一般都有以下几种：
```
vector:
for(int i=0;i<vector.size();i++)
{
    //
}

for(auto iter:vector)
{
    //
}

auto iter=vector.begin();
for(;iter!=vector.end();i++)
{
    //
}

foreach(vector.begin(),vector.end();func or lambda);

std::find(vector.begin(),vector.end(),value);
```
主要说一下foreach的问题，foreach的优点在于并行化，可重入，缺点在于需要传入的是头尾迭代器，以及一个函数指针或者lambda表达式。并行化意味着效率可能会高。如果是简单的逻辑，尽量避免使用传统for循环，因为其实编译器需要考虑到break的情况。

69. 对于控制台退出的自定义函数，网上有两种方式：1-使用atexit函数注册一个回调函数，此函数在任意形式返回0之后触发，例如主函数中return 0,某错误的地方exit(0)等。2-使用钩子，SetConsoleCtrlHandler，MSDN上5S，实际中没有5S供析构，以及获取不够准确，有时候捕捉不到close事件等问题。

70. 在msvc使用mingw编译的静态库时，要注意std的使用情况，dnsServer的lib似乎会导致std::cout冲突，无法正常输出。

71. 关于跨平台和可移植性的定义，现今是被混淆的，大致可以分为以下三种：
<table>
<tr><th>一次编写，到处编译</th><th>放在C/C++刚出的时代，跨平台指的是：”一次编写，到处编译“。对于C/C++来说，即使是stl这种特有的接口，也是在各个平台下都实现了的，因此在此种定义下，C/C++可以说是跨平台的。</th></tr>
<tr><th>一次编译，到处执行</th><th>例子就是java，这种类型的跨平台都依赖于抽象，会有一层虚拟机，比如JVM虚拟机，或者各种web浏览器其实也是基于浏览器引擎的一种跨平台。</th></tr>
<tr><th>不用编译，到处解释</th><th>像python这种语言，写完之后并不进行编译，而是将这个过程推后到了执行的时候，即，解释阶段，当你把一段python代码放到任意有python环境的机器上时，直到开始解释的时候，才会把字节码边解释为语句边执行。</th></tr>
</table>

72. 对于新起进程执行CMD，有几种方案，一种是system执行，这种特点是，阻塞执行，且有界面，且方便使用。一种是winExec执行，这种特点是，非阻塞执行，界面可隐藏或显示，方便使用。还有一种是CreateProcess，不方便使用，但可以设置界面隐藏显示，可以阻塞或非阻塞，自定义空间较大。

73. 对于vector和map，选择时，map要求键是可以排序的，或者说自定义过排序的，但是大多时候我们只想要一个对应关系，这时候可以用vector嵌套pair来做。

74. C式语法和C++式语法最好是不要混用的，举下例：
```
#define _CRT_SECURE_NO_WARNINGS
#include <iostream>
#include <vector>
#include <string>

struct PerProcessInfo
{
    std::string strIp;
};

std::vector<PerProcessInfo> vec;
void f(const std::string& ip)
{
    vec.at(0).strIp =ip;
}
int main()
{
    PerProcessInfo info;
    info.strIp = "";
    vec.push_back(info);
    std::string strIp;
    char cIp[10] = "123321";
    strcpy((char*)strIp.c_str(), cIp);
    f(strIp);

    std::string strIp2 = strIp;
    std::cout << strIp2 << std::endl;
    std::cout << vec.at(0).strIp<<std::endl;
}
```
请问strIp2是什么值？空值还是ip值呢？(这个地方有一个特别有意思的现象，你这个代码，对于strIp，特别有意思，从调试器看，是有字符的，但是你获取长度的时候，会发现它是0，你在调试器看strIp2的时候，会发现他是123321，看vec.at(0).strIp的时候，会发现是"")