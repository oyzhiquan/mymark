C++ 慎用无符号

```c++
int main()
{
	unsigned long long xx = 666666666666666654;
    unsigned long long yy = 666666666666666655;
    
    if ((xx - yy) > 0)
    {
        cout << "判断错误" << endl;
    }
    return;
}
```

在Windows上运行上述代码，输出“判断错误”。

原因：

无符号数的相减，实际结果是负数，但内存中的数是undefined。