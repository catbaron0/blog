---
title: "一个基于Jquery的打字效果插件TypingCat"
date: "2011-12-21"
categories: 
  - "技"
tags: 
  - "javascript"
---

前段时间，要写一个页面，需要用到打字效果。在网上找到几个基于jQery的插件，但是有些太复杂，有些不太可控。于是干脆就自己写了一个。  
用法和效果都比较简单，直接把html的代码写出来，看注释应该就知道怎么用了首先需要在 `<head></head>` 中加载必要的插件

```html
<head>
…
<script type=”text/javascript” src=”js/jquery.js”></script>
<script type=”text/javascript” src=”js/TypingCat.js”></script>
</head>
```

而html的body中有如下标签内容（用来承载打印效果的位置）：

```html
<body>
<div><a href=’#’>back</a></div>
<div id=’pra1′>
<div id=’word1′></div>
<div id=’word2′></div>
<div id=’word3′></div>
</div>
<div id=’pra2′>
<div class=’word1′></div>
<div class=’word2′></div>
</div>
</body>
```

然后需要做的是，在任意位置加入下面的代码（开始使用插件，示例是放在了head标签里面）：

```html
<head>
<script type=”text/javascript” src=”js/jquery.js”></script>
<script type=”text/javascript” src=”js/type.js”></script>
<script>
$(function(){
l_blink_speed = 300;    //’‘闪烁速度 speed ” blinking
l_blink = 14;           //‘’闪烁次数 times ” blink
l_blink_s = 8;          //‘’闪烁次数（少）times ” blink (fewer)
l_start = 2;            //how long to wait before a new line starts
l_start_q = 1;          //how long to wait before a new line starts (quick)
w_blink = 80;           //汉字打印时间间隔 speed of tying
hide_time = 2000;      //字体层隐藏速度 speed of hiding
wait = 1000;            //weit 3 second before start
typeWriter(“this is Pra1’s 1st words”,”#pra1″,”#word1″,l_start,l_blink_s);
typeWriter(“this is Pra1’s 2nd words”,”#pra1″,”#word2″,l_blink_s,l_blink);
typeWriter(“this is Pra1’s 3nd words”,”#pra1″,”#word3″,l_blink,l_blink_s);
typeWriter(“this is Pra1’s 4nd words”,”#pra1″,”#word3″,l_blink_s,-1);   //retype in the same div. -1 means this pragraph  will disappear after finishing typing
typeWriter(“this is Pra1’s 1st words”,”#pra2″,”.word1″,l_start,l_blink_s);
typeWriter(“this is Pra1’s 2nd words”,”#pra2″,”.word2″,l_blink_s,-2);   //-2 means this pragraph will not disappear and ” will blink without stop
})
</script>
</head>
```

真正产生效果的是typeWriter(str,pra\_id,div\_id,wait\_before,cur\_wait)  
参数的意义分别是：  
str：需要打印的文字，注意最好不要打印‘’这个符号，因为它作为光标使用了  
pra\_id：段落的id或者类名，从例子中可以看出，如果是传入id则用"#id"的形式，如果传入样式的类名则用“.class”的形式。文字的消失是以段落为单位的。  
`div_id`：每一行用div层引起来，这里是段落中各个div的id或者样式类名。用法同上。  
`wait_before`：上一行的光标闪烁次数，也就是说上一行打印结束后多久才开始打印本行。  
`cur_wait`：本行结束是，光标的闪烁次数。  
上面的一些变量有说明，可以自己修改调整各种效果的时间和速度。  
效果可以看[这里](http://catbaron.tk/TypingCat/example.html)  
下载在[这里](http://catbaron.tk/TypingCat/js/TypingCat.js)
