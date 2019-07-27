---
title: css实现环形进度条
date: 2019-07-27 09:40:07
tags:
  - css
  - 前端

category:
  - css
---

#### 原理

需要一个圆，和两个半圆(左右，挡住底色)，通过旋转左右半圆漏出底色,再在顶层加一个原型覆盖层就可以实现环形，具体细节可参考[CSS3+JS 实现静态圆形进度条【清晰、易懂】](https://segmentfault.com/a/1190000012655551)

#### 实现

思路:两个元素实现环形进度条，第一个通过背景色加背景图片渐变实现遮罩，第二个元素实现顶层遮罩形成环形；第一个背景图通过两次线性渐变配合渐变角度，实现两个半圆及旋转,当进度<50%,固定左侧半圆，旋转右侧半圆，当进度>50%这相反并改变渐变色实现遮罩效果，代码如下。

```js
class Demo extends React.Component {
  generateBgImgCss = percent =>
    percent < 0.5
      ? `linear-gradient(90deg, #fff 50%, transparent 50%, transparent), linear-gradient(${percent *
          360 +
          90}deg,  transparent 50%, #fff 50%, #fff)`
      : `linear-gradient(${percent * 360 +
          90}deg, #0084FF 50%, transparent 50%, transparent),linear-gradient(90deg, #fff 50%, transparent 50%, transparent)`;

  render() {
    return (
      <div
        className="circle"
        style={{
          backgroundImage: this.generateBgImgCss(this.props.curProgress / 100)
        }}
      >
        <div className="top-cover" />
      </div>
    );
  }
}
```
