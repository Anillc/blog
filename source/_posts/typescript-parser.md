---
title: TypeScript 类型体操，实现编程语言的解析并求值
date: 2022-11-29 18:33:34
---

之前看到了 [TypeScript 类型体操天花板，用类型运算写一个 Lisp 解释器](https://zhuanlan.zhihu.com/p/427309936) 这篇文章，感到非常震撼。这两天在研究一些文本解析的问题突然想起了这篇文章，然后自己也想糊一个。

众所周知，TypeScript 的类型系统是图灵完备的，于是我们可以用 ts 的类型系统来做到一些奇妙的事情，比如上面那篇用类型系统实现的 lisp 解释器。而我想要做一个更通用的东西。

<!-- more -->

先看一下效果：

```typescript
type E<I extends string> = Eval<Parse<producers, table, I>>

type T = E<`(1 + 2) * 3 + 4 * 5`>
//   ^? type T = 29
```

代码在 [Anillc/yatcc](https://github.com/Anillc/yatcc)。

先糊一个词法分析器，

```typescript
type LexTrim<I extends string> = I extends `${' ' | '\t' | '\n'}${infer R}` ? LexTrim<R> : I

type Num = '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9'
type LexNumber<I extends string> = I extends `${infer N extends Num}${infer R}`
  ? [`${N}${LexNumber<R>[0]}`, LexNumber<R>[1]]
  : I extends `.${infer R}`
    ? [`.${LexNumber<R>[0]}`, LexNumber<R>[1]]
    : ['', I]

type LexString<I extends string, In extends false | string = false, Ignore extends boolean = false> = In extends false
  ? I extends `${infer S extends "'" | '"' | '`'}${infer R}` ? LexString<R, S> : ['', I]
  : Ignore extends true
    ? I extends `${infer L}${infer R}` ? [`${L}${LexString<R, In>[0]}`, LexString<R, In>[1]] : ['', I]
    : I extends `\\${infer R}`
      ? LexString<R, In, true>
      : I extends `${In}${infer R}`
        ? ['', R]
        : I extends `${infer L}${infer R}`
          ? [`${L}${LexString<R, In>[0]}`, LexString<R, In>[1]]
          : never

type Id =
  | 'a' | 'b' | 'c' | 'd' | 'e' | 'f' | 'g'
  | 'h' | 'i' | 'j' | 'k' | 'l' | 'm' | 'n'
  | 'o' | 'p' | 'q' | 'r' | 's' | 't' | 'u'
  | 'v' | 'w' | 'x' | 'y' | 'z' | 'A' | 'B'
  | 'C' | 'D' | 'E' | 'F' | 'G' | 'H' | 'I'
  | 'J' | 'K' | 'L' | 'M' | 'N' | 'O' | 'P'
  | 'Q' | 'R' | 'S' | 'T' | 'U' | 'V' | 'W'
  | 'X' | 'Y' | 'Z' | '_'
type LexId<I extends string, F extends boolean = true> = F extends true
  ? I extends `${infer L extends Id}${infer R}` ? [`${L}${LexId<R, false>[0]}`, LexId<R, false>[1]] : ['', I]
  : I extends `${infer L extends Id | number }${infer R}` ? [`${L}${LexId<R, false>[0]}`, LexId<R, false>[1]] : ['', I]

type Equal<A, B> = [A] extends [B] ? true : false

export type Lex<I extends string> =
  LexTrim<I> extends infer Trimmed extends string
    ? Trimmed extends '' ? []
    : LexNumber<Trimmed> extends [infer N extends string, infer R extends string] ? Equal<N, ''> extends false ? [{ type: 'num', value: ToNumber<N> }, ...Lex<R>]
    : LexString<Trimmed> extends [infer S extends string, infer R extends string] ? Equal<S, ''> extends false ? [{ type: 'str', value: S }, ...Lex<R>]
    : LexId<Trimmed>     extends [infer I extends string, infer R extends string] ? Equal<I, ''> extends false ? [{ type: 'id' , value: I }, ...Lex<R>]
    : Trimmed extends `${infer L}${infer R}` ? [{ type: L }, ...Lex<R>]
  : never : never : never : never : never
```

LexTrim 这个类型用于去除字符串前的空白字符，LexNumber LexString LexId 用于解析数字、字符串和标识符，Lex 把他们组合起来。

然后糊语法分析器。LR 是一种解析上下文无关文法的算法，可以预先把解析文本所需要的所有可能的状态保存到表中，再使用一个状态机来解析他。分为 Action 表和 Goto 表，Action 表中保存移近和规约的规则，Goto 表保存规约过后应该往栈中加入的状态。简单糊一个 LR 表生成器过后，就可以愉快地用类型写语法分析器啦。

```typescript
export type Parse<
  P extends Record<number, Producer>,
  T extends Table,
  S extends string[],
  B extends (Node | Token)[],
  F extends boolean,
  Tokens,
> = F extends true ? B[0] :
  Tokens extends Token[]
    ? GetAction<T, S, Tokens> extends [infer Type extends number, infer Action extends number]
      ? Type extends 0 ?
        Parse<P, T, [...S, ToString<Action>], [...B, Tokens[0]], false, Tail<Tokens>>
      : Type extends 1 ?
        Parse<
          P, T,
          [...Pop<S, P[Action]['tokens']>[0],
            ToString<T[Last<Pop<S, P[Action]['tokens']>[0], string>][P[Action]['name']][1]>],
          [...Pop<B, P[Action]['tokens']>[0], { id: Action, nodes: Pop<B, P[Action]['tokens']>[1] }],
          P[Action]['name'] extends 'G' ? true : false, Tokens
        >
      : never : never
    : never
```

完事之后还需要对抽象语法树求值。

```typescript
type CastEval<N, T> = Eval<N> extends T ? Eval<N> : never 
type Eval<N> = N extends Node
  ? N['id'] extends 1 ? Eval<N['nodes'][0]>
  // @ts-ignore
  : N['id'] extends 2 ? Add<CastEval<N['nodes'][0], number>, CastEval<N['nodes'][2], number>>
  : N['id'] extends 3 ? Subtract<CastEval<N['nodes'][0], number>, CastEval<N['nodes'][2], number>>
  : N['id'] extends 4 ? Eval<N['nodes'][0]>
  : N['id'] extends 5 ? Multiply<CastEval<N['nodes'][0], number>, CastEval<N['nodes'][2], number>>
  : N['id'] extends 6 ? DividedBy<CastEval<N['nodes'][0], number>, CastEval<N['nodes'][2], number>>
  : N['id'] extends 7 ? Eval<N['nodes'][0]>
  : N['id'] extends 8 ? Eval<N['nodes'][1]>
  : N['id'] extends 9 ? N['nodes'][0] extends NumberToken ? N['nodes'][0]['value'] : never
  : never : N
```

TypeScript 已经觉得递归太深不开心了，我们这里让他闭嘴。加减乘除直接抄了上面的 lisp 解释器的加减乘除。然后我们就得到了一个类型系统上的计算器。
​
![类型体操](/img/6.png)