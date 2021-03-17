---
title: 测试mark down图床功能
date: 2021-03-10 13:35:47
tags:
- mark down
---

终于开始在pages服务上写文章了，学了一下hexo的用法，挑了一个对眼的主题，现在就差Mark Down图片怎么搞了。
付费方案有不少，什么腾讯云、阿里云、网易云的各种对象存储，有的还有不少免费空间，但是都需要自己注册一个备案域名。我只是想在Pages上写写文章而已，于是作罢。

后来看到有两个免费方案，微博和B站的，就是不知道稳定性怎么样，要是用着用着挂了，那真是GG了，不管怎么样，先测测看。

微博的：

![test](https://tvax1.sinaimg.cn/large/006uGrbwly1goln45myagj33282aonpf.jpg)

B站的

![test](https://images.weserv.nl/?url=https://i0.hdslb.com/bfs/article/1ef201715d96f76d115ae78a74531dbac299c2f8.jpg)

目前看着微博的似乎靠谱一点，需要登录自己账号，B站的都不用登录，也不知道怎么办到的。希望能多用一段时间吧，这就涉及到我要随时把自己的图片给备份好了，打算先归档，放到OneDrive里面。

写到这里，突然想到，OneDrive不是也有文件共享链接的功能吗！何不尝试一下用OneDrive做图床呢！如下：

![test](https://bn1305files.storage.live.com/y4mAQsk8ihAJqLBH3ykNKbg5RPVUEmLaYif6LOroE9sottNo05DlwGROWaznfF0XuoEo0Qa2wfWU27NsRVELzn7Inl5UtgNDu5OeyQ9p1vmcfkCgFen1bOWjQcHLT8AHvrWfq0CmwyS_4ggYqyd0FQH-Voxlh9VuyKbKw8N3d6V6SSBbTdyLzEF0r3deL-U5vAH?width=3968&height=2976&cropmode=none)

实测了一下，确实可以，不过有两步，把文件放进OneDrive之后，先分享得到一个链接，但是这个链接并不能直接使用。于是还需要第二步，打开OneDrive的分享链接，选择顶部的嵌入，这样就可以生成一个能直接访问的图片链接，直接插入Mark Down即可。两步也不算很麻烦，但是OneDrive的网页版需要梯子才能打开，所以还是有些不便。就数据安全性，绝对是完胜上面的B站和微博了，不担心突然就跪了。

既然OneDrive可以，那百度云盘可不可以呢（邪恶笑）。尝试了一下百度云的分享链接：

![test](https://xacu02.baidupcs.com/file/282723864o221f917c8576dae56ae40e?bkt=en-864c1d195a8f2f4123f8f22d6d5c5d7e0b1c9856977fa9e16dd37b7f7a6d12266a356418b4a85229864d16108b860074d637bb9bd6a36bca5b8e3aa0bbc00d7e&fid=2772381170-250528-188088861539946&time=1615955913&sign=FDTAXUGERQlBHSKfWqi-DCb740ccc5511e5e8fedcff06b081203-nVU4chyHm6FBDkS05sJAvfQtSNQ%3D&to=129&size=3878674&sta_dx=3878674&sta_cs=1&sta_ft=jpg&sta_ct=0&sta_mt=0&fm2=MH%2CXian%2CAnywhere%2C%2Cbeijing%2Ccnc&ctime=1615870827&mtime=1615870827&resv0=-1&resv1=0&resv2=rlim&resv3=5&resv4=3878674&vuk=2772381170&iv=2&htype=&randtype=&newver=1&newfm=1&secfm=1&flow_ver=3&pkey=en-95e9b6e14b667950e2c0aa7aea3bf1eb3e20b0640d5a9e7ac061feff0f01ef5bea159b6324ff6221aec3b9a95776a30eaa9ce1906d00661c305a5e1275657320&expires=8h&rt=sh&r=717144358&vbdid=3365647290&fin=test.jpg&fn=test.jpg&rtype=1&dp-logid=8669208128570586831&dp-callid=0.1&hps=1&tsl=0&csl=0&fsl=-1&csign=oE6AFZG%2FofVu7knzZuEt6HmjNrg%3D&so=0&ut=1&uter=4&serv=0&uc=516136545&ti=c90c91e6ed22f03829f4c621b2edb08f3ba368559903b263&hflag=30&from_type=1&adg=c_9325d1b9d7dbdafd1e89286a1198bdb5&reqlabel=250528_f_4eae437c11366568f15f4ef1d1a46331_-1_a0f2d67f117179d4e6f2d1bf4e74828c&by=themis)

也可以！不用梯子这一点比OneDrive强，原先以为现在百度网盘只能分享私密链接链接，必须用提取码才能访问原始文件，后来发现原来点击下载的任务链接也可以直接用来作为图床链接！不过就是不知道百度这个链接是否是永久的，虽然我选的是永久有效，链接会不会变就不知道了，至少目前是OK的！

总结一下就是，OneDrive官方提供获取直接访问原图的链接，但是需要打开OneDrive网页版需要梯子。百度的目前看来直接复制下载链接就能当图床链接，但官方并没有直接提供这个功能，不知道以后会不会有变化。至于B站和微博的，就变数更多了。