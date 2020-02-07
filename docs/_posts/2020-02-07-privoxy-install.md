---
layout: post
title:  在windows 10上安裝Privoxy建立local proxy 轉發http request
date:   2020-02-07 11:40:00 +0800
categories: privoxy windows docker
author: yuchen
---

# Privoxy
建立local proxy pass to evertrust proxy with credentials  
為了在 docekr pull 不會被 evertrust proxy 一直回應 Proxy Authrization Required.

---

## Step 1 > 安裝chocolatey (以下選一種即可)

### Step 1.1 > 啟動 cmd.exe with Administrator
並執行以下指令
```
@“%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe” -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command “iex ((New-Object System.Net.WebClient).DownloadString(’https://chocolatey.org/install.ps1'))” && SET “PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin”
```
### Step 1.2 > 啟動 powershell.exe with Administrator
並執行以下指令
```
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString(’https://chocolatey.org/install.ps1'))
```
  

---
  

## Step 2 > 確認安裝成功
安裝完成後，在命令列打出下方指令確認安裝成功
```
choco --version
```
應會回傳安裝的版本號碼

### Step 2.1 > (Optional) 安裝GUI
在命令列打出下方指令安裝GUI視覺化，給不習慣用指令碼的朋友
```
choco install chocolateygui
```
  
  
---
  


## Step 3 > 安裝 privoxy
在命令列打出下方指令安裝privoxy
```
choco install privoxy
```
  
  
---
  


## Step 4 > 將 privoxy 設定為 以系統管理員身分執行
1. 在 開始工具列右鍵privoxy ，選擇以系統管理員身分執行
2. 在下方工具列右鍵privoxy > 再右鍵privoxy選內容 > 進階 > 以系統管理員身分執行

## Step 5 > 重啟privoxy
  
  
---
  

## Step 6 > privoxy > options > Edit Main Configuration
  
### Step 6.1 > 將 listen-address 改為 10.0.75.1:8118
> 貼心備註 10.0.75.1 為 docker nat 用的 對外監聽 port
> 該設定意思為 privoxy listen(監聽) 10.0.75.1 過來的 request
> #開頭的文字皆為註解，請改沒有開頭為#的文字行

### Step 6.2 > 將 enable-proxy-authentication-forwarding  由0改為1

> 貼心備註: 將proxy request 帶入credentials(即是公司AD帳號密碼), 帶入帳密會讓 evertrust proxy 比較容易通過...

### Step 6.2 > 文末 加入 forward / secproxy.evertrust.com.tw:6588

> 貼心備註: 將剛剛監聽的request(10.0.75.1) 轉發到 secproxy.evertrust.com.tw(evertrust proxy)

### Step 6.3 Main Configuration設定完成
  
### Notice 小注意
> listen-address 改為 10.0.75.1:8118
10.0.75.1 是為 docker for windows 使用 HYPER-V 虛擬機器的占用IP, 所以如果docker更新後可能會調整IP

此IP查詢的方式為 先查詢所有網路使用狀況
> ipconfig /all
找到下述的 yper-V Virtual Ethernet Adapter #3

```Ethernet adapter vEthernet (DockerNAT):

   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Hyper-V Virtual Ethernet Adapter #3
   Physical Address. . . . . . . . . : 00-15-5D-12-B3-04
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   IPv4 Address. . . . . . . . . . . : 10.0.75.1(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . :
   DNS Servers . . . . . . . . . . . : fec0:0:0:ffff::1%1
                                       fec0:0:0:ffff::2%1
                                       fec0:0:0:ffff::3%1
```

此處的10.0.75.1(Preferred), 即是docker 對外ip

備註 更新後(Docker Engine v19.03.5)有發現已經變成 172.27.32.1

```乙太網路卡 vEthernet (nat):

   連線特定 DNS 尾碼 . . . . . . . . :
   描述 . . . . . . . . . . . . . . .: Hyper-V Virtual Ethernet Adapter #3
   實體位址 . . . . . . . . . . . . .: 00-15-5D-A8-F4-DC
   DHCP 已啟用 . . . . . . . . . . . : 是
   自動設定啟用 . . . . . . . . . . .: 是
   IPv4 位址 . . . . . . . . . . . . : 172.27.32.1(偏好選項)
   子網路遮罩 . . . . . . . . . . . .: 255.255.240.0
   預設閘道 . . . . . . . . . . . . .:
   NetBIOS over Tcpip . . . . . . . .: 啟用
```

