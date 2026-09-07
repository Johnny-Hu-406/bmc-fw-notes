# [筆記] IPMI SPEC 41~43章 Sensor & Event

## 章節內容
IPMI 41-43 章可以一起讀:
- Chapter 41：說明 Sensor Type Code、Event/Reading Type Code、event offset 這些 code 在 SDR 和 Event Message 裡怎麼用。
- Chapter 42：列出 code table。你看到 Sensor Type = 07h、Event Type = 6Fh、offset 07h 這類數字時，要在這章節的表格查詢解析。
- Chapter 43：定義 SDR record format。SDR 是 BMC 告訴 host 或 management software 怎麼解析的格式，定義了：有哪些 sensor、怎麼讀、單位是什麼、哪些事件會發生、threshold 數值轉換公式等內容。

## 41 章 : 分析 SEL record 或 Event Message
SEL record 或 Event Message 裡的事件語意，通常由 Sensor Type、Event/Reading Type Code、Event Dir、以及 Event Offset / Event Data 共同決定。

分析步驟如下:
1. 透過 Sensor Number 找到對應 SDR record
2. 從 SDR 查 Sensor Type、Event/Reading Type Code，以及 Assertion/Deassertion/Reading masks
3. 依 Chapter 42 code table 解 event offset 或 reading bits
4. 依 Chapter 43 SDR 欄位解 entity、unit、threshold、record type

首先，每個 sensor 都有一個 Sensor Type Code 定義了 Sensor 類型，例如 spec 舉例 07h 代表 Processor sensor。這個 code 同時會出現在 SDR 與 Event Message 裡。

接著分析 Event/Reading Type，描述 sensor 會回報哪一類 reading、會產生哪一類 event，Chapter 41 把 Event/Reading Type Code 分成三類:
- `Generic (02h~0Ch)`：通用離散事件類型，許多不同 sensor 都可使用。
- `Sensor Specific (6Fh)`：Event/Reading Type Code = 6Fh，Sensor Type 決定 Event Offset 的意義，查表對應 *Table 42-, Sensor Type Codes*。
例如 Processor sensor 的 offset 由 Sensor Type table 裡 Processor (07h)那列定義。
- `OEM (70h-7Fh)`：offset 意義由 OEM 定義，通常還要搭配 Manufacturer ID 才能正確解讀。

Note:

Chapter 29 有提到 `Event Dir = 0/1`，`Event Dir = 0` 通常代表 assertion，`Event Dir = 1` 代表 deassertion。

## 42 章 : 查表章節
Chapter 42 把 Event/Reading Type Code 的範圍、generic offset，以及 Sensor Type Code 對應的 sensor-specific offset 列出來。

例如 Sensor Type Code 常見值:
- 01h Temperature
- 02h Voltage
- 03h Current
- 04h Fan
- 05h Physical Security / Chassis Intrusion
- 06h Platform Security Violation Attempt
- 07h Processor

## 43 章 : SDR record byte layout / record format 定義章節
Chapter 43 定義 SDR record 的 byte layout，一個 SDR 由三個部分組成:
- RECORD HEADER
    RECORD HEADER 中的 Record Type 決定不同格式與類型的 Sensor Record，例如 Full Sensor Record (01h)、 Compact Sensor Record (02h)、 Event-Only Sensor Record(03h)......等等，參考 *CH43. Sensor Data Record Formats*。
- RECORD ‘KEY’ FIELDS
    RECORD ‘KEY’ FIELDS 是一個固定值，由 Sensor Owner ID、Sensor Owner LUN、Sensor Number 組成。
- RECORD BODY

Note :
對 analog reading，Full Sensor Record 會提供 M、B、B exp、R exp 等欄位，用於線性轉換公式: 

$$
y = L\left[\left(Mx + B \times 10^{B_{\text{exp}}}\right) \times 10^{R_{\text{exp}}}\right]
$$

*L[ ]* 是 Linearization 欄位指定的 function，當 sensor 為 linear sensor 時， *L[ ]* 等同不做額外轉換 (null)。