# metric, tag 使用慣例

監控項可以使用 metric 加上 tag 來定義。好的命名法，可以讓監控項的語意容易被了解。

取例來說： `net.if.in.bits/iface=eth1` 就是既有的監控項。它是指網路 (net) 通過 `eth1` 網卡，接收 (in) 的數據量，單位是 `bits` 。它同時使用了 metric 和 tag 。

metric 的部分是 `net.if.in.bits` 的部分。如果監控項的特性，是有普遍性，不依賴於特定機器，那就都用 metric 來描述。

tag 的部分是 `iface=eth1` 的部分。如果監控項的特性，是依賴於機器本身的，那就用 tag 來描述。( 因為每一台機器的網卡名稱可能都不一樣，所以有的機器的網卡名稱可能是 eth0 或是 eth2 等等。)

## 命名慣例

1. metric 的名稱用句點 . 分隔。
2. metric 的第一個欄位是「分類」。目前常用的分類是 `cpu`, `mem`, `net`, `disk` ，或者是自訂。 自訂的類別常見的有   `http`, `vfcc` 等。常常是服務或是程式的名稱。
3. metric 的最後一個欄位是「單位」。常見的單位有 `percent`, 'bits', `sec`, 
4. metric 透過句點分隔的分類概念，由左而右依序是由大而小。

## 參考命名
[open falcon 既有的監控項名稱](http://book.open-falcon.org/zh/faq/linux-metrics.html)
