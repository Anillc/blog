---
title: Nix, Nixpkgs, NixOS 从入门到入土
date: 2022-08-10 16:05:46
---

nix 是个非常棒的工具，这篇文章将会讲解如何入门 nix。

<!-- more -->

> 注：在学习 nix 之前，请保证你已经有了一定的 Linux 使用经验，如果对本文一些内容不太理解，你可能暂时还不太适合使用 nix。

<!-- toc -->

---

## 1. nix 是什么

根据 nix 在 GitHub [仓库](https://github.com/NixOS/nix)中的定义，nix 是一个用于 Linux 和其他的 Unix 的强大的包管理器，他使包管理可靠和可重现。官网中列出了 nix 的以下特性:

- 可重现 (reproducible)
- 声明式 (declarative)
- 可靠 (reliable)

我将会在这篇文章中向你讲解如何使用 nix。

> 一些刚开始学习的人可能会搞混 Nix, Nixpkgs, NixOS，请记住他们是不同的东西。

## 2. 安装 nix

> 这个小节中讲述的 nix 安装方法是在使用 NixOS 以外的发行版的情况下的安装方法。如果你正在使用 NixOS，可以参考之后的 NixOS 部分。

nix 与你系统上已经有的东西并不冲突，即使你系统里有 apt, pacman，也能愉快地安装 nix。nix 的安装过程会新建一个 /nix 文件夹并建立几个符号链接。

### 安装 nix

如果你想要让多个用户可以运行 nix, 可以使用以下命令安装:

```bash
$ sh <(curl -L https://nixos.org/nix/install) --daemon
```

如果你想要单用户运行 nix 或者你没有可用的 systemd (比如 wsl1 环境), 可以使用以下命令安装:

```bash
$ sh <(curl -L https://nixos.org/nix/install) --no-daemon
```

> 如果你的网络情况不是很好，可以尝试将 `https://nixos.org/nix/install` 更换为 `https://mirrors.tuna.tsinghua.edu.cn/nix/latest/install`

> ~~使用以上命令时不应该使用 root 账户，他会调用 sudo 来创建需要的文件，请保证你的系统里有 sudo。如果你需要使用 root 安装，请参考[附录](#附录)中的安装方法。~~ 由于 [nix#6882](https://github.com/NixOS/nix/pull/6882)，现在已经可以直接使用 root 安装 nix。

### 更换 Channel 和 Binary Cache

安装完 nix 之后我们可能想要给 nix 换一下源。我们需要更换 Channel 和 Binary Cache 两个东西的源。我将会在之后讲解 Channel 和 Binary Cache 的作用。

```bash
$ nix-channel --add https://mirrors.tuna.tsinghua.edu.cn/nix-channels/nixpkgs-unstable nixpkgs
$ nix-channel --update
```

如果你是多用户安装 nix, 请修改 /etc/nix/nix.conf，如果你是单用户安装 nix, 请修改 ~/.config/nix/nix.conf:

```conf
substituters = https://mirror.sjtu.edu.cn/nix-channels/store https://mirrors.tuna.tsinghua.edu.cn/nix-channels/store https://anillc.cachix.org https://cache.nixos.org/
trusted-public-keys = cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY=
```

修改完成后如果你是多用户安装 nix, 请重启 nix-daemon:

```bash
# systemctl restart nix-daemon
```

这样就安装好 nix 了。

## 3. nix 初体验

```bash
$ nix-shell -p hello

[nix-shell:~]$ hello
Hello, world!

[nix-shell:~]$ exit

$ hello
bash: hello: command not found
```

可以看到我们通过 `nix-shell` 得到了一个 hello 程序，退出后就没有了。如果想要知道能使用哪些包，可以到 <https://search.nixos.org> 查询。

> 你可能会在别的地方看到类似于 `nix-env -iA nixpkgs.hello` 的安装指令。我不建议使用 `nix-env`，他会导致你使用的包在不同的 nixpkgs commit 上等问题，之后会给出一些替代方案。

__nix 做了什么？__

在上面的例子中，我们使用 `nix-shell` 获得了一个带有 [GNU Hello](https://www.gnu.org/software/hello/manual/) 的 shell 环境，退出后便没有 `hello` 了。

查看 `/nix/store`，可以看到类似以下的内容：

```bash
$ ls /nix/store
0006xvyj2xdqj7sh41x5r9j911gwjykq-5fe5595a5a46eab069dcd2d6813c1b68cd9ade4f.drv
000n05b5xig1av867dk9vcizi37rwai1-X11-1.10.2.drv
000q965rl5mr33blq57538n906jj7222-readline63-008.drv
000vv6vpv1ng2fkwmgh5k5i2qpcm2v2i-ruby2.7.6-creole-0.5.0.drv
001gp43bjqzx60cg345n2slzg7131za8-nix-nss-open-files.patch
001qi4pcc8pk2dkw4q6m4lvh9m2cbmv5-diff___diff_4.0.2.tgz.drv
002mlfdwrf9d94ix738zgnncpd01hdnw-table___table_4.0.2.tgz.drv
...
```

而 `nix-shell` 做的事情就是构建一个 `hello`，然后将 `/nix/store` 中的 `hello` 放到 shell 的 `PATH` 变量中，你就可以使用 `hello` 了。

观察 `/nix/store` 中文件的结构，会发现有两种文件，一种是 `${hash}-${name}.drv`，一种是 `${hash}-${name}`。前者是一个 `derivation` 的声明，后者是一个 `derivation` 的构建结果。

在 nix 中，其实没有包的概念，可以把 `derivation` 理解为别的构建系统中的包。nix 语言就是用来声明这些包的。一个 `derivation` 可以依赖别的 `derivation`，最后在 `/nix/store` 中形成「闭包」。

如果你好奇 `hello` 的 nix 表达式是在哪里定义的，可以查看 nixpkgs 中的[定义](https://github.com/NixOS/nixpkgs/blob/b784c5ae63dd288375af1b4d37b8a27dd8061887/pkgs/applications/misc/hello/default.nix)。

## 4. 深入 nix

### 特性

我们在[第一小节](#1-nix-是什么)中写出了 nix 的以下几个特性:

- 可重现 (reproducible)
- 声明式 (declarative)
- 可靠 (reliable)

如果你曾经学习过 Haskell 等纯函数式语言，你可能知道 `纯函数` 这样一个概念。纯函数有以下特点

1. 对于同样的输入，纯函数将会返回同样的输出。
2. 纯函数中不会对外产生任何副作用，比方说一个纯函数不会修改外部的任何一个变量。

nix 的可重现性，便是尽力去实现纯函数的效果。你在 nix 表达式中声明了某个东西要怎么构建，nix 会提供同样的环境 (例如在构建环境中没有办法访问网络) 来构建他 (由于谁也没办法保证某个 Makefile 里生成了个随机数这种操作，所以 nix 只能去接近同样的构建环境)。同时，nix 也是一门函数式的语言，在达成一些条件的情况下他就是纯函数式的语言，在其中的任何求值都是「纯」的。这保证了 nix 的可重现性。在这样的情况下，nix 构建出来的文件应该是不可变的，因为他对应了一组输入，要保证输入和输出是对应的。

由于纯函数式的特性，我们写的 nix 代码比起命令式语言一行一行地执行，更像是在声明某个东西，所以 nix 也是声明式的，你可以通过 nix 文件来声明包的构建流程。你可以很轻松地分享你的构建流程或者开发环境给别人，因为他们都声明在 nix 文件中。

之前也讲到，nix store 中的文件格式是 `${hash}-${name}` 这样的格式，这与 Linux 传统的目录结构 (FHS) 有所不同。在传统的结构中，二进制文件存放在 `/usr/bin`, `/usr/lib` 这些位置，一旦某个软件有更新就必须要覆盖之前的文件。这可能导致很多问题，比如 A B 软件都依赖 C，A 必须要低于 1.1.0 的 C，B 必须要高于 1.2.0 的 C，这时便会出现难以解决的依赖冲突问题。而在 nix 中不存在这样的问题，因为 nix store 中可以存放多个版本的 C。同时由于这个特性，我们更新某个软件并不会影响到别的软件，出现问题也能快速回滚到以前的文件，实现了可靠性。

### Channel

[Nixpkgs](https://github.com/NixOS/nixpkgs) 是一个软件集合，包含了很多的软件，可以理解为其他包管理器中的官方源的目录，任何人都可以通过 Pull Request 的方式向其中新增包。其实 Nixpkgs 就是一个大型的 nix 表达式，里面声明了很多软件的构建流程以及一组打包软件用的工具和模板流程用于简化打包。

Channel 是一种用于管理 nix 表达式的东西，通过 `nix-channel` 可以做到从网络中获取 nix 表达式并完成更新等操作。这些下载下来的表达式被储存在 `~/.nix-defexpr`。

而[上文](#更换-channel-和-binary-cache)中提到的更换 Channel 便是将官方的 Nixpkgs 下载链接换到了别的地方，而我们通过 `nix-shell` 加载的软件，也是根据 Nixpkgs 中的定义获取的。

### Binary Cache

之前也说到过 nix 的纯函数式特性，对于同样的输入，一定会输出同样的结果。那么 nix 表达式的执行结果也会是一样的。所以我们可以把同样的 `derivation` 的构建结果储存起来再提供给别人下载，这样就能大大减少 `derivation` 的构建时间。

### Profile

Profile 是一个用于管理用户、系统环境的功能，它可以实现将 `/nix/store` 中的文件链接到 `/nix/var/nix/profiles` 中，并且进行版本管理。当你使用类似 `nix-env -iA` 安装程序的时候，便会更新 profile，产生新的 `generation`。可以发现，在安装 nix 的时候 `/nix/var/nix/profiles/per-user/${username}/profile` 就已经被链接到了 `~/.nix-profile`，而在非 NixOS 环境中 `~/.bashrc` 中也被加入了 `source ~/.nix-profile/etc/profile.d/nix.sh`，这样就能实现了运行 profile 中的程序。

> 不推荐手动管理 profile，原因已经在上文中讲过，从不同 commit 的 nixpkgs 中安装软件可能导致一些问题 (比如输入法无法使用)。

## 5. nix 语言

该部分单独写到了 [nix 语言教程](/2022/08/27/nix-lang-tutorial/) 中。

下面的内容将会假设你已经学会了 nix 语言。

### callPackage override overrideAttrs

### docker 容器

## flake

## home-manager

## tmpfs as root

## 附录

常用指令

## 参考资料

以上对 Nix, Nixpkgs, NixOS 的讲解并不全面。如果你想要了解更多，可以参考以下内容

nixos.wiki
nixpkgs manual
nix manual
nixos manual
nixos-cn
flakes
nix pills

ripgrep

<!-- TODO: pkgsCross -->

