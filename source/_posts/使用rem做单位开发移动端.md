---
title: 简单粗暴的使用rem做单位开发移动端
date: 2018-01-23 20:12:23
updated: 2018-01-23 20:12:23
tags: [HTML5,JavaScript]
categories: [前端]
---
rem开发移动端再适合不过，省时省心省力

<!--more-->

```javascript
(function(doc,win){ 
    var docEl=doc.documentElement,//html
        resizeEvt='orientationchange' in window?'orientationchange':'resize',
        recalc=function(){
            var clientWidth=docEl.clientWidth;
            if(!clientWidth)return;
            docEl.style.fontSize=20/375*clientWidth+"px";
        };
        if(!doc.addEventListener)return;
        win.addEventListener(resizeEvt,recalc,false);
        doc.addEventListener('DOMContentLoaded',recalc,false); 
})(document,window);
```



```scss
@charset "UTF-8";
$rem:20;
$color:#fa0;
@function r($px){
    @return $px/$rem+rem;
}
```

