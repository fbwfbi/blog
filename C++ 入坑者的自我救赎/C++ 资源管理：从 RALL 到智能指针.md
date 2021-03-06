# C++ 资源管理：从 RALL 到智能指

C++ 比 Java、C# 麻烦的一个方面在于内存管理。Java、C# 都自带垃圾回收机制，不用程序员自己管理内存。而 C++ 需要程序员手动管理内存。这是 C++ 灵活的地方，灵活自然也是有代价的。我们可以使用一些技巧，让 C++ 的内存管理变得轻松一些，虽然不会像 Java 那么轻松，但这些技巧能大大减轻我们的负担。而且这些技巧也可以用于管理其他资源，包括网络连接、锁、数据库连接、文件描述符等等。

>   TR1：一份规范，描述加入 C++ 标准程序库的诸多新功能。这些功能以新的 class templates 和 function templates 形式提供。
>
>   Boost：一个 C++ 开发者集结的社群，提供高质量、源码开放、平台独立、编译器独立的程序库。和 C++ 标准委员会关系密切。

## 用对象管理资源、RALL

假如有如下一段代码：

```c++
void do_something()
{
    int *p = new B();
   // ...do somethings
   // ...
   delete p;
}
```

像不像你本科时写的 C++ 代码。看上去好像没毛病，函数的开头申请了一个 B 对象，最后也没忘记销毁了它，肯定没内存泄漏。但是这里有一个隐患，如果中间的代码执行了一个 return 或者调用的某个函数触发的异常，使得这个函数执行到一半就退出了，`delete p` 就没机会执行了，于是内存泄漏了。

C++ 果然变态，一不小心就内存泄漏了。于是饱受折磨的前辈们想出了用对象管理资源的方法。回忆一下 C++ 的局部变量，当代码执行离开一个局部作用域时，属于该作用域的局部变量都会被销毁。既然这样，我们可以将资源和一个局部变量的生命周期绑定在一起。让局部对象代替我们进行资源的释放不就可以了。代码如下：

```c++
class Handler
{
public:
  Handler(B *p)
  {
      this->p = p;
  }
  
  ~Handler()
  {
      if(p)
      {
          delete p;
      }
  }
 private:
  B *p;
}

void do_something()
{
    int *p = new B();
    Handler handler(p);
   // ...do somethings
   // ...
}
```

在上面的代码中，我们定义了一个资源管理类 Handler。并在原来的函数中用局部变量 handler 绑定了指针 p。这样我们就不用自己去 delete p 了。代码执行离开该函数时，不管是中途 return，还是抛出异常，都会清理局部变量 handler，会调用 handler 的析构函数，而我们在 handler 的析构函数中释放了指针 p 指向的内存。这样就达到了自动清理资源的目的。

是不是很完美？但你可能会问我们需要为每个类型的资源都定义一个对应的资源处理器吗？比如上面的 Handler 是针对类型 B 定义的。这样且不是很麻烦。聪明如你，肯定已经想到将 Handler 定义成模板类就能完美的解决这个问题。

还有一个小问题。我们观察一下上面示例的这两行代码：

```c++
int *p = new B();
Handler handler(p);
```

B 类对象资源的申请和托管给资源管理类 handler 是分开的。这里其实有个小隐患，如果你习惯了这样先申请资源，再托付给管理器。如果你在这两个步骤间不小心将申请到的资源用作它途是存在安全隐患的。比如：

```c++
int *p = new B();
// ... 这里我将 p 不小心用作了它途
some_func(p);
//... p 好像被 some_func 释放了
Handler handler(p); // 现在托管给 handler 的是无效指针了
```

虽然不一定会发生这种事，但可能性是存在的。既然已经决定将资源的管理托管给资源管理类对象，那就彻底点，将资源的申请和托管合成一步就没这隐患了。所以我们将代码改成如下：

```c++
Handler handler(new B());
```

我们在获取资源 B 的同时初始化资源管理类对象。这就是 RALL。

>   下面引用《Effective C++》的一段关于 RALL 的解释：“RALL:资源的获取时机便是初始化时机”。
>
>   也就是他们常说的“获取及初始化”。
>
>   初始化指的是资源管理类对象的初始化。
>
>   关于 RALL 更详细的解释：在资源管理类对象的构造函数中申请资源，在资源管理类的析构函数中释放资源。

OK，我们已经知道了 RALL 这个“高大上”的概念，在和别人吹牛之前，我们先完成我们的资源管理类的初始版本。记得改掉上面说的两个问题。

```c++
// 资源管理类 1.0
#include <iostream>
using std::cout;
using std::endl;

class B
{
public:
	~B()
	{
		cout << "我是 B 的析构函数" << endl;
	}
};

template <class T>
class Handler
{
public:
	Handler(T *p) :_p(p){}

	~Handler()
	{
		delete _p;
	}

private:
	T *_p;
};


int main()
{
	Handler<B> handler(new B);

	return 0;
}
```

我们在这个 1.0 版本中用模板实现了资源管理类，并将资源的获取和资源管理类对象的初始化合成一步。并添加了测试代码。在管理类的析构函数中输出打印语句，测试资源是否被正确的销毁。测试结果为：

```shell
我是 B 的析构函数
```

OK，没毛病。

我们已经知道了怎么用对象来管理资源（目前只是内存资源），知道了 RALL 的概念。是时候走的更远了。

