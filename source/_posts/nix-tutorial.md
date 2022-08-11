---
title: nix, nixpkgs, nixos 从入门到入土
date: 2022-08-10 16:05:46
---

nix 是个非常棒的工具，这篇文章将会讲解如何入门 nix。

<!-- more -->

> 注：在学习 nix 之前，请保证你已经有了一定的 Linux 使用经验，如果对本文一些内容不太理解，你可能暂时还不太适合使用 nix。

<!-- toc -->

---

## 1. nix 是什么，他有什么作用

首先我给 nix 下一个不准确的定义，以便大家对 nix 有一个初步的印象。我们可以先把 nix 理解为构建工具和包管理器。同时，nix 也是一门语言，用于声明包是如何打出来的。为什么要用 nix 而不用 apt 或者 pacman 呢，这个我们会在之后讨论。

## 2. 安装 nix

nix 与你系统上已经有的东西并不冲突，即使你系统里有 apt, pacman，也能愉快地安装 nix。nix 的安装过程会新建一个 /nix 文件夹并建立几个符号链接。

### 安装 nix

如果你想要让多个用户可以运行 nix, 可以使用以下命令安装

```bash
$ sh <(curl -L https://nixos.org/nix/install) --daemon
```

如果你想要单用户运行 nix 或者你没有可用的 systemd (比如 wsl1 环境), 可以使用以下命令安装

```bash
$ sh <(curl -L https://nixos.org/nix/install) --no-daemon
```

> 如果你的网络情况不是很好，可以尝试将 `https://nixos.org/nix/install` 更换为 `https://mirrors.tuna.tsinghua.edu.cn/nix/latest/install`

> 注：使用以上命令时不应该使用 root 账户，他会调用 sudo 来创建需要的文件，请保证你的系统里有 sudo。如果你执意要使用 root 安装，请参考[附录](#附录)中的安装方法。

### 更换 channel 和 binary cache

安装完 nix 之后我们可能想要给 nix 换一下源。我们需要更换 channel 和 binary cache 两个东西的源。我将会在之后讲解 channel 和 binary cache 的作用。

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

## nix 初体验

```bash
$ nix-shell -p hello

[nix-shell:~]$ hello
Hello, world!

[nix-shell:~]$ exit

$ hello
bash: hello: command not found
```

可以看到我们通过 `nix-shell` 得到了一个 hello 程序，退出后就没有了。可以到 <https://search.nixos.org> 查询包。

> 你可能会在别的地方看到类似于 `nix-env -iA nixpkgs.hello` 的安装指令。我不建议使用 `nix-env`，他会导致你使用的包在不同的 nixpkgs commit 上等问题，之后会给出一些替代方案。

## nix 做了什么

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

## 附录

<!-- TODO: pkgsCross, install as root -->
<!-- https://github.com/NixOS/nix/pull/6882 -->