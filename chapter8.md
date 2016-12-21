# 活用 awk 與函式編程 (functional programming)

open-falcon 的 plugin script 常常用於下列的情境：　
1. 讀取 /proc 下的檔案。這些檔案內容的特色是「表格化」的內容
2. 做一些數值的運算。做一些字串的運算。引入 date 和 hostname
3. 格式化，做成 json 

上述的情境如果使用 awk 來做處理，最容易讓邏輯可以簡化。因為 awk 程式的隱含 (implicit) 語意，就是「開啟檔案(或是串流)，讀每一列、處理」。這邊的隱含語意就已經是「迴圈」 (loop)

活用 awk 來做 plugin script ，可以將本來需要的迴圈邏輯塞入 awk 的隱含語意中。在程式碼中只寫最關鍵的部分。這也是為何用 awk 寫，行數會精簡的原因。

##範例問題
* 輸入： `/proc/sys/net/traffic_counter/status` 這個檔案。
* 輸出： json 格式
* 需要做的計算是： 每一列會對應到一個 `metric` 。 `metric` 對應的 `value` 是將每一列的第 2 個數值乘以 1000 後，再除以第 4 個數值，並做四捨五入。

### `/proc/sys/net/traffic_counter/status` 的內容

```
all_out 730111 3525411980 55177
all_in 1753460 1739877139 55177
multicast_out 28 23660 55177
multicast_in 124 104780 55177
broadcast_out 0 0 55177
broadcast_in 37 21312 55177
local_out 398309 1568063904 55177
local_in 263802 1425914567 55177
private_out 0 0 55177
private_in 0 0 55177
routed_all_out 331774 1957324416 55177
routed_all_in 1489497 313836480 55177
routed_line1_out 331774 1957324416 55177
routed_line1_in 1489497 313836480 55177
routed_line2_out 0 0 55177
routed_line2_in 0 0 55177
routed_line3_out 0 0 55177
routed_line3_in 0 0 55177
routed_other_out 0 0 55177
routed_other_in 0 0 55177
```


###輸出
```
[
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.all.pps","value":13232,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.all.bps","value":105857,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.all.pps","value":31779,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.all.bps","value":254231,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.multicast.pps","value":1,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.multicast.bps","value":4,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.multicast.pps","value":2,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.multicast.bps","value":18,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.broadcast.pps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.broadcast.bps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.broadcast.pps","value":1,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.broadcast.bps","value":5,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.local.pps","value":7219,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.local.bps","value":57750,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.local.pps","value":4781,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.local.bps","value":38248,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.private.pps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.private.bps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.private.pps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.private.bps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.routed_all.pps","value":6013,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.routed_all.bps","value":48103,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.routed_all.pps","value":26995,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.routed_all.bps","value":215959,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.routed_line1.pps","value":6013,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.routed_line1.bps","value":48103,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.routed_line1.pps","value":26995,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.routed_line1.bps","value":215959,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.routed_line2.pps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.routed_line2.bps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.routed_line2.pps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.routed_line2.bps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.routed_line3.pps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.routed_line3.bps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.routed_line3.pps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.routed_line3.bps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.routed_other.pps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.out.routed_other.bps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.routed_other.pps","value":0,"counterType":"GAUGE","step":60},
{"endpoint":"owl-docker","tags":"",timestamp:1481182883,"metric":"traffic.in.routed_other.bps","value":0,"counterType":"GAUGE","step":60}
]
```

###用 awk 寫的範例程式
```
#!/bin/bash 
s='{"endpoint":"%s","tags":"","timestamp":%s,"metric":"%s","value":%s,"counterType":"GAUGE","step":60}\n'  
cat /proc/sys/net/traffic_counter/status \
| awk '{data=gensub(/(.+)_(in|out)/, "\\2 \\1","g",$1);print data" "int($2*1000/$4 + 0.5);print data" "int($2*1000*8/$4 + 0.5)}'\
| awk '{if(NR%2==1){suffix=".pps"}else{suffix=".bps"}{print "traffic."$1"."$2suffix" "$3}}' \
| awk -v date="$(date +%s)" -v hostname="$(hostname -s)" '{print hostname" "date" "$0}' \
| awk -v format=$s '{printf(format,$1,$2,$3,$4) }' \
| awk -v size=$(cat proc-sys-net-traffic_counter-status | wc -l) 'BEGIN{print "["}{if(NR<size*2){print $0","} else {print $0}}END{print "]"}'
```

###程式碼詳析


* 第一行的 awk 做字串分割、重組字串、並且做數值的運算。
* 第二行的 awk 將字串處理成 metric 名稱
* 第三行的 awk 將時間 (timestamp) 、主機名稱 (hostname) 引入資料流
* 第四行的 awk 將表格形式的輸出，轉換成 json 形式的輸出。
* 第五行的 awk 將 json 物件，轉換成 json list 

###中間的運算過程

