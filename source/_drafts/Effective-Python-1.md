---
title: Effective Python - 1
tags:
- Python 
categories: Effective Python
---

## 0. 系列前言
《Effective Python》是某阳推荐的，他所在部门的必读书籍，虽然我不用Python进行一些大型应用的开发，但是作为一个热爱Python的脚本小子，我对这本书也非常感兴趣。最新第二版今年刚出，还热乎着，还比第一版多了整整31条建议，这不血赚？

这本书的子标题叫90 Specific Ways to Write Better Python，中译名叫编写高质量Python代码的90个有效方法，一共有10章，打算写5篇左右的文章学习记录一下，每篇覆盖2章内容。

## 1. 培养Pythonic思维

### 1.1 查询自己使用的Python版本
第一条居然是查版本，不会吧不会吧，不会真的有人不知道怎么查版本吧？

咳咳，其实我理解作者的用意主要还是在介绍整本书的环境基础，整本书的代码示例是基于Python 3.7的语法并介绍一些Python 3.8的新特性。

在撰写现在这篇博文的时候，Python 3.10刚发布不久，它是在2021-10-04发布的，并且久违地引入了新的语法特性——Structural Pattern Matching，后面有时间的话我会读一读[PEP-634 -- Structural Pattern Matching: Specification](https://www.python.org/dev/peps/pep-0634/)、[PEP 635 -- Structural Pattern Matching: Motivation and Rationale](https://www.python.org/dev/peps/pep-0635/)和[PEP 636 -- Structural Pattern Matching: Tutorial](https://www.python.org/dev/peps/pep-0636/)并对这个新特性做一定的分析。

### 1.2 遵循PEP-8风格指南
这一条的内容是关于[PEP-8](https://www.python.org/dev/peps/pep-0008/)代码规范的，其实PEP-8也是我特别喜欢的一条PEP（Python Enhancement Proposals），它以最权威的方式定义了Python的代码风格指南，使所有的代码至少看上去是Pythonic的，顺便这里需要吹一波Pycharm，让我在没有仔细读过PEP-8的情况下，也对PEP-8的细节知道得大概了，毕竟作为轻微强迫症，看到黄色波浪线就会十分不爽。当然，类似的工具应该还有Pylint，常用VS Code的同学可能接触得比较多吧。

但是读的时候发现关于命名规范还是有一点不太清楚的，主要还是由于我没用过Python写过大型的面向对象项目，很少去实现一些类，所以忽略了实例属性（发现书中在这里称其为实例属性，顺手就要写成类字段或者类属性了，但是仔细想想可能不对，如果是类属性的话，那就是属于类对象本身的属性了，存在比较大的差异）的命名规范，PEP-8建议了受保护的实例属性用一个下划线开头而私有的实例属性用两个下划线开头，也就是`foo._protected`和`bar.__private`。

其实不太清楚Python的受保护属性和私有属性是怎么样的一个机制，毕竟之前碰到过一大堆Python沙箱逃逸，也没碰到有说啥私有属性不能访问之类的呀，所以粗略猜测Python并不存在像C++那样的语言语法层面的限制，而是作为一种逻辑约束。

PS: 看了这篇[Stack Overflow的帖子](https://stackoverflow.com/questions/1641219/does-python-have-private-variables-in-classes)，所以确实是一种逻辑上的约束。

### 1.3 了解bytes和str的区别
这条也比较有意思，从我个人角度来说的话，由于在CTF中经常会写脚本处理一些编码相关的问题，不论是Web还是Crypto还是Pwn还是Misc还是Reverse，根据出题人的脑洞，都会或多或少地接触到编码的转换，其中最常见也最基本的就是base64编码。  

虽然书上没有提这块内容在Python 2和Python 3上的差异，但是我觉得这个有必要在这展开一下。  

在Python 3里面一个比较令人头疼的地方是，会出现大量使用`encode`和`decode`的情况，比如
```base64.b64decode(foo.encode())).decode()```和```base64.b64encode(bar.encode())).decode()```
这里面涉及到的就是bytes和str之间的相互转化，刚开始接触到这种转换的时候那真是被绕晕了，不像Python 2里面那样直接`base64.b64encode(foo)`和`base64.b64decode(bar)`就行了。

我认为Python 3这么做的原因其实很简单，那就是更好地支持本地化，Python 3程序需要把解码和编码操作放在程序界面的最外层进行，就是为了把输入文本的编码和内部数据的编码区分开来。

Python 2时代，如果直接输入非Ascii字符的字符串，会自动根据当前编码转换成字节存储，例如`'你好'`在OS默认编码为UTF-8的情况下会直接变成`'\xe4\xbd\xa0\xe5\xa5\xbd'`，那么随之而来的问题其实已经可以想到了，万恶的GBK又要出来了，Windows的默认编码应该是GBK或者GB2312（这俩家伙的差异俺也不太清楚，开个坑先），如果在不同平台接受输入就需要进行转换，就需要使用Python 2的unicode类型来处理。

正因如此，Python 2的程序在编写时出现了不一致的地方，处理相同逻辑，会因跨平台的需求，有的时候需要做处理，有的时候不需要做处理，所以Python社区直接决定：所有时候都要做处理！

这也就是Python 3中的运作方式，被称为Unicode三明治（Unicode Sandwich）。

Python 2没有bytes和str的区分，2.6之后添加的bytes只是str的别名，从行为上来说，Python 2的str就类似C的char字符数组型的字符串，和Python 3的bytes类型一致，而Python 3的str有点类似Python 2的unicode，这里的细节就不再过多展开。

#### 1.3.1 互相不兼容
众所周知，Python属于强类型语言，类型直接不会自动转换，bytes和str虽然看上去在好多地方用法差不多，但并不能互相替换。

书中举的例子一共有这么几种：
- `+`，拼接，bytes和str不能相互拼接
- `>`、`<`，`==`二元比较，bytes和str不能相互比较
- `%`操作符，格式化字符串的行为

书中在说明二元比较情况的时候提到了bytes与str实例是否相等，总是会被评估为假，翻翻cpython源码应该能知道是为啥

反编译结果`dis.dis('a == b')`，可以发现用到了COMPARE_OP
```text
LOAD_NAME                0 (a)
LOAD_NAME                1 (b)
COMPARE_OP               2 (==)
```

cpython/Python/ceval.c:3859
```c
...
        TARGET(COMPARE_OP) {
            assert(oparg <= Py_GE);
            PyObject *right = POP();
            PyObject *left = TOP();
            PyObject *res = PyObject_RichCompare(left, right, oparg);
            SET_TOP(res);
            Py_DECREF(left);
            Py_DECREF(right);
            if (res == NULL)
                goto error;
            PREDICT(POP_JUMP_IF_FALSE);
            PREDICT(POP_JUMP_IF_TRUE);
            DISPATCH();
        }
...
```

发现会调用`PyObject_RichCompare`，继续看到这个函数
cpython/Objects/object.c:731
```c
PyObject *
PyObject_RichCompare(PyObject *v, PyObject *w, int op)
{
    PyThreadState *tstate = _PyThreadState_GET();

    assert(Py_LT <= op && op <= Py_GE);
    if (v == NULL || w == NULL) {
        if (!_PyErr_Occurred(tstate)) {
            PyErr_BadInternalCall();
        }
        return NULL;
    }
    if (_Py_EnterRecursiveCall(tstate, " in comparison")) {
        return NULL;
    }
    PyObject *res = do_richcompare(tstate, v, w, op);
    _Py_LeaveRecursiveCall(tstate);
    return res;
}
```

看`do_richcompare`，
cpython/Objects/object.c:679
```c
static PyObject *
do_richcompare(PyThreadState *tstate, PyObject *v, PyObject *w, int op)
{
...
}
```

到这其实有一点超出范围了，本着求真务实的态度，我决定挖个坑后面看，就叫《Python 3的==实现》。

回到正题就是Python的bytes和str之间也不能进行二元比较，而==会直接返回False。

关于第三点`%`格式化字符串的行为，存在这些行为：
- `bytes`类型的格式字符串中的`%s`可以用`bytes`填充，不能使用`str`进行填充，会报错提示不是`bytes-like`的对象或没有`__bytes__`方法
- `str`类型的格式字符串中的`%s`可以用`str`填充，可以使用`bytes`进行填充，但行为不一致，书上说是调用了对象的`__repr__`方法

其实觉得第二条调用对象的`__repr__`方法有点蹊跷，因为和上面的`__bytes__`方法对应的话这里应该是`__str__`方法才比较像。

所以写个demo来测试一下
```python
class Foo:
    def __str__(self):
        raise Exception('str')
    
    def __repr__(self):
        raise Exception('repr')

foo = Foo()
print('%s' % foo)
```
输出结果是：
```text
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in __str__
Exception: str
```
所以确实用的是`__str__`方法而不是`__repr__`方法


