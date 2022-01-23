---
title: "Rclone備份筆記"
tags: ["Rclone", "筆記", "GUI"]
date: 2022-01-23T13:33:00+08:00
draft: false
author: "永格天(武則天、wxex)"
featuredImagePreview: "https://dr.sudo.host/iLrxhB+"
featuredImage: "https://dr.sudo.host/iLrxhB+"
---

# Rclone 備份筆記

如題，紀錄一下自己怎麼使用 Rclone 備份東東的
這裡主要是備份影片，然後散在多個硬碟內，結構如下 2 個硬碟

E:.
├─ 備份
│ ├─OOO 的獅子
│ ├─ 為美 VVVVVVV！
│ ├─ 魔法 OOOOOOO
.......

F:.
├─ 備份
│ ├─OOO 的獅子
│ ├─XXX 不存在 XXX－
│ ├─AAAAA Beats！
.......

看的出來他有**部分資料夾是重複的**，本來應該是要塞在同一個資料夾的，但我原本硬碟沒空間了，所以才臨時改塞第二顆硬碟內。

有想說把多顆硬碟做[JBOD](https://zh.wikipedia.org/wiki/RAID)來擴充容量，不過手邊沒有多的硬碟可以臨時存放資料
→ 只好先把檔案丟雲端硬碟
→ 等丟好後再格式化自己的幾顆實體硬碟來串成 JBOD
→ 爾後再把東西從雲端載回來。

---

## Rclone 安裝

### 載點

![Rclone Downloads](https://dr.sudo.host/9vIUoz+)
請直接去[官網下載頁面](https://rclone.org/downloads/)找你要下載的版本下載，或是官方 Github[版本下載點](https://github.com/rclone/rclone/releases)找

### windows 環境設定(參考用)

如果你是 Windows 的版本可以跟我一樣，在電腦 C 槽中找個自己平常不會亂動的資料夾(或任何你爽的地方)解壓縮下載回來的檔案，就可以直接用了。
![](https://dr.sudo.host/6PtgwA+)

然後如果要在任何地方都可以使用 Rclone 指令
記得改一下電腦的環境變數，如下圖"本機"內按右鍵，剩下跟著圖做，只是最後"新增 Path"時，把你的資料夾位置丟上去就好

(下面這張圖用是舊圖示意，圖中新增的位置是"C:\\Octave\\Octave-5.2.0\\mingw64\\bin"，在這篇文章中應該要改成"C:\\My_Programs\\rclone-v1.57.0-windows-amd64")
![加入系統環境變數中](https://dr.sudo.host/v8LBvD+)

---

## Rclone 設定

逛過一輪網路上的教學後發現絕大多數都是用命令列來設定
這裡說一下用 Rclone 內建的 GUI 設定

### 開啟 Rclone GUI

打開你的命令列窗(cmd、powershell、Terminal...)貼上執行

```
rclone rcd --rc-web-gui --rc-user=admin --rc-pass=password
```

會在自動打開你的瀏覽器瀏覽 localhost:5572 (Rclone 的控制台)
![](https://dr.sudo.host/UqJp92+)

### 設定新的 config

1. 點左邊的 "config"，並點 "Create a New Config"
   ![](https://dr.sudo.host/icwlyZ+)
2. 幫你的雲端硬碟取個你喜歡的名子跟選擇你的雲端硬碟的廠牌，並按"Next"
   ![](https://dr.sudo.host/0H7sdC+)
3. 創建自己的 Google Drive 客戶端，這方面直接貼個我看過覺得很清楚的[創建教學](https://hackmd.io/@adeliae/H1ZRCUfaU#Google-Application-Client-Id-%E5%BB%BA%E7%AB%8B)，拿到後的 2 條字串貼上來就行了
   ![](https://dr.sudo.host/QI79Qy+)

4. 會出現一個帳戶的授權介面，選自己要連結的帳號，最後出現 Success 就成功了
   ![](https://dr.sudo.host/bqhKjo+)
   ![](https://dr.sudo.host/NfEBXR+)
   ![](https://dr.sudo.host/AyBDw6+)

---

## Rclone 上傳指令

OK 終於到重點了~
總體而言可以用一個指令帶過

```
rclone copy "C:\Users\we684123\Downloads\備份" "hex:test上船合併機制" -P --size-only --bwlimit 1.78m --transfers 1 --local-encoding None
```

格式：

```
rclone copy source:sourcepath dest:destpath [-flags]
```

flags 意義：

- -P , 在終端機顯示進度
- --size-only , 比對遠端檔案時只用檔案大小來判斷是否為同一個檔案
- --bwlimit 1.78m , 限制上傳速度(這裡單位是用 MiB/s,1.78m 大概是 14.93 Mbps)
- --transfers 1 , 最大同時上傳檔案數量
- --local-encoding None , 不轉換特殊字元編碼

## Rclone 操作參考連結

- [Rclone Commands](https://rclone.org/commands/)
- [Global Flags](https://rclone.org/flags/)
- [Overview of cloud storage systems](https://rclone.org/overview/#restricted-filenames)

- [win10 Rclone 不思考ㄉ安裝與掛載](https://hackmd.io/@adeliae/H1ZRCUfaU)
- [rclone-across-transfer-each-cloud-drive-without-local-machine-bandwidth](https://caloskao.org/rclone-across-transfer-each-cloud-drive-without-local-machine-bandwidth/)
- [子風的知識庫 - [Tool] rclone 介紹 ](https://zwindr.blogspot.com/2020/01/tool-rclone.html)
- [wei115 - [心得] 用 rclone 的筆記](https://www.ptt.cc/bbs/Free_box/M.1548436333.A.194.html)
