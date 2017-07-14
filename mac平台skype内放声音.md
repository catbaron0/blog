---
title: mac平台skype内放声音
tags:
  - Geek
  - Mac
  - skype
  - soudflower
  - 录音
  - 技
id: 338
categories:
  - 技
date: 2014-09-03 20:15:26
---

好吧，先说一下需求。

我希望通过skype共享桌面。共享桌面的时候能同时传输电脑的声音。

这样就能用skype做游戏实况了，或者能和朋友一起看电影。

这个需要装soundflower 1.6，soundflower 1.6 的配置参照这里：

[https://www.ptt.cc/bbs/MAC/M.1320517815.A.8EF.html](https://www.ptt.cc/bbs/MAC/M.1320517815.A.8EF.html)

简单点说，就是把电脑的声音作为skype的声音输入。

但是我希望同时我还能通过话筒和朋友讲话。所以需要另一个软件，lineIn。

下载地址在这：

[http://www.macupdate.com/info.php/id/11333/linein](http://www.macupdate.com/info.php/id/11333/linein)

通过这个软件，可以修改声音的输入和输出。这里输入是麦克风，输出改为soundflower，而soundflower又是skype的声音输入，这样麦克风的声音就能传到skpe里面了。

==============================

今天有同学问我具体怎么设置。

稍微讲明百点，其实是这样。

&nbsp;

&nbsp;

lineln不需要特别的配置，打开之后，可以选择声音的输出端，选定之后开启生效就行了。

soundflowerbed的配置，主要是建立多重输出接口。我很久没用，细节记不清楚了。你可以参考我给出台湾PTT社区的帖子链接配置。

========

再次更新。

XOS 10.11 之后soundflower 1.6就不兼容了。好在Github上soundflower
[更新了2.0版本](https://github.com/mattingalls/Soundflower/releases/tag/2.0b2)。需要注意的是，如果你没有装过soundflowerb1.6，你需要先安装这个版本，因为soundflower 2.0 并没有附带soundflowerbed。你需要用soundflowerbed设置多重输出接口。