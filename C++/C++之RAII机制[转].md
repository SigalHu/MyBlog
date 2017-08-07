原文：[C++中的RAII机制](http://www.jellythink.com/archives/101)

#### 什么是`RAII`？

`RAII`是`Resource Acquisition Is Initialization`的简称，是C\+\+语言的一种管理资源、避免泄漏的惯用法。利用的就是C\+\+构造的对象最终会被销毁的原则。`RAII`的做法是使用一个对象，在其构造时获取对应的资源，在对象生命期内控制对资源的访问，使之始终保持有效，最后在对象析构的时候，释放构造时获取的资源。

#### 为什么要使用`RAII`？

在计算机系统中，资源是数量有限且对系统正常运行具有一定作用的元素。比如：网络套接字、互斥锁、文件句柄和内存等等，它们属于系统资源。由于系统的资源是有限的，所以，我们在编程使用系统资源时，都必须遵循一个步骤：

1. 申请资源
2. 使用资源
3. 释放资源

第1步和第3步缺一不可，因为资源必须要申请才能使用的，使用完成以后，必须要释放，如果不释放的话，就会造成资源泄漏。

一个最简单的例子：
```cpp
#include <iostream>
using namespace std;

int main() {
    int *testArray = new int[10];
    // Here, you can use the array
    delete[] testArray;
    testArray = NULL;
    return 0;
}
```
我们使用`new`开辟的内存资源，如果我们不进行释放的话，就会造成内存泄漏。所以，在编程的时候，`new`和`delete`操作总是匹配操作的。如果总是申请资源而不释放资源，最终会导致资源全部被占用而没有资源可用的场景。但是，在实际的编程中，我们总是会各种不小心的就把释放操作忘了，就是编程的老手，在几千行代码，几万行中代码中，也会犯这种低级的错误。

再来一个例子：
```cpp
#include <iostream>
using namespace std;

bool OperationA();
bool OperationB();

int main() {
    int *testArray = new int[10];

    // Here, you can use the array
    if (!OperationA()) {
        // If the operation A failed, we should delete the memory
        delete[] testArray;
        testArray = NULL;
        return 0;
    }

    if (!OperationB()) {
        // If the operation A failed, we should delete the memory
        delete[] testArray;
        testArray = NULL;
        return 0;
    }

    // All the operation succeed, delete the memory
    delete[] testArray;
    testArray = NULL;
    return 0;
}

bool OperationA() {
    // Do some operation, if the operate succeed, then return true, else return false
    return false;
}

bool OperationB() {
    // Do some operation, if the operate succeed, then return true, else return false
    return true;
}
```
上述这个例子的模型，在实际中是经常使用的，我们不能期待每个操作都是成功返回的，所以，每一个操作，我们需要做出判断，上述例子中，当操作失败时，然后，释放内存，返回程序。上述的代码，极度臃肿，效率下降，更可怕的是，程序的可理解性和可维护性明显降低了，当操作增多时，处理资源释放的代码就会越来越多，越来越乱。如果某一个操作发生了异常而导致释放资源的语句没有被调用，怎么办？这个时候，`RAII`机制就可以派上用场了。

#### 如何使用`RAII`？

当我们在一个函数内部使用局部变量，在退出这个局部变量的作用域时，这个变量也就被销毁了；当这个变量是类对象时，这个时候，就会自动调用这个类的析构函数，而这一切都是自动发生的，不要程序员显示的去调用完成。`RAII`就是这样去完成的。由于系统的资源不具有自动释放的功能，而C\+\+中的类具有自动调用析构函数的功能。如果把资源用类进行封装起来，对资源操作都封装在类的内部，在析构函数中进行释放资源。当定义的局部变量的生命结束时，它的析构函数就会自动的被调用，如此，就不用程序员显示的去调用释放资源的操作了。现在，我们就用`RAII`机制来完成上面的例子。代码如下：
```cpp
#include <iostream>
using namespace std;

class ArrayOperation {
public :
    ArrayOperation() {
        m_Array = new int[10];
    }

    void InitArray() {
        for (int i = 0; i < 10; ++i) {
            *(m_Array + i) = i;
        }
    }

    void ShowArray() {
        for (int i = 0; i < 10; ++i) {
            cout << m_Array[i] << endl;
        }
    }

    ~ArrayOperation() {
        cout << "~ArrayOperation is called" << endl;
        if (m_Array != NULL) {
            delete[] m_Array;  // 非常感谢益可达非常犀利的review，详细可以参加益可达在本文的评论 2014.04.13
            m_Array = NULL;
        }
    }

private :
    int *m_Array;
};

int main() {
    ArrayOperation arrayOp;
    arrayOp.InitArray();
    arrayOp.ShowArray();
    return 0;
}
```
上面这个例子没有多大的实际意义，只是为了说明`RAII`的机制问题。下面说一个具有实际意义的例子：
```cpp
/*
** FileName     : RAII
** Author       : Jelly Young
** Date         : 2013/11/24
** Description  : More information, please go to http://www.jellythink.com
*/

#include <iostream>
#include <windows.h>
#include <process.h>
using namespace std;

CRITICAL_SECTION cs;
int gGlobal = 0;

class MyLock {
public:
    MyLock() {
        EnterCriticalSection(&cs);
    }

    ~MyLock() {
        LeaveCriticalSection(&cs);
    }

private:
    MyLock(const MyLock &);
    MyLock operator=(const MyLock &);
};

void DoComplex(MyLock &lock) {
}

unsigned int __stdcall ThreadFun(PVOID pv) {
    MyLock lock;
    int *para = (int *) pv;

    // I need the lock to do some complex thing
    DoComplex(lock);

    for (int i = 0; i < 10; ++i) {
        ++gGlobal;
        cout << "Thread " << *para << endl;
        cout << gGlobal << endl;
    }
    return 0;
}

int main() {
    InitializeCriticalSection(&cs);

    int thread1, thread2;
    thread1 = 1;
    thread2 = 2;

    HANDLE handle[2];
    handle[0] = (HANDLE) _beginthreadex(NULL, 0, ThreadFun, (void *) &thread1, 0, NULL);
    handle[1] = (HANDLE) _beginthreadex(NULL, 0, ThreadFun, (void *) &thread2, 0, NULL);
    WaitForMultipleObjects(2, handle, TRUE, INFINITE);
    return 0;
}
```
这个例子可以说是实际项目的一个模型，当多个线程访问临界变量时，为了不出现错误的情况，需要对临界变量进行加锁；上面的例子就是使用的Windows的临界区域实现的加锁。但是，在使用`CRITICAL_SECTION`时，`EnterCriticalSection`和`LeaveCriticalSection`必须成对使用，很多时候，经常会忘了调用`LeaveCriticalSection`，此时就会发生死锁的现象。当我将对`CRITICAL_SECTION`的访问封装到`MyLock`类中时，之后，我只需要定义一个`MyLock`变量，而不必手动的去显示调用`LeaveCriticalSection`函数。

#### 使用`RAII`的陷阱

在使用`RAII`时，有些问题是需要特别注意的。先举个例子：
```cpp
#include <iostream>
#include <windows.h>
#include <process.h>
using namespace std;

CRITICAL_SECTION cs;
int gGlobal = 0;

class MyLock {
public:
    MyLock() {
        EnterCriticalSection(&cs);
    }

    ~MyLock() {
        LeaveCriticalSection(&cs);
    }

private:
    //MyLock(const MyLock &);
    MyLock operator=(const MyLock &);
};

void DoComplex(MyLock lock) {
}

unsigned int __stdcall ThreadFun(PVOID pv) {
    MyLock lock;
    int *para = (int *) pv;

    // I need the lock to do some complex thing
    DoComplex(lock);

    for (int i = 0; i < 10; ++i) {
        ++gGlobal;
        cout << "Thread " << *para << endl;
        cout << gGlobal << endl;
    }
    return 0;
}

int main() {
    InitializeCriticalSection(&cs);

    int thread1, thread2;
    thread1 = 1;
    thread2 = 2;

    HANDLE handle[2];
    handle[0] = (HANDLE) _beginthreadex(NULL, 0, ThreadFun, (void *) &thread1, 0, NULL);
    handle[1] = (HANDLE) _beginthreadex(NULL, 0, ThreadFun, (void *) &thread2, 0, NULL);
    WaitForMultipleObjects(2, handle, TRUE, INFINITE);
    return 0;
}
```
这个例子是在上个例子上的基础上进行修改的。添加了一个`DoComplex`函数，在线程中调用该函数，该函数很普通，但是，该函数的参数就是我们封装的类。你运行该代码，就会发现，加入了该函数，对`gGlobal`全局变量的访问整个就乱了。

由于`DoComplex`函数的参数使用的传值，此时就会发生值的复制，会调用类的复制构造函数，生成一个临时的对象，由于`MyLock`没有实现复制构造函数，所以就是使用的默认复制构造函数，然后在`DoComplex`中使用这个临时变量。当调用完成以后，这个临时变量的析构函数就会被调用，由于在析构函数中调用了`LeaveCriticalSection`，导致了提前离开了`CRITICAL_SECTION`，从而造成对`gGlobal`变量访问冲突问题，如果在`MyLock`类中添加以下代码，程序就又能正确运行：
```cpp
MyLock( const MyLock & temp ) {
    EnterCriticalSection(&cs);
}
```
这是因为`CRITICAL_SECTION`允许多次`EnterCriticalSection`，但是，`LeaveCriticalSection`必须和`EnterCriticalSection`匹配才能不出现死锁的现象。

为了避免掉进了这个陷阱，同时考虑到封装的是资源，由于资源很多时候是不具备拷贝语义的，所以，在实际实现过程中，`MyLock`类应该如下：
```cpp
class MyLock {
public:
	MyLock() {
		EnterCriticalSection(&cs);
	}

	~MyLock() {
		LeaveCriticalSection(&cs);
	}

private:
	MyLock(const MyLock &);
	MyLock& operator =(const MyLock &);
};
```
这样就防止了背后的资源复制过程，让资源的一切操作都在自己的控制当中。如果要知道复制构造函数和赋值操作符的调用，可以好好的阅读一下《深度探索C++对象模型这本书》。

#### 总结

`RAII`的本质内容是用对象代表资源，把管理资源的任务转化为管理对象的任务，将资源的获取和释放与对象的构造和析构对应起来，从而确保在对象的生存期内资源始终有效，对象销毁时资源一定会被释放。说白了，就是拥有了对象，就拥有了资源，对象在，资源则在。所以，`RAII`机制是进行资源管理的有力武器，C\+\+程序员依靠`RAII`写出的代码不仅简洁优雅，而且做到了异常安全。
