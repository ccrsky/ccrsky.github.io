---
title: Hexo插入媒体资源
date: 2017-08-15 00:59:44
tags:
    - hexo

---

通常我们会在Hexo文章中插入媒体资源如图片，音频，视频，那么让我们来看看如何插入

### 插入图片

#### 本地图片
在/hexo/source目录下新建文件夹，命名为images或其他

```markdown
![图片描述](/images/图片名字.png)  
```

#### 使用远程图片
这种是大家常用的方法
```markdown
![图片描述](http://ouonaooa5.bkt.clouddn.com/demo.jpg)  
```

>可使用七牛存储自己的图片资源

### 插入音频

#### 插入网易云音乐

```html
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86   
    src="http://music.163.com/outchain/player?type=2&id=25706282&auto=0&height=66">  
</iframe> 
```
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86   
    src="http://music.163.com/outchain/player?type=2&id=25706282&auto=0&height=66">  
</iframe> 


#### 插入喜马拉雅

```html
<iframe height="36" width="260" src="http://www.ximalaya.com/swf/sound/red.swf?id=36371956" frameborder=0 allowfullscreen></iframe>
```


<object type="application/x-shockwave-flash" id="ximalaya_player" data="http://www.ximalaya.com/swf/sound/red.swf?id=36371956" width="260" height="36"></object>


### 插入视频

```html
<iframe class="ql-video ql-align-center" frameborder="0" allowfullscreen="true" src="https://www.youtube.com/embed/QHH3iSeDBLo?showinfo=0" height="238" width="560"></iframe>
```

<iframe class="ql-video ql-align-center" frameborder="0" allowfullscreen="true" src="https://www.youtube.com/embed/QHH3iSeDBLo?showinfo=0" height="238" width="560"></iframe>





