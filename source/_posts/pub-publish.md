---
title: dart pub上传失败如何解决（转载）
date: 2019-12-04 17:54:41
cover: http://qiniu.zzzz1997.com/c83d70cf3bc79f3d068c6661b6a1cd11728b2976.png
categories: 
- 技术
tags:
- Dart
---

问题： Flutter Exception: Pub will wait for a while before trying to connect again.

<!--more-->

1.设置终端代理

```bash
export http_proxy=http://127.0.0.1:1080;
export https_proxy=http://127.0.0.1:1080;
```

2.使用curl google.com测试是否连通

```bash
curl google.com
```

3.禁用设置的镜像源

```bash
unset FLUTTER_STORAGE_BASE_URL;
unset PUB_HOSTED_URL;
```

4.检查是否可上传

```bash
flutter packages pub publish --dry-run --server=https://pub.dartlang.org
```

5.上传

```bash
flutter packages pub publish --server=https://pub.dartlang.org
```

（转载自：[dart pub上传失败如何解决](https://www.cnblogs.com/meetqy/p/11250024.html)）