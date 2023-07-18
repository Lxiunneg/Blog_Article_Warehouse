---
title: 关于如何替换Hexo背景的操作指南
top: false

date: 2023-07-11 22:30:06
tags:
- 站点建设
- 站点美化
categories:
- 站点信息
---

根据设备类型显示不同的背景图片。  

<!--more-->

打开主题配置文件即themes/next下的_config.yml，将 style: source/_data/styles.styl 取消注释。

打开根目录Blog/source创建文件_data/styles.styl，添加以下内容：

```css
// 添加背景图片
body {
      background: url(images/background.png);//自己喜欢的图片地址
      background-size: cover;
      background-repeat: no-repeat;
      background-attachment: fixed;
      background-position: 50% 50%;
}
```

但是由于桌面端和移动端的分辨率不同，可以设置桌面端和移动端设置不同的背景。  

在Blog/source/_data/styles.styl中添加：  
```css
@media only screen and (max-width: 1000px) {
  body{
    background:url(/images/bg.jpg);
    background-size:cover;
    background-repeat:no-repeat;
    background-attachment:fixed;
    background-position:center;  
  } 
}
```

ios端修正：

```css
@media only screen and (max-width: 1000px) {
  body:before{
  content:"";
  display:block;
  position:fixed;
  top:0;
  left:0;
  bottom: 0;
  z-index:-1;
  width:100%;
  height:100vh;
  background:url(/images/bg2.png) center 0 no-repeat;
  background-size:cover;
  }
}
```