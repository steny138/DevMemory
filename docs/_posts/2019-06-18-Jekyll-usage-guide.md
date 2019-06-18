---
layout: post
title:  "Jekyll usage guide!"
date:   2019-06-18 15:36:26 +0800
categories: jekyll guide
author: yuchen
---

# Jekyll usage guide

## Install

### MacOS

#### brew 安裝 ruby

``` 
    brew install ruby
    ruby -v
    // ruby 2.6.3p62 (2019-04-16 revision 67580) [x86_64-darwin18]

    gem -v
    gem install jekyll bundler

```

#### 另外一種安裝方式

```
1. 安裝RVM
\curl -sSL https://get.rvm.io | bash -s stable
2. 安裝ruby
rvm use ruby --install --default

＊ 若是無法安裝成功 macOS可能是xcrun 啟用失敗，試試下列語法安裝
xcode-select --install

ruby -v
須為2.4以上版本

gem -v
gem install bundler jekyll

```

####