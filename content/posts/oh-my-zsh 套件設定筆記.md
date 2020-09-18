---
title: "oh-my-zsh 套件設定筆記"
tags: ["code", "linux", "shell", "zsh"]
date: 2019-12-23T17:16:00+08:00
draft: false
author: "tdc"
aliases: [ "/545"]
featuredImagePreview: "[https://p176.p0.n0.cdn.getcloudapp.com/items/eDu9yQkN/collage.png][1]"
featuredImage: "[https://p176.p0.n0.cdn.getcloudapp.com/items/eDu9yQkN/collage.png][2]"

---
# oh-my-zsh 套件設定筆記

# 序

有時候看到朋友初入 Linux 坑不久，看他們還在辛苦的用 `bash` 看得我就很心累。示範了一下自己常用的環境之後居然被當作神一樣（Nani the F\*ck）  

對此就生出一篇簡單的筆記。  

# 小孩子才做選擇

在這篇文章可以學到  

- [x] oh-my-zsh
- [x] 主題
- [x] oh-my-zsh 套件
- [x] 套件管理

總之先來安裝 `oh-my-zsh` 吧。  

```shell
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

另外忘記說他必須依賴 `git` 以及 `curl` 或 `wget`

接著來挑選自己喜歡的主題，例如 [repo][3] 提供的，如果找到的主題並未在 repo 內，只需要將想要的主題下載到 `~/.oh-my-zsh/themes` 內，並且編輯 `~/.zshrc` 內的設定如下。

```ini
ZSH_THEME="ys"
```

而 `zsh` 最好玩的部分就是 `zsh tab completion select` ，但是應該有不少人會發現有些程式無法使用 `tab` 大法，實際上是需要安裝 套件 的，支援的套件列表在 `~/.oh-my-zsh/plugins` ，並且將想要啟用的套件編輯在 `~/.zshrc` 

```shell
plugins=(git heroku)
```

但這太麻煩了，如果我需要安裝別人的 `repo` 提供的套件這就非常的麻煩，我還要先 `git cone` 放到 `~/.oh-my-zsh/plugins` 內。  

這時候就需要用到 `antigen` 來處理比較方便（？）。  

```shell
curl -L git.io/antigen > ~/.oh-my-zsh/plugins/antigen.zsh
```

接著打開 `~/.zshrc` 在底下加入想要使用的插件，例如。

```shell
# 填入你下載的 antigen.zsh 位置
source $HOME/.oh-my-zsh/plugins/antigen.zsh

# 想要安裝的套件
antigen bundle git
antigen bundle pip
antigen bundle command-not-found

# 外部來源也可以！
antigen bundle zsh-users/zsh-syntax-highlighting
antigen bundle zsh-users/zsh-autosuggestions

# 安裝主題
antigen theme ys

# 啟用
antigen apply
```

接著重新讀取一下確認是否正常操作。  

```shell
source ~/.zshrc
```

# 結

結果這邊我想很久要寫了，結果最近又發現一個 `zplug` 好像更有潛能...

  


[1]:	https://raw.githubusercontent.com/zsh-users/antigen/develop/antigen.png
[2]:	https://raw.githubusercontent.com/zsh-users/antigen/develop/antigen.png
[3]:	https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes