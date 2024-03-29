---
title: 2023 年末总结
date: 2023-12-22 00:46:00
---

又是一年快过去咯，看看今年都干了些什么吧

<!-- more -->


<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/aplayer/dist/APlayer.min.css">
<script src="https://cdn.jsdelivr.net/npm/aplayer/dist/APlayer.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/meting@2/dist/Meting.min.js"></script>
<meting-js server="netease" type="song" id="1844981660"></meting-js>
<br/>

要说今年都干了些什么事情，好像还挺多的，但是每次写年末总结的时候总是只能想起最近干了些什么事情。

去年年末的时候才刚开始看 RISC-V, 现在已经开始尝试写 CPU 了。在此之前真的是一点硬件设计都没接触过，目前还在慢慢摸索。去年在研究 umlwin32 无果后，决定放弃直接跑操作系统，而是写一个模拟器。我的目标是希望能够给人发一个可执行文件，别人就能跑起一套网络栈，在这上面进行组网。目前这个[模拟器](https://github.com/Anillc/Yuri)已经写好大部分了，应该已经可以运行 Linux 了（还没测试），现在还差一个 virtio 的 block 设备。计划未来还会写点内核模块，用于 bypass 数据包，提高转发效率。

今年知道了一生一芯这个项目。就在知道一生一芯前几天，我还在跟朋友聊天说有机会试试自己做一个 CPU, 没想到机会真的就来了。写硬件跟写软件思维差别挺大的，感觉一点也不优雅。不过看人类跟物理学斗智斗勇还是挺有意思的。在跟着一生一芯做了几周 CPU 过后，又得知了“中国芯”训练营，我做出了一个大胆的决定：休学去参与这个活动。参加这个活动需要至少九个月，所以下学期肯定没办法回学校上课了，因此选择了休学。目前还不知道究竟能不能去，但是已经跟家里和学校商量好了，一旦通过就办理休学。

今年还作为导师，参与了 [OSPP](https://summer-ospp.ac.cn) 这个活动，这个其实也没有什么好说的。

除此之外似乎在写代码这方面没有更多的值得拿出来说的内容了。

在这一年里，还学了一些数学相关的东西。很感兴趣，但是还是挺困难的，需要花很多时间来学习一下。起初是在学习类型论相关的东西，想要尝试设计语言，但是没有想到类型论跟数学关系这么大。今年参与了由 [∞-type Café](https://infinity-type-cafe.github.io/ntype-cafe-summer-school/) 举办的暑期学校，学到了很多东西，不过也只能算作刚开始学。毕竟本来就不是数学专业的，很多数学方面的概念都还没学过。前段时间买了一本拓扑学的教材来看，不过至今还没看几页。

关于弹琴方面的话，今年少练了许多。主要原因是室友嫌我弹琴太吵，我目前在室友在寝室的时候都不能弹琴了。其实经常都会很想弹，但是只会在嫌我吵的室友不在的时候弹琴。网易云年度报告里也有大量指弹，都很想学，但是还是很有难度。最近最想学的松井祐贵的 Sunny Day, 在学了好久才搞清楚了从扫弦开始的第一段的旋律。最后今年学会了等待的风、Wings You are the hero、Memories Clock 这几首，比去年少了很多。

在 10 月和 11 月的时候，我还依次去了伍伍慧、押尾的演奏会，能够亲耳听到尾叔弹琴，也算是一种圆梦了。还拿到了伍伍慧和押尾的签名。伍伍慧的签名签到了专辑上，最后送给了一位朋友；押尾的签到了琴上面。听演奏会的时候还在上学，伍伍慧那会儿正好是运动会，请了好几天假和室友到武汉玩了几天；押尾那会儿周五出发去杭州，周日就回学校。去杭州那三天面了五个人，举行了我一直以来都很想做的申必仪式：让大家在手机上把资料页打开，再对着手机拍照。

年初的时候，我还学会了日麻，在今年花了很多时间来打麻将，打麻将还是很有趣的。学打麻将的初衷其实是希望能够和大家一起打，不过基本上还是一个人在打，还挺伤心的。今年还写了一个麻将的 npm 包，不过 bug 有点多，后面也没什么时间修。写的时候才发现，麻将规则真复杂啊。虽然每条规则都还比较浅显，但是组合起来就很乱，还要处理许多边界情况。

暑假的时候花了大量时间玩戴森球，导致那两个月代码基本没写。不过戴森球最后也没造出来，点满科技树就没有玩了，各个设施也是用的蓝图，并不是自己设计的。戴森球之后玩了尼尔。再之后玩了 Ender Lilies, 虽然我还是玩不来这种要打怪的游戏，但是还是把他打通关了。很不错的游戏，美术、音乐、剧情我都很喜欢。最近还打完了星空列车与白的旅行，很棒的游戏。

去年年末总结的时候说了希望今年能整什么大活，现在看来这个目标确实实现了，做了很多有趣的事情，还做了一些大胆的决定。最后在年末总结部分再附上今年的 GitHub 瓷砖和 Wakatime 瓷砖：

![GitHub 瓷砖](/img/8.png)

![Wakatime 瓷砖](/img/9.png)

关于明年的话，首要目标肯定是完成 CPU 的设计。短期的目标是先把普通的流水线 CPU 做出来，在之后想要尝试设计超标量处理器。也希望明年能有更多的时间弹琴，有空的话还希望能够填一点以前的坑。大概就是这么多。

希望明年也能开开心心的～
