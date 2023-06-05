# 强制进入课程

课程中心强制进入课程的脚本，如遇到课程关闭可以通过改变url的方式进入课程(课程被删除的不行)。

```javascript
// ==UserScript==
// @name         进入课程脚本
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  可以进入已经关闭的课程，其实就是调了一下参数罢了。
// @author       You
// @match        https://kczx.cuit.edu.cn/learn/course/preview/spoc/*
// @icon         https://www.google.com/s2/favicons?sz=64&domain=cuit.edu.cn
// @grant        none
// ==/UserScript==

(function() {
    'use strict';
    // 获取第一个class为"content_bottom"的div
    var contentBottomDiv = document.querySelector('.content_bottom');

    // 创建一个按钮元素
    var button = document.createElement('button');
    button.style.width = '100px';
    button.style.height = '50px';
    button.textContent = '强制进入课程';

    // 将按钮添加到div
    contentBottomDiv.appendChild(button);

    // 添加点击事件监听器
    button.addEventListener('click', function() {
        // 打印当前页面的URL
        let url = window.location.href;
        console.log(url);
        let param = url.split('/')[7];
        // 在新标签页中跳转到一个指定的URL
        window.open('https://kczx.cuit.edu.cn/learn/course/detail/spoc/courseWare/' + param + '?type=', '_blank');
    });
})();