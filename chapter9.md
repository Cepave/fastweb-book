# plugin script 範例

## 範例 1
### 監控項 

`net.if.in.bits/iface=gateway` 和 `net.if.out.bits/iface=gateway` 是用來取得對外網卡 (gateway) 速率的監控項。會需要客制化 plugin script 來實現。


### 輸出
```
[
    {
        "counterType": "COUNTER",
        "endpoint": "owl-docker",
        "metric": "net.if.in.bits",
        "step": 60,
        "tags": "iface=gateway",
        "timestamp": 1481192881,
        "value": 5952843096
    },
    {
        "counterType": "COUNTER",
        "endpoint": "owl-docker",
        "metric": "net.if.out.bits",
        "step": 60,
        "tags": "iface=gateway",
        "timestamp": 1481192881,
        "value": 704075232
    }
]

```

### 原始碼

```
#/bin/bash

s='{"endpoint":"%s","tags":"iface=gateway","timestamp":%s,"metric":"%s","value":%s,"counterType":"COUNTER","step":60}\n'
cat /proc/net/dev \
| awk -F: -v iface=$(ip r | awk '/^default/{print $5}' | head -1) '{if($1~iface) { print $2}}' \
| awk '{print $1, $9}'  \
| awk '{inbits=$1*8;outbits=$2*8;print "net.if.in.bits", inbits; print "net.if.out.bits", outbits}' \
| awk -v date="$(date +%s)" -v hostname="$(hostname -s)" '{print hostname" "date" "$0}' \
| awk -v format=$s '{printf(format,$1,$2,$3,$4) }' \
| awk -v size=2 'BEGIN{print "["}{if(NR<size){print $0","} else {print $0}}END{print "]"}' \
| python -mjson.tool

```

## 範例 2
### 監控項

`vmstat.procs.r` 和 `vmstat.procs.b` 是 vmstat 指令可以取得的監控項。然而， `vmstat.procs.r` 在取值時，必須要考慮有時候會有瞬間有大量的 runnable processes ，這種極端值會導致採樣的 bias 。

### 輸出 

```
[
    {
        "counterType": "GAUGE",
        "endpoint": "owl-docker",
        "metric": "vmstat.procs.r",
        "step": 60,
        "tags": "",
        "timestamp": 1481789576,
        "value": 0.375
    },
    {
        "counterType": "GAUGE",
        "endpoint": "owl-docker",
        "metric": "vmstat.procs.b",
        "step": 60,
        "tags": "",
        "timestamp": 1481789576,
        "value": 0
    }
]

```
### 原始碼

```
#/bin/sh

s='{"endpoint":"%s","tags":"","timestamp":%s,"metric":"%s","value":%s,"counterType":"GAUGE","step":60}\n'
# first line takes values from vmstat
# second line sorts and removes the biggest and smallest
# calculates the average
count=10
vmstat 1 $count | tail -n$count |  awk '{print $1, $2}' \
| sort -nk1 | sed -e '1d' -e '$d' \
| awk '{r+=$1; b+=$2} END{print "vmstat.procs.r", r/NR;print "vmstat.procs.b", b/NR}' \
| awk -v date="$(date +%s)" -v hostname="$(hostname -s)" '{print hostname, date, $0}' \
| awk -v format=$s '{printf(format,$1,$2,$3,$4) }' \
| awk -v size=2 'BEGIN{print "["}{if(NR<size){print $0","} else {print $0}}END{print "]"}'
```