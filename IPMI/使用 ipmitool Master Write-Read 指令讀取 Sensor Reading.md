# [筆記] 使用 ipmitool Master Write-Read 指令讀取 Sensor Reading 

利用 [ipmitool](https://linux.die.net/man/1/ipmitool) Master Write‑Read 讀取 TMP75 的 I²C 暫存器，取得溫度值。

## ipmitool 指令格式

當存取 intelligent device (例如 Satellite Controller）時，可以透過 IPMB (IPMI Bus) 發送標準的 IPMI 指令或請求。

而存取底層 I²C 裝置 (non-intelligent device) 時，需使用 Master Write‑Read 指令：

```
ipmitool raw <netfn> <cmd> [<data>]
```

- <netfn> 欄位參考 *IPMI2.0 Table 5-, Network Function Codes* ，數值設為 06h (request)。
- <cmd> 欄位參考 *Table G-, Command Number Assignments and Privilege Levels*，數值設為 52h。
- <data> 欄位參考 *IPMI2.0 22.11 Master Write-Read*，格式如下:

    ![image](https://hackmd.io/_uploads/ByOoN5uwMl.png)

    <data> 指令格式為:

    ```
    <bus id> <slave‑addr> <read count> [ <write‑data> ]
    ```

    其中 <write‑data> 需參考 [*TMP75 SPEC 7.5 Programming*](https://www.ti.com/lit/ds/symlink/tmp75.pdf?ts=1787539945315&ref_url=https%253A%252F%252Fwww.ti.com%252Fproduct%252Fzh-tw%252FTMP75%252Fpart-details%252FTMP75AIDGKR) ，[P1 P0] 設為 00 代表選擇 Temperature register。

最後以 bus 0x20 為例，假設 Slave address 0x90 (8-bit format)，完整 ipmitool raw 指令如下:

```
ipmitool raw 0x06 0x52 0x20 0x90 0x02 0x00
```


## Q&A

Q : IPMI Master Write-Read (22.11) 與 IPMI Get Sensor Reading (35.14) 的差異?

- Master Write‑Read (Cmd 0x52) 讓 BMC 扮演 I²C Master，用來讀取底層 I²C 裝置（例如 TMP75、EEPROM 等），使用值需要先指定暫存器指標，再讀取資料。
- Get Sensor Reading 讀取已經登錄在 BMC SDR 的感測器，使用者指定 Sensor Number 後 BMC 會自動處理底層 I²C 讀取與資料轉換。