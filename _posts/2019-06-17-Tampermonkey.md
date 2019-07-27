---
layout:     post
title:      Tampermonkey
subtitle:   
date:       2019-06-17
author:     gary
header-img: 
catalog: true
tags:
    - Frontend
---

# Tampermonkey

## 自定义脚本

### csdn/iteye/yq等网站自动点击查看更多
```
// ==UserScript==
// @name         readmore
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  csdn/iteye/yq等网站自动点击查看更多
// @author       gary
// @match        *://*.csdn.net/*
// @match        *://*.iteye.com/*
// @match        *://yq.aliyun.com/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';
    function readmore(){
        var btn = document.getElementById("btn-readmore");
        if(btn){
            btn.click();
        }
        // csdn改成class了
        var btns = document.getElementsByClassName("btn-readmore");
        if(btns.length>0){
            btns[0].click();
        }
    }
    // 为了尽快看到内容，未加载完时也尝试执行一次
    readmore();
    window.onload=function(){
        readmore();
    }
})();
```

## 推荐直接下载
- AC-baidu:重定向优化百度搜狗谷歌搜索_去广告_favicon_双列
- 百度网盘直链下载助手