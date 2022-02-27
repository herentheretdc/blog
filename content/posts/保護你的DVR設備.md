---
title: "保護你的DVR設備"
tags: ["cloudfalre", "security", "資安", "網路安全", "DVR", "監視器", "warp"]
date: 2022-02-27T03:44:24Z
draft: false
author: "tdc"
featuredImagePreview: https://blog.cloudflare.com/content/images/2019/04/warp-illustration-update@2x.png

featuredImage: https://blog.cloudflare.com/content/images/2019/04/warp-illustration-update@2x.png

---

# 序
之前有一陣子台灣網路環境變得很糟，打開 YouTube 或看 Netflix 什麼的都會很卡，後來才知道是因為台灣某 DVR 監視器主機被惡意攻擊被利用成 Botnet，許多房東或是公司安裝的 DVR 主機根本就是插上網路線後，管理密碼都沒有改過就放了好幾年。改了密碼還是有可能會被別人一直測試密碼有完沒完，那最安全的做法可能就是建立一個 VPN 環境，並把 DVR 主機的連上公開網路的封包用防火牆給阻斷。但每次要看監視器就要連回去其實就很麻煩，如果一直連線的話 VPN 網路很慢用起來也抓狂，那有沒有一種東西可以一直連著舒服還能確保 DVR 連線安全。  
經過研究，Cloudflare 的 Zero Trust 可以完美符合上述需求，Cloudflare 有一個專屬的 WARP 網路，在各個平台都能使用，原本 WARP 被大家當做是免費的 VPN 但其實有更厲害的用法，連上 WARP 還能保護你的網路環境安全，蘋果上的「iCloud 私密轉送
」其實也是跟 Cloudflare 租的，~~差不多你可以以解成這是同一個東西。~~

