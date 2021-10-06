---
title: Nix Pills 翻译
date: 2021-10-03 17:08:48
---

最近在学习 `NixOS` , 但是中文资料非常少，于是我决定边看边翻译 <https://nixos.org/guides/nix-pills/> . 翻译水平不高, 请谅解. 如果有错误, 欢迎到 [GIthub]() 上创建 `issues` .

Version 20210221125542-f1acd7e

> Working In Progress

<!-- more -->

---

<!-- toc -->

# 前言

这是由 **Luca Bruno** (Lethalman) 在 2014 和 2015 年在博客上写的一系列文章. 这些文章以很短的章节 (被称为 `Pill`) 介绍了 `Nix` 包管理器和 `Nixpkgs` 包集合.

`Nix Pills` 被认为是对 `nix` 的经典介绍，所以 2017 年 Graham Christensen (grahamc / gchristensen) 和其他贡献者试图把它迁移到现在的格式.

如果你遇到问题, 请将他们报告到 [nixos/nix-pills](https://github.com/NixOS/nix-pills/issues) .

> 注意: 以 `#` 开头的命令必须使用 `root` 用户运行, 比如以 `root` 身份登录或者使用 `sudo` 临时切换到 `root`

---

# 第一章 为什么你应该试一下 Nix

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

`Nix` 对全局状态不做任何的猜测, 这有很多的好处, 也有一些缺点. `Nix` 的核心是 `Nix store` , 通常情况下位于 `/nix/store` , 还有一些工具来管理 `store` . 在 `Nix` 里有一个 `derivation` 的概念而不是包. 一开始这些区别可能很微妙, 所以我会经常交替使用这些词.

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

在 `Nix` 中没有这样的全局路径给 `Firefox` 查找插件, 所以 `Firefox` 必须准确知道 `flash` 的准确安装位置. 我们处理这个问题的方式是封装 `Firefox` 的二进制文件以配置必要的变量来保证 `Firefox` 能找到 `Nix store` 中的 `flash` . 这会产生一个新的 `Firefox derivation` , 这需要几秒钟的时间.

您的数据没有升级/降级脚本, 这种方法没有意义, 因为没有任何 `derivation` 升级了. 使用 `Nix` , 您可以切换到任意用它自己的依赖编译的软件, 但这样做到时候并没有升级或降级的概念.

如果数据的格式发生变化, 您应该自己负责迁移到新的数据格式.

## 1.6 结论

`Nix` 允许您写的软件在构建时有最高的灵活性, 并且构建尽可能具有可重现性 (`reproducible `). 不仅如此, 因为它的性质, 它在云中部署的时候非常简单, 一致, 可靠, 以至于在 `Nix` 世界中现有自包含和编排工具都因为 [NixOps](http://nixos.org/nixops/) 而弃用.

这听上去可能很吓人, 但是在服务器和笔记本电脑上运行 `NixOS` 到目前为止我非常满意. 一些架构问题只需要一些人力就能解决, 其他的设计问题仍需要等待社区解决.

考虑到 [Nixpkgs](https://nixos.org/nixpkgs/) ([Github 链接](https://github.com/NixOS/nixpkgs)) 是一个为所有软件创建的全新的仓库, 拥有全新的概念. 核心的开发人员很少, 但总体上逐年增加. 换句话说, 这是值得您投资的.

## 1.7 下一个 Pill

... 我们将会在您当前的系统上安装 Nix (我假设各位使用 GNU/Linux, 但是我们也有 OSX 用户) 并开始学习工具.

---

# 第二章 在您的系统上安装 Nix

欢迎来到第二章. 我们先简要地介绍一下 `Nix`.

现在我们即将在您的电脑上安装 `Nix` 并且让您明白接下来会在系统上发生些什么变化. 如果您正在使用 `NixOS` , 那么 `Nix` 已经在您的电脑上安装完成了, 您可以跳到[下一章](#第三章-进入环境).

安装 `Nix` 和安装其他软件一样简单, 它不会剧烈地改变我们的系统, 它将会远离我们的现有系统.

## 2.1 安装

以非 `root` 账户运行 `curl -L https://nixos.org/nix/install | sh` 然后根据提示完成安装. 如果您更喜欢下载安装脚本并使用 `GPG` 签名检测脚本, 您可以跟随 <https://nixos.org/nix/download.html> 完成安装.

这些文章并不是使用 `Nix` 的教程, 我们会通过 `Nix` 来了解他们的基本原理.

第一件要注意的事是: `derivations` 会在 `Nix store` 中引用其他的 `derivations` . 他们不适用我们系统中已经存在的 `libc` 或者其他任何东西. 这是一个包含运行软件所需要的所有包的自包含的 `store` .

> 注意: 在为多用户安装时, 例如使用 `NixOS` 的时候, `store` 是被 `root` 所拥有的, 其他用户可以通过 `Nix daemon` 来安装和构建软件. 你可以通过阅读这篇文章来了解更多关于多用户安装 `Nix` 的信息: <https://nixos.org/nix/manual/#ssec-multi-user>

## 2.2 了解 Nix store

观察安装命令的输出:

```bash
copying Nix to /nix/store..........................
```

这正是我们在第一篇文章提到的 `/nix/store` . 我们拷贝必要的文件以启动 `Nix` . 可以看到 `bash` , `coreutils` , `C` 语言工具链, `perl` 库,  `sqlite` 以及 `Nix` 本身以及 `libnix` .

您可能已经注意到了, `/nix/store` 并不只包含文件夹, 也会有一些文件, 这些文件也是以 `hash-name` 格式的名字存在的.

## Nix 数据库

当拷贝完 `store` 之后, 安装程序将会初始化数据库:

```bash
initialising Nix database...
```

`Nix` 在 `/nix/var/nix/db` 有一个数据库, 他是一个用来跟踪每两个 `derivation` 间依赖的数据库.

数据库的结构非常简单: 有一张从一个自增整数映射到一个有效的 `store` 路径的表.

你可以通过安装 sqlite (`nix-env -iA sqlite -f '<nixpkgs>'`) 并运行 `sqlite3 /nix/var/nix/db/db.sqlite` 来查看数据库.

> 注意: 如果这是您在安装后第一次运行 `Nix` , 您需要先重新打开您的终端以更新您的 `shell` 环境

> 注意: 除非您__真的__知道您在做什么, 永远不要手动更改 `/nix/store` . 如果您这样做了将不会更新到 `sqlite` 数据库.

## 2.4 第一个 Profile

在接下来的安装中, 我们会遇到 `profile` 的概念

```bash
creating /home/nix/.nix-profile
installing 'nix-2.1.3'
building path(s) `/nix/store/a7p1w3z2h8pl00ywvw6icr3g5l9vm5r7-user-environment'
created 7 symlinks in user environment
```

`Nix` 中的 `profile` 是实现回滚通用和实用的概念. `profiles` 用于组合散布在多个路径下的组件. 不仅如此, `profile` 还由很多版本化的 `generation` 组成, 当你更改一个 `profile` 的时候, 就会生成新的 `generation` .

`generations` 可以原子性地切换和回滚, 这让管理系统变得非常方便.

让我们仔细看看 `profile` :

```bash
$ ls -l ~/.nix-profile/
bin -> /nix/store/ig31y9gfpp8pf3szdd7d4sf29zr7igbr-nix-2.1.3/bin
[...]
manifest.nix -> /nix/store/q8b5238akq07lj9gfb3qb5ycq4dxxiwm-env-manifest.nix
[...]
share -> /nix/store/ig31y9gfpp8pf3szdd7d4sf29zr7igbr-nix-2.1.3/share
```

`nix-2.1.3 derivation` 就是包含二进制文件和库的 `Nix store` 本身. 安装 `derivation` 到 `profile` 的过程通过符号链接基本上重现了 `nix-2.1.3 derivation` 的层次结构.

`profile` 的内容非常特别, 因为只有一个程序被安装进了我们的 `profile` , `bin` 目录指向了唯一一个程序, 也就是 `Nix` 本身.

但是这只是我们最新的 `generation` 的 `profile` . 事实上 `~/.nix-profile` 是指向 `/nix/var/nix/profiles/default` 的符号链接.

同样, 这也是一个指向 `default-1-link` 的符号链接. 对, 这意味着这是第一个 `generation` 的 `profile` .

最后, `default-1-link` 指向了 `nix store` 用户环境的 `derivation ` , 就如您在安装过程中看到的那样.

我们将在下一篇文章中更加详细地讲解 `manifest.nix` .

## 2.5 Nixpkgs 表达式

从安装程序那里看到的更多的输出:

```bash
downloading Nix expressions from `http://releases.nixos.org/nixpkgs/nixpkgs-14.10pre46060.a1a2851/nixexprs.tar.xz'...
unpacking channels...
created 2 symlinks in user environment
modifying /home/nix/.profile...
```

[Nix 表达式](https://nixos.org/nix/manual/#chap-writing-nix-expressions)通常用来描述一个包以及如何构建他们. [Nixpkgs0(https://nixos.org/nixpkgs/) 是包含这些所有表达式的仓库.

安装程序从 `commit a1a2851` 中获取到包的描述.

我们可以发现第二个 `profile` 是 `channels profile` . `~/.nix-defexpr/channels` 指向了 `/nix/var/nix/profiles/per-user/nix/channels` , 后者也指向了 `channels-1-link` , 最后指向了包含已经下载好的 `Nix` 表达式的 `Nix store` 文件夹.

`channels` 是一个可以被下载的包和表达式的集合. 类似 `Debian` 的 `stable` 和 `unstable` , `channels` 也有 `stable` 和 `unstable` .

不要担心 `Nix` 表达式, 我们之后会获取他们.

最后, 为了方便, 安装程序会修改 `~/.profile` 来自动进入 `Nix` 环境. `~/.nix-profile/etc/profile.d/nix.sh` 做的只是简单地把 `~/.nix-profile/bin` 加入到 `PATH` , 把 `~/.nix-defexpr/channels/nixpkgs` 加入到 `NIX_PATH` . 我们之后会讨论 `NIX_PATH` .

您可以阅读一下 `nix.sh` , 它非常短.

## 2.6 FAQ: 我能不能修改 /nix 到其他位置?

能. 但是有使用 `/nix` 有比使用其他文件夹更好的理由. 所有的 `derivation` 通过绝对路径依赖其他的 `derivation` . 我们在第一篇文章的时候就已经看到 `bash` 通过绝对路径引用了一个 `/nix/store` 里的 `glibc` .

您可以自己看看, 不要担心如果您看到了多个 `bash` 的 `derivation` :

```bash
$ ldd /nix/store/*bash*/bin/bash
[...]
```

保持 `store` 在 `/nix` 中意味着我们可以从 `nixos.org` 中获取 `binary cache` (就如同您从 `Debian` 镜像中获取包)

想象一下:

- `glibc` 被安装在 `/foo/store`
- 因此 `bash` 需要引用 `/foo/store` 下的 `glibc` 而不是 `/nix/store` 文件夹
- `binary cache` 没有办法提供帮助, 因为我们需要不一样的 `bash` , 我们必须要自己重新编译所有东西

所以把 `store` 放在 `/nix` 中是明智之举.

## 2.7 结论

我们已经在我们的系统上安装了由 `nix` 用户拥有的完全隔离的 `Nix` .

我们学习了一些新的概念, 比如 `profile` 和 `channel` . 我们可以通过 `profile` 来管理不同 `generation` 的不同包的组合, 通过 `channel` 来从 `nixos.org` 下载二进制文件.

安装程序把所有东西都放在了 `/nix` 文件夹并创建了指向 `Nix` 用户的 `home` 的符号链接, 这是让每个用户都能在他自己的环境下安装和使用软件.

我希望我没有遗漏任何东西以至于让您感觉到幕后有魔法. 一切组件都放在 `store` 中并使用符号链接将这些组件组合在一起.

## 2.8 下一个 Pill

... 我们将会进入 `Nix` 环境并学习如何与 `store` 交互.

---

# 第三章 进入环境

欢迎来到第三章. 在[第二篇](#第二章-在您的系统上安装-Nix)文章中我们在您的系统上安装了 `Nix` . 现在我们终于可以开始玩玩了, 这些内容也适用于 `NixOS` 用户.

## 3.1 进入环境

__如果您正在使用 `NixOS` , 您可以[跳过](#3.2-安装一些东西)这个步骤__

在上一篇文章中我们创建了一个 `Nix` 用户, 所以让我们用 `su - nix` 切换到它. 如果您的 `~/.profile` 已经被执行了, 您应该就可以使用 `nix-env` 和 `nix-store` 指令了.

如果不能执行, 尝试

```bash
$ source ~/.nix-profile/etc/profile.d/nix.sh
```

## 3.2 安装一些东西

终于有实用的东西了! 安装东西进 `Nix` 环境是一个有趣的过程. 让我们来安装一下 `hello` 吧, 这是一个简单的命令行工具, 它会打印 `Hello world` 到终端上. 这个软件主要是用来测试编译器和包是否有安装好的.

```bash
$ nix-env -i hello
installing 'hello-2.10'
[...]
building '/nix/store/0vqw0ssmh6y5zj48yg34gc6macr883xk-user-environment.drv'...
created 36 symlinks in user environment
```

现在您可以运行 `hello` . 注意以下几点:

- 我们以用户的身份安装了软件, 并且只适用于 `Nix` 用户.
- 它会创建一个新的用户环境, 这是我们 `Nix` 用户的 `profile` 的一个新 `generation`
- `nix-env` 指令是管理环境, `profile` , 以及它们的 `generation` 的工具
- 我们通过 `derivation` 名与版本安装上了 `hello` . 我重复一遍: 我们__指定__了 `derivation` 名 (和版本) 来安装它

我们可以不用进入 `/nix` 文件夹也能查看 `generation` :4

```bash
$ nix-env --list-generations
   1   2014-07-24 09:23:30
   2   2014-07-25 08:45:01   (current)
```

列出已经安装的 `derivation` :

```bash
$ nix-env -q
nix-2.1.3
hello-2.10
```

那么, `hello` 究竟被安装到了哪里呢? 使用 `which hello` 指令得到的结果是指向 `Nix store` 的 `~/.nix-profile/bin/hello` . 我们同样也可以通过 `nix-env -q --out-path` 来列出 `derivation` 路径. 这些 `derivation` 路径就被称作构建的__输出 (output)__.

## 3.3 路径合并

这个时候你可能想要运行 `man` 来获得一些文档. 即使您在 `Nix` 环境之外已经有了 `man` , 您也可以通过 `nix-env -i man-db` 在 `Nix` 中安装和使用它. 像之前一样, 它将会创建一个新的 `generation` , 然后 `~/.nix-profile` 将会指向它.

让我们查看一下 `profile` :

```bash
$ ls -l ~/.nix-profile/
dr-xr-xr-x 2 nix nix 4096 Jan  1  1970 bin
lrwxrwxrwx 1 nix nix   55 Jan  1  1970 etc -> /nix/store/ig31y9gfpp8pf3szdd7d4sf29zr7igbr-nix-2.1.3/etc
[...]
```

现在这很有趣. 当我们只安装了 `nix-2.1.3` 的时候 `bin` 被符号链接到了 `nix-2.1.3` . 现在我们实际上已经安装了一些东西 (`man, hello`), 它变成了一个真正的文件夹而不是符号链接.

```bash
$ ls -l ~/.nix-profile/bin/
[...]
man -> /nix/store/83cn9ing5sc6644h50dqzzfxcs07r2jn-man-1.6g/bin/man
[...]
nix-env -> /nix/store/ig31y9gfpp8pf3szdd7d4sf29zr7igbr-nix-2.1.3/bin/nix-env
[...]
hello -> /nix/store/58r35bqb4f3lxbnbabq718svq9i2pda3-hello-2.10/bin/hello
[...]
```

OK, 现在更清晰了. `nix-env` 从已经安装的 `derivation` 中合并了这些路径. `which man` 指向了 `Nix profile` 而不是系统里的 `man` , 因为 `~/.nix-profile/bin` 在 `$PATH` 的开头.

## 3.4 回滚与切换 generation

上一条命令我们安装了 `man`, 除非您在中途更改了一些东西, 我们现在处在第三个 `generation` . 让我来告诉你如何混滚到旧版本:

```bash
$ nix-env --rollback
switching from generation 3 to 2
```

现在 `nix-env -q` 不再列出 `man` 了. ``ls -l `which man` `` 现在应该是您系统里的 `man` 了.

回滚已经足够了, 让我们回到最近的 `generation` :

```bash
$ nix-env -G 3
switching from generation 2 to 3
```

我希望您能阅读一下 `nix-env` 的 `manpage`. `nix-env` 需要一个操作符来执行, 有对每个操作通用的选项也有每个操作特定的选项.

您当然也可以[卸载](https://nixos.org/nix/manual/#operation-uninstall)和[升级](https://nixos.org/nix/manual/#operation-upgrade)这些包.

## 3.5 查询 store

至今我们已经学习了如何查询和管理环境, 但是所有这些环境的组件都指向 `store` .

为了查询和管理 `store` , 我们有一条 `nix-store` 指令, 可以用它来做一些有趣的事情, 但是我们目前将只会用来查询.

查询 `hello` 的直接运行依赖:

```bash
$ nix-store -q --references `which hello`
/nix/store/fg4yq8i8wd08xg3fy58l6q73cjy8hjr2-glibc-2.27
/nix/store/58r35bqb4f3lxbnbabq718svq9i2pda3-hello-2.10
```

`nix-store` 的参数可以是任何东西, 只要它指向 `Nix store` .

这可能现在对您来说没有意义, 但是我们查询依赖 `hello` 的文件:

```bash
$ nix-store -q --referrers `which hello`
/nix/store/58r35bqb4f3lxbnbabq718svq9i2pda3-hello-2.10
/nix/store/fhvy2550cpmjgcjcx5rzz328i0kfv3z3-env-manifest.nix
/nix/store/mp987abm20c70pl8p31ljw1r5by4xwfw-user-environment
```

这是预期的吗? 事实证明, 我们的环境依赖 `hello` . 是的, 这意味着我们的环境在 `store` 里, 并且弹包含了到 `hello` 的符号链接, 因此环境依赖 `hello` .

两个环境被列出来了, 第二个 `generation` 和第三个 `generation` , 因为这些环境安装了 `hello` .

`manifest.nix` 文件包含了关于环境的信息, 比如哪些 `derivation` 被安装了, 所以 `nix-env` 可以列出, 升级或删除这些 `derivation` . 另外, 当前的 `manifest.nix` 可以在 ``~/.nix-profile/manifest.nix` 找到.

## 3.6 闭包

`derivation` 的闭包是它所有的依赖列表, 递归地包含了 `derivation` 需要的一切.

```bash
$ nix-store -qR `which man`
[...]
```

拷贝所有以上 `derivation` 到另一台机器的 `Nix store` 中可以让您在另一台机器上实现 `man` 的开箱即用. 这是使用 `Nix` 进行部署的基础, 相信您已经看出来了 `Nix` 在云中部署软件的潜力 (提示: `nix-copy-closures` 和 `nix-store --export`).

一个更漂亮的闭包输出:

```bash
$ nix-store -q --tree `which man`
[...]
```

使用以上命令, 您可以更清晰地找出 `derivation` 运行时存在直接或间接依赖的原因.

这同样也适用于环境. 作为练习, 尝试运行 `nix-store -q --tree ~/.nix-profile` , 可以看到第一个子项都依赖于用户环境: 已安装的用户环境和 `manifest.nix` .

## 3.7 依赖的解决方案

`Nix` 没有像需要解决 `SAT` 问题来满足版本上下行依赖关系的 `apt` 的这样的问题. `Nix` 没有必要这样做, 因为所有依赖都是静态的: 让一个 `derivation X` 依赖于 `derivation Y` , `X` 将永远依赖 `Y` . 依赖于 `Z` 的某个版本的 `X` 将会是不一样的 `derivation` .

## 3.8 艰难的恢复

```bash
$ nix-env -e '*'
uninstalling 'hello-2.10'
uninstalling 'nix-2.1.3'
[...]
```

oh, 这删除了所有的 `derivation` , 包括 `Nix` . 这意味着我们没有办法使用 `nix-env` 了, 怎么办?

之前我们从环境中获取了 `nix-env` . 对用户来说, 环境非常方便, 但是 `Nix` 仍在 `store` 里!

首先, 选一个 `nix-2.1.3` 的 `derivation` : `ls /nix/store/*nix-2.1.3` , 比如 `/nix/store/ig31y9gfpp8pf3szdd7d4sf29zr7igbr-nix-2.1.3` .

第一个选项是回滚:

```bash
$ /nix/store/ig31y9gfpp8pf3szdd7d4sf29zr7igbr-nix-2.1.3/bin/nix-env --rollback
```

第二个选项是安装 `Nix` , 创建一个新的 `generation` :

```bash
$ /nix/store/ig31y9gfpp8pf3szdd7d4sf29zr7igbr-nix-2.1.3/bin/nix-env -i /nix/store/ig31y9gfpp8pf3szdd7d4sf29zr7igbr-nix-2.1.3/bin/nix-env
```

## 3.9 Channels

我们的包从哪里来? 我们在[第二章](#第二章-在您的系统上安装-Nix)的时候已经聊过了, 我们可以从一个 `channels` 列表里获取包, 尽管我们通常只用一个 `channel` . 这个管理 `channel` 的工具就是 `nix-channel` .

```bash
$ nix-channel --list
nixpkgs http://nixos.org/channels/nixpkgs-unstable
```

如果您正在使用 `NixOS` , 那么您可能看到的结果跟上边有一点区别 (默认情况下), 您可能会看到 `channel` 的名字以 `nixos-` 开头而不是 `nixpkgs` .

这实际上是 `~/.nix-channels `的内容.

> 注意: `~/.nix-channels` 并不是指向 `store` 的符号链接.

更新 `channel` 可以使用 `nix-channel --update` , 这会下载新的 `Nix` 表达式 (包的描述), 创建 `~/.nix-defexpr/channels` 下的新的 `channels profile` 的 `generation` .

这很类似于 `apt update` . (有关 Ubuntu 和 NixOS 包管理直接的粗略对比可以查看[这篇文章](https://nixos.wiki/wiki/Cheatsheet))

## 3.10 结论

我们已经学习到如何查询用户环境并通过安装和卸载软件来管理它. 如果您阅读了[手册](https://nixos.org/nix/manual/#operation-upgrade), 升级软件也同样直截了当.

每次我们更改环境的时候, 新的 `generation` 就会被创建, 切换 `generation` 也简单快速.

然后然后我们学习了如何查询 `store` . 我们查看了 `store` 路径中的依赖和被依赖的关系.

我们看到了如果使用符号链接来组合 `Nix store` 的路径, 这是很有用的技巧.

与编程语言的快速类比: 您拥有包含所有对象的堆, 这类似于 `Nix store` ; 您拥有对象到对象的映射, 这很像 `derivation` . 这是一个暗示性的比喻, 但是这会不会是正确的道路呢?

## 3.11 下一个 Pill

... 我们将会学习 `Nix language` 的基本语法. `Nix language` 是一个用来描述如何构建 `derivation` , 它是一切的基础, 包括 `NixOS` . 因此理解它的语法和语义非常重要.

> Working In Progress

