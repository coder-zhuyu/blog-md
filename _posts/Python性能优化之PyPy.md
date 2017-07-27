---
title: Python性能优化之PyPy
date: 2017-07-27 17:36:27
categories: Python
tags: [Python, 性能优化, PyPy, JIT]
---

Python等动态语言是通过将程序编译成字节码，用虚拟机执行字节码来运行程序的，并不是直接执行的本地机器码，所以解释型动态语言一般会比编译型静态语言执行速度慢。
python等动态类型语言之所以慢，就是因为每一个简单的操作都需要大量的指令才能完成。JIT（即时编译，一种混合了解释器和编译器好处的技术）在字节码执行过程中标识被经常执行的字节码，并将其编译成本地机器码，缓存该结果，当同样的字节码再次被执行的时候，会执行预编译的机器码，从而提高性能。
[PyPy][1]就是为了提升python性能而设计的，其使用RPython实现的解释器。RPython是Python的子集， 具有静态类型。

## PyPy源码安装
以ubuntu操作系统为例，[参加官方文档][2]

1. 依赖安装
```bash
$ sudo apt-get install gcc make libffi-dev pkg-config libz-dev libbz2-dev \
libsqlite3-dev libncurses-dev libexpat1-dev libssl-dev libgdbm-dev \
tk-dev libgc-dev python-cffi \
liblzma-dev libncursesw-dev      # these two only needed on PyPy3
```

2. 源码下载
https://pypy.org/download.html

3. 编译
确保有足够的内存，执行编译过程有点漫长。
```bash
make
```
。。。
编译成功后会生成一个可执行文件pypy-c和一个动态库libpypy-c.so。新建目录/opt/pypy，将源码目录下的如下文件复制到该目录下:

 - pypy-c
 - libpypy-c.so
 - include/
 - lib_pypy/
 - lib-python/2.7
 - site-packages/

将pypy-c放到PATH所在目录，将动态库放入搜索路径下:
```bash
$ cd /usr/local/lib
$ sudo ln -s /opt/pypy/libpypy-c.so libpypy-c.so
$ cd /usr/local/bin
$ sudo ln -s /opt/pypy/pypy-c pypy
```

## PyPy使用
结合虚拟环境使用
```bash
$ mkvirtualenv -p /usr/local/bin/pypy venv-pypy
```
一个Flask框架写的项目[flask-api][3]，压测一个简单数据库查询请求，使用PyPy运行性能提升了一倍左右。
 


  [1]: https://pypy.org/index.html
  [2]: https://pypy.readthedocs.io/en/latest/build.html
  [3]: https://github.com/coder-zhuyu/flask-api.git
