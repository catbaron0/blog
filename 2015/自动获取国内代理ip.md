---
title: "自动获取国内代理IP"
date: "2015-12-19"
categories: 
  - "技"
tags: 
  - "linux"
  - "代理"
  - "脚本"
---

前面写了一个用[代理翻回国内的指南](http://catbaron.com/blog/2015/12/15/%e6%b5%b7%e5%a4%96%e7%94%a8%e6%88%b7%e7%9a%84%e5%9b%bd%e5%86%85%e5%9c%a8%e7%ba%bf%e9%9f%b3%e4%b9%90%e6%9c%8d%e5%8a%a1%e4%bd%bf%e7%94%a8%e6%8c%87%e5%8d%97/ "代理翻回国内的指南")，需要从proxy-list.org这里找免费代理IP。  
这里拿到的IP不稳定，所以总要去重新查询，回来更新pac文件。于是写了个脚本：

- 查询免费
- ping测
- 找出来最快的更新pac文件

然后在服务器跑了个定时任务，每小时更新一次。

```
#!/bin/sh
date
echo -n > ip.txt
echo -n > ip_sort.txt
for i in $(seq 1 3)
do
    echo "reading page "$i"..."
    url='http://proxy-list.org/english/search.php?search=CN&country=CN&p='$i
    for ip64 in $(curl --silent $url | grep -P "Proxy\('.*'\)" | cut -d"'" -f2)
    do
        ip_port=$(echo $ip64|base64 -d )
        ip=$(echo $ip_port|cut -d":" -f1)
        time=$(ping -c1 $ip|grep from|cut -d" " -f7|cut -d"=" -f2)
        echo $ip_port":"$time >> ip.txt
    done
done
echo "sorting..."
cat ip.txt|sort -t: -k3 -nu|grep -v -P :$ > ip_sort.txt
ip=$(head -n1 ip_sort.txt|cut -d":" -f1,2)
sed -i-e 's/\([0-9]\{1,3\}\.\)\{3\}[0-9]\{1,3\}:[0-9]\{1,\}/'$ip'/g' SwitchyPac.pac
echo "========"
```

\===============  
更新了一版，时隔半年，修正了那个错误的测速……[看这里](http://catbaron.com/blog/2016/07/04/%e6%b7%b7%e4%b8%80%e6%ac%a1%e6%9b%b4%e6%96%b0%ef%bc%88%e7%94%a8curl%e8%bf%9b%e8%a1%8c%e6%b5%8b%e9%80%9f%ef%bc%89/)
