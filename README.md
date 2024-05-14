# 【專2024消006】手機認證優化APP-後端系統設計文件

| 項目   | 內容           |     |
| ---- | ------------ | --- |
| 建立日期 | 2023/03/11   |     |
| 公司   | 中租控股         |     |
| 單位   | 數位發展部數位科技發展科 |     |
| 專案代號 | 專2024消006    |     |

## 版本紀錄

| 日期         | 修改者 | 說明     |
| ---------- | --- | ------ |
| 2023/03/25 | Sky | 初版     |
| 2023/05/06 | Sky | v1.0版本 |
| 2023/05/08 | Sky | v1.1版本 |
| 2023/05/10 | Sky | v1.2版本 |

---
# 目錄
 * [前言](#前言)
 * [模組功能架構](#模組功能架構)
 * [資料表結構schema](#資料表結構schema)
 * [新介接Kong相關設定](#新介接Kong相關設定)
 * [Recaptcha金鑰](#recaptcha金鑰)
 * [注意事項](#注意事項)
 * [功能設計與實作](#功能設計與實作)
    * [1. 電子合約待辦專區](#電子合約待辦專區)
        * [1-1.取得電子合約代辦清單](#取得電子合約代辦清單)  
        * [1-2. 取得電子合約代辦明細](#取得電子合約代辦明細)  
    * [2.FAQ常見問題專區](#FAQ常見問題專區)
        * [2-1.取得單筆FAQ資訊](#取得單筆FAQ資訊)        
        * [2-2.取得FAQ清單](#取得FAQ清單)
        * [2-3.FAQ搜尋](#FAQ搜尋)
    * [3.取得合約、憑證資訊](#取得合約、憑證資訊)
	    * [3-1.取得同意條款](#取得同意條款)
	    * [3-2.取得合約清單](#取得合約清單)
	    * [3-3.取得單筆合約](#取得單筆合約)
	    * [3-4.取得憑證清單](#取得憑證清單)
	    * [3-5.取得憑證明細](#取得憑證明細)
	*  [4.加簽程序](#加簽程序)
	    * [4-1.Login](#Login)
	    * [4-2.憑證安裝完成](#憑證安裝完成)
		* [4-3.加簽完成](#加簽完成)


---

# 前言
此專案範圍包含「移動生活APP系統」、「電子合約加簽網站(mid-sign-service)」、「台網Portal系統(mid-service)」，主要目的是讓App能執行電子合約加簽流程。

---
# 模組功能架構

檔案存放位置:
H:\20-AP開發單位\20-數位發展部\20-數位科技發展科\工時預估\商模\R202312136375_手機認證優化APP

## 系統架構圖

![[Pasted image 20240513093919.png]]
## 系統流程圖

![[憑證APP_20240402-一般與增補待確認.jpg]]


---
# 資料表結構schema

檔案存放位置:
mid_sign_app.json
mid_sing_app.vuerd.json

編輯/閱讀工具:
VS code的ERD Editor 套件


---
# 新介接Kong相關設定

|說明|環境|來源網址|代理網址|Key-value|
|-|-|-|-|-|
| 待確認|待確認 | 待確認|待確認 | 待確認|

---
# Recaptcha金鑰

|版本|金鑰|Secret|
|-|-|-|
|待確認| 待確認| 待確認|
# 注意事項
文件內命名皆為*大駝峰*（後端命名原則），Swagger文件輸出皆為*小駝峰*。
    例：後端命名**R**eturnCode，swagger輸出為**r**eturnCode。
``` json
後端命名 
{
	"ReturnCode":0,
	"RedirectUrl":"string",
	"ReturnType":"0",
}

swagger顯示
{
	"returnCode":0,
	"redirectUrl":"string",
	"returnType":"0",
}

```

---
# 功能設計與實作


# 電子合約待辦專區

## 取得電子合約代辦清單

#### ==== GET/GetSignMainList ====
### 需求描述:
> 取得電子合約代辦清單(一般、增補)，需顯示(對保日期、車號/簽約人姓名或公司名稱、代辦數量、是否已完成)，並篩選二個月內資料。
### 功能設計與實作
- 模組：小車電子合約APP化
- 類別：`MidSigAPPController`
- 方法：`Task<SignMainListViewModel> GetSignMainListAsync(string userMobileNumber)`
- 資料表：[MidSigMain]

### 處理商業邏輯
1. 查詢資料表：MidSigMain
2. SQL取得MidSigMain資料表，依據[UserMobileNumber]篩選手機號碼 userMobileNumber，[CreateDate]篩選二個月內資料，[SystemId]篩選為'MidSig'，用[SignatureDate]來DESC排序，對保日期越新的在最上面。
3. 計算尚未加簽[SignatureComplete]='N'的數量
4. 若DB異常，拋BusinessException，錯誤訊息Exception

### SQL語法
SQL
```sql
/****** SSMS 中 SelectTopNRows 命令的指令碼  ******/
SELECT * FROM [APP_COMPRE].[dbo].[MidSigMain]
  WHERE SystemId='MidSig'
	  AND [UserMobileNumber] = @UserMobileNumber
      AND [CreateDate] >= DATEADD(MONTH, -2, GETDATE())
  ORDER BY SignatureDate DESC
```
### Request

#### Header
|項目|說明|
|---|---|
|HTTP Method|Get|
|Header|Content-Type:application/json; charset=UTF-8|
|Uri|api/MidSigAPP/GetSignMainListAsync|
#### Body 欄位
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

無
#### Body 範例
無

### Response

#### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

|欄位名稱|IsNullable|資料型別|名稱|說明|
|---|---|---|---|---|
|ReturnCode|N|int|回傳代碼|成功:0/錯誤:-1||
|RedirectUrl|Y|string|轉導網址|下一步驟的route|
|ReturnType|N|string|結果顯示類型|0:pop視窗 1:滿版頁面 2:警告文字|
|ReturnTitle|Y|string|結果標題|成功/錯誤標題|
|ReturnMsg|Y|string|結果訊息|成功/錯誤訊息|
|Data|Y|object|回傳資料|回傳Data欄位，無資料回傳null|
#### 回傳Data物件

|欄位名稱|資料型別|IsNullable|長度|預設值|說明|
|---|---|---|---|---|---|
|SignMainInfo|List<objec\>|Y|-|-|電子合約主檔|
- SignMainInfo欄位

| 欄位名稱              | 資料型別     | IsNullable | 長度  | 預設值 | 說明             |
| ----------------- | -------- | ---------- | --- | --- | -------------- |
| MidSigId          | string   | N          | 40  | -   | MidSigId合約主key |
| UserName          | string   | Y          | 50  | -   | 簽約人姓名/公司名稱     |
| PlateNumber       | string   | N          | 10  | -   | 車號             |
| SignatureDate     | Datetime | Y          | -   | -   | 對保日期           |
| CreateDate        | Datetime | N          | -   | -   | 建立日期           |
| SignatureComplete | string   | N          | 10  | -   | 對保狀態           |

#### Response 範例

``` json
{
	"ReturnCode":0,
	"RedirectUrl":"string",
	"ReturnType":"0",
	"ReturnTitle":"string",
	"ReturnMsg":"string",
	"Data":
	{
	    "SignMainInfo":[
            {
                "UserName":"陳建成",
                "PlateNumber":"1191-MHL",
                "SignatureDate":"2024-04-30 07:28:53.963",
                "CreateDate" :"2024-04-30 07:27:41.860",
                "SignatureComplete" :"Y"
            },
            {
                "UserName":"中租迪和 陳建成",
                "PlateNumber":"1867-BV",
                "SignatureDate":"2024-04-30 07:28:53.963",
                "CreateDate" :"2024-04-30 07:27:41.860",
                "SignatureComplete" :"N"
            },
            ///...
        ]
	}
}
```

#### 待確認
- 電話欄位、UserName欄位，跟車輛科確認是否會存資料近來
- 電話欄位需要新開一個欄位
- UserName是既有欄位，但都沒存資料，需特別注意儲存內容有無符合需求。(姓名/公司名稱+負責人姓名)
#### 已確認

- 小車會從後台呼叫API (api/MidSig/v1POST )，新增[MidSigMain]、[MidSigFile]、[MidSigHtml]至DB。

[返回目錄](#目錄)


## 取得電子合約代辦明細

無須開發


# FAQ常見問題專區

## 取得單筆FAQ資訊
#### ==== GET/GetFAQInfo ====
### 需求描述:
> 依據QaArticleId，取得單筆FAQ資訊
### 功能設計與實作
- 模組：小車電子合約APP化
- 類別：`MidSigAPPController`
- 方法：`Task<FAQInfoViewModel> GetFAQInfoAsync(string qaArticleId)`
- 資料表：[QaArticle]

### 處理商業邏輯
1. 查詢資料表：QaArticle
2. SQL取得QaArticle資料表，依據QaArticleId篩選對應資料，篩選IsOn的資料，用SortNumber來排序。
3. 若DB異常，拋BusinessException，錯誤訊息Exception


### SQL語法
SQL
```sql
SELECT * FROM [dbo].[QaArticle]
WHERE QaArticle.IsOn = 1 AND QaArticle.QaArticleId = @QaArticleId
ORDER BY QaArticle.SortNumber DESC
```

### Request

#### Header
|項目|說明|
|---|---|
|HTTP Method|Get|
|Header|Content-Type:application/json; charset=UTF-8|
|Uri|api/MidSigAPP/GetFAQInfoAsync|
#### Body 欄位
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱        | IsNullable | 資料型別   | 名稱     | 說明           |
| ----------- | ---------- | ------ | ------ | ----------------------- |
| qaArticleId | N          | string | QA文章ID | uniqueidentifier轉string |
|             |            |        |        |                         |

#### Body 範例

``` json
{
    "qaArticleId":"string"
}
```

### Response

#### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱        | IsNullable | 資料型別   | 名稱     | 說明                    |     |
| ----------- | ---------- | ------ | ------ | --------------------- | --- |
| ReturnCode  | N          | int    | 回傳代碼   | 成功:0/錯誤:-1            |     |
| RedirectUrl | Y          | string | 轉導網址   | 下一步驟的route            |     |
| ReturnType  | N          | string | 結果顯示類型 | 0:pop視窗 1:滿版頁面 2:警告文字 |     |
| ReturnTitle | Y          | string | 結果標題   | 成功/錯誤標題               |     |
| ReturnMsg   | Y          | string | 結果訊息   | 成功/錯誤訊息               |     |
| Data        | Y          | object | 回傳資料   | 回傳Data欄位，無資料回傳null    |     |
#### 回傳Data物件

|欄位名稱|資料型別|IsNullable|長度|預設值|說明|
|---|---|---|---|---|---|
|FAQInfo|Y|-|-|Qa文章|
- FAQInfo欄位

| 欄位名稱               | 資料型別   | IsNullable | 長度  | 預設值 | 說明     |
| ------------------ | ------ | ---------- | --- | --- | ------ |
| FAQInfoSortNumber  | int    | N          | -   | -   | 問題排序編號 |
| FAQInfoTitle       | string | Y          | 50  | -   | 問題主旨   |
| FAQInfoDescription | string | Y          | 500 | -   | 問題描述   |
| FAQInfoContent     | string | Y          | max | -   | 問題詳細內容 |

#### Response 範例

``` json
{
	"ReturnCode":0,
	"RedirectUrl":"string",
	"ReturnType":"0",
	"ReturnTitle":"string",
	"ReturnMsg":"string",
	"Data":
	{
	    "FAQInfo":
        {
            "QaArticleSortNumber" :1,
            "QaArticleTitle":"問題主旨",
            "QaArticleDescription":"問題描述",
            "QaArticleContent":"問題詳細內容"
        },
	}
}
```

[返回目錄](#目錄)

---
## 取得FAQ清單
#### ==== GET/GetFAQList ====
### 需求描述:
> 取得所有FAQ清單
### 功能設計與實作
- 模組：小車電子合約APP化
- 類別：`MidSigAPPController`
- 方法：`Task<FAQListViewModel> GetFAQListAsync()`
- 資料表：[Category]、[QaArticle]

### 處理商業邏輯
1. 查詢資料表：Category、QaArticle
2. SQL取得所有Category資料表並Join QaArticle資料表，篩選IsOn的資料，用SortNumber來排序。
3. 若DB異常，拋BusinessException，錯誤訊息Exception

### SQL語法
SQL
```sql
SELECT * FROM [dbo].[Category]
LEFT JOIN [dbo].[QaArticle] ON Category.CategoryId = QaArticle.CategoryId
WHERE Category.IsOn = 1 AND QaArticle.IsOn
ORDER BY Category.SortNumber DESC, FAQList.SortNumber DESC
```
### Request

#### Header
|項目|說明|
|---|---|
|HTTP Method|Get|
|Header|Content-Type:application/json; charset=UTF-8|
|Uri|api/MidSigAPP/GetFAQListAsync|
#### Body 欄位
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

無
#### Body 範例
無

### Response

#### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

|欄位名稱|IsNullable|資料型別|名稱|說明|
|---|---|---|---|---|
|ReturnCode|N|int|回傳代碼|成功:0/錯誤:-1||
|RedirectUrl|Y|string|轉導網址|下一步驟的route|
|ReturnType|N|string|結果顯示類型|0:pop視窗 1:滿版頁面 2:警告文字|
|ReturnTitle|Y|string|結果標題|成功/錯誤標題|
|ReturnMsg|Y|string|結果訊息|成功/錯誤訊息|
|Data|Y|object|回傳資料|回傳Data欄位，無資料回傳null|
#### 回傳Data物件

|欄位名稱|資料型別|IsNullable|長度|預設值|說明|
|---|---|---|---|---|---|
|FAQInfo|List<objec\>|Y|-|-|Qa文章|
- FAQInfo欄位

| 欄位名稱                | 資料型別   | IsNullable | 長度  | 預設值 | 說明     |
| ------------------- | ------ | ---------- | --- | --- | ------ |
| CategoryType        | int    | N          | -   | -   | 類別Type |
| CategorySortNumber  | int    | N          | -   | -   | 類別排序編號 |
| CategoryTitle       | string | Y          | 50  | -   | 類別主旨   |
| CategoryDescription | string | Y          | 500 | -   | 類別描述   |
| FAQInfoSortNumber   | int    | N          | -   | -   | 問題排序編號 |
| FAQInfoTitle        | string | Y          | 50  | -   | 問題主旨   |
| FAQInfoDescription  | string | Y          | 500 | -   | 問題描述   |
| FAQInfoContent      | string | Y          | max | -   | 問題詳細內容 |

#### Response 範例

``` json
{
	"ReturnCode":0,
	"RedirectUrl":"string",
	"ReturnType":"0",
	"ReturnTitle":"string",
	"ReturnMsg":"string",
	"Data":
	{
	    "FAQInfo":[
            {
                "CategoryType":1,
                "CategorySortNumber":1,
                "CategoryTitle":"類別主旨",
                "CategoryDescription" :"類別描述",
                "FAQInfoSortNumber" :1,
                "FAQInfoTitle":"問題主旨",
                "FAQInfoDescription":"問題描述",
                "FAQInfoContent":"問題詳細內容"
            },
            {
                "CategoryType":1,
                "CategorySortNumber":1,
                "CategoryTitle":"類別主旨",
                "CategoryDescription" :"類別描述",
                "FAQInfoSortNumber" :2,
                "FAQInfoTitle":"問題主旨",
                "FAQInfoDescription":"問題描述",
                "FAQInfoContent":"問題詳細內容"
            },
            ///...
        ]
	}
}
```

[返回目錄](#目錄)

---
## FAQ搜尋
#### ==== GET/SearchFAQList ====
### 需求描述:
> 比對問題的關鍵字，取得符合查尋條件的FAQ清單。
### 功能設計與實作
- 模組：小車電子合約APP化
- 類別：`MidSigAPPController`
- 方法：`Task<FAQListViewModel> SearchFAQListAsync(string query)`
- 資料表：[Category]、[QaArticle]

### 處理商業邏輯 
1. 查詢資料表：Category、QaArticle
2. SQL取得所有Category資料表並Join QaArticle資料表，篩選IsOn的資料，用SortNumber來排序。
3. 根據傳入的query，篩選出包含在主旨（FAQInfoTitle）、描述（FAQInfoDescription）或關鍵字（Keywords）中任何一項的文章清單。
4. 若DB異常，拋BusinessException，錯誤訊息Exception
### SQL語法
SQL
```sql
SELECT * FROM [dbo].[Category]
LEFT JOIN [dbo].[QaArticle] ON Category.CategoryId = QaArticle.CategoryId
WHERE Category.IsOn = 1 AND QaArticle.IsOn = 1
AND (
    QaArticle.FAQInfoTitle LIKE '%' + @query + '%' 
    OR QaArticle.FAQInfoDescription LIKE '%' + @query + '%' 
    OR QaArticle.Keywords LIKE '%' + @query + '%'
)
ORDER BY Category.SortNumber DESC, QaArticle.SortNumber DESC

```
### Request

#### Header
| 項目          | 說明                                           |
| ----------- | -------------------------------------------- |
| HTTP Method | Get                                          |
| Header      | Content-Type:application/json; charset=UTF-8 |
| Uri         | api/MidSigAPP/GetFAQInfoAsync                |
#### Body 欄位
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱        | IsNullable | 資料型別   | 名稱     | 說明           |
| ----------- | ---------- | ------ | ------ | ----------------------- |
| query | N          | string | 查詢內容 | string |
|             |            |        |        |                         |

#### Body 範例

``` json
{
    "query":"string"
}
```

### Response

#### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

|欄位名稱|IsNullable|資料型別|名稱|說明|
|---|---|---|---|---|
|ReturnCode|N|int|回傳代碼|成功:0/錯誤:-1||
|RedirectUrl|Y|string|轉導網址|下一步驟的route|
|ReturnType|N|string|結果顯示類型|0:pop視窗 1:滿版頁面 2:警告文字|
|ReturnTitle|Y|string|結果標題|成功/錯誤標題|
|ReturnMsg|Y|string|結果訊息|成功/錯誤訊息|
|Data|Y|object|回傳資料|回傳Data欄位，無資料回傳null|
#### 回傳Data物件

|欄位名稱|資料型別|IsNullable|長度|預設值|說明|
|---|---|---|---|---|---|
|FAQInfo|List<objec\>|Y|-|-|Qa文章|
- FAQInfo欄位

| 欄位名稱                | 資料型別   | IsNullable | 長度  | 預設值 | 說明     |
| ------------------- | ------ | ---------- | --- | --- | ------ |
| CategoryType        | int    | N          | -   | -   | 類別Type |
| CategorySortNumber  | int    | N          | -   | -   | 類別排序編號 |
| CategoryTitle       | string | Y          | 50  | -   | 類別主旨   |
| CategoryDescription | string | Y          | 500 | -   | 類別描述   |
| FAQInfoSortNumber   | int    | N          | -   | -   | 問題排序編號 |
| FAQInfoTitle        | string | Y          | 50  | -   | 問題主旨   |
| FAQInfoDescription  | string | Y          | 500 | -   | 問題描述   |
| FAQInfoContent      | string | Y          | max | -   | 問題詳細內容 |

#### Response 範例

``` json
{
	"ReturnCode":0,
	"RedirectUrl":"string",
	"ReturnType":"0",
	"ReturnTitle":"string",
	"ReturnMsg":"string",
	"Data":
	{
	    "FAQInfo":[
            {
                "CategoryType":1,
                "CategorySortNumber":1,
                "CategoryTitle":"類別主旨",
                "CategoryDescription" :"類別描述",
                "FAQInfoSortNumber" :1,
                "FAQInfoTitle":"問題主旨",
                "FAQInfoDescription":"問題描述",
                "FAQInfoContent":"問題詳細內容"
            },
            {
                "CategoryType":1,
                "CategorySortNumber":1,
                "CategoryTitle":"類別主旨",
                "CategoryDescription" :"類別描述",
                "FAQInfoSortNumber" :2,
                "FAQInfoTitle":"問題主旨",
                "FAQInfoDescription":"問題描述",
                "FAQInfoContent":"問題詳細內容"
            },
            ///...
        ]
	}
}
```

[返回目錄](#目錄)

# 取得合約、憑證資訊

## 取得同意條款

#### ==== GET/**GetStaticDocument** ====
### 需求描述:
> 回傳同意條款Html，包含css樣式，此內容經PM確認不常更新。

(  如有效能考量可以將HTML部屬在CDN 上，加快加載速度並減輕主服務器的負擔。)
### 功能設計與實作
- 模組：小車電子合約APP化
- 類別：`MidSigAPPController`
- 方法：`Task<IActionResult> GetStaticDocumentAsync(string documentName)`
### 處理商業邏輯 
1.  從服務或文件系統獲取指定的 HTML 內容
	-  `var htmlContent = await GetHtmlContentByDocumentName(documentName);`
```
private async Task<string> GetHtmlContentByDocumentName(string documentName)
{
    string filePath = Path.Combine(Directory.GetCurrentDirectory(), "StaticDocuments", $"{documentName}.html");

    if (File.Exists(filePath))
    {
        return await File.ReadAllTextAsync(filePath);
    }

    return null;
}
```
2.  如果文件不存在或documentName不正確，返回適當的 HTTP 狀態碼
	- `if (htmlContent == null) { return NotFound($"Requested document '{documentName}' not found."); }`
3.  直接返回 HTML 內容
	- `return Content(htmlContent, "text/html");`

### Request

#### Header
| 項目          | 說明                                    |
| ----------- | ------------------------------------- |
| HTTP Method | Get                                   |
| Header      | Content-Type:text/html; charset=UTF-8 |
| Uri         | api/MidSigAPP/GetStaticDocument       |
|             |                                       |
#### Body 欄位
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱         | IsNullable | 資料型別   | 名稱   | 說明  |
| ------------ | ---------- | ------ | ---- | --- |
| documentName | N          | string | 檔案名稱 |     |

#### Body 範例

``` json
{
    "documentName":"TermsService"
}
```

### Response
#### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

|欄位名稱|IsNullable|資料型別|名稱|說明|
|---|---|---|---|---|
|ReturnCode|N|int|回傳代碼|成功:0/錯誤:-1||
|RedirectUrl|Y|string|轉導網址|下一步驟的route|
|ReturnType|N|string|結果顯示類型|0:pop視窗 1:滿版頁面 2:警告文字|
|ReturnTitle|Y|string|結果標題|成功/錯誤標題|
|ReturnMsg|Y|string|結果訊息|成功/錯誤訊息|
|Data|Y|object|回傳資料|回傳Data欄位，無資料回傳null|
#### 回傳Data物件

| 欄位名稱     | 資料型別             | IsNullable | 長度  | 預設值 | 說明     |
| -------- | ---------------- | ---------- | --- | --- | ------ |
| HtmlFile | HtmlFileResponse | Y          | -   | -   | Html檔案 |

- HtmlFile欄位

| 欄位名稱        | 資料型別   | IsNullable | 長度  | 預設值 | 說明   |
| ----------- | ------ | ---------- | --- | --- | ---- |
| FileName    | string | N          | 100 | -   | 檔案名稱 |
| FileContent | string | N          | max | -   | 檔案內文 |

Response 範例

``` json
{
	"ReturnCode":0,
	"RedirectUrl":"string",
	"ReturnType":"0",
	"ReturnTitle":"string",
	"ReturnMsg":"string",
	"Data":
	{
	    "HtmlFile":
            {
                "FileName":"數位簽章使用與手機門號認證",
                "FileContent" :"<h2class="mb-3">數位簽章使用與手機門號認證</h2><divclass="articles-sectionmb-2"><article>一、中租控股(以下稱本公司)依據電子簽章法規定，向台端告知下列事項，請台端詳閱：<br><br>1.依電子簽章法第四條第一項及第九條第一項規定，本人同意以使用本公司系統產製具有電子簽章之電子文件作為交易往來之基礎，取代傳統書面文件及簽名、蓋章確定之相關法律責任。<br><br>2.本人同意使用電子文件為表示方法，有關彼我雙方權利義務之合約，依法律規定應使用書面或應簽名、蓋章者，本人同意使用電子簽章/數位簽章簽署。<br><br>3.凡使用本公司規定方式傳送之任何電子文件，經主管機關許可之憑證機構所簽發或核可之數位簽章簽署後，本人同意視同為書面，且將經前述數位簽章簽署後之電子文件完整列印後視為原件。<br><br>4.本人同意接受且不得否認，不論相關法律是否規定特定文件需以書面形式或以主管機關許可之憑證機構簽發或核可之憑證簽署，本公司合約電子文件經電子簽章或數位簽章簽署後對合約雙方具有拘束力及證據力。<br><br>二、本作業係依循電子簽章法，建置安全及可信賴之網路環境，透過網際網路進行線上申辦。<br><br>三、依<ahref="https://www.twca.com.tw/product/8a08562a-ed45-4420-85ec-2eaba0ce5f22"target="_blank"rel="noreferrernoopener">門號認證條款</a>，使用您的手機門號進行身分驗證，並請確認使用4G/5G網路。</article></div><style>h2{font-size:1.5rem;font-weight:600;text-align:center;color:#535353;}.articles-section{max-height:500px;overflow-y:scroll;padding-right:5px;@includemedia.pad{max-height:400px;}@includemedia.phone{max-height:300px;}&::-webkit-scrollbar{//整體width:8px;//粗度background-color:transparent;}&::-webkit-scrollbar-track{//底層background-color:transparent;border-radius:20px;}&::-webkit-scrollbar-thumb{//有在動的background-color:variables.$grey_400;border-radius:20px;}}.mb-1{margin-bottom:.5rem!important}</style>"
	        }
	}
}

```

[返回目錄](#目錄)

## 取得合約清單
#### ==== GET/GetContractList ====
### 需求描述:
> 根據[MidSigId]篩選出相關合約清單，回傳檔案名稱、檔案路徑(URL)清單。

### 功能設計與實作
- 模組：小車電子合約APP化
- 類別：`MidSigAPPController`
- 方法：`Task<ContractViewModel> GetContractListAsync(string verifyToken)`

### 處理商業邏輯 
1.  可參考既有程式(AcceptV)、(BaseCheck)、(GetContractFileUrl)
2.  基礎檢核(BaseCheck)
3.  根據[sigMain.MidSigId]取得合約清單(sigMain.MidSigId)
4.  如果清單為空，顯示錯誤訊息。
### SQL語法
SQL
```sql
SELECT * FROM [dbo].[MidSigHtmls]
WHERE MidSigHtmls.MidSigId = @midSigId
ORDER BY MidSigHtmls.[SubSequenceId] DESC
```
### Request

#### Header
| 項目          | 說明                                           |
| ----------- | -------------------------------------------- |
| HTTP Method | Get                                          |
| Header      | Content-Type:application/json; charset=UTF-8 |
| Uri         | api/MidSigAPP/GetContractListAsync           |
#### Body 欄位
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱        | IsNullable | 資料型別   | 名稱       | 說明                        |
| ----------- | ---------- | ------ | -------- | ------------------------- |
| verifyToken | N          | string | MidSigId | verifyToken此步驟為[MidSigId] |

#### Body 範例

``` json
{
    "verifyToken":"string"
}
```

### Response

#### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

|欄位名稱|IsNullable|資料型別|名稱|說明|
|---|---|---|---|---|
|ReturnCode|N|int|回傳代碼|成功:0/錯誤:-1||
|RedirectUrl|Y|string|轉導網址|下一步驟的route|
|ReturnType|N|string|結果顯示類型|0:pop視窗 1:滿版頁面 2:警告文字|
|ReturnTitle|Y|string|結果標題|成功/錯誤標題|
|ReturnMsg|Y|string|結果訊息|成功/錯誤訊息|
|Data|Y|object|回傳資料|回傳Data欄位，無資料回傳null|
#### 回傳Data物件

| 欄位名稱             | 資料型別                        | IsNullable | 長度  | 預設值 | 說明   |
| ---------------- | --------------------------- | ---------- | --- | --- | ---- |
| ContractFileList | List<ContractFileResponse\> | Y          | -   | -   | 合約清單 |

- ContractFileList欄位

| 欄位名稱     | 資料型別   | IsNullable | 長度  | 預設值 | 說明    |
| -------- | ------ | ---------- | --- | --- | ----- |
| FileName | string | N          | 100 | -   | 檔案名稱  |
| FileUrl  | string | N          | 500 | -   | 檔案Url |

#### Response 範例

``` json
{
	"ReturnCode":0,
	"RedirectUrl":"string",
	"ReturnType":"0",
	"ReturnTitle":"string",
	"ReturnMsg":"string",
	"Data":
	{
	    "ContractFileList":[
            {
                "FileName":"雲端帳號申請合約",
                "FileUrl" :"http://localhost:10434/MidSigPage/v1/ContractFile?verifyToken=0032aa8d5874435db0fbd136d92b019a&fileId=78D9D5C4-689D-424B-8DA6-2AE0BE8D812E"
            },
            {
                "FileName":"個人法定告知事項",
                "FileUrl" :"http://localhost:10434/MidSigPage/v1/ContractFile?verifyToken=0032aa8d5874435db0fbd136d92b019a&fileId=2A8DF313-C78A-468D-91F0-356BDC31C691"
            },
            ///...
        ]
	}
}
```

[返回目錄](#目錄)

## 取得單筆合約
#### ==== GET/GetContract ====
### 需求描述:
> 根據[MidSigId]和[ContractFileUuid]取得檔案名稱、檔案內容(Html to String)。

### 功能設計與實作
- 模組：小車電子合約APP化
- 類別：`MidSigAPPController`
- 方法：`Task<ContractViewModel> GetContractAsync(string verifyToken,Guid fileId)`
### 處理商業邏輯 
1. 可參考既有程式(ContractFile、BaseCheck、GetContractFile)
2. 基礎檢核(BaseCheck)
3. 根據[sigMain.MidSigId]和[ContractFileUuid]取得檔案資料(MidSigHtml)
5. 呼叫公版檔案中心API，取得檔案資料，將Content轉換成String並回傳。
6. 公版檔案中心API，傳參如下：
7. 如果清單為空，顯示錯誤訊息。
### 外部API
1. 公版檔案中心:GetFileDownloadAsync
#### Request
##### Header

| 項目          | 說明                                                                                                                                                                  |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| HTTP Method | Get                                                                                                                                                                 |
| Header      | Content-Type:application/json; charset=UTF-8                                                                                                                        |
| X-API-KEY   | apikey-when-MidSignWeb-is-client                                                                                                                                    |
| Uri         | https://sit01-intra-clg-apigw.chailease.com.tw/file-center/service/FileCenter/GetFileDownloadAsync?fileDetailId=78D9D5C4-689D-424B-8DA6-2AE0BE8D812E&downloader=SYS |
- URL格式：{_fileCenterApiUrl}{detailUrl}/FileCenter/GetFileDownloadAsync?fileDetailId={contractFile.ContractFileUuid}&downloader=SYS
	- _fileCenterApiUrl：https://sit01-intra-clg-apigw.chailease.com.tw/file-center/service
	- detailUrl：/FileCenter/GetFileDownloadAsync?fileDetailId={contractFile.ContractFileUuid}&downloader=SYS
		- contractFile.ContractFileUuid：78D9D5C4-689D-424B-8DA6-2AE0BE8D812E
#####  Body 
無

##### Response
#### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱    | IsNullable | 資料型別 | 名稱  | 說明   |     |
| ------- | ---------- | ---- | --- | ---- | --- |
| Content | N          | int  | 內容  | 檔案內容 |     |

##### Response 範例

``` json
{
	//標準 HttpResponseMessage
	"HttpStatusCode":0,
	"HttpContent":"string",
	...
}
```



### SQL語法
SQL
```sql
SELECT TOP 1 * FROM [dbo].[MidSigHtmls]
WHERE MidSigHtmls.MidSigId = @midSigId
AND MidSigHtmls.ContractFileUuid = @fileId
```

### 補充：
- 小車會從後台呼叫API (api/MidSig/v1POST )，新增[MidSigMain]、[MidSigFile]、[MidSigHtml]至DB。
	-  POST的時候會傳入MidSigHtmls資訊，此時會將檔案UUID等資訊存入MidSigHtmls
	- 上傳檔案的部分事前就上傳了，在小車或微企系統。

### Request

#### Header
| 項目          | 說明                                           |
| ----------- | -------------------------------------------- |
| HTTP Method | Get                                          |
| Header      | Content-Type:application/json; charset=UTF-8 |
| Uri         | api/MidSigAPP/GetContractAsync               |
#### Body 欄位
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱        | IsNullable | 資料型別   | 名稱               | 說明                        |
| ----------- | ---------- | ------ | ---------------- | ------------------------- |
| verifyToken | N          | string | MidSigId         | verifyToken此步驟為[MidSigId] |
| fileId      | N          | string | ContractFileUUID | 檔案UUID                    |
|             |            |        |                  |                           |

#### Body 範例

``` json
{
    "verifyToken":"string",
    "fileId":"string"
}
```
### Response

#### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

|欄位名稱|IsNullable|資料型別|名稱|說明|
|---|---|---|---|---|
|ReturnCode|N|int|回傳代碼|成功:0/錯誤:-1||
|RedirectUrl|Y|string|轉導網址|下一步驟的route|
|ReturnType|N|string|結果顯示類型|0:pop視窗 1:滿版頁面 2:警告文字|
|ReturnTitle|Y|string|結果標題|成功/錯誤標題|
|ReturnMsg|Y|string|結果訊息|成功/錯誤訊息|
|Data|Y|object|回傳資料|回傳Data欄位，無資料回傳null|
#### 回傳Data物件

| 欄位名稱         | 資料型別                 | IsNullable | 長度  | 預設值 | 說明   |
| ------------ | -------------------- | ---------- | --- | --- | ---- |
| ContractFile | ContractFileResponse | Y          | -   | -   | 合約檔案 |
- ContractFile欄位

| 欄位名稱        | 資料型別   | IsNullable | 長度  | 預設值 | 說明              |
| ----------- | ------ | ---------- | --- | --- | --------------- |
| FileName    | string | N          | 100 | -   | 檔案名稱            |
| FileContent | string | N          | max | -   | Html檔案轉換成String |

#### Response 範例

``` json
{
	"ReturnCode":0,
	"RedirectUrl":"string",
	"ReturnType":"0",
	"ReturnTitle":"string",
	"ReturnMsg":"string",
	"Data":
	{
	    "ContractFile":{	
            "FileName":"雲端帳號申請合約",
            "FileContent" :"<Html><style>...</style><title>Crystal Report Viewer</title><body>...</body></Html>"
	    }        
	}
}
```

[返回目錄](#目錄)
## 取得憑證清單 
(待確認-小車)
打小車api
[返回目錄](#目錄)

## 取得憑證明細
(待確認-小車)
打小車api
[返回目錄](#目錄)

# 加簽程序

## Login
#### ==== Post/Login====

### 需求描述:
> 1. 取得app端用SDK呼叫台網[裝置認證-Sign]前，需要帶入的Token等相關資訊。
> 2. 取得app端用SDK呼叫台網[檢查憑證-Pkcs7]前，需要帶入的Token等相關資訊。
> 3. 取得app端用SDK呼叫台網[建立憑證-Cert]前，需要帶入的Token等相關資訊。
> 4. 取得app端用SDK呼叫台網[加簽-Pdf]前，需要帶入的Token等相關資訊。

### 功能設計與實作
- 模組：小車電子合約APP化
- 類別：`MidSigAPPController`
- 方法：`Task<LoginViewModel> Login(string verifyToken)`
- 資料表：[MidSigMain]
### 處理商業邏輯 
1. 基礎檢核AppBaseCheck(參考既有BaseCheck)
	-  帶入參數有分二種
		- verifyToken=midSigId
		- TwcaPostbackModel=台網執行後回傳資訊
			- 檢核 CustomerInfo是否為空
			- 檢核 台網端回傳resultCode & 是否為最新命令
	-  用midSigId去找[MidSigMain]主檔資料並做檢核
		- 檢核
			- midSigMain是否有資料
			- 台網執行後回傳資訊ReturnCode是否成功(0、0000)
			- midSigMain.ExperidDate < DateTime.UtcNow 比對是否過期
		- 根據動作，依據MidSigMain前往DB取得
			- 身分證字號[UserIdentityNumber]
			- 電話號碼[UserIdentityNumber]
			- 檔案清單[MidSigFiles]
				- 檔案清單根據主表[MidSigMain]，[MidSigId]欄位篩選取得
		- CAType固定帶"12"，12=MID 裝置確認，不選電信業者
2. 執行Login動作，呼叫對應的mid共用。
	1. [取得建立裝置認證參數-SignPost](https://dev.azure.com/chaileaseholding/MID/_wiki/wikis/MID.wiki/16914/%E5%BE%8C%E7%AB%AFMID%E5%85%B1%E7%94%A8%E7%B3%BB%E7%B5%B1%E6%96%87%E4%BB%B6?anchor=%E5%8F%96%E5%BE%97%E5%BB%BA%E7%AB%8B%E8%A3%9D%E7%BD%AE%E8%AA%8D%E8%AD%89%E5%8F%83%E6%95%B8)
	2. [取得檢查是否有憑證流程參數-Pkcs7Post](https://dev.azure.com/chaileaseholding/MID/_wiki/wikis/MID.wiki/16914/%E5%BE%8C%E7%AB%AFMID%E5%85%B1%E7%94%A8%E7%B3%BB%E7%B5%B1%E6%96%87%E4%BB%B6?anchor=%E5%8F%96%E5%BE%97%E6%AA%A2%E6%9F%A5%E6%98%AF%E5%90%A6%E6%9C%89%E6%86%91%E8%AD%89%E6%B5%81%E7%A8%8B%E5%8F%83%E6%95%B8)
	3. [取得憑證發放流程參數-CertPost](https://dev.azure.com/chaileaseholding/MID/_wiki/wikis/MID.wiki/16914/%E5%BE%8C%E7%AB%AFMID%E5%85%B1%E7%94%A8%E7%B3%BB%E7%B5%B1%E6%96%87%E4%BB%B6?anchor=%E5%8F%96%E5%BE%97%E6%86%91%E8%AD%89%E7%99%BC%E6%94%BE%E6%B5%81%E7%A8%8B%E5%8F%83%E6%95%B8)
	4. [取得建立檔案加簽流程參數-PdfPost](https://dev.azure.com/chaileaseholding/MID/_wiki/wikis/MID.wiki/16914/%E5%BE%8C%E7%AB%AFMID%E5%85%B1%E7%94%A8%E7%B3%BB%E7%B5%B1%E6%96%87%E4%BB%B6?anchor=%E5%8F%96%E5%BE%97%E5%BB%BA%E7%AB%8B%E6%AA%94%E6%A1%88%E5%8A%A0%E7%B0%BD%E6%B5%81%E7%A8%8B%E5%8F%83%E6%95%B8)
3. 新增MidSigDetails資料
4. 整理並回傳從mid共用取得的Token等相關資訊。
### 外部API

1. MID共用：[取得建立裝置認證參數-SignPost](https://dev.azure.com/chaileaseholding/MID/_wiki/wikis/MID.wiki/16914/%E5%BE%8C%E7%AB%AFMID%E5%85%B1%E7%94%A8%E7%B3%BB%E7%B5%B1%E6%96%87%E4%BB%B6?anchor=%E5%8F%96%E5%BE%97%E5%BB%BA%E7%AB%8B%E8%A3%9D%E7%BD%AE%E8%AA%8D%E8%AD%89%E5%8F%83%E6%95%B8)
2. MID共用：[取得檢查是否有憑證流程參數-Pkcs7Post](https://dev.azure.com/chaileaseholding/MID/_wiki/wikis/MID.wiki/16914/%E5%BE%8C%E7%AB%AFMID%E5%85%B1%E7%94%A8%E7%B3%BB%E7%B5%B1%E6%96%87%E4%BB%B6?anchor=%E5%8F%96%E5%BE%97%E6%AA%A2%E6%9F%A5%E6%98%AF%E5%90%A6%E6%9C%89%E6%86%91%E8%AD%89%E6%B5%81%E7%A8%8B%E5%8F%83%E6%95%B8)
3. MID共用：[取得憑證發放流程參數-CertPost](https://dev.azure.com/chaileaseholding/MID/_wiki/wikis/MID.wiki/16914/%E5%BE%8C%E7%AB%AFMID%E5%85%B1%E7%94%A8%E7%B3%BB%E7%B5%B1%E6%96%87%E4%BB%B6?anchor=%E5%8F%96%E5%BE%97%E6%86%91%E8%AD%89%E7%99%BC%E6%94%BE%E6%B5%81%E7%A8%8B%E5%8F%83%E6%95%B8)
4. MID共用：[取得建立檔案加簽流程參數-PdfPost](https://dev.azure.com/chaileaseholding/MID/_wiki/wikis/MID.wiki/16914/%E5%BE%8C%E7%AB%AFMID%E5%85%B1%E7%94%A8%E7%B3%BB%E7%B5%B1%E6%96%87%E4%BB%B6?anchor=%E5%8F%96%E5%BE%97%E5%BB%BA%E7%AB%8B%E6%AA%94%E6%A1%88%E5%8A%A0%E7%B0%BD%E6%B5%81%E7%A8%8B%E5%8F%83%E6%95%B8)

###  SQL語法

共用：
```sql
SELECT * FROM [APP_COMPRE].[dbo].[MidSigMain]
WHERE [MidSigId] = @midSigId
```

```sql=
insert into MidSigDetails
```

加簽：
```sql
SELECT * FROM [APP_COMPRE].[dbo].[MidSigFiles]
WHERE [MidSigId] = @midSigId
```

### Request

#### Header
| 項目          | 說明                                           |
| ----------- | -------------------------------------------- |
| HTTP Method | Post                                         |
| Header      | Content-Type:application/json; charset=UTF-8 |
| Uri         | api/MidSigAPP/Login                          |
#### Body 欄位
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱        | IsNullable | 資料型別   | 名稱       | 說明                        |
| ----------- | ---------- | ------ | -------- | ------------------------- |
| verifyToken | N          | string | MidSigId | verifyToken此步驟為[MidSigId] |
|             |            |        |          |                           |
#### Body 範例

``` json
{
    "verifyToken":"0008f1fd61f0499aa3e67e2e6ae22c19"
}
```

### Response

#### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱        | IsNullable | 資料型別   | 名稱     | 說明                    |     |
| ----------- | ---------- | ------ | ------ | --------------------- | --- |
| ReturnCode  | N          | int    | 回傳代碼   | 成功:0/錯誤:-1            |     |
| RedirectUrl | Y          | string | 轉導網址   | 下一步驟的route            |     |
| ReturnType  | N          | string | 結果顯示類型 | 0:pop視窗 1:滿版頁面 2:警告文字 |     |
| ReturnTitle | Y          | string | 結果標題   | 成功/錯誤標題               |     |
| ReturnMsg   | Y          | string | 結果訊息   | 成功/錯誤訊息               |     |
| Data        | Y          | object | 回傳資料   | 回傳Data欄位，無資料回傳null    |     |

#### 回傳Data物件

| 欄位名稱      | 資料型別          | IsNullable | 長度  | 預設值 | 說明  |
| --------- | ------------- | ---------- | --- | --- | --- |
| LoginInfo | LoginResponse | Y          | -   | -   |     |
- LoginInfo欄位

| 欄位名稱            | 資料型別   | IsNullable | 長度  | 預設值        | 說明                                                                                                 |
| --------------- | ------ | ---------- | --- | ---------- | -------------------------------------------------------------------------------------------------- |
| ReturnCode      | int    | N          |     | 回傳代碼       | 成功:0/錯誤:-1                                                                                         |
| ReturnMsg       | string | Y          |     | 結果訊息       | 成功/錯誤訊息                                                                                            |
| VerifyNo        | string | N          |     | 驗證編號(唯一值)  | DateTime.Now.ToString("yyyyMMdd") + Guid.NewGuid().ToString("N")                                   |
| BusinessNo      | string | N          |     | 應用平台代號     | 由TWID Portal產生後提供                                                                                  |
| ApiVersion      | string | N          |     | API 版本     | 固定值"1.0"                                                                                           |
| HashKeyNo       | string | N          |     | HashKey 組別 | 預設"1"                                                                                              |
| IdentifyNo      | string | N          |     | 驗證碼        | Identify ( BusinessNo + ApiVersion + HashKeyNo + VerifyNo + ReturnParams + InputParams + HashKey ) |
| Token           | string | N          |     | 通行證        | login後的response                                                                                    |
| RequestUrl      | string | N          |     | Do網址       | 中租台網server+功能URL                                                                                   |
| TWIDPortalUrl   | string | N          |     | 台網網址       | 中租台網server                                                                                         |
| MIDOutputParams | object | Y          |     | MID資訊      | 當Platform為1時存在                                                                                     |
#### Response 範例

``` json
{
	"ReturnCode":0,
	"RedirectUrl":"string",
	"ReturnType":"0",
	"ReturnTitle":"string",
	"ReturnMsg":"string",
	"Data":
	{
	    "LoginInfo":{			            
			"ReturnCode":0,
			"ReturnMsg":"string",
			"VerifyNo":"string",
			"BusinessNo":"string",
			"ApiVersion":"string",
			"HashKeyNo":"string",
			"IdentifyNo":"string",
			"Token":"string",
			"RequestUrl":"string",
			"TWIDPortalUrl":"string",
			"MIDOutputParams":	{
				"ReqSeq":"string",
				"SessionKey":"string",
				"Profile":"string"
			}
	    }        
	}
}
```

[返回目錄](#目錄)



## 憑證安裝完成 
原：CertBack
建議與4-3共用，僅開一隻API。
 !!!待確認(原本是展期Type04，才會進入這動作)
#### ==== Post/CertBack ====
### 需求描述:
> 取得Cert憑證資料並回傳(APP)，呼叫小車API回傳相關資訊。
### 功能設計與實作
- 模組：小車電子合約APP化
- 類別：`MidSigAPPController`
- 方法：`Task<CertBackViewModel> CertBack(TwcaPostbackModel request)`
- 資料表：[MidSigMain]
### 處理商業邏輯 
1. 基礎檢核AppBaseCheck(參考既有BaseCheck)
	-  帶入參數有分二種
		- verifyToken=midSigId
		- TwcaPostbackModel=台網執行後回傳資訊
			- 檢核 CustomerInfo是否為空
			- 檢核 台網端回傳resultCode & 是否為最新命令
	-用midSigId去找[MidSigMain]主檔資料並做檢核
		- midSigMain是否有資料
		- 台網執行後回傳資訊ReturnCode是否成功(0、0000)
		- midSigMain.ExperidDate < DateTime.UtcNow 比對是否過期
3. 如果是展期Type04才執行以下步驟 !!!待確認
4. 呼叫mid共用GetMidCertInfo((​/Mid​/v1​/Transaction​/GetMidCertInfo)，取得Cert憑證資料。
5. 呼叫小車API
	- 呼叫路徑存在DB[CallBackUrl]欄位，目前有二個分別是"FAS_CIS_API"、"FASUCAR_API"。
		- 如果小車提供的api路徑跟DB存放的不同，要確認是不是打不同組
6. 解析後回傳更新DB(MidSigMain)
```C#
var result = await _commonService.GetApiResultModel<LittleCarReturnModel>(apiResult);
if ((result?.Result ?? null) != null  && result.Result.ReturnCode == LittleCarReturnCode.Success.GetHashCode())
{
    sigMain.SignatureDate = DateTime.UtcNow;
    sigMain.SignatureComplete = MidSigMainSignatureComplete.Complete;
    //sigMain.SourceIp = _httpContextAccessor.HttpContext?.Connection?.RemoteIpAddress?.ToString();
    if (_httpContextAccessor.HttpContext?.Request?.Headers?.TryGetValue("X-Forwarded-For", out var forwardedForValues) ?? false)
    {
        sigMain.SourceIp = forwardedForValues.FirstOrDefault()?
                                    .Split(',').FirstOrDefault()?
                                    .Split(':').FirstOrDefault();
    }
    
    _midSigRepository.UpdateMidSigMain(sigMain);

    return true;
}
```
5. 回傳Cert憑證資料(MidCertInfo))
### 外部API
1. MID共用：GetMidCertInfo
#### Request
##### Header

| 項目          | 說明                                           |
| ----------- | -------------------------------------------- |
| HTTP Method | Get                                          |
| Header      | Content-Type:application/json; charset=UTF-8 |
| X-API-KEY   | apikey-when-MidSignWeb-is-client             |
| Uri         | MID/v1/Transaction/GetMidCertInfo            |
| Name        | MID-System-Id                                |
#####  Body 
| 欄位名稱          | IsNullable | 資料型別   | 名稱       | 備註        |
| ------------- | ---------- | ------ | -------- | --------- |
| TransactionId | N          | string | Mid交易流水號 | FromQuery |
```json=
{
	"TransactionId":"string"
}
```

#### Response
##### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱         | IsNullable | 資料型別   | 名稱       | 說明    |
| ------------ | ---------- | ------ | -------- | ----- |
| Id           | N          | int    | 身分證字號    | AES加密 |
| PhoneNumber  | N          | string | 手機號碼     | AES加密 |
| CustomerInfo | N          | string | 客戶資料JSON | AES加密 |

##### Response 範例

```json=
{
	"Id":"string",
	"PhoneNumber":"string",
	"CustomerInfo":"string"
}
```
### SQL語法

```sql=
UpdateMidSigMain
  _context.Update(model);
  _context.SaveChanges();
```
### Request

#### Header
| 項目          | 說明                                           |
| ----------- | -------------------------------------------- |
| HTTP Method | Post                                         |
| Header      | Content-Type:application/json; charset=UTF-8 |
| Uri         | api/MidSigAPP/CertBack                       |
#### Body 欄位
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱        | IsNullable | 資料型別   | 名稱       | 說明                        |
| ----------- | ---------- | ------ | -------- | ------------------------- |
| verifyToken | N          | string | MidSigId | verifyToken此步驟為[MidSigId] |
|             |            |        |          |                           |
#### Body 範例

``` json
{
    "verifyToken":"string"
}
```

### Response

#### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱        | IsNullable | 資料型別   | 名稱     | 說明                    |     |
| ----------- | ---------- | ------ | ------ | --------------------- | --- |
| ReturnCode  | N          | int    | 回傳代碼   | 成功:0/錯誤:-1            |     |
| RedirectUrl | Y          | string | 轉導網址   | 下一步驟的route            |     |
| ReturnType  | N          | string | 結果顯示類型 | 0:pop視窗 1:滿版頁面 2:警告文字 |     |
| ReturnTitle | Y          | string | 結果標題   | 成功/錯誤標題               |     |
| ReturnMsg   | Y          | string | 結果訊息   | 成功/錯誤訊息               |     |
| Data        | Y          | object | 回傳資料   | 回傳Data欄位，無資料回傳null    |     |

#### 回傳Data物件

| 欄位名稱        | 資料型別         | IsNullable | 長度  | 預設值 | 說明  |
| ----------- | ------------ | ---------- | --- | --- | --- |
| midCertInfo | CerViewModel | Y          | -   | -   |     |
- LoginInfo欄位

| 欄位名稱          | 資料型別     | IsNullable | 長度  | 預設值    | 說明         |
| ------------- | -------- | ---------- | --- | ------ | ---------- |
| MidCertInfoId | string   | N          |     | 回傳代碼   |            |
| TransactionId | string   | Y          |     | 結果訊息   |            |
| X509Cert      | string   | N          |     | 憑證     | 文件簽署憑證     |
| CertCN        | string   | N          |     | 憑證CN   |            |
| CertSN        | string   | N          |     | 憑證序號   | 憑證序號(16進位) |
| CertNotBefore | DateTime | Y          |     | 憑證生效日期 |            |
| CertNotAfter  | DateTime | Y          |     | 憑證截止日期 |            |

#### Response 範例

``` json
{
	"ReturnCode":0,
	"RedirectUrl":"string",
	"ReturnType":"0",
	"ReturnTitle":"string",
	"ReturnMsg":"string",
	"Data":
	{
	    "LoginInfo":{			            
			"MidCertInfoId":"string",
			"TransactionId":"string",
			"X509Cert":"string",
			"CertCN":"string",
			"CertSN":"string",
			"CertNotBefore":DateTime,
			"CertNotAfter":DateTime
	    }        
	}
}
```

[返回目錄](#目錄)
`
## 加簽完成

原：Success
建議與4-3共用，僅開一隻API。
#### ==== Post/PdfSuccess ====
### 需求描述:
> 取得Cert憑證資料並回傳(APP)，呼叫小車API回傳相關資訊。
### 功能設計與實作
- 模組：小車電子合約APP化
- 類別：`MidSigAPPController`
- 方法：`Task<PdfSuccessViewModel> PdfSuccess(TwcaPostbackModel request)`
- 資料表：[MidSigMain]
### 處理商業邏輯 
1. 基礎檢核AppBaseCheck(參考既有Success)
	-  帶入參數有分二種
		- verifyToken=midSigId
		- TwcaPostbackModel=台網執行後回傳資訊
			- 檢核 CustomerInfo是否為空
			- 檢核 台網端回傳resultCode & 是否為最新命令
	-用midSigId去找[MidSigMain]主檔資料並做檢核
		- midSigMain是否有資料
		- 台網執行後回傳資訊ReturnCode是否成功(0、0000)
		- midSigMain.ExperidDate < DateTime.UtcNow 比對是否過期
2. 呼叫mid共用GetMidCertInfo((​/Mid​/v1​/Transaction​/GetMidCertInfo)，取得Cert憑證資料。
3. 呼叫mid共用GetSignFile((​/Mid​/v1​/Transaction​/GetSignFile)，取得取得加簽過的PDF檔案UUID。
	- 取得檔案，並檢核檔按是否為空
4. 呼叫小車API
	- 呼叫路徑存在DB[CallBackUrl]欄位，目前有二個分別是"FAS_CIS_API"、"FASUCAR_API"。
		- 如果小車提供的api路徑跟DB存放的不同，要確認是不是打不同組
5. 解析後回傳更新DB(MidSigMain)
```C#
var result = await _commonService.GetApiResultModel<LittleCarReturnModel>(apiResult);
if ((result?.Result ?? null) != null  && result.Result.ReturnCode == LittleCarReturnCode.Success.GetHashCode())
{
    sigMain.SignatureDate = DateTime.UtcNow;
    sigMain.SignatureComplete = MidSigMainSignatureComplete.Complete;
    //sigMain.SourceIp = _httpContextAccessor.HttpContext?.Connection?.RemoteIpAddress?.ToString();
    if (_httpContextAccessor.HttpContext?.Request?.Headers?.TryGetValue("X-Forwarded-For", out var forwardedForValues) ?? false)
    {
        sigMain.SourceIp = forwardedForValues.FirstOrDefault()?
                                    .Split(',').FirstOrDefault()?
                                    .Split(':').FirstOrDefault();
    }
    
    _midSigRepository.UpdateMidSigMain(sigMain);

    return true;
}
```
5. 回傳Cert憑證資料(MidCertInfo))
### 外部API

1. MID共用：GetMidCertInfo
#### Request
##### Header

| 項目          | 說明                                           |
| ----------- | -------------------------------------------- |
| HTTP Method | Get                                          |
| Header      | Content-Type:application/json; charset=UTF-8 |
| X-API-KEY   | apikey-when-MidSignWeb-is-client             |
| Uri         | MID/v1/Transaction/GetMidCertInfo            |
| Name        | MID-System-Id                                |
#####  Body 
| 欄位名稱          | IsNullable | 資料型別   | 名稱       | 備註        |
| ------------- | ---------- | ------ | -------- | --------- |
| TransactionId | N          | string | Mid交易流水號 | FromQuery |
```json=
{
	"TransactionId":"string"
}
```

#### Response
##### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱         | IsNullable | 資料型別   | 名稱       | 說明    |
| ------------ | ---------- | ------ | -------- | ----- |
| Id           | N          | int    | 身分證字號    | AES加密 |
| PhoneNumber  | N          | string | 手機號碼     | AES加密 |
| CustomerInfo | N          | string | 客戶資料JSON | AES加密 |

##### Response 範例

```json=
{
	"Id":"string",
	"PhoneNumber":"string",
	"CustomerInfo":"string"
}
```
2. MID共用：GetSignFile
#### Request
##### Header

| 項目          | 說明                                           |
| ----------- | -------------------------------------------- |
| HTTP Method | Get                                          |
| Header      | Content-Type:application/json; charset=UTF-8 |
| X-API-KEY   | apikey-when-MidSignWeb-is-client             |
| Uri         | MID/v1/Transaction/GetSignFile               |
| Name        | MID-System-Id                                |
#####  Body 
| 欄位名稱          | IsNullable | 資料型別   | 名稱       | 備註        |
| ------------- | ---------- | ------ | -------- | --------- |
| TransactionId | N          | string | Mid交易流水號 | FromQuery |
```json=
{
	"TransactionId":"string"
}
```

#### Response
##### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱          | IsNullable | 資料型別   | 名稱         | 說明  |
| ------------- | ---------- | ------ | ---------- | --- |
| PdfUUid       | N          | string | PDF檔案 UUID | 加簽前 |
| SignedPdfUUid | N          | string | PDF檔案 UUID | 加簽後 |

##### Response 範例

```json=
{
	"PdfUUid":"string",
	"SignedPdfUUid":"string"
}
```
### SQL語法

```sql=
UpdateMidSigMain
  _context.Update(model);
  _context.SaveChanges();
```
### Request

#### Header
| 項目          | 說明                                           |
| ----------- | -------------------------------------------- |
| HTTP Method | Post                                         |
| Header      | Content-Type:application/json; charset=UTF-8 |
| Uri         | api/MidSigAPP/CertBack                       |
#### Body 欄位
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱        | IsNullable | 資料型別   | 名稱       | 說明                        |
| ----------- | ---------- | ------ | -------- | ------------------------- |
| verifyToken | N          | string | MidSigId | verifyToken此步驟為[MidSigId] |
|             |            |        |          |                           |
#### Body 範例

``` json
{
    "verifyToken":"string"
}
```

### Response

#### Body
> Y 為必要傳送欄位，N 為自由選擇傳送欄位。空白則輸入-

| 欄位名稱        | IsNullable | 資料型別   | 名稱     | 說明                    |     |
| ----------- | ---------- | ------ | ------ | --------------------- | --- |
| ReturnCode  | N          | int    | 回傳代碼   | 成功:0/錯誤:-1            |     |
| RedirectUrl | Y          | string | 轉導網址   | 下一步驟的route            |     |
| ReturnType  | N          | string | 結果顯示類型 | 0:pop視窗 1:滿版頁面 2:警告文字 |     |
| ReturnTitle | Y          | string | 結果標題   | 成功/錯誤標題               |     |
| ReturnMsg   | Y          | string | 結果訊息   | 成功/錯誤訊息               |     |
| Data        | Y          | object | 回傳資料   | 回傳Data欄位，無資料回傳null    |     |

#### 回傳Data物件

| 欄位名稱        | 資料型別         | IsNullable | 長度  | 預設值 | 說明  |
| ----------- | ------------ | ---------- | --- | --- | --- |
| midCertInfo | CerViewModel | Y          | -   | -   |     |
- LoginInfo欄位

| 欄位名稱          | 資料型別     | IsNullable | 長度  | 預設值    | 說明         |
| ------------- | -------- | ---------- | --- | ------ | ---------- |
| MidCertInfoId | string   | N          |     | 回傳代碼   |            |
| TransactionId | string   | Y          |     | 結果訊息   |            |
| X509Cert      | string   | N          |     | 憑證     | 文件簽署憑證     |
| CertCN        | string   | N          |     | 憑證CN   |            |
| CertSN        | string   | N          |     | 憑證序號   | 憑證序號(16進位) |
| CertNotBefore | DateTime | Y          |     | 憑證生效日期 |            |
| CertNotAfter  | DateTime | Y          |     | 憑證截止日期 |            |

#### Response 範例

``` json
{
	"ReturnCode":0,
	"RedirectUrl":"string",
	"ReturnType":"0",
	"ReturnTitle":"string",
	"ReturnMsg":"string",
	"Data":
	{
	    "LoginInfo":{			            
			"MidCertInfoId":"string",
			"TransactionId":"string",
			"X509Cert":"string",
			"CertCN":"string",
			"CertSN":"string",
			"CertNotBefore":DateTime,
			"CertNotAfter":DateTime
	    }        
	}
}
```

[返回目錄](#目錄)


待確認
1. 加簽完成、憑證安裝完成
	要先拿到TransactionId，確認如何拿到。
2. Login CAType
	1. ign之後，看資料要我存還是app存。
	2. 存進去才會有，TransactionId



[【中租迪和】TWID_Portal_應用平台整合手冊3.4.1.0(Sign+Cert+P7+PDF)_20240513(含SDK).docx](https://github.com/Vincent3054/doc/files/15303464/TWID_Portal_.3.4.1.0.Sign%2BCert%2BP7%2BPDF._20240513.SDK.docx)



From: 李竣翔(Sean) <sean@twca.com.tw> 
Sent: Monday, May 13, 2024 3:47 PM
To: MisuSu【蘇亭郡】 <MisuSu@chailease.com.tw>; 陳昱翰(jasperchen) <jasperchen@twca.com.tw>
Cc: ArthurYu【游超能】 <ArthurYu@chailease.com.tw>; SkyChen【陳建成】 <SkyChen@chailease.com.tw>
Subject: RE: 關於TWID Portal 介接SDK問題詢問

Hi Misu,

	API Login response內容MIDaOutputParams，會在action為SIGN，且CAType8、9、12，且Platform = 1時，有回傳值。
	以本案範圍，執行MID身分識別時，CAType應為12，Login response wording有部分筆誤，已更新如附件，再請參考，
另CAType可以參考3.1.2章節。
 



Best Regards,
Sean Lee

From: MisuSu@chailease.com.tw <MisuSu@chailease.com.tw> 
Sent: Monday, May 13, 2024 2:49 PM
To: 李竣翔(Sean) <sean@twca.com.tw>; 陳昱翰(jasperchen) <jasperchen@twca.com.tw>
Cc: ArthurYu@chailease.com.tw; SkyChen@chailease.com.tw
Subject: 關於TWID Portal 介接SDK問題詢問

Hi Sean,
有個Login參數問題想請教：
Request的InputParams裡有一部分是MIDInputParams，描述是「InputParams 子項目（ 當 Action 為 SIGN 且 CAType 包含 8、 9 時需存在）」
Platform（1: SDK 版、2: Web 版），對應的Response 內會有MIDaOutputParams回傳參數
以下問題
  

        1.     想請問描述的意思是只有在Action為SIGN才會有回傳，假如是CERT、PDF、PKCS7的狀況下，
即使傳了Platform = 1，也不會有相關的MIDaOutputParams回傳嗎？
2.     CAType的8、9分別代表哪些裝置？

以上再麻煩協助了，謝謝！

Best Regards,
中租迪和股份有限公司
數位科技發展科 蘇亭郡
電話：(02)87526388#76079

