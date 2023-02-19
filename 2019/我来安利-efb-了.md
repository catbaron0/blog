---
title: "我来安利 EFB 了"
date: "2019-04-29"
categories: 
  - "技"
tags: 
  - "telegram"
  - "wechat"
---

我知道看我 blog 的人中有人用过 [Telegram](https://telegram.org/)，比如[@水八口](https://shuiba.co/) 曾经就把自己的 Telegram 账号堂而皇之地挂在自己的主页上。

Telegram 是俄罗斯人写的免费即时通讯软件。从最开始，Telegram 就把「加密」当做产品重点。这导致了两个结果。

1. 被学习强国化。
2. 被大量币圈人事使用。

顺便说一句，因为国内币圈人士大量群发广告消息，Telegram 限制了使用 +86 电话注册的用户主动发起私聊的权限。但这都不能影响 Telegram 优秀的使用体验。

1. 界面干净。无广告，没有摇一摇朋友圈也没有今日看点。聊天也不会掉金币。
2. 功能齐全。一款 IM 该有的功能基本上都有。URL 预览，**已发出消息的编辑/删除**，@用户，回复/转发消息。语音/视频消息，（海量的）免费 sticker。**语音消息有进度条**。
3. 全平台全内容同步。手机上写了一半的消息可以在电脑上接着写。
4. 多平台同时登录。电脑/手机/iPad/网页 每种终端只能登录一台？这什么狗屎逻辑。

丰富的机器人接口，可以让人轻松地写一个通信机器人。这给了 Telegram 很大的扩展性。 我知道这么好用的东西你是不会用的。因为你身边没人用。

企鹅的通讯软件最好用的部分是它的用户，除了「大家都在用」这一点之外，Wechat 简直一无是处。

虽然Telegram 上有一些有意思的群，你可以加群来玩。

但终究还是要回去用 wechat。

所以我今天来安利 EFB 了。

我前面提到了 Telegram 的机器人，有人就用它写了一个框架。[EFB (ehforwarderbot)](https://ehforwarderbot.readthedocs.io) 是这样一个项目，它希望能用 Telegram 来接管所有的其他服务的消息。 ![EFB 的框架](https://ehforwarderbot.readthedocs.io/en/latest/_images/EFB-docs-0.png) 在这个框架下，你可以用它来代为收发其他平台的聊天消息。EFB 本身只是一个框架，在此基础之上可以针对某个平台开发独立的插件（channel）来管理消息。EFB 的作者提供了 Wechat（[EWS](efb-wechat-slave)，利用了网页版微信 ）和 Facebook Messager（[EFMS](https://github.com/blueset/efb-fb-messenger-slave)） 的插件。另一位开发者开发了针对 QQ 的（[EQS](https://github.com/milkice233/efb-qq-slave)）插件。

我已经用了很久的 EWS，久到不记得什么时候开始用的了。EWS 不只是接管微信的通信，甚至有了增强。比如你可以在 Telegram 端直接编辑已经发出的消息，在微信端则体现为撤回+重发。你可以回复某条消息，在微信端则体现为在你的回复前插入消息的引用。可以说，EWS 非常明显地提升了我的网络生活幸福感。

我之所以忽然想起来写安利，是因为我刚写了一个中间件。

我有一个很热闹的微信群，我很少讲话，只是偶尔会看他们的讨论内容。但是有些人会频繁地发表情包。其频繁程度已经影响了正常消息的阅读，我不得不在表情包的间隙寻中找有意义的只言片语。而且一些人在用表情包的时候可能根本没考虑过用法是不是合适，如果表情包不能传达正确的表情，那就只是普通的沙雕图片而已。

我实在不堪其扰，终于决定自己写了一个 EFB 上的中间件（[MessageBLocker](https://github.com/catbaron0/efb-msg_blocker-middleware)），用来对消息进行过滤。你看，EFB 提供的是解决问题的可能性。

![](https://i.loli.net/2019/04/29/5cc6cf20388e1.jpeg)

图中上面是微信收到的消息。下面是 Telegram 收到的消息。

于是世界又重新变得美好起来。
