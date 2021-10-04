---
title: Nix Pills 翻译
date: 2021-10-03 17:08:48
---

最近在学习 `NixOS` , 但是中文资料非常少，于是我决定边看边翻译 <https://nixos.org/guides/nix-pills/> . 翻译水平不高, 请谅解. 如果有错误, 欢迎到 [GIthub]() 上创建 `issues` .

> Working In Progress

<!-- more -->

---

<!-- toc -->

# 前言

这是由 **Luca Bruno** (Lethalman) 在 2014 和 2015 年在博客上写的一系列文章. 这些文章以很短的章节 (被称为 `pill`) 介绍了 `Nix` 包管理器和 `Nixpkgs` 包集合.

`Nix Pills` 被认为是对 `nix` 的经典介绍，所以 2017 年 Graham Christensen (grahamc / gchristensen) 和其他贡献者试图把它迁移到现在的格式.

如果你遇到问题, 请将他们报告到 [nixos/nix-pills](https://github.com/NixOS/nix-pills/issues) .

> 注意: 以 `#` 开头的命令必须使用 `root` 用户运行, 比如以 `root` 身份登录或者使用 `sudo` 临时切换到 `root`

---

# 第一章 为什么你应该试一下 `Nix`

## 1.1 介绍

欢迎来到 `Nix in pills` 的第一篇文章. `Nix` 是一个为纯函数式的包管理器和部署系统 <!-- for POSIX 不知道怎么翻译 -->

介绍 `Nix`, `NixOS` 和相关项目的文档有很多, 但是这些文章的目的是说服你试一试 `Nix` . 安装 `NixOS` 并不是必要的, 但是有时候我可能会举使用 `Nix` 构建 `NixOS` 整个操作系统的真实示例. <!-- 没看出转折关系 -->

## 1.2 该系列的基本原理

[Nix](https://nixos.org/nix/manual/), [Nixpkgs](https://nixos.org/nixpkgs/manual/), [NixOS](https://nixos.org/nixos/manual/) 手册以及 wiki 都是解释 `Nix/NixOS` 如何运行, 如何使用, 以及如何用它完成很多很酷的事情的非常好的资料. 但是一开始, 你可能会感觉幕后发生的一些魔法很难掌控.

这个系列旨在补充现有的更正式的文档中的解释.

以下是对 `Nix` 的描述. 我会尽可能缩短它.

## 1.3 非纯函数式

大多数被广泛使用的软件包管理器 ([dkg](https://wiki.debian.org/dpkg), [rpm](http://www.rpm.org/), ...) 都会修改系统的全局状态. 如果一个包 `foo-1.0` 安装了一个二进制文件到 `/usr/bin/foo` , 除非你修改安装路径或者二进制文件的名字, 你就不能安装 `foo-1.1` . 但是更改二进制文件的名字意味着破坏使用这个二进制文件的程序.

有很多发行版尝试缓解这个问题. 比如 `Debian` , 用 [alternatives](https://wiki.debian.org/DebianAlternatives) 系统解决了部分问题.

因此, 虽然理论上在一个系统上安装同一个软件的多个版本是可能的, 但实际上这是非常痛苦的.

假设您需要一个 `nginx` 和一个 `nginx-openresty` 服务, 你必须创建一个更改了所有路径 (比如 `-openresty` 后缀) 的新包.

或者假设您需要运行两个不同的 `mysql` 示例: `5.2` 和 `5.5` , 也会有同样的情况. 此外, 您还必须保证两个 `mysqlclient` 不会发生冲突.

这并不是不可能, 但这非常不方便. 如果你想要安装像 `GNOME 3.10` 和 `GNOME 3.12` 两套软件, 可想而知工作量是多么的巨大.

从一个运维的角度来看: 您可以使用容器. 如今通常的解决方案是每个服务创建一个容器, 特别是两个不同版本的软件都需要的时候. 这在某种程度上解决了问题, 但是却在一些其他层面上带来了另外的缺点, 例如需要管理工具, 设置包的共享缓存和新机器来监控而不是简单的服务.

从一个开发者的角度来看: 您可以使用 `python` 的 `virtualenv` , `gnome` 的 `jhbuild` 或者其他的. 但是你如何混合两套程序呢? 你要如何避免能共享的文件被重复编译? 另外你需要配置你的开发工具到不同的文件夹, 这些文件夹包含了开发工具所需要的库. 不只是这样, 有些包也会错误地使用系统库, 这是很危险的.

`Nix` 很好地解决了所有这些人在包的层面上的问题.

## 1.4 纯函数式

`Nix` 对全局状态不做任何的猜测, 这有很多的好处, 但当然也有一些缺点 `Nix` 的核心是 `Nix store` , 通常情况下位于 `/nix/store` , 还有一些工具来管理 `store` . 在 `Nix` 里有一个 `derivation` 的概念而不是包. 一开始这些区别可能很微妙, 所以我会经常交替使用这些词.

`derivations/packages` 以 `/nix/store/hash-name` 的格式存储在 `Nix store` , 其中 `hash` 是用来标识 `derivation` 的唯一散列 (这样的说法不是很准确, 这个问题有一点复杂), `name` 是 `derivation` 的名字.

让我们以 bash 为例子看看 `derivation` : `/nix/store/s4zia7hhqkin1di0f187b79sa2srhv6k-bash-4.2-p45/` . 这是一个在 `Nix store` 的文件夹, 里边包含 `bin/bash` .

这意味着没有 `/bin/bash` . `store` 中只有独立的构建输出. `coreutils` 和其他所有东西都是这样. 为了方便在 `shell` 中使用他们, `nix` 将适当地将他们加入到 `PATH` 环境变量中.

我们有的基本上是所有包 (包括不同路径下的不同版本) 的 `store` , 所有在 `Nix store` 里的包都是不可变的.

事实上同样也没有 `ldconfig cache` . `bash` 要如何找到 `libc` 呢?

```bash
$ ldd  `which bash`
libc.so.6 => /nix/store/94n64qy99ja0vgbkf675nyk39g9b978n-glibc-2.19/lib/libc.so.6 (0x00007f0248cce000)
```

事实证明当 `bash` 构建的时候它是针对 `Nix store` 里特定版本的 `glibc` 构建的, 并且在运行的时候它会需要准确的 `glibc` 版本.

不要被 `derivation` 里 `name` 的版本号混淆: 它只是为了人类可读而取的名字, 你可能最终得到两个 `name` 相同 `hash` 不同的 `derivation` , 真正重要的是 `hash` .

这意味着什么? 这意味着你可以运行用 `glibc-2.18` 编译的 `mysql 5.2` , `glibc-2.19` 编译的 `mysql 5.5` . 你可以用 `gcc 4.6` 编译的 `python 2.7` 来运行你的 `python` 模块和 `gcc 4.8` 编译的 `python 3` 来运行同样的模块, 全部都在同一个系统里.

换句话说: 没有依赖地狱, 甚至没有一个依赖解决算法, 直接从一个 `derivations` 依赖到另一个 `derivations` .

从运维的角度看: 如果您想要一个旧版本的 `PHP` 来运行一个应用, 但是想要升级系统的剩余部分, 就不会再这么痛苦了.

从开发者的角度看: 如果想开发 `llvm 3.3 和 3.4` 上的 `webkit` , 您就不会再感到那么痛苦了.

## 1.5 可变 vs 不可变

当你升级一个库的时候, 大多数包管理器会将它们替换, 之后所有程序都使用新库运行而不需要重新编译. 之后, 所有程序都动态引用 `libc6.so` .

因为 `Nix derivations` 都是不可变的, 所以升级一个像 `glibc` 的库意味着重新编译所有的程序, 因为 `glibc` 在 `Nix store` 的路径是被硬编码到程序里的.

所以我们要如何解决安全更新呢? 在 `Nix` 里, 我们有一些把戏 (依然是纯的) 来解决这些问题, 但是这是另一个故事.

另一个问题是除非软件有一个纯函数式模型或者能适应这个模型, 否则很难在运行时组合应用程序.

以 `Firefox` 为例子, 当你在大部分系统上安装 `flash` 之后启动 `Firefox` , `flash` 就开始工作了, 因为 `Firefox` 会在全局路径查找插件.

在 `Nix` 中没有这样的全局路径给 `Firefox` 查找插件, 所以 `Firefox` 必须准确知道 `flash` 的准确安装位置. 我们处理这个问题的方式是封装 `Firefox` 的二进制文件以配置必要的变量来保证 `Firefox` 能找到 `Nix store` 中的 `flash` . 这会产生一个新的 `Firefox derivation` : 注意, 这需要几秒钟的世界并且在运行时让合成变得更困难.

您的数据没有升级/降级脚本, 这种方法没有意义, 因为没有任何 `derivation` 升级了. 使用 `Nix` , 您可以切换到任意用它自己的依赖编译的软件, 但这样做到时候并没有升级或降级的概念.

如果数据的格式发生变化, 那么迁移数据到新的格式仍然是您自己的责任

## 1.6 结论

`Nix` 允许您写的软件在构建时有最高的灵活性, 并且构建尽可能具有可重现性 (`reproducible `). 不仅如此, 因为它的性质, 它在云中部署的时候非常简单, 一致, 可靠, 以至于在 `Nix` 世界中现有自包含和编排工具都因为 [NixOps](http://nixos.org/nixops/) 而弃用.

这听上去可能很吓人, 但是在服务器和笔记本电脑上运行 `NixOS` 到目前为止我非常满意. 一些架构问题只需要一些人力就能解决, 其他的设计问题仍需要等待社区解决.

考虑到 [Nixpkgs](https://nixos.org/nixpkgs/) ([Github 链接](https://github.com/NixOS/nixpkgs)) 是一个为所有软件创建的全新的仓库, 拥有全新的概念. 核心的开发人员很少, 但总体上逐年增加. 换句话说, 这是值得您投资的.

## 1.7 下一个 pill

... 我们将会在您当前的系统上安装 Nix (我家是是 GNU/Linux, 但是我们也有 OSX 用户) 并开始检查现有软件.

---

> Working In Progress
