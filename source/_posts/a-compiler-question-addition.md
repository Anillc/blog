---
title: 续-一个编译原理问题
date: 2021-01-29 05:07:04
---

前天晚上晚自习又摸鱼想了一下  

再次考虑这个文法  

<!-- more -->

<p>
$$ \begin{aligned}
&1.\ S \to A a \\
&2.\ A \to a \\
&3.\ A \to \epsilon
\end{aligned} $$
</p>

上次说到这个文法不是一个LL文法，这次我闲的蛋疼，把他的LR表写出来了  

|   | a | $ | A | S |
| - | - | - | - | - |
| 0 | s3/r3 |   | 2 | 1 |
| 1 |   | acc |   |   |
| 2 | s4 |   |   |   |
| 3 | r2 |   |   |   |
| 4 |   | r1 |   |   |

可以看到，LR表中同样有冲突，但是这却不属于二义性，如果直接抛弃一个操作那么就会出现错误  

之前我一直没有理解到GLR的作用，这个就体现了LR的局限性，如果使用GLR就能正确分析文本返回一个唯一的结果  

我们再来考虑一个文法  

<p>
$$ \begin{aligned}
&If \to if\ B\ Elseif\ Else \\
&Elseif \to else\ if\ B\ Elseif \\
&Elseif \to \epsilon \\
&Else \to else\ B \\
&Else \to \epsilon
\end{aligned} $$
</p>

这是一个if语句的简化版文法，这里就遇到了冲突，如果只靠LL(1)或者LR(1)是没有办法进行很好的处理的  

但是如果我们优化一下文法  

<p>
$$ \begin{aligned}
&If \to if\ B\ Else \\
&Else \to else\ B \\
&Else \to else\ If \\
&Else \to \epsilon
\end{aligned} $$
</p>

这样不就解决冲突了吗。而且如果我们把if语句当做B的一部分，文法还能进一步简化

总之编译原理这玩意儿还是挺有意思的XD  
