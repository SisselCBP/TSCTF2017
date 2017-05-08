## Misc logo

天枢的logo，用十六进制编辑器打开在图片末尾发现了一个Base64字符串

VFNDVEZ7QmFzZTY0X2RlY29kaW5nISEhfQ==，解码得到flag:TSCTF{Base64_decoding!!!}



## Misc  easyCrypto 

简单的crypto，做到这个题的时候有些困了，所以耽误了很久

首先理解pack与unpack这里是用来将四个字符与一个整数相互转换用的，而整数则用来后续

```python
def crypto(data):
	return data ^ data >> 16 

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

