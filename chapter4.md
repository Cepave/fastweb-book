# 寫 plugin script ，需要特別注意的指令

## ```top``` 

在 plugin script 裡，用一般寫 shell script 的習慣使用 ```top -n1``` 指令，會出錯。得到的錯誤是：```TERM environment variable not set``` 。原因是 "Using a terminal command i.e. "clear", in a script called from cron (no terminal) will trigger this error message."

合理的作法是用 ```top -bn 1``` 

## ```vmstat```

open-falcon agent 在採集資料的時候，會 plugin script 的啟動是同時啟動的，所以會有一瞬間產生大量的執行中的程序 (running processes) 。而 `vmstat` 採集到的資料有一項是 runnable processes 。這個數值如果只採集一次，就會造成嚴重的誤差。 

```
Procs
  r: The number of runnable processes (running or waiting for run time).
  b: The number of processes in uninterruptible sleep.

```

在有執行 open-falcon agent 的實體主機上同時執行 `vmstat` 的結果如下：注意第 7 列的 r 數值

```
[root@bgp-bj-xxxxxxxxxxxxxxx]# vmstat 1 10
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 893184 116876 27763716    0    0  1921   901    2    0  3  7 87  3  0
 0  1      0 886736 116876 27768912    0    0  4724    87 21718 7203  2  8 89  2  0
 0  0      0 873560 116876 27777912    0    0  8508    98 14129 6144  2  7 89  2  0
 2  0      0 855612 116876 27799224    0    0 10204    70 15366 5918  1  6 92  1  0
 0  0      0 841732 116876 27812980    0    0 13472    70 16005 5974  1  6 91  2  0
 0  2      0 820404 116880 27835584    0    0  5764   128 17350 6449  1  6 91  1  0
51  0      0 760648 116884 27849956    0    0  5628   104 28971 12076  8 22 66  4  0
 1  0      0 790028 116884 27857456    0    0  6680   102 23405 16582 23 34 43  1  0
 0  0      0 761420 116924 27886104    0    0 28016   456 19636 6745  2  8 84  6  0
 0  0      0 751624 116936 27890244    0    0  3432   221 17453 6490  3  6 88  3  0

```


## ```route```

像 `route` 這種指令，它所在的資料夾不是單純的 /bin ，而是 /sbin 。當使用這一類取得系統資訊的指令時，都要特別注意 $PATH 變數的內容。否則 plugin script 就會因為路徑問題而無法順利調用指令。