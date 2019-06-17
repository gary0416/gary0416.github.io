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
    window.onload=function(){
        document.getElementById("btn-readmore").click();
    }
})();
```

## 推荐直接下载
- AC-baidu:重定向优化百度搜狗谷歌搜索_去广告_favicon_双列
- 百度网盘直链下载助手