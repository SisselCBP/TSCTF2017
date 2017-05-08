python版本为3.6，部分题目追求效率运行中可能有错误。

### Code 小明的二进制

code的签到题，非常简单，题目给出一个十进制整数n，将其用最少的二进制数看作十进制相加得到。

比如103 需要 101+1+1。易知，只需要判断 给出数字的各位中最大位即可。

50轮后获得flag:TSCTF{this_is_coding}

```python
import socket
import time

HOST, PORT = "10.105.42.5", int(41111)

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((HOST, PORT))

time.sleep(0.1)
response = sock.recv(24)
print(response.decode())
time.sleep(0.1)
response = sock.recv(10240)
number = response.decode().split('\n')[2]
while(1):
    a = list(number)
    mymax = max(a)

    sendBuf = mymax+"\n"
    sock.send(sendBuf.encode())
    print(sendBuf)

    time.sleep(0.1)
    response = sock.recv(10240)
    print(response)
    number = response.decode().split('\n')[3]
```

### Code 泽哥的算数

10s内给出A的B次幂的后四位，快速幂

50轮以后获得flag:TSCTF{Y0uR_m4th_1s_Vry_G0od}

```python
import socket
import time

HOST, PORT = "10.105.42.5", int(42222)

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((HOST, PORT))

time.sleep(0.1)
response = sock.recv(69)
print(response.decode())
time.sleep(0.1)
response = sock.recv(10240)

number = response.decode().split('\n')[2]
k = number.split(' ')
print(k)
while(1):
    # 3 Send the Answer to Server
    a = int(k[0])
    b = int(k[1])
    t = 1
    while(b):
        if b % 2 == 1:
            t = (t * a) % 10000
        a = ((a % 10000) * (a % 10000)) % 10000
        b = b//2
        print(b)

    sendBuf = str(t)+"\n"
    sock.send(sendBuf.encode())
    print(sendBuf)

    time.sleep(0.1)
    response = sock.recv(10240)
    print(response)
    number = response.decode().split('\n')[3]
    k = number.split(' ')

```

### Code Las Vegas

简单的求必胜算法的题目，先手必胜。考察桌上剩B+1个石头的情况，甲无论取1~B任意数量的石子，乙只需取剩下的石子即可获胜。由此可递推得到，谁可以将石子取为B+1的倍数，谁必胜。又由我们是先手，所以只需第一次将石子数量取为B+1的倍数，之后按上述方法完成游戏。遇到开始即为B+1的情况？再跑一次程序。

50轮后获得flag:TSCTF{u\_R\_K!ng\_0f\_L4s\_Vegas}

```python
import socket
import time

HOST, PORT = "10.105.42.5", int(43333)

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((HOST, PORT))

time.sleep(0.1)
response = sock.recv(62)
print(response.decode())
time.sleep(0.1)
response = sock.recv(10240)
number = response.decode().split('\n')[1]
k = number.split(' ')
print(k)
while(1):
    a = int(k[0])
    b = int(k[1])
    k = a//(b+1)
    c = a % (b+1)

    sendBuf = str(c) + "\n"
    sock.send(sendBuf.encode())

    while(1):
        #time.sleep(0.03)
        response = sock.recv(10240)
        a = int(response.decode().split('\n')[0].split(' ')[1])
        c = str(a % (b+1))
        sendBuf = c + "\n"
        sock.send(sendBuf.encode())
        if a == int(c):
            break
    time.sleep(0.1)
    response = sock.recv(10240)
    print(response)
    number = response.decode().split('\n')[2]
    k = number.split(' ')

```

### Code 修路

并查集问题，判断两个地点是否连通，直接用以前的轮子就是好hh

1000轮后获得flag:TSCTF{R04ds\_BeI0ng\_2\_yoU!1!}

```python
import socket
import time

HOST, PORT = "10.105.42.5", int(44444)

class UnionSet(object):
    def __init__(self):
        self.parent = {}

    def init(self, key):
        if key not in self.parent:
            self.parent[key] = key

    def find(self, key):
        self.init(key)
        while self.parent[key] != key:
            self.parent[key] = self.parent[self.parent[key]]
            key = self.parent[key]
        return key

    def join(self, key1, key2):
        p1 = self.find(key1)
        p2 = self.find(key2)
        if p1 != p2:
            self.parent[p2] = p1

    def judege(self, key1, key2):
        p1 = self.find(key1)
        p2 = self.find(key2)
        if p1 == p2:
            return "yes\n"
        else:
            return "no\n"

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((HOST, PORT))

time.sleep(0.1)
response = sock.recv(10240)
print(response.decode())
sendBuf = "\n"
sock.send(sendBuf.encode())

time.sleep(1)

response = sock.recv(10240)
print(response)
mylist = response.decode('ascii').split('\n')

a = UnionSet()
a.__init__()

for i in mylist:
    if i[0]=='-':
        break
    k = i.split(' ')
    a.join(k[0], k[1])

que = mylist[801].split(' ')
m = 0
while(1):
    try:
        print(a.judege(que[0], que[1]).encode('ascii'))
        sock.send(a.judege(que[0], que[1]).encode('ascii'))
        response = sock.recv(10240)
        print(response)
        que = response.decode('ascii').split('\n')[1].split(' ')
    except Exception:
        m += 1 #没用的
```

本届预赛的code题目的难度较去年来说低一些，AK很开心。

