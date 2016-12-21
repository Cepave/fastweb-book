# 自訂監控項應做正規化 (normalization)

open-falcon 本來的設計中，有一小部分的監控項是沒有做「正規化」的。比方說：
* cpu.idle：Percentage of time that the CPU or CPUs were idle and the system did not have an outstanding disk I/O request.
* cpu.busy：与cpu.idle相对，他的值等于100减去cpu.idle。

由於只要使用者知道了 `cpu.idle` 或是 `cpu.busy` ，就可以推導出另一個監控項的值。其實系統也只需要儲存一個監控項而已。

也因此，考慮到自訂監控項因為需要客製化 plugin script 來開發，應該要讓「自訂監控項」做正規化。一方面節省資料庫的空間。更重要的是，減少重複的程式碼邏輯，讓 plugin script 容易被維護。