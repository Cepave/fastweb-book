# counterType 的使用

寫 plugin script 時，最常用的 counterType 就是 GAUGE。
```"counterType":"GAUGE"```然而還有其它的 counterType 可以使用。 `cpu.idle`, `mem.memfree.percent` 這種常見的 metric 都是使用 GAUGE 這種 counterType


## 可使用的 counterType 列表：
1. GAUGE
2. COUNTER

## [counterType 的語意](http://book.open-falcon.org/zh/usage/data-push.html)
* GAUGE：即用户上传什么样的值，就原封不动的存储
* COUNTER：指标在存储和展现的时候，会被计算为 speed ，即（当前值 - 上次值）/ 时间间隔

## COUNTER 使用實例
`net.if.in.bits/iface=eth1` 就是用 `COUNTER` 做為 counterType 上報的監控項。 `net.if.in.bits/iface=eth1` 的資料來自於
`cat /proc/net/dev` 的輸出。

```
Inter-|   Receive                                                |  Transmit
 face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
eth1:     648       8    0    0    0     0          0         0    26408     380    0    0    0     0       0          0
docker0:    7448     111    0    0    0     0          0         0      648       8    0    0    0     0       0          0
```
以上方的例子， `net.if.in.bits/iface=eth1` 就是會取到 `648 * 8` 的值。這個 648 bytes 是指，電腦自開機以來，**累計**收到了 648 bytes 的資料。

然而，實務上的使用， `net.if.in.bits/iface=eth1` 是用來呈現網卡收封包的**速率**。所以上報這個監控項時，就指定它的 counterType 為 `COUNTER` ，表達這個監控項的本質是一個「**累計值**」 (accumulated value) 。這就是透過指定合適的 counterType 來讓 open-falcon 內部的 RRDtool 來做對應的數值微分運算。