# 搞事
搞事之前先解釋一下整個架構  
![cf](https://www.cloudflare.com/static/1a1b6050c964dab0a1e764893d3f8a35/CF_SWG_Illustration_2022_Q1.png)  
簡單說其實就是使用者的裝置連上 Cloudflare 的網路後，可以存取公司內部的資源，其實就很像傳統 VPN，但他對 IT 來說可以做到更多厲害方便的功能。

需要事前準備以下東西
> - 一台不關機的電腦當作中介 *必須在同一個 LAN 下*（Raspberry Pi, NAS, Server）
> - Cloudflare 帳號

打開 [Cloudflare Zero Trust](dash.teams.cloudflare.com
) 並登入自己的帳號，首次啟用的話可能會要你設定「團隊名稱」取一個自己喜歡的就可以了，事後可以在設定頁面裡面更改。  
![1.png](https://dr.sudo.host/v6BPuU+
)

方案選擇的部分選 Free 免費即可，接著他還是會跟你要付款方式，不過結帳金額會是零不用怕，但他可能會刷個一塊錢測試
![2.png](https://dr.sudo.host/vFBL9u+
)

接著在負責當轉介的電腦上安裝 [cloudflared](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/
)  
接著在 shell/pwshell 輸入
```shell
cloudflared tunnel login
```
他會打開一個網頁（如果你是 ssh 進去 headless 主機他會給你網址），打開並登入後選擇你的帳號跟網域，完成後他會提示你可以關閉網頁了，回到主機會顯示已下載 `cert.pem`  

接著繼續輸入
```shell
cloudflared tunnel create <NAME>
```
`<Name>` 的部分須替換成自己的（記得去掉括號）  
完成後應該會長這樣
```shell
❯ cloudflared tunnel create tochu
Tunnel credentials written to /Users/tdc/.cloudflared/xxxxxxx-eec0-481f-xxxxxx.json. cloudflared chose this file based on where your origin certificate was found. Keep this file secret. To revoke these credentials, delete the tunnel.

Created tunnel tochu
 with id xxxxxxx-eec0-481f-xxxxxx.json

```

接著在中介主機上打開一個資料夾看個人喜好，Linux 可以開在 `~/.cloudflared` 下，接著創建一個檔案 `xxx.yml` xxx 可以自己取。
打開剛剛建立的檔案將以下的東西複製貼上
```yaml
tunnel: xxxxxxx-eec0-481f-xxxxxx
credentials-file: ~/.cloudflared/xxxxxxx-eec0-481f-xxxxxx
.json
warp-routing:
  enabled: true
```
其中的 `tunnel` 就如同上方 create tunnel 輸出的內容直接複製貼上，`credentials-file` 的話也是剛剛輸出的內容複製貼上就可以了。

回到 shell 輸入以下指令
```shell
cloudflared tunnel route ip add 192.168.50.255/32 xxxxxxx-eec0-481f-xxxxxx
```

其中該 IP 是要輸入目標對象的 IP，並以 CIDR 的格式呈現，如果我 DVR 的主機是 `192.168.50.123` 那就輸入 `192.168.50.123/32` 即可，而後面一樣是剛剛建立的 tunnel id。

緊接著就是建立連線
```shell
cloudflared tunnel --config xxx.yml run
```
大概幾秒後就會出現類似的畫面
```shell
2022-02-27T05:14:59Z INF Starting tunnel tunnelID=xxxxxxx-eec0-481f-xxxxxx
2022-02-27T05:14:59Z INF Version 2022.2.2
2022-02-27T05:14:59Z INF Warp-routing is enabled
2022-02-27T05:15:00Z INF Connection xxxxx-89c7-xxxx-bce3-xxxxx registered connIndex=0 location=SIN
2022-02-27T05:15:01Z INF Connection xxxxx
-1a6c-xxxx-b480-xxxxx registered connIndex=1 location=NRT
2022-02-27T05:15:02Z INF Connection xxxxx
-1a6c-xxxx-b480-xxxxx registered connIndex=2 location=SIN
2022-02-27T05:15:03Z INF Connection xxxxx
-1a6c-xxxx-b480-xxxxx registered connIndex=3 location=NRT
```

表示你已經成功讓他連上 Cloudflare tunnel 網路。

接著打開 Cloudflare Zero Trust 設定頁面  
![3.png](https://dr.sudo.host/MpHmpo+)  
![4.png](https://dr.sudo.host/b71ZBl+)  

在 Split tunnels 內預設了一些常見的內網網段，要將剛剛自己設定的 IP 那段給刪除。  
如果你不知道哪一個的話可以用[轉換器](https://www.ipaddressguide.com/cidr)查一下  
一般常見應該是下面兩個，將它刪除即可，建議是將沒用到的給補回去。
![5.png](https://dr.sudo.host/0dsEMb+)

接著打開手機下載 Warp App [iOS](<https://apps.apple.com/us/app/1-1-1-1-faster-internet/id1423538627>
) [Android](<https://play.google.com/store/apps/details?id=com.cloudflare.onedotonedotonedotone>
)
打開之後點右上角進入帳號設定，最底下有一個 `Login with Cloudflare for Teams` 點下去之後他會要你輸入組織名稱，就是最一開始取的名稱，如果忘記的話可以進去 `Zero Trust` 一般設定裡面找一下，接著就正常登入，回到主畫面後剛剛紅色字的 Warp 會變成藍色的 Teams，新版的會顯示 Zero Trust。  
![6.png](<https://dr.sudo.host/U6pSyQ+>)

點擊連線，接著打開你平常用的 DVR 連線 App，IP 就是輸入你 DVR LAN 下的 IP 即可。  
收工。  


但你如果覺得 Warp 網路時好時壞的話你可以改成只有使用 DVR App 才接入 Warp 網路，方法也很簡單，只要切換為 `Include IPs and domains`，注意切換的時候他會提示會刪除預設清單並可能會造成危害，預設清單可以恢復為預設，但如果你原本有設定自己的清單請記得備份。  
![7.png](https://dr.sudo.host/ZYSNvx+)  
切換過去後輸入剛剛的 DVR IP 依然是 CIDR 格式按儲存就可以了。  
![8.png](https://dr.sudo.host/TjEuav+)


# 結論
其實 Cloudflare Zero Trust 還有很多超酷的功能，比如說自訂 DNS，L7 過濾什麼的，如果你是公司的 MIS 的話其實可以考慮一下，免費版也很大方的給 50 人使用，以後如果我熊熊沒事做想到再來補充更酷的玩法。