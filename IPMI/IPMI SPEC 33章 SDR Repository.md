# [筆記] IPMI SPEC 33章 SDR Repository
SDR(Sensor Data Record) Repository 用於告訴 system management software 中預期有哪些 sensors、management controllers、FRU devices。

SDR 描述的是平台預期存在的管理物件與 sensor 資訊，不等於即時硬體掃描結果。若某硬體缺失， system management software 可以根據/檢查 SDR 後執行對應的動作。

在 SDR 中有 Record key 與 Record ID 欄位:
- 對 sensor record 來說，實務上常用 Sensor Owner ID、Sensor Owner LUN、Sensor Number 這些 key fields 來識別同一顆 sensor。
- Record ID 會隨著 SDR 新增/刪除/更新後改變，在實際操作中常用來連接下一個 Record ID，例如執行 "Get SDR Command"，會回傳下一筆紀錄的 Record ID。
(Record ID 是 SDR Repository 用來列舉/存取 record 的 ID，不等於 Sensor Number，也不等於 sensor identity)
![image](https://hackmd.io/_uploads/SkN-oH3uMl.png)


33.12 Get SDR Command 讀取流程：

1. 先用 Get SDR Repository Info 取得 SDR Version、Record Count、最近新增/清除 timestamp，以及 operation support。
2. 若需要分段讀取，先用 Reserve SDR Repository 取得 Reservation ID。
3. 用 Get SDR, Record ID = 0000h，取得第一筆 record。
4. 使用 response 的 Next Record ID 繼續讀，直到 Next Record ID = FFFFh。

最後在 CH33 中，也有提到其他操作 SDR Repository 的指令(略):
![image](https://hackmd.io/_uploads/Bk7EsB3_Mg.png)
