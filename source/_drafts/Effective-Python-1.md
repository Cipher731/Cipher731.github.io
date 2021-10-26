---
title: Effective Python - 1
tags:
- Python
---

## 0. 前言
《Effective Python》是某阳推荐的，他所在部门的必读书籍，虽然我不用Python进行一些大型应用的开发，但是作为一个热爱Python的脚本小子，我对这本书也非常感兴趣。最新第二版今年刚出，还热乎着，还比第一版多了整整31条建议，这不血赚？  
这本书的子标题叫90 Specific Ways to Write Better Python，中译名叫编写高质量Python代码的90个有效方法，打算写5篇左右的文章学习记录一下，每篇分析18个左右的方法。

## 1. 培养Pythonic思维

### 1.1 查询自己使用的Python版本
第一条居然是查版本，不会吧不会吧，不会真的有人不知道怎么查版本吧？  
咳咳，其实我理解作者的用意主要还是在介绍整本书的环境基础，整本书的代码示例是基于Python 3.7的语法并介绍一些Python 3.8的新特性。  
在撰写现在这篇博文的时候，Python 3.10刚发布不久，它是在2021-10-04发布的，并且久违地引入了新的语法特性——Structural Pattern Matching，后面有时间的话我会读一读[PEP-634 -- Structural Pattern Matching: Specification](https://www.python.org/dev/peps/pep-0634/)、[PEP 635 -- Structural Pattern Matching: Motivation and Rationale](https://www.python.org/dev/peps/pep-0635/)和[PEP 636 -- Structural Pattern Matching: Tutorial](https://www.python.org/dev/peps/pep-0636/)并对这个新特性做一定的分析。

### 1.2 遵循PEP-8风格指南
这一条的内容是关于[PEP-8](https://www.python.org/dev/peps/pep-0008/)代码规范的，其实PEP-8也是我特别喜欢的一条PEP（Python Enhancement Proposals），它以最权威的方式定义了Python的代码风格指南，使所有的代码至少看上去是Pythonic的，顺便这里需要吹一波Pycharm，让我在没有仔细读过PEP-8的情况下，也对PEP-8的细节知道得大概了，毕竟作为轻微强迫症，看到黄色波浪线就会十分不爽。当然，类似的工具应该还有Pylint，常用VS Code的同学可能接触得比较多吧。  
但是读的时候发现关于命名规范还是有一点不太清楚的，主要还是由于我没用过Python写过大型的面向对象项目，很少去实现一些类，所以忽略了实例属性（书中在这里称其为实例属性，写文章的时候顺手就写成类字段或者类属性了，但是仔细想想可能不对，如果是类属性的话，那就是属于类对象本身的属性了，存在比较大的差异）的命名规范，PEP-8建议了受保护的实例属性用一个下划线开头而私有的实例属性用两个下划线开头，也就是`foo._protected`和`bar.__private`，其实不太懂所谓的受保护和私有是怎么样的一个机制，毕竟之前碰到过一大堆Python沙箱逃逸，我粗略猜测Python并不存在像C++那样的语言语法层面的限制，而是作为一种逻辑约束。
于是看了这篇[Stack Overflow上的问题](https://stackoverflow.com/questions/1641219/does-python-have-private-variables-in-classes)，所以确实是一种逻辑上的约束。

### 1.3 了解bytes和str的区别
这条也比较有意思，从个人角度来说的话，由于在CTF中经常会写脚本处理一些编码相关的问题，不论是Web还是Crypto还是Pwn还是Misc还是Reverse，根据出题人的脑洞，都会或多或少地接触到编码的转换，其中最常见也最基本的就是base64编码。  
虽然书上没有提这块内容在Python 2和Python 3上的差异，但是我觉得这个有必要在这展开一下。  
在Python 3里面一个比较令人头疼的地方是，会出现大量使用`encode`和`decode`的情况，比如`base64.b64decode(foo.encode())).decode()`或者`base64.b64encode(bar.encode())).decode()`，这里面涉及到的就是bytes和str之间的相互转化，刚开始接触到这种转换的时候那真是被绕晕了，不像Python 2里面那样直接`base64.b64encode(foo)`和`base64.b64decode(bar)`就行了。  
Python 3把事情搞复杂的原因其实不复杂，那就是更好地支持本地化，Python 3程序需要把解码和编码操作放在程序界面的最外层进行，就是为了把输入文本的编码和内部数据的编码区分开来。Python 2时代，如果直接输入非Ascii字符的字符串，会自动根据当前编码转换成字节存储，例如`'你好'`在OS默认编码为UTF-8的情况下会直接变成`'\xe4\xbd\xa0\xe5\xa5\xbd'`，那么随之而来的问题其实已经可以想到了，万恶的GBK又要出来了，Windows的默认编码应该是GBK或者GB-2312（这俩家伙的差异俺也不太清楚，开个坑先）
Python 2没有bytes和str的区分，2.6之后添加的bytes只是str的别名，从行为上来说，Python 2的str就类似C的char字符数组型的字符串，和Python 3的bytes类型一致。
