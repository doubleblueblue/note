# 交叉编译
## MINGW的static_a在MSVC使用
* 要使mingw编译出来的库在msvc可以使用，需要补充其他两个文件，libmingwex.a以及libgcc.a。两者分别是从mingw的lib目录下可找到，以及lib\gcc。然后在要引用的头文件中，比如lib.h中，将这两个文件使用静态引用的方式引用进即可。
## MINGW的dll在msvc使用
* DLL本身没什么影响，可以尝试直接使用。