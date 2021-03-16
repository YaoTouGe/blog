---
title: 测试mark down图床功能
date: 2021-03-10 13:35:47
tags:
---

终于开始在pages服务上写文章了，学了一下hexo的用法，挑了一个对眼的主题，现在就差Mark Down图片怎么搞了。
付费方案有不少，什么腾讯云、阿里云、网易云的各种对象存储，有的还有不少免费空间，但是都需要自己注册一个备案域名。
我只是想在Pages上写写文章而已，于是作罢。

后来看到有两个免费方案，微博和B站的，就是不知道稳定性怎么样，要是用着用着挂了，那真是GG了，不管怎么样，先测测看咯。

微博的：

![test](https://tvax1.sinaimg.cn/large/006uGrbwly1goln45myagj33282aonpf.jpg)

B站的

![test](https://images.weserv.nl/?url=https://i0.hdslb.com/bfs/article/1ef201715d96f76d115ae78a74531dbac299c2f8.jpg)

目前看着微博的似乎靠谱一点，需要登录自己账号，B站的都不用登录，也不知道怎么办到的。希望能多用一段时间吧，这就涉及到我要随时把自己的图片给备份好了，打算先归档，放到OneDrive里面。

写到这里，突然想到，OneDrive不是也有文件共享链接的功能吗！何不尝试一下用OneDrive做图床呢！如下：

![test](https://bn1305files.storage.live.com/y4mAQsk8ihAJqLBH3ykNKbg5RPVUEmLaYif6LOroE9sottNo05DlwGROWaznfF0XuoEo0Qa2wfWU27NsRVELzn7Inl5UtgNDu5OeyQ9p1vmcfkCgFen1bOWjQcHLT8AHvrWfq0CmwyS_4ggYqyd0FQH-Voxlh9VuyKbKw8N3d6V6SSBbTdyLzEF0r3deL-U5vAH?width=3968&height=2976&cropmode=none)

实测了一下，确实可以，不过有两步，把文件放进OneDrive之后，先分享得到一个链接，但是这个链接并不能直接使用。于是还需要第二步，打开OneDrive的分享链接，选择顶部的嵌入，这样就可以生成一个能直接访问的图片链接，直接插入Mark Down即可。两步也不算很麻烦，但是OneDrive的页面有时候需要梯子才能打开，所以还是有些不便。就稳定性而言，绝对是完胜上面的B站和微博了，不担心突然就跪了。

既然OneDrive可以，那百度云盘可不可以呢（邪恶笑）。尝试了一下百度云的分享链接：

![test](https://thumbnail0.baidupcs.com/thumbnail/282723864o221f917c8576dae56ae40e?fid=2772381170-250528-188088861539946&time=1615870800&rt=sh&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-mjVOsOz%2FsblDFyW0F4kHDlH6R7U%3D&expires=8h&chkv=0&chkbd=0&chkpc=&dp-logid=575938486921483110&dp-callid=0&file_type=0&size=c710_u400&quality=100&vuk=-&ft=video)

也可以！网络稳定性来讲，比OneDrive强，可是有个缺点，百度网盘分享链接里面只能复制预览图片的网址，而非原图，对于高清大图来讲就不太友好了，也不知道有没有办法能够直接访问百度网盘原图链接的。后面有时间再继续探索一下，目前看OneDrive是个相当不错的选择。