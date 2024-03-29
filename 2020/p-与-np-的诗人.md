---
title: "P 与 NP 的诗人"
date: "2020-03-26"
categories: 
  - "作"
tags: 
  - "math"
---

如果我的记忆没有偏差，《吞噬者》是刘慈欣在2001年发表在《科幻世界》上的一篇短篇小说。这篇小说讲述了人类与外星人抗衡的故事，虽然篇幅短小，但写得波澜壮阔。

之后不久，他写了这篇小说的续篇《诗云》。《诗云》中出现了一个更高级文明，一个可以为所欲为的神级文明，顺手就把外星文明给灭了。听起来是不是很像《地球往事》的套路。

神级文明和人类接触后，了解到了中文古诗这一艺术形式，并倾心于此。李白被神级文明视为最为伟大的诗人，也成为神级文明要超越的目标。而他们的做法是：

> 我要写出所有的五言和七言诗，这是李白所擅长的；另外我还要写出常见词牌的所有的词！你怎么还不明白？我要在符合这些格律的诗词中，试遍所有汉字的所有组合！

> 事实上我终结了诗词艺术，直到宇宙毁灭，所出现的任何一个诗人，不管他们达到了怎样的高度，都不过是个抄袭者，他的作品肯定能在我那巨大的存贮器中检索出来。

这听起来确实是一番值得干一票的伟大事业。为了完成这项工作，大刘让神级文明毁灭了几颗恒星来获取物质和能量。

然而，这些一切都毫无意义。

在继续讨论之前，我需要先介绍一个概念：迭代器(`iterator`)，这是编程语言中常见的一个概念。是的，编程。但稳住，别慌，不难。

假设你有一个机器人，可以判断一个数字是不是平方数（或者随便什么数）。如果你想知道 1 到 100 中有多少数字是平方数，就可以提供一个数字列表给他。他每次从列表中拿一个数字，告诉你是不是平方数，然后再拿下一个，以此往复。

如果你想让他在 1 到 10000 中寻找平方数，你就需要准备10000 个数字给他。但你意识到你并不需要提前准备这个列表。你只需要告诉他，每次处理完一个数字，就把这个数字加一，然后再处理一次就可以了。

这就是迭代器。和列表不同，迭代器里放的不是元素，而是根据当前元素生成下一个元素的规则（比如加一）。

好了，我们回过头来看神级文明做的事情。汉字的数量非常有限，比如常用汉字大约有 3000 字左右。如果我们要用常用汉字写下所有的五言绝句（20 字），则有 3000^20 首，这还不包括七言和律诗，甚至古体诗。因此神级文明才需要如此兴师动众来做成这件事。

但如同我们前面迭代器的例子，这件事是不必要的。要写出所有诗歌规律并不复杂，一个编程的初学者就可以很快写出类似的「迭代器」。实际上，这些诗歌只是汉字的排列组合，因此无论是不是把它们写出来，它们都已经存在，如同你不需要写出任何一个数字写出来就知道这个数字肯定存在一样。

你可以想象，神级文明写下来的诗歌中充斥着大量的垃圾。毫无意义的文字组合必然占据绝对多数，剩下为数不多的部分中也必然充斥着毫无美感可言的作品。我们前面说过，诗歌如同数字，所有的组合早已存在。一个伟大的诗人之所以伟大，并非因为他们能够写足够的诗歌，而是他们能在浩如烟海的排列组合中，找出最有价值的那些。神级文明通过写出所有诗歌来超越李白的做法，就如同妄图通过写出最多的数字来超越高斯一样，无稽之谈。

> 事实上我终结了诗词艺术

别闹了先生，你除了几颗恒星之外，什么都没有终结。

我们说了这么多，终于要说到标题的内容了： P/NP 问题。

简单地说，P 问题就是可以快速算出答案的问题。P 指的是「多项式时间」。在计算机科学中，「多项式时间」之内能解决的问题可以被视为「快速解决」。NP 问题，就是「不能在多项式时间内解决的问题」，但同时，这个问题的答案可以在多项式时间内被验证是否正确。也就是说，你可能一时半会算不出答案，但只要正确答案在你眼前，你一下就能认出它来。毕竟，如果一个问题即使给出答案也无法在可接受的时间内验证是否正确，那么这个问题对我们来说也是没有意义的。

一个常见的例子是素数分解。给出一个很大的数字，你很难判断这个数字是不是两个素数的乘积；但如果给出两个素数，你可以很快地判断出大数是不是他们的乘积。因此这是一个 NP 问题：我们不能快速解决，但可以快速验证。

回到作诗的问题上：写出一首好诗很难，但给出一首诗你可以快速判断出这首诗是不是「好诗」。因此这也是一个 NP 问题。（这个例子并不严谨）因此，诗人的价值实际上体现在他们在多项式时间内解决了 NP 问题。是的，我在强行扣题。

你也许常常听到一个说法

> P=NP

这是一个猜想，即：

> 所有 NP 问题都有一个对应的 P 问题。

如果这个猜想是正确的，这意味着对于我们现在看来无法快速解决的问题，实际上都存在一个可以快速解决的方案（只是我们还没找到它），这将是一个充满希望的未来；相反，如果这个猜想是错误的，这意味着有一些问题我们几乎永远无法解决，预示着一个绝望的未来。现在的情况是，即没有人能证明，也没有人能证伪。

因此，我们活在一个薛定谔的希望中。

### 2022/7/3 更新

最近开始恢复更新的 Matrix67 刚写了一篇关于 [P与NP 问题的文章](http://www.matrix67.com/blog/archives/7084)。
