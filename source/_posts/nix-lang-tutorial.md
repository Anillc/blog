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

<!-- toc -->

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

## 数字 (Number)

```nix
> 1
1

> 2
2
```

## 路径 (Path)

路径可以是绝对路径 (`/foo/bar`)，也可以是相对路径 (`./foo/bar`)。需要注意的是，使用这种路径会把文件拉到 `/nix/store` 中。

```nix
> /foo/bar
/foo/bar

> "${./foo/bar}"
"/nix/store/ll3iwpqwhk11ms7mdzmb1533g7rkfk7r-bar"

> bar = "baz"
> /foo/${bar}
/foo/baz
```

还可以使用 `<>` 来表示路径，比如 `<nixpkgs>`, `<nixpkgs/nixos>`。`<>` 会优先在 `NIX_PATH` 这个环境变量中的路径查找其中的文件或者目录。如果 `NIX_PATH` 不存在，nix 会在 `~/nix-defexpr` 中查找文件或目录。

> 在 pure 的情况下无法使用 `<>`，因为他会引入可变的量，这对于系统的纯度有很大的影响。同理，path 在 pure 的情况下无法引用当前目录以外的文件，因为这也会影响系统的纯度。我将在 [flake](/2022/08/10/nix-tutorial/#flake) 部分详细说明。

## 布尔 (Boolean)

```nix
> true
true

> false
false
```

## 空 (Null)

```nix
> null
null
```

## 字符串 (String)

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

需要注意的是，像 `/` 和 `-` 这些符号，如果没有加空格的话可能会被解析成其他东西，建议使用时在运算符间加上空格。

```nix
> a - b
1

> a-b
error: undefined variable 'a-b'
```

这里直接翻译[这里](https://nixos.org/manual/nix/stable/expressions/language-operators.html)的表格。

| 名字 | 语法 | 结合性 | 描述 | 优先级 |
|-|-|-|-|-|
| 选取 | *e* `.` *attrpath* [ `or` *def* ] | 无 | 从 *e* 中根据 *attrpath* (由 `.` 分割的属性名) 访问值。如果属性不存在并且提供了 *def*，将会返回 *def*，否则将终止求值。 | 1 |
| 函数调用 | *e1* *e2* | 左 | 将 *e2* 作为参数，调用 *e1* | 2 |
| 负 | `-` *e* | 无 | 算术取负 | 3 |
| 存在属性 | *e* `?` attrpath | 无 | 根据 *attrpath* 检测 *e* 中是否存在属性，返回 `true` 或 `false`。 | 4 |
| List 连接 | *e1* `++` *e2* | 右 | List 连接。 | 5 |
| 乘 | *e1* `*` *e2* | 左 | 算术乘法。 | 6 |
| 除 | *e1* `/` *e2* | 左 | 算术除法。 | 6 |
| 加 | *e1* `+` *e2* | 左 | 算术加法。 | 7 |
| 减 | *e1* `-` *e2* | 左 | 算术减法。 | 7 |
| 字符串相加 | *e1* `+` *e2* | 左 | 字符串相加。 | 8 |
| 非 | `！` *e* | 无 | 逻辑非。 | 8 |
| 更新 (Update) | *e1* `//` *e2* | 右 | 返回由 *e1* 和 *e2* 共同组成的 Set (在有同名属性的时候，优先使用后者的属性)。 | 9 |
| 小于 | *e1* `<` *e2* | 无 | 算术比较或按照字典顺序比较。 | 10 |
| 小于等于 | *e1* `<=` *e2* | 无 | 算术比较或按照字典顺序比较。 | 10 |
| 大于 | *e1* `>` *e2* | 无 | 算术比较或按照字典顺序比较。 | 10 |
| 大于等于 | *e1* `<=` *e2* | 无 | 算术比较或按照字典顺序比较。 | 10 |
| 等于 | *e1* `==` *e2* | 无 | 等于。 | 11 |
| 不等于于 | *e1* `！=` *e2* | 无 | 不等于。 | 11 |
| 逻辑与 | *e1* `&&` *e2* | 左 | 逻辑与。 | 12 |
| 逻辑或 | *e1* <code>&vert;&vert;</code> *e2* | 左 | 逻辑或。 | 13 |
| 逻辑蕴含 | *e1* `->` *e2* | 无 | 逻辑蕴含 (与 <code>!e1 &vert;&vert; e2</code> 等价)。 | 14 |

# let ... in

let 表达式允许你绑定一个表达式 (定义变量)。

```nix
> let
    x = "foo";
    y = "bar";
in x + y
"foobar"
```

# inherit

当我们需要拷贝变量的时候使用 `inherit` 会比较方便。

```nix
let
    x = 123;
in {
    inherit x;
    y = 456;
}
```

与之等价的是:

```nix
let
    x = 123;
in {
  x = x;
  y = 456;
}
```

如果在 `inherit` 后用括号包裹一个表达式，可以从这个表达式的结果中继承属性。

```nix
> let
    x = { y = 123; };
in {
   inherit (x) y;
   z = 456;
}
{ y = 123; z = 456; }
```

`inherit` 也可以在 let ... in 中使用:

```nix
let
    inherit hello;
in hello
```

# with

`with e1; e2` 可以将 Set `e1` 中的属性加入到 `e2` 的作用域中。

```nix
> let
    x = { y = "hello"; };
in with x; y
"hello"
```

# if

`if` 表达式写作 `if e1 then e2 else e3`。当 `e1` 为 `true` 时以 `e2` 为 `if` 表达式的值；当 `e1` 为 `false` 时以 `e3` 为 `if` 表达式的值。

```nix
> if true then 1 else 2
1
```

# 函数 (Function)

nix 中，一个函数写作 `pattern: body`。

```nix
> x: x + 1
«lambda @ (string):1:1»

> (x: x + 1) 1
2
```

如果参数是个 Set，他还能够被解构:

```nix
> { x, y, z }: x + y + z
«lambda @ (string):1:1»

> ({ x, y, z }: x + y + z) { x = 1; y = 2; z = 3; }
6

> ({ x, y }: x + y) { x = 1; y = 2; z = 3; }
error: anonymous function at (string):1:2 called with unexpected argument 'z'
```

如果有多余的参数不想写出来时，可以在后面加上 `...`

```nix
> ({ x, y, ... }: x + y) { x = 1; y = 2; z = 3; }
3
```

如果你想要参数有默认值，可以使用以下写法:

```nix
> ({ x, y ? 2 }: x + y) { x = 1; }
3
```

你可能还想要在解构的同时将参数绑定到一个标识符上:

```nix
> (args@{ x, ... }: x + args.y) { x = 1; y = 2; }
3

> ({ x, ... }@args: x + args.y) { x = 1; y = 2; }
3
```

需要注意的是，默认值并不会被加入绑定的标识符中:

```nix
> (args@{ x, y ? 2 }: x + args.y) { x = 1; }
error: attribute 'y' missing
```

# 导入 (import)

使用 `import` 可以导入其他 nix 文件。

```nix
> let
    pkgs = import <nixpkgs> {};
in pkgs.hello
«derivation /nix/store/d4h6nprhdkjsyj93aapimw780jfak78s-hello-2.12.drv»
```

当 `import` 的目标是一个目录的时候，会默认导入其中的 `default.nix`。

# derivation

使用 `derivation` 关键字可以生成一个 `derivation`。通常我们不使用这个方法来构建 `derivation`，而是使用 `pkgs.stdenv.mkDerivation` 等封装好的函数。如果你对 `mkDerivation` 的原理感兴趣，可以阅读 [nix pills](https://nixos.org/guides/nix-pills/) 第六章到第八章的内容。

> 使用 `:b <expression>` 可以在 repl 环境中构建一个 `derivation`。

# 惰性求值

nix 语言是惰性求值的，这与命令式语言不同。惰性求值指的是只有你用到某一个值的时候他才会被计算。

```nix
> a = { x = 1 + 1; }
> a.x
2
```

当我们定义 a 的时候，x 的值是没有被计算的。只有在使用 `a.x` 的时候 `1 + 1` 才会被计算。

这样的好处有很多，其中一个是我们拥有一个非常大的 Nixpkgs。如果我们每次求值都要求整个 Nixpkgs 的话，那将会浪费非常多的时间。

---

至此，nix 的基本语法已经讲完。如果你不太理解，可以多看一下别人写的 nix 代码并参考一些官方文档进行学习。现在你可以返回 [Nix, Nixpkgs, NixOS 从入门到入土](/2022/08/10/nix-tutorial/#6-nixpkgs) 继续学习了。