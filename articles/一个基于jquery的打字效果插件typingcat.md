---
title: 一个基于Jquery的打字效果插件TypingCat
tags:
  - coding
  - JavaScript
id: 281
categories:
  - 技
date: 2011-12-21 16:00:53
---

前段时间，要写一个页面，需要用到打字效果。在网上找到几个基于jQery的插件，但是有些太复杂，有些不太可控。于是干脆就自己写了一个。

&nbsp;

用法和效果都比较简单，直接把html的代码写出来，看注释应该就知道怎么用了

首先需要在&lt;head&gt;&lt;/head&gt;中加载必要的插件
> &nbsp;
> 
> &lt;head&gt;
> 
> &nbsp;
> 
> ...
> 
> &nbsp;
> 
> &lt;script type="text/javascript" src="js/jquery.js"&gt;&lt;/script&gt;
> 
> &nbsp;
> 
> &lt;script type="text/javascript" src="js/TypingCat.js"&gt;&lt;/script&gt;
> 
> &nbsp;
> 
> &lt;/head&gt;
> 
> &nbsp;
&nbsp;

而html的body中有如下标签内容（用来承载打印效果的位置）：
> &nbsp;
> 
> &lt;body&gt;
> 
> &nbsp;
> 
> &nbsp;
> 
> &nbsp;
> 
> &lt;div&gt;&lt;a href='#'&gt;back&lt;/a&gt;&lt;/div&gt;
> 
> &nbsp;
> 
> &lt;div id='pra1'&gt;
> 
> &nbsp;
> 
> &lt;div id='word1'&gt;&lt;/div&gt;
> 
> &nbsp;
> 
> &lt;div id='word2'&gt;&lt;/div&gt;
> 
> &nbsp;
> 
> &lt;div id='word3'&gt;&lt;/div&gt;
> 
> &nbsp;
> 
> &lt;/div&gt;
> 
> &nbsp;
> 
> &lt;div id='pra2'&gt;
> 
> &nbsp;
> 
> &lt;div class='word1'&gt;&lt;/div&gt;
> 
> &nbsp;
> 
> &lt;div class='word2'&gt;&lt;/div&gt;
> 
> &nbsp;
> 
> &lt;/div&gt;
> 
> &nbsp;
> 
> &lt;/body&gt;
> 
> &nbsp;
&nbsp;

然后需要做的是，在任意位置加入下面的代码（开始使用插件，示例是放在了head标签里面）：

&nbsp;
> &nbsp;
> 
> &lt;head&gt;
> 
> &nbsp;
> 
> &nbsp;
> 
> &nbsp;
> 
> &lt;script type="text/javascript" src="js/jquery.js"&gt;&lt;/script&gt;
> 
> &nbsp;
> 
> &lt;script type="text/javascript" src="js/type.js"&gt;&lt;/script&gt;
> 
> &nbsp;
> 
> &lt;script&gt;
> 
> &nbsp;
> 
> $(function(){
> 
> &nbsp;
> 
> l_blink_speed = 300;    //’‘闪烁速度 speed '' blinking
> 
> &nbsp;
> 
> l_blink = 14;           //‘’闪烁次数 times '' blink
> 
> &nbsp;
> 
> l_blink_s = 8;          //‘’闪烁次数（少）times '' blink (fewer)
> 
> &nbsp;
> 
> l_start = 2;            //how long to wait before a new line starts
> 
> &nbsp;
> 
> l_start_q = 1;          //how long to wait before a new line starts (quick)
> 
> &nbsp;
> 
> w_blink = 80;           //汉字打印时间间隔 speed of tying
> 
> &nbsp;
> 
> hide_time = 2000;      //字体层隐藏速度 speed of hiding
> 
> &nbsp;
> 
> &nbsp;
> 
> &nbsp;
> 
> wait = 1000;            //weit 3 second before start
> 
> &nbsp;
> 
> typeWriter("this is Pra1's 1st words","#pra1","#word1",l_start,l_blink_s);
> 
> &nbsp;
> 
> typeWriter("this is Pra1's 2nd words","#pra1","#word2",l_blink_s,l_blink);
> 
> &nbsp;
> 
> typeWriter("this is Pra1's 3nd words","#pra1","#word3",l_blink,l_blink_s);
> 
> &nbsp;
> 
> &nbsp;
> 
> &nbsp;
> 
> &nbsp;
> 
> &nbsp;
> 
> typeWriter("this is Pra1's 4nd words","#pra1","#word3",l_blink_s,-1);   //retype in the same div. -1 means this pragraph  will disappear after finishing typing
> 
> &nbsp;
> 
> &nbsp;
> 
> &nbsp;
> 
> typeWriter("this is Pra1's 1st words","#pra2",".word1",l_start,l_blink_s);
> 
> &nbsp;
> 
> typeWriter("this is Pra1's 2nd words","#pra2",".word2",l_blink_s,-2);   //-2 means this pragraph will not disappear and '' will blink without stop
> 
> &nbsp;
> 
> &nbsp;
> 
> &nbsp;
> 
> })
> 
> &nbsp;
> 
> &lt;/script&gt;
> 
> &nbsp;
> 
> &lt;/head&gt;
> 
> &nbsp;
> 
> &nbsp;
> 
> &nbsp;
真正产生效果的是typeWriter(str,pra_id,div_id,wait_before,cur_wait)

参数的意义分别是：

str：需要打印的文字，注意最好不要打印‘’这个符号，因为它作为光标使用了

pra_id：段落的id或者类名，从例子中可以看出，如果是传入id则用"#id"的形式，如果传入样式的类名则用“.class”的形式。文字的消失是以段落为单位的。

div_id：每一行用div层引起来，这里是段落中各个div的id或者样式类名。用法同上。

wait_before：上一行的光标闪烁次数，也就是说上一行打印结束后多久才开始打印本行。

cur_wait：本行结束是，光标的闪烁次数。

&nbsp;

上面的一些变量有说明，可以自己修改调整各种效果的时间和速度。

&nbsp;

效果可以看[这里](http://catbaron.tk/TypingCat/example.html)

下载在[这里](http://catbaron.tk/TypingCat/js/TypingCat.js)

&nbsp;