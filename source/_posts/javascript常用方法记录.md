---
title: javascript常用方法记录
date: 2018-04-10 13:34:12
updated: 2018-04-10 13:34:23
tags: [JavaScript]
categories: ['前端']
---
记录一些javascript常用方法

<!--more-->

1. 截取一定长度的字符串（前端展示为了美观，通常需要截取相同长度的字符串作为填充，要保证空间上长度基本一致，这段js用派上用场）

```javascript
/*
 * js截取字符串，中英文都能用
 * @param len: 需要截取的长度
 */
String.prototype.gbslice = function(len) {
    var str_length = 0;
    var str_len = 0;
    str_cut = new String();
    str_len = this.length;
    for (var i = 0; i < str_len; i++) {
        a = this.charAt(i);
        str_length++;
        if (escape(a).length > 4) {
            //中文字符的长度经编码之后大于4
            str_length++;
        }
        str_cut = str_cut.concat(a);
        if (str_length >= len) {
            str_cut = str_cut.concat("..");
            return str_cut;
        }
    }
    //如果给定字符串小于指定长度，则返回源字符串；
    if (str_length < len) {
        return this;
    }
}
```



2. echars图片下载（echars生成的图片上有自己带的下载按钮，如果需要在图片外做一个按钮下载相应的echars，这个方法就是你需要的）

```javascript
// echars图片下载
function myBrowser() {
    var userAgent = navigator.userAgent; //取得浏览器的userAgent字符串
    var isOpera = userAgent.indexOf("OPR") > -1; if (isOpera) { return "Opera" }; //判断是否Opera浏览器 OPR/43.0.2442.991

    if (userAgent.indexOf("Firefox") > -1) { return "FF"; } //判断是否Firefox浏览器  Firefox/51.0
    if (userAgent.indexOf("Trident") > -1) { return "IE"; } //判断是否IE浏览器  Trident/7.0; rv:11.0
    if (userAgent.indexOf("Edge") > -1) { return "Edge"; } //判断是否Edge浏览器  Edge/14.14393
    if (userAgent.indexOf("Chrome") > -1) { return "Chrome"; }// Chrome/56.0.2924.87
    if (userAgent.indexOf("Safari") > -1) { return "Safari"; } //判断是否Safari浏览
}
function base64Img2Blob(code){
    var parts = code.split(';base64,');
    var contentType = parts[0].split(':')[1];
    var raw;
    if(window.atob){
        raw = window.atob(parts[1]);
        var rawLength = raw.length;
        var uInt8Array = new Uint8Array(rawLength);
        for (var i = 0; i < rawLength; ++i) {
            uInt8Array[i] = raw.charCodeAt(i);
        }
        return new Blob([uInt8Array], {type: contentType});
    } else {
        raw = BaseCode(parts[1]);
    }
}
function downloadFile(fileName, content) {
    var blob = base64Img2Blob(content);
    // 支持IE11  base64Img2Blob
    window.navigator.msSaveBlob(blob, fileName);
}
var Global_interDown = true;
/*
 * echars保存为图片
 * @param tag: 按钮id，例："#btn_click"
 * @param mychart: echars对象
 * @param image_name: 保存的图片名字
 */
function initSaveImg(tag, mychart, image_name) {
    $(tag).unbind();
    $(tag).click(function() {
        var hasData = $(this).parents('.wrapper-echars').find('canvas').length == 0 ? false : true;
        if (!hasData) {
            return false;
        }

        setTimeout(function() {
            Global_interDown = true;
        }, 1000);

        if (!Global_interDown) {
            return false;
        }
        Global_interDown = false;

        var aTag = document.createElement("a");
        var dataurl = mychart.getDataURL({
            type: 'png'
        });

        if (myBrowser() == "IE") {
            aTag.href = "#";
            downloadFile(image_name + '.png', dataurl);
        } else {
            var MIME_TYPE = 'image/png';
            aTag.href = dataurl;
            aTag.target = "_self";
            aTag.download = image_name + '.png';
        }

        document.body.appendChild(aTag);
        aTag.click();
        document.body.removeChild(aTag);
    });
}

```

3. 日期格式化

```javascript
// 日期格式化 var yesterDay = new Date(yesterdayTime).Format("yyyy-MM-dd");
Date.prototype.Format = function (fmt) { //author: meizz
    var o = {
        "M+": this.getMonth() + 1, //月份
        "d+": this.getDate(), //日
        "h+": this.getHours(), //小时
        "m+": this.getMinutes(), //分
        "s+": this.getSeconds(), //秒
        "q+": Math.floor((this.getMonth() + 3) / 3), //季度
        "S": this.getMilliseconds() //毫秒
    };
    if (/(y+)/.test(fmt)) fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length));
    for (var k in o)
        if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
    return fmt;
};
```

4. 日期排序
```javascript
//日期排序
function sortDownDate(a, b) {
    return Date.parse(a) - Date.parse(b);
}
function sortUpDate(a, b) {
    return Date.parse(b) - Date.parse(a);
}

var arr = ['2018-02-02','2018-03-03','2017-03-03','2019-01-01'];
arr.sort(sortDownDate)
arr.sort(sortUpDate)

```


