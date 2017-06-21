---
layout:     post
title:      "Jekyll MarkDown 文件date 当前时间不展示文档"
date:       UTC 2017-06-21 09:06:00
author:     "Pearpai"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - 开发环境
    - Blog
    - Jekyll
---

在使用Jekyll 制作个人网站的过程中遇到个问题，在书写title信息时候，如果使用的是当前时间，页面上将不会显示，非要调整时间后才能显示文章，经过调试发现与当前北京时间相差8小时，这时就知道了，因为默认使用的是**格林威治时间**所以因为时间差的问题文章将不显示。
### 此时我们将date修改为UTC
```
layout:     post
title:      "Jekyll MarkDown 文件date 当前时间不展示文档"
date:       UTC 2017-06-21 09:06:00
author:     "Pearpai"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - 开发环境
    - Blog
```
