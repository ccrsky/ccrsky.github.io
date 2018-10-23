---
title: macOS Mojave字体变细
date: 2018-10-23 22:51:00
category:
    - 随记

tags:
    - mac
    - os
---

升级新版 macOS Mojave后字体变细发虚的解决办法：
```shell
# 解决字体变虚
defaults write -g CGFontRenderingFontSmoothingDisabled -bool NO

# 默认为0 ,可设置1,2,3
defaults -currentHost write -globalDomain AppleFontSmoothing -int 1

```