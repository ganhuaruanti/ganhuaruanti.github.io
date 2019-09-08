---
layout: post
title: Jekyll 使用心得
author: miles
tags: [blog]
---

目前使用 [Jekyll](https://jekyllrb.com/) + [Hacker theme](https://pages-themes.github.io/hacker/)，覺得意外的好用。

之前雖然有自己玩過一陣子，但一直搞不懂它背後運作的原理，所以最後還是做罷。

現在總算了解裡面設定的原理了，其實它只是一個很自由的設定檔 `_config.yml`，再加上每個 markdown 檔都能有自己的 metadata，而這些 metadata 又能在 layout 的樣版引擎做自定義變數操作，所以才能有無窮的變化。

> 會用到的參數都在[說明文件](https://jekyllrb.com/docs/variables/)裡。

若有興趣，可以參考[原始碼](https://github.com/ganhuaruanti/ganhuaruanti.github.io)裡面寫的設定和樣版。
