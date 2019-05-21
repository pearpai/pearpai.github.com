---
layout:     post
title:      "xcode nw_connection_get_connected_socket_block 不停止"
date:       UTC2019-05-21 13:39:00
author:     "Pearpai"
header-img: "img/head/other_think.png"
catalog: true
tags:
    - react-native
---
1. Xcode menu -> Product -> Edit Scheme...
 ![添加网关架构图](/img/blog/rn/connection_get_connected_socket_block_01.jpg)

2. Environment Variables -> Add -> Name: "OS_ACTIVITY_MODE", Value:"disable"
 ![添加网关架构图](/img/blog/rn/connection_get_connected_socket_block_02.png)
3. close 后 重新run