```

# the first awk out is 
out all 13232
out all 105857
in all 31779
in all 254231
out multicast 1
out multicast 4
in multicast 2
in multicast 18
out broadcast 0
out broadcast 0
in broadcast 1
in broadcast 5
out local 7219
out local 57750
in local 4781
in local 38248
out private 0
out private 0
in private 0
in private 0
out routed_all 6013
out routed_all 48103
in routed_all 26995
in routed_all 215959
out routed_line1 6013
out routed_line1 48103
in routed_line1 26995
in routed_line1 215959
out routed_line2 0
out routed_line2 0
in routed_line2 0
in routed_line2 0
out routed_line3 0
out routed_line3 0
in routed_line3 0
in routed_line3 0
out routed_other 0
out routed_other 0
in routed_other 0
in routed_other 0

# the second awk out is 
traffic.out.all.pps 13232
traffic.out.all.bps 105857
traffic.in.all.pps 31779
traffic.in.all.bps 254231
traffic.out.multicast.pps 1
traffic.out.multicast.bps 4
traffic.in.multicast.pps 2
traffic.in.multicast.bps 18
traffic.out.broadcast.pps 0
traffic.out.broadcast.bps 0
traffic.in.broadcast.pps 1
traffic.in.broadcast.bps 5
traffic.out.local.pps 7219
traffic.out.local.bps 57750
traffic.in.local.pps 4781
traffic.in.local.bps 38248
traffic.out.private.pps 0
traffic.out.private.bps 0
traffic.in.private.pps 0
traffic.in.private.bps 0
traffic.out.routed_all.pps 6013
traffic.out.routed_all.bps 48103
traffic.in.routed_all.pps 26995
traffic.in.routed_all.bps 215959
traffic.out.routed_line1.pps 6013
traffic.out.routed_line1.bps 48103
traffic.in.routed_line1.pps 26995
traffic.in.routed_line1.bps 215959
traffic.out.routed_line2.pps 0
traffic.out.routed_line2.bps 0
traffic.in.routed_line2.pps 0
traffic.in.routed_line2.bps 0
traffic.out.routed_line3.pps 0
traffic.out.routed_line3.bps 0
traffic.in.routed_line3.pps 0
traffic.in.routed_line3.bps 0
traffic.out.routed_other.pps 0
traffic.out.routed_other.bps 0
traffic.in.routed_other.pps 0
traffic.in.routed_other.bps 0

# the third awk output is 
owl-docker 1481183143 traffic.out.all.pps 13232
owl-docker 1481183143 traffic.out.all.bps 105857
owl-docker 1481183143 traffic.in.all.pps 31779
owl-docker 1481183143 traffic.in.all.bps 254231
owl-docker 1481183143 traffic.out.multicast.pps 1
owl-docker 1481183143 traffic.out.multicast.bps 4
owl-docker 1481183143 traffic.in.multicast.pps 2
owl-docker 1481183143 traffic.in.multicast.bps 18
owl-docker 1481183143 traffic.out.broadcast.pps 0
owl-docker 1481183143 traffic.out.broadcast.bps 0
owl-docker 1481183143 traffic.in.broadcast.pps 1
owl-docker 1481183143 traffic.in.broadcast.bps 5
owl-docker 1481183143 traffic.out.local.pps 7219
owl-docker 1481183143 traffic.out.local.bps 57750
owl-docker 1481183143 traffic.in.local.pps 4781
owl-docker 1481183143 traffic.in.local.bps 38248
owl-docker 1481183143 traffic.out.private.pps 0
owl-docker 1481183143 traffic.out.private.bps 0
owl-docker 1481183143 traffic.in.private.pps 0
owl-docker 1481183143 traffic.in.private.bps 0
owl-docker 1481183143 traffic.out.routed_all.pps 6013
owl-docker 1481183143 traffic.out.routed_all.bps 48103
owl-docker 1481183143 traffic.in.routed_all.pps 26995
owl-docker 1481183143 traffic.in.routed_all.bps 215959
owl-docker 1481183143 traffic.out.routed_line1.pps 6013
owl-docker 1481183143 traffic.out.routed_line1.bps 48103
owl-docker 1481183143 traffic.in.routed_line1.pps 26995
owl-docker 1481183143 traffic.in.routed_line1.bps 215959
owl-docker 1481183143 traffic.out.routed_line2.pps 0
owl-docker 1481183143 traffic.out.routed_line2.bps 0
owl-docker 1481183143 traffic.in.routed_line2.pps 0
owl-docker 1481183143 traffic.in.routed_line2.bps 0
owl-docker 1481183143 traffic.out.routed_line3.pps 0
owl-docker 1481183143 traffic.out.routed_line3.bps 0
owl-docker 1481183143 traffic.in.routed_line3.pps 0
owl-docker 1481183143 traffic.in.routed_line3.bps 0
owl-docker 1481183143 traffic.out.routed_other.pps 0
owl-docker 1481183143 traffic.out.routed_other.bps 0
owl-docker 1481183143 traffic.in.routed_other.pps 0
owl-docker 1481183143 traffic.in.routed_other.bps 0

# the fourth awk output is
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.all.pps","value":13232,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.all.bps","value":105857,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.all.pps","value":31779,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.all.bps","value":254231,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.multicast.pps","value":1,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.multicast.bps","value":4,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.multicast.pps","value":2,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.multicast.bps","value":18,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.broadcast.pps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.broadcast.bps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.broadcast.pps","value":1,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.broadcast.bps","value":5,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.local.pps","value":7219,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.local.bps","value":57750,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.local.pps","value":4781,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.local.bps","value":38248,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.private.pps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.private.bps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.private.pps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.private.bps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.routed_all.pps","value":6013,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.routed_all.bps","value":48103,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.routed_all.pps","value":26995,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.routed_all.bps","value":215959,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.routed_line1.pps","value":6013,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.routed_line1.bps","value":48103,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.routed_line1.pps","value":26995,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.routed_line1.bps","value":215959,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.routed_line2.pps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.routed_line2.bps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.routed_line2.pps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.routed_line2.bps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.routed_line3.pps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.routed_line3.bps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.routed_line3.pps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.routed_line3.bps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.routed_other.pps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.out.routed_other.bps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.routed_other.pps","value":0,"counterType":"GAUGE","step":60}
{"endpoint":"owl-docker","tags":"",timestamp:1481183098,"metric":"traffic.in.routed_other.bps","value":0,"counterType":"GAUGE","step":60}

```