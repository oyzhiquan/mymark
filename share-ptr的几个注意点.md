---
title: share_ptr的几个注意点
date: 2019-12-27 09:24:52
tags:程序开发
---

[转载](https://cppfans.org/1641.html)

智能指针在boost中很早就有了，在tr1上也很早，但是没怎么用，后来0x标准出来之后，智能指针变成了标准库，所以现在用起来就不区分boost和std了。

主要说下share_ptr的几个注意点，待补全。

1.环状的链式结构可能会形成内存泄露

例如：

```
class BaseClass;
class ChildClass;

typedef std::shared_ptr<BaseClass> BaseClassPtr;
typedef std::shared_ptr<ChildClass> ChildClassPtr;

class BaseClass
{
public:
    ChildClassPtr childClass;
protected:
private:
};

class ChildClass
{
public:
    BaseClassPtr baseClass;
protected:
private:
};

int _tmain(int argc, _TCHAR* argv[])
{
    BaseClassPtr base(new BaseClass());
    ChildClassPtr child(new ChildClass());

    base->childClass = child;
    child->baseClass = base;


    system("pause");
    return 0;
}
```

2.多线程环境下使用代价更大。

因为share_ptr内部有两个数据成员，一个是指向对象的指针 ptr，另一个是 ref_count 指针，指向堆上的 ref_count 对象，读写操作不能原子化，（具体的结构图可以查看 [陈硕的文章《为什么多线程读写 shared_ptr 要加锁？》](http://blog.csdn.net/solstice/article/details/8547547)）所以多线程下要么加锁，要么小心翼翼使用share_ptr。

例如：

```
class Test
{
public:
    Test() {}

    ~Test() {}

    // ...
protected:
private:
};

void func(std::shared_ptr test_ptr)
{
    // 大量使用test_ptr

    std::shared_ptr temp_ptr = test_ptr;
}

int _tmain(int argc, _TCHAR* argv[])
{
    std::shared_ptr sp(new Test());
    boost::thread th1(std::bind(&func, sp));
    boost::thread th2(std::bind(&func, sp));

    th1.join();
    th2.join();

    return 0;
}
```

上面的代码不知道什么时候可能就宕了，而且不容易找到问题，这个时候你就得硬看代码了。

你也可以通过使用weak_ptr来解决这个问题，例如上述例子可以修改为：

```
class Test
{
public:
    Test() {}

    ~Test() {}

    // ...
protected:
private:
};

void func(std::weak_ptr test_ptr)
{
    // 大量使用test_ptr

    std::weak_ptr temp_ptr = test_ptr;
}

int _tmain(int argc, _TCHAR* argv[])
{
    std::shared_ptr sp(new Test());
    std::weak_ptr wp(sp);

    boost::thread th1(std::bind(&func, wp));
    boost::thread th2(std::bind(&func, wp));

    th1.join();
    th2.join();

    return 0;
}
```

weak_ptr是一种可构造可赋值的不增加引用计数来管理share_ptr的智能指针，它可以非常方便的通过weak_ptr.lock()转为share_ptr，通过weak_ptr.expired()来判断智能指针是否被释放，还是非常方便的。条目1中的例子使用weak_ptr就可以解决问题

3.share_ptr包装this的时候使用enable_shared_from_this

```
class Test
{
public:
    Test() {}

    ~Test() {}

    std::shared_ptr get_ptr()
    {
        return std::shared_ptr(this);
    }

    // ...
protected:
private:
};

int _tmain(int argc, _TCHAR* argv[])
{
    Test t;
    std::shared_ptr t_ptr(t.get_ptr());
    return 0;
}
```

这样就会发生析构两次的问题，可以使用enable_shared_from_this来做共享，上面例子修改为

```
class Test : public std::enable_shared_from_this<Test>
{
public:
    Test() {}

    ~Test() {}

    std::shared_ptr get_ptr()
    {
        return shared_from_this();
    }

    // ...
protected:
private:
};

int _tmain(int argc, _TCHAR* argv[])
{
    std::shared_ptr t_ptr(new Test());
    t_ptr->get_ptr();

    return 0;
}
```

4.share_ptr多次引用同一数据会导致内存多次释放

```
int _tmain(int argc, _TCHAR* argv[])
{
    int* int_ptr = new int[100];
    std::shared_ptr s_int_ptr1(int_ptr);

    // do something

    std::shared_ptr s_int_ptr2(int_ptr);

    return 0;
}
```

而且C++之父对share_ptr的初衷是：“shared_ptr用于表示共享拥有权。然而共享拥有权并不是我的初衷。在我看来，一个更好的办法是为对象指明拥有者并且为对象定义一个可以预测的生存范围。”

 

总结：智能指针虽然好，但是还需要谨慎的使用，C++的新特性很不错，但是写C++代码还是得非常注意，各位共勉吧