---
title: nix 语言教程
date: 2022-08-27 14:24:05
---

这是上一篇 [Nix, Nixpkgs, NixOS 从入门到入土](/2022/08/10/nix-tutorial/) 的一部分，由于篇幅过长，将此部分单独分出来。

<!-- more -->

我们已经大致了解了 nix 的原理，接下来我将会讲解 nix 语言的一些基本语法。

之前也提到过 nix 是一门函数式语言，同时他也是惰性求值的。如果你曾经没有写过函数式语言，可能会对 nix 的写法感到奇怪。如果你无法理解我这里讲解的 nix 语言，可以尝试先了解一下别的函数式语言再回来阅读，或者查看一些[官方的文档](https://nixos.org/manual/nix/stable/expressions/writing-nix-expressions.html)，可能会有更好的理解。

> 你可以使用 `nix repl` 打开 repl 来对 nix 语言进行实验。

请注意，在 nix 语言中没有「语句」的概念，也没有「变量」的概念，nix 语言中的值都是不可变的。偶尔可能会说「变量」这只是因为习惯所以大家会用这种表述来代替，其实他是一种「绑定」。

nix 表达式的后缀是 `.nix`。

# 注释

```nix
# 这是单行注释
/* 这是多行注释 */
```

# 标识符

值得注意的是，nix 里的标识符跟其他语言不一样，`-` 也会被作为标识符的一部分。

例如: `a-b` 会被认为是一个标识符而不是 `a` 减 `b`。

# 值

nix 中的值有多种类型

## 数字

```nix
> 1
1

> 2
2
```

## 字符串

__单行字符串__

```nix
> "这是一个单行字符串"
"这是一个字符串"
```

__多行字符串__

```nix
> ''
    这是一个多行字符串
''
"qwq\n"
```

多行字符串中的缩进会被自动删除。

__字符串插值__

```nix
> let world = "World"; in "Hello ${world} !"
"Hello World !"
```

```nix
> let world = "World"; in ''
    Hello ${world} !
''
"Hello World !\n"
```

如果要在单行字符串中使用 `${`，请在 `$` 前加上 `\` 进行转义: `"\${}"`;
如果要在多行字符串中使用 `${`，请在 `$` 前加上 `''` (两个单引号) 进行转义:

```nix
> ''
    ''${}
''
"${}\n"
```

## 列表 (List)

nix 中的列表以 `[` 和 `]` 标志开始和结束，元素以空格分隔:

```nix
> [ 1 2 (3) ]
[ 1 2 3 ]
```

## 集合 (Set)

```nix
> { hello = "world"; foo.bar = "foobar"; }
{ foo = { ... }; hello = "world"; }

> { hello = "world"; }.hello
"world"
```

其中，`foo.bar = "foobar";` 会被展开为 `foo = { bar = "foobar"; };`。

__递归集合 (Recursive set)__

```nix
> rec { x = "hello"; y = x; }
{ x = "hello"; y = "hello"; }
```

递归集合中可以引用自身的属性。

# 运算

# let ... in

# if

# with

# 函数

# derivation

> 使用 `:b <expression>` 可以构建一个 `derivation`

# 惰性求值