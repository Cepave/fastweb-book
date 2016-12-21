# 讓問題可以被追蹤

在 plugin script 撰寫時，應該在 plugin script 的開頭寫入「用途」「維護者的訊息」。否則，出了問題都不知道要找誰。

例如：
```
#!/bin/bash
# -------------------------------------------------------------------------------
# Filename:     300_fastmedia_worker_process.sh
# Revision:     1.0
# Date:         2016/11/28
# Author:       作者名稱
# Email:        作者的電郵信箱
# Description:  采集fastmedia work进程占用cpu百分比
#--------------------------------------------------------------------------------
```

# 取得 hostname 的方式

* 不好的作法：

  直接使用 ```$HOSTNAME``` 環境變數。

* 好的作法：
  
  使用 ```$(hostname -s)``` 輸出的結果

# 簡化 json 的輸出方式

plugin script 裡常用的 json 輸出方式是這樣子：
```
echo  "{\
      \"endpoint\"   : \"$HOST\",\
      \"tags\"       : \"TYPE=$TYPE\",\
      \"timestamp\"  : $DATE,\
      \"metric\"     : \"$METRICNAME\",\
      \"value\"      : \"$VALUE\",\
      \"counterType\": \"GAUGE\",\
      \"step\"       : 60}"
```
如果覺得太多的逸脫符號很難閱讀，可以改成採用 here document 的寫法
```
cat << EOF
    {
        "endpoint"      :"$HOST",
        "tags"          :"TYPE=${TYPE}",
        "timestamp"     :$DATE,
        "metric"        :"$METRICNAME",
        "value"         :"$VALUE",
        "counterType"   :"GAUGE",
        "step"          :60
    }
EOF

```