---

## Step 7 > privoxy > options > Edit User Actions

### Step 7.1 > 文末加入以下文字

```
#Http proxy auth (user.action)
{ \
#+hide-forwarded-for-headers \
+hide-from-header{block} \
+set-image-blocker{pattern} \
+add-header{Proxy-Connection: Keep-Alive}\
+add-header{Proxy-Authorization: Basic [這是你的base64加密文字格式為AD帳號:AD密碼]} \
}
/
```
ps: 帳號帶入員編就好, 不需要evertrsut\


[Base64加密連結](https://www.base64decode.org/) <br>
或是[懶人用法](https://siderite.blogspot.com/2014/02/using-prixovy-to-forward-to.html)
在下方有個輸入框，分別輸入帳密按下“Help me configure privoxy”, 在複製第一段 user.action 
* ※請注意　斜線都是有用的～～不要自行略過不貼 *

### Step 7.2 > User Actions設定完成
  
  
---
  

## Step 8 > 重啟 Privoxy
  
  
---
  
## Step 9 > docker > Settings > Proxies
### Step 9.1 > 選擇 Manual proxy configuration
### Step 9.2 > Web Server (Http) 輸入 10.0.75.1:8118
### Step 9.2 > 勾選 Use same for both
### Step 9.3 > Bypass for these hosts and domains
~~聽說該設定無效因為Linux不是這種格式~~
```
localhost,10.*;172.16.*;172.17.*;172.30.*;172.18.*;172.19.*;172.30.*;172.31.*;192.168.*.*;127.0.0.1;*.evertrust.tw;*.yongqing.com.cn;*.evertrust.com.cn;*.yongqing.local;*.housefun.com.tw;*.yungching.com;*.ycut.com.tw;*.evertrust.com.tw;*.yungching.org.tw;*.u-trust.com.tw;*.housefun.com.tw;*.yungching.tw;*.yungching.com.tw;*.taiching.com.tw;pic.hfcdn.com;static.hfcdn.com;w3cache.hfcdn.com;cdn.hfcdn.com;cdnyc.hfcdn.com
```
### Step 9.4 > 按下Apply按鈕存檔本次修改
### Step 9.5 > 在command line 輸入指令確認
```
docker info
```
出現 一堆文字說明，找到
> HTTP Proxy: 10.0.75.1:8118

> HTTPS Proxy: 10.0.75.1:8118
只要是 10.0.75.1:8118 就設定成功囉
```
 Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 32
Server Version: 18.06.1-ce
Storage Driver: overlay2
 Backing Filesystem: extfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: 468a545b9edcd5932818eb9de8e72413e616e86e
runc version: 69663f0bd4b60df09991c08812a60108003fa340
init version: fec3683
Security Options:
 seccomp
  Profile: default
Kernel Version: 4.9.93-linuxkit-aufs
Operating System: Docker for Windows
OSType: linux
Architecture: x86_64
CPUs: 2
Total Memory: 3.837GiB
Name: linuxkit-00155d6e1d27
ID: RXRC:VK23:T47S:3TSB:SXFH:DZ2G:IO4K:FUVE:T35D:KFEA:PWE6:2BYA
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): true
 File Descriptors: 22
 Goroutines: 47
 System Time: 2018-11-09T02:45:20.3574548Z
 EventsListeners: 1
HTTP Proxy: 10.0.75.1:8118
HTTPS Proxy: 10.0.75.1:8118
No Proxy: localhost,... 
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false
``` 

---  
  

## Step 10 > 設定完成，可以下 docker pull 不會失敗囉~~~


> 參考來源
> 
> 安裝說明
>> https://siderite.blogspot.com/2014/02/using-prixovy-to-forward-to.html

>  Base64 encode online
>>　https://www.base64decode.org/

>  官網
>> https://chocolatey.org/

>> http://www.privoxy.org/