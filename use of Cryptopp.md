# 函数用法
1. StringSource
```
/// \param 传入的第一个参数是需要进行操作的字符串，
/// \param 传入的第二个参数是需要操作的字符串大小
/// \param 传入的第三个参数是是否将所有数据立刻进行转换（？存疑）
/// \param 实际需要进行的操作，此处这种传入方法是转换编码，后面解密也是用另一种方式传入解密器
StringSource((BYTE*) EncryptValue.c_str(),EncryptValue.size(),true,new HexEncoder(new StringSink(Decoded)));
```
