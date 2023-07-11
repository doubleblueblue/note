1. 错误代码：只需要记住返回为void的是系统认为不会出错的函数，同时我们编码也应该遵循这个规范。非void的基本都有可能出错，并且返回NULL。

2. VS调试技巧，可以通过在监视窗口加入$err,hr来实时监视windows错误代码。

3. WINDOWS API是否使用unicode主要取决于你项目设置的字符集,如果设置为unicode，对应的API的默认版本就是unicode，至于W和A版本则分别还是unicode和ansi版本。

4. 对于字符数组，或者其他类型的数组，尽量使用_countof代替sizeof

5. 