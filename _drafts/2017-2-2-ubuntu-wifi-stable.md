---
layout:         post
title:          Ubuntu 16.04 Wifi经常断开的解决方法
categories:     [Ubuntu, Wifi]
description:    Ubuntu Wifi不稳定的解决方法
keywords:       Ubuntu, Wifi
---

从去年3月装了win10 + ubuntu双系统之后，在ubuntu下wifi一直有各式各样的问题，找了好久相关的解决方法，这两天终于找到了真正有效的solution！在Blog里记录下来分享给大家。


### Wifi症状

* 半年前在学校时联SJTU，可以联上但无法正常上网，也无法ping通任何外网；还好当时还能联SJTU-Web，勉强在宿舍和ylm之外的地方可以上网，但问题是经常断开需要重联。脸好的话关闭wifi之后再联就ok了，脸黑的时候就呵呵了，直接重启不知道多少次。。
* 上个暑假因为自己ubuntu里换了好几次包管理器下载源，在使用过程中还添加了不少后来失效的PPA源，索性删了它重新安装了个ubuntu 16.04的版本，各种配置也都正常了。上学期开学后一个更奇妙的现象发生了，除了学校图书馆之外的所有地方SJTU和SJTU-Web都正常，唯独在图书馆不能上，了解下来是驱动有问题，但具体怎么个解决方法也无从下手。
* 这两天在ubuntu中文社区里终于找到了靠谱的solution来这share下。


### Solution

先说个可能的solution，反正在我这是没


### 引用

* Solution in Ubuntu中文社区: http://forum.ubuntu.org.cn/viewtopic.php?t=476291
* New Wifi Driver Repository: https://github.com/lwfinger/rtlwifi_new/
* 不靠谱的解决方法: http://www.th7.cn/system/lin/201508/123986.shtml