# MISC

## 至尊宝

首先拿到题目的图片文件，根据题目“你的头箍呢”，猜测图片上部被做了手脚。

检测图片文件后，发现不仅图片高度改小了，文件最后还加了个rar包上去。恢复图片后得到：

![Image of zhizhunbao](https://raw.githubusercontent.com/Henryzhao96/TSCTF2017/master/img/misc_zhizunbao.bmp "大圣，你的头上是绿色的！")

取每个色块的 16 进制颜色代码，根据 HINT 猜测这些颜色是 MD5

> 437b93 0db84b 8079c2 dd804a 71936b 5f3ab9 78c64a 766522
24e2e2 31e05f ca3571 f262d7 96bed1 ab30e8 a2d5a8 ddee6f

查表得

 MD5 | Text
 ----|----
 437b930db84b8079c2dd804a71936b5f | something
 3ab978c64a76652224e2e231e05fca35 | atthe
 71f262d796bed1ab30e8a2d5a8ddee6f | bottom

然后我就盯着图片的 bottom 盯了 1 个小时，赵胖，卒。
其实压缩包的密码就是 `somethingatthebottom` 我也很绝望啊！

解压得到一个 pptx 我直接把它拆包一个一个 XML 研究去了，其实也可以把背景上的空白白色图片一个一个拖开，最底下嵌入了一个文件，就是 flag 的 txt。

`可把出题人牛逼坏了.jpg`

-----

## 神秘的文件

这次的 Wireshark 抓包比去年的简单多了（死）

首先从 FTP-DATA 里面得到两个文件

```
MD5(pass) = 24885fdab795c41166d6f0067782dc9f
pass = ['a','h','k','q','y','$','%']
```

还有一个是 zip 压缩包，拿 ARCHPR 跑一下密码组合，秒出密码。md5是啥，不知道不知道。

之后解一下base64，就可以拿到 flag 了。

-----

## ZIPCRC

用脚本去跑 CRC ，可以跑出三个 key txt 的原文。

```
zipcrc_f62487ec8bde25d8b652e076487d49b3.zip / key1.txt : 'EcRy'
zipcrc_f62487ec8bde25d8b652e076487d49b3.zip / key2.txt : 'C3Cd'
zipcrc_f62487ec8bde25d8b652e076487d49b3.zip / key3.txt : 'Ea4Y'
```

组合一下 key3+key2+key1 就是解压密码。

主文件是一个 python 脚本，这道题变成了一道 Crypto 题目。有请我的队友逸飞大佬！ `鼓掌.jpg`

-----

## 四维码

用 PS 读取闪闪烁烁的二维码的每一张图，得到一个 twitter 地址（这年头，不科学上网都没法打 CTF 了）。

> https://twitter.com/pinkotsctf

扫一扫 twitter 唯一的一条推文，拿到一个 base32 解得

> key=qrcode

然后我们打了2小时 switch 。（大误）

终于等到hint，说是要谷歌搜图，于是搜出来一个工具 [Matroschka](https://github.com/fbngrm/Matroschka) 用两个相同的密钥 `qrcode` 解密出来一个图片，上面有一个二维码，扫出来 231 个 0 和 1 ，然后我们又打了 2 小时的 switch 。

接下来有请我的队友 boke 大佬！ `鼓掌.jpg`