---
title: 混一次更新（用curl进行测速）
tags:
  - Linux
  - 代理
  - 脚本
id: 509
categories:
  - 技
date: 2016-07-04 15:37:11
---

前文介绍过一个[在海外如何翻墙回国内的代理配置方法](http://catbaron.com/blog/2015/12/15/%e6%b5%b7%e5%a4%96%e7%94%a8%e6%88%b7%e7%9a%84%e5%9b%bd%e5%86%85%e5%9c%a8%e7%ba%bf%e9%9f%b3%e4%b9%90%e6%9c%8d%e5%8a%a1%e4%bd%bf%e7%94%a8%e6%8c%87%e5%8d%97/)。然后又写了一个[自动抓取免费代理服务器地址的脚本](http://catbaron.com/blog/2015/12/19/%e8%87%aa%e5%8a%a8%e8%8e%b7%e5%8f%96%e5%9b%bd%e5%86%85%e4%bb%a3%e7%90%86ip/)。

这个脚本是有测速的，但是之前用的是ping测速，这就有两个问题

1.  ping 不稳定，毕竟不是TCP连接，所以这个延时不准确
2.  ping的是从自己的电脑到代理直接的延时，而非到目的地址的延时。

不过当时懒，就这么用了。

最近感觉这个功能不好用还不如没有，于是用curl代替ping重新测了一下速度。更新的脚本如下：<!--more-->

    #!/bin/sh
    date
    echo -n &gt; ip.txt
    echo -n &gt; ip_sort.txt
    for i in $(seq 1 3)
    do
    echo "reading page "$i"..."
    url='http://proxy-list.org/english/search.php?search=CN&amp;country=CN&amp;p='$i
    for ip64 in $(curl --silent $url | grep -P "Proxy\('.*'\)" | cut -d"'" -f2)
    do
    ip_port=$(echo $ip64|base64 -d )
    ip=$(echo $ip_port|cut -d":" -f1)
    #time=$(ping -c1 $ip|grep from|cut -d" " -f7|cut -d"=" -f2)
    time=$(curl -m 3 -x $ip_port -o /dev/null -s -w "%{time_total}\n" "http://img.xiami.net/images/album/img23/78523/21003555471467278253_5.jpg")
    echo $ip_port":"$time &gt;&gt; ip.txt
    done
    done
    echo "sorting..."
    cat ip.txt|sort -t: -k3 -nu|grep -v -P :$ &gt; ip_sort.txt
    ip=$(head -n1 ip_sort.txt|cut -d":" -f1,2)
    sed -i-e 's/\([0-9]\{1,3\}\.\)\{3\}[0-9]\{1,3\}:[0-9]\{1,\}/'$ip'/g' SwitchyPac.pac
    echo "========"`</pre>

    执行脚本后，会在当前目录生成一个ip_sort.txt 文件。内容类似如下：

    <pre>`115.25.138.245:3128:0.341
    112.81.157.119:81:0.352
    106.48.49.6:80:0.361
    113.205.43.182:8888:0.373
    221.10.85.63:808:0.386
    183.207.129.190:80:0.397
    117.81.68.12:808:0.400
    114.103.31.57:8998:0.402
    218.18.5.1:8118:0.404
    117.135.251.74:80:0.408
    111.59.117.102:8118:0.433
    112.239.236.229:8888:0.438

每行三列，分别是IP，端口和相应时间。

哦对了，这里的时间是curl从本机通过代理访问xiami.com一张图片的时间。奇怪的是虾米与网易云不同，就算不用代理也可以直接访问他们的静态资源。我猜他们是在网页里面的JavaScript中判断Ip地址的。没仔细看，就这么用吧。

如果你按照开头提到的两篇文章配置了SwitchyPac.pac且保存在当前目录，它会自动更新里面的代理IP和端口。

就这样。