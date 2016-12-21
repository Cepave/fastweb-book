# 將告警條件的判斷移出 plugin script

下方這個範例，透過 plugin 來取得 fastmedia 這個程序的 cpu 使用量，並且根據這個 cpu 使用量的數值輸出 0 或是 1 。超過 90 輸出 1 。小於等於 90 輸出 0 。負責寫範例的團隊收到運維的建議就直接照做。於是就寫出了這樣子的程式。這樣子的 plugin 是很不好的範例。

因為這個 plugin 一口氣將「數據的採集」和「告警條件的判斷」都做在 plugin 裡了。 open-falcon 的告警條件，本身就可以做 if 與數值比較的判斷。也因此，這個 plugin 合理的輸出，應該就是直接輸出「 fastmedia 這個程序的 cpu 使用量」，這樣子的話， 運維團隊就可以透過 graph 模組來對這個監控項來繪圖。與 90 做比較的數值條件判斷，應該要做在告警的策略模板。

```
#!/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
export PATH

function fastmedia_worker_process()
{
CPUUSE=`top -bn 1 | awk '/fastmedia/{count=NF-4;if($count>90)print $count}'`
if [ -n "$CPUUSE" ];then
    return 1
else
    return 0
fi
}
#---------------------------------------------------------------------------------

# Call function
msg=$(fastmedia_worker_process)
retval=$?
date=`date +%s`
host=$HOSTNAME
tag=""

# Send JSON message
echo "[{\
  \"endpoint\"   : \"$host\",\
  \"tags\"       : \"$tag\",\
  \"timestamp\"  : $date,\
  \"metric\"     : \"fastmedia.worker.process\",\
  \"value\"      : $retval,\
  \"counterType\": \"GAUGE\",\
  \"step\"       : 60}]"
```

改成下方的，就會好很多。
```
#!/bin/bash

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
export PATH

function fastmedia_worker_process()
{
CPUUSE=`top -bn 1 | awk '/fastmedia/{count=NF-4;print $count}'`
echo $CPUUSE
}
#---------------------------------------------------------------------------------

# Call function
msg=$(fastmedia_worker_process)
retval=$?
date=`date +%s`
host=$HOSTNAME
tag=""

# Send JSON message
echo "[{\
  \"endpoint\"   : \"$host\",\
  \"tags\"       : \"$tag\",\
  \"timestamp\"  : $date,\
  \"metric\"     : \"fastmedia.worker.process\",\
  \"value\"      : $msg,\
  \"counterType\": \"GAUGE\",\
  \"step\"       : 60}]"
```