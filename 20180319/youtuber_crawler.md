補充 - Youtuber 相關資料爬蟲程式參考
================

-   資料來源網站: [SocialBlade](https://socialblade.com/)
-   資料擷取時間: 2018/03/14

R code
------

``` r
## 安裝並載入 rvest 套件
# install.packages("rvest")
library(rvest)

## 指定目標網頁網址
url = "https://socialblade.com/youtube/top/5000/mostsubscribed"

## 讀取網頁內容
page = read_html(url)

## 根據 CSS Selector 獲取節點資訊
node = html_nodes(page, css="div div div div a")

## 擷取超連結網址路徑
href = html_attr(node, "href")

## 由於無法透過 CSS 選擇器單純選擇 top 5000 資訊，因此透過人工手動判斷節點位置
## 這邊的 top 5000 就是我們新的爬蟲目標網址
top5000 = href[74:5073]

## 記錄開始執行時間
t0 = proc.time()

## 先準備一個空的矩陣物件
info = matrix(, 5000, 6)

## 用迴圈依序爬取 5000 個網頁的內容(注意：可能需時數小時！)
for (i in 1:5000) {
  url = paste0("https://socialblade.com", top5000[i])
  page = read_html(url)
  node = html_nodes(page, css="#YouTubeUserTopInfoBlock br+ span")
  
  ## 如果擷取的節點不是預期的 6 個，可能代表擷取內容有誤，就先跳過一次迴圈
  if (length(node) != 6) next
  
  ## 把擷取文字內容存放到 info 矩陣的第 i 列
  info[i,] = html_text(node)
  
  ## Print 出訊息來掌控迴圈進度及內容正確性
  message(i, ":", info[i,])
  
  ## 每執行 50 次迴圈就隨機暫停 1~3 秒
  if (i %% 50 == 0) Sys.sleep(runif(1,1,3))
}

## 計算執行經過時間
t1 = proc.time() - t0

## 轉換為 data frame 型態以便後續分析運用
dat = as.data.frame(info)

## 重新命名欄位名稱
names(dat) = c("UPLOADS", "SUBSCRIBERS", "VIEWS", "COUNTRY", "TYPE", "CREATED")
```
