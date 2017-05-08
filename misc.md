## Misc logo

![Misc1-1](https://raw.githubusercontent.com/boke1208/TSCTF2017/master/img/Misc1-1.png)

天枢的logo，用十六进制编辑器打开在图片末尾发现了一个Base64字符串

VFNDVEZ7QmFzZTY0X2RlY29kaW5nISEhfQ==，解码得到flag:TSCTF{Base64_decoding!!!}

---

## Misc  easyCrypto 

简单的crypto，做到这个题的时候有些困了，所以耽误了很久

首先理解pack与unpack这里是用来将【二进制串格式的】四个字符与对应它的整数相互转换用的，而整数则用来后续的加密运算。

```python
def crypto(data):
	return data ^ data >> 16  #将一个数二进制右移16位后与其本身异或，解码函数就是这个函数本身

def encode(datas, iv):
	cypher = []
	datas_length = len(datas)
	cypher += [crypto(datas[0] ^ iv)] #获得cypher里的第一项

	for i in range(1, datas_length):
		cypher += [crypto(cypher[i-1] ^ datas[i])] #由前一个crypto与data异或得到新的一项crypto

	cyphertext = b''
	for c in cypher:
		cyphertext += struct.pack("I", c) #将其转回四位的字符

	return base64.b64encode(cyphertext) #base64编码一下
```

![easyCrypto](https://raw.githubusercontent.com/boke1208/TSCTF2017/master/img/easyCrypto.png)

```python
import struct
import base64

cypher_text = 'DgYiZFttExBafXJPPn8BNhI9cwEhaUMgPmg+IA=='
#cyphertext = base64.b64decode(cypher_text)
cyphertext = [b'\x0e\x06"d',b'[m\x13\x10',b'Z}rO',b'>\x7f\x016',b'\x12=s\x01',b'!iC ',b'>h> ']

flag = b'TSCT'

for i in range(6):
	flag += struct.pack("I", struct.unpack("I", cyphertext[i])[0] ^ struct.unpack("I", cyphertext[i+1])[0] ^ (struct.unpack("I", cyphertext[i+1])[0] >> 16))
print(flag)
```

flag:TSCTF{1ts_a_e4sy_Cr7pt0!!!}

---

## Misc 至尊宝

首先拿到题目的图片文件，根据题目“你的头箍呢”，猜测图片上部被做了手脚。

检测图片文件后，发现不仅图片高度改小了，文件最后还加了个rar包上去。恢复图片后得到：

![Image of zhizhunbao](https://raw.githubusercontent.com/Henryzhao96/TSCTF2017/master/img/misc_zhizunbao.bmp "大圣，你的头上是绿色的！")

取每个色块的 16 进制颜色代码，根据 HINT 猜测这些颜色是 MD5

> 437b93 0db84b 8079c2 dd804a 71936b 5f3ab9 78c64a 766522
> 24e2e2 31e05f ca3571 f262d7 96bed1 ab30e8 a2d5a8 ddee6f

查表得

| MD5                              | Text      |
| -------------------------------- | --------- |
| 437b930db84b8079c2dd804a71936b5f | something |
| 3ab978c64a76652224e2e231e05fca35 | atthe     |
| 71f262d796bed1ab30e8a2d5a8ddee6f | bottom    |

然后我就盯着图片的 bottom 盯了 1 个小时，赵胖，卒。
其实压缩包的密码就是 `somethingatthebottom` 我也很绝望啊！

解压得到一个 pptx 我直接把它拆包一个一个 XML 研究去了，其实也可以把背景上的空白白色图片一个一个拖开，最底下嵌入了一个文件，就是 flag 的 txt。

`可把出题人牛逼坏了.jpg`

-----

## Misc 神秘的文件

这次的 Wireshark 抓包比去年的简单多了（死）

首先从 FTP-DATA 里面得到两个文件

```
MD5(pass) = 24885fdab795c41166d6f0067782dc9f
pass = ['a','h','k','q','y','$','%']
```

还有一个是 zip 压缩包，拿 ARCHPR 跑一下密码组合，秒出密码。md5是啥，不知道不知道。

之后解一下base64，就可以拿到 flag 了。

-----

## Misc ZIPCRC

用脚本去跑 CRC ，可以跑出三个 key txt 的原文。

```
zipcrc_f62487ec8bde25d8b652e076487d49b3.zip / key1.txt : 'EcRy'
zipcrc_f62487ec8bde25d8b652e076487d49b3.zip / key2.txt : 'C3Cd'
zipcrc_f62487ec8bde25d8b652e076487d49b3.zip / key3.txt : 'Ea4Y'
```

组合一下 key3+key2+key1 就是解压密码。

主文件是一个 python 脚本，这道题变成了一道 Crypto 题目。有请我的队友逸飞大佬！ `鼓掌.jpg`

这个题的加密算法我没有仔细去阅读，我把crc的工具丢给他，他把压缩包里的脚本还给我的时候，我正准备去打一会游戏，所以尝试碰撞解法，发现意外的可行。我们知道flag的前几位是TSCTF{，这个算法目测是几位一组，各组之间不互相影响，于是暴力尝试。下面是最后一轮尝试的原码。

注意，因为python历史遗留问题，这个题我把环境切到了py2.7

```python
# -*- coding: utf-8 -*-  
import random, base64  
from hashlib import sha1  

key = 'TSCTF2017'

ctMessage = 'k6QqE3TU2qfqytHIatHD6DUOT+7D6vPXFUofQyF7dXjPhkPX9OnN/W5OxkvMfa0='

def crypt(data, key):  
    x = 0  
    flow = range(256)  
    for i in range(256):  
        x = (x + flow[i] + ord(key[i % len(key)])) % 256  
        flow[i], flow[x] = flow[x], flow[i]  
    x = y = 0  
    out = []  
    for char in data:  
        x = (x + 1) % 256  
        y = (y + flow[x]) % 256  
        flow[x], flow[y] = flow[y], flow[x]  
        out.append(chr(ord(char) ^ flow[(flow[x] + flow[y]) % 256]))  
  
    return ''.join(out)  
  
def tsencode(data, salt, key=key, encode=base64.b64encode):
    data = salt + crypt(data, sha1(key + salt).digest())
    if encode:
        data = encode(data)
    return data  

a = base64.b64decode(ctMessage)
#print(a)
salt = a[:16]

m = range(255)
for i in m:
    for j in m:
        str = 'TSCTF{rc4_1s_n0t_D1f5ic4lt__'+chr(j)+chr(i)+'}'
        if tsencode(str, salt=salt)[:61] == ctMessage[:61]:
            print(tsencode(str, salt=salt))
            print(str)
```

flag:TSCTF{rc4_1s_n0t_D1f5ic4lt__!!}

---

## Misc 四维码

用 PS 读取闪闪烁烁的二维码的每一张图，得到一个 twitter 地址（这年头，不科学上网都没法打 CTF 了）。

> https://twitter.com/pinkotsctf

扫一扫 twitter 唯一的一条推文，拿到一个 base32 解得

> key=qrcode

然后我们打了2小时 switch 。（大误）

终于等到hint，说是要谷歌搜图，于是搜出来一个工具 [Matroschka](https://github.com/fbngrm/Matroschka) 用两个相同的密钥 `qrcode` 解密出来一个图片，上面有一个二维码，扫出来 231 个 0 和 1 ，然后我们又打了 2 小时的 switch 。

接下来有请我的队友 星原 大佬！ `鼓掌.jpg`

最后剩下的就是一个条形码了，用matlab作图即可。

![Misc-four](https://raw.githubusercontent.com/boke1208/TSCTF2017/master/img/Misc-four.png)

---

## Misc 密码学穷举攻击

这道题完成于比赛第一天的深夜，凭借这1000分还在个人榜榜首待了一晚上，很开心。此题数量级较大，单纯的穷举显然没办法获得结果。首先将题目化简为：

> 输入两个数 m1小于2^32 m2小于2^96
> e = m1 按位异或 m2 % 16^6
> c1 = (m1^e) % (2^32-1)
> c2 = (m2^e) % (2^32-1)
>
> 已知
> c1 = 0x84A721F4
> c2 = 0x1133086
> 求m1 m2

我们注意到，此题格式非常像RSA加密算法，m ^ e = c mod n，对称的：c ^ d = m mod n，我们尝试类似的做法，设存在一个d = 2^16*a + b满足c ^ d = m mod n，带入运算后得到m1，m2。将m1，m2异或得到e，再检验这个e是否正确。

模数2^32-1可分解为 3\*5\*17\*257\*65537，根据欧拉定理可知 φ(2^32-1) = 2\*4\*16\*256\*65536 = 2^16，与 c2 互质，则c2 ^ d = c2 ^ b mod 2^32-1，这样可以简便运算

```python
import base64
import math

c1 = 0x84A721F4
c2 = 0x1133086
mod = 2**32-1

print(pow(3, (2**16), mod))

for b in range(1, 2**16):
    for a in range(1, 2**8):
        m2 = pow(c2, b, mod)
        m1 = pow(c1, (2**16)*a+b, mod)

        e = (m1 ^ m2) % 2**24
        if pow(m1, e, mod) == c1 and pow(m2, e, mod) == c2:
            print(m1,m2)
```

得到结果m1：F59AAB33，m2：9BA432B8