---
title: 基于erlang动态编译的简单配置方案：econfig
date: 2017-11-01 23:31:30
tags:
---

## 前提
在使用erlang 编程的时候，经常需要抽离配置文件。

通常配置方式有以下三种:
1. 使用ets存储。
2. 使用erlang 的 application environment，实际上也是使用ets存储的。[文档参考](http://erlang.org/doc/man/config.html)
3. 使用动态编译，将配置编译成erlang模块。这样程序可以直接访问代码区，并且无锁，另外由于erlang本身将代码保存两份的热更新机制，在更新配置的时候不会出现短暂读取不到配置的情况。单这种方案有个缺点，在配置过多的情况下，动态编译会比较慢。

## 使用动态编译构建配置

* 我根据ejabberd的dynamic_compile.erl 构建了一个简单的基于动态编译的配置方案： [econfig](https://github.com/YYChildren/econfig)
* 使用例子
```erlang
KVs = [{1,1}, {2,2}, {3,3}],
TestCase = [
    {test1, KVs},
    {test2, fun() -> KVs end}
],

econfig:reload_config(TestCase),
%% or
[econfig:reload_config(TestCase) || C <- TestCase],

%% 访问单个配置
1 = econfig:find(test1, 1),

%% 访问不存在的配置
undefined = econfig:all(4),

%% 访问所有配置（根据key排序）
KVs = econfig:all(test1).
```