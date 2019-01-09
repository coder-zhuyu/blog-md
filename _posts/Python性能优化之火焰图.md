---
title: Python性能优化之火焰图
date: 2017-07-26 13:36:25
categories: Python
tags: [Python, 性能优化, 火焰图]
---
性能优化首先要知道程序的性能瓶颈在哪里，python已经提供了profile工具可以看出哪些函数耗时较长。本文介绍的[火焰图][1]会以图的形式给出程序更直观的函数调用及开销，这里将介绍的是Uber开源的python火焰图工具[pyflame][2]。

## 首先给出一个直观的火焰图
![火焰图示例][3]
可以很形象的看出函数调用耗时情况，横向越宽说明耗时越长，从下往上看，最下面表示程序从开始到结束总的耗时。从下往上是一级级函数的调用情况。

## pyflame安装(以ubuntu为例)
对Debian/Ubuntu/Fedora系统支持的比较好，[centos6参考][4]。

 1. 依赖安装
 ```bash
 $ sudo apt-get install autoconf automake autotools-dev g++ pkg-config python-dev python3-dev libtool make
 ```
 
 2. 编译安装
 ```bash
 ./autogen.sh
 ./configure      # Plus any options like --prefix.
 make
 ```
 生成的pyflame工具位于src目录下，可以将其放到/usr/local/bin目录下
 
## 火焰图生成工具
火焰图[github][5], 主要使用脚本flamegraph.pl，可以将其放到/usr/local/bin目录下

## pyflame使用
```bash
$ pyflame 12345                             # Attach to PID 12345 and profile it for 1 second
$ pyflame -s 5 -r 0.01 768                  # Attach to PID 768 and profile it for 5 seconds, sampling every 0.01 seconds
$ pyflame -o prof.txt -t py.test tests/     # Run py.test against tests/, emitting sample data to prof.txt
$ flamegraph.pl prof.txt>prof.svg           # 生成图
```
 - -s 指定pyflame运行时间
 - -r 抽样频率
 - -o 抽样数据输出
 - --abi option to force a particular Python ABI.(uwsgi可能会用到这个选项)

  [1]: http://www.brendangregg.com/flamegraphs.html
  [2]: https://github.com/uber/pyflame
  [3]: https://github.com/coder-zhuyu/images/blob/master/blog/flame.svg
  [4]: http://blog.motitan.com/2017/04/15/python%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90%E5%B7%A5%E5%85%B7%E4%B9%8Bpyflame
  [5]: https://github.com/brendangregg/FlameGraph
