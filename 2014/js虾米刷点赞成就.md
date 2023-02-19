---
title: "js虾米刷点赞成就"
date: "2014-04-08"
categories: 
  - "技"
tags: 
  - "geek"
  - "javascript"
  - "js"
  - "点赞"
  - "脚本"
  - "虾米"
---

今天发现虾米有一个点赞成就，达到成就后可以开通VIP

虽然我几乎用不到VIP，但是这个给好友状态点赞的感觉还挺好拿的，但是三星成就就需要100个赞……嗯，用 js 刷一下好了。

先打开好友状态页面，往下滚动尽可能加载出来更多的状态，然后在chrome控制台执行下面js代码：

```
i=1;
setInterval("i++;$('.comment_like')[0].children[0].click();$('.item_wrap')[i].click();alert(i)",5000)
```

```
print('test')
```

嗯，由于需要等待ajax，所以每个赞需设置为等待5s。去干点别的等等好了╮(╯▽╰)╭
