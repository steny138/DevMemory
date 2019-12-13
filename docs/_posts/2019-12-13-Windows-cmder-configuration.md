---
layout: post
title:  在windows 10上使用Cmder與WSL建立與Mac+Iterm2相同的開發體驗
date:   2019-12-13 10:40:00 +0800
categories: zsh oh-my-zsh windows wsl cmder
author: yuchen
---


# 使用oh-my-zsh 在Cmder上 用windows 10

## Install

- 安裝Cmder

> C:\> choco install cmdermini

- 安裝相關字體 ttf

[參考下載](https://github.com/ryanoasis/nerd-fonts)

- 安裝WSL

    使用power shell 執行

> Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux

- 安裝一個 Linux distribution
[參考下載](ms-windows-store://pdp/?ProductId=9n9tngvndl3q)

若上述的store 下載被擋住可在微軟找到下載檔

[點選此處](https://docs.microsoft.com/en-us/windows/wsl/install-manual)

- 執行 WSL，並設定 Linux 的帳號密碼

打開就會了吧...

---
以下安裝步驟可能不適用於企業內部網路

1. 可以自行從github migrate 專案到github.everxxxst.com.tw
2. apt-get 設定防火牆

```
sudo vim /etc/apt/apt.conf
Acquire::http::Proxy "http://proxy.seed.net.tw:8080";

或設定Linux Proxy
echo "export http_proxy=http://proxy.example.com"
 >> ~/.bashrc
echo "export https_proxy=https://proxy.example.com"
 >> ~/.bashrc
```
---

- 在 WSL 上 安裝zsh

`sudo apt-get install zsh`

- 在 WSL 上 安裝oh-my-zsh

`sudo sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`

- 在 WSL 上 安裝oh-my-zsh 相關套件

```
sh -c 
"$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

- 更換oh-my-zsh theme  
參考路徑
https://github.com/robbyrussell/oh-my-zsh/wiki/themes

> vim ~/.zshrc  
> 編輯 ZSH_THEME=“robbyryssell” 改為喜歡的

- 設定 docker 連結

1. 開啟 Docker on Windows 的 Setting 在 General 勾選 Expose daemon on tcp://localhost:2375 without TLS。

2. 使用 bash 修改 ~/.bashrc。

```
$ vim ~/.bashrc
# docker
PATH="$PATH:/mnt/c/Program\ Files/Docker/Docker/resources/bin"
alias docker=docker.exe
alias docker-machine=docker-machine.exe
alias docker-compose=docker-compose.exe
export DOCKER_HOST="tcp://localhost:2375"
```

## 參考連結

[在 Windows 上復刻 Mac 使用習慣](https://william-yeh.net/post/2019/03/wsl-cmder-zsh/)

[安裝powerleve9k](https://github.com/bhilburn/powerlevel9k#installation)
