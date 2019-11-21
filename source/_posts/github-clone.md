---
title: github clone慢的解决办法（转载）
date: 2019-11-21 11:07:23
cover: http://qiniu.zzzz1997.com/c7ed878f4ba13f0d89458c73db85b6b5.gif
categories: 
- 技术
tags:
- Github
---

从github下载项目下来，由于项目提交历史过多等各种原因导致文件太大，clone的时候非常的慢，或者直接出现

<font color='#ff0000'>
error: RPC failed; curl 18 transfer closed with outstanding read data remaining

fatal: The remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
</font>

报错，终止下载

<!--more-->

**两种办法：**

1、修改hosts文件，增加映射，这样可以加快clone速度：

```txt
151.101.44.249 github.global.ssl.fastly.net
192.30.253.112 github.com
```

2、避免报错导致下载终止：在clone后面加上参数:`--depth 1`，设置clone深度为1，来解决这个问题

```bash
git clone https://github.com/xxx/xxx.git  --depth 1
```

（转载自：[git clone克隆项目太慢，或者直接导致克不下来的解决办法](https://www.cnblogs.com/moonLightcy/p/11812757.html)）
