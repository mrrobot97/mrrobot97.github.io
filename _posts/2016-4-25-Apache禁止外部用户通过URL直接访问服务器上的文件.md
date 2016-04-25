---
layout: post
author: mrrobot
title: Apache禁止外部用户通过URL直接访问服务器上的文件
---

编辑Apache的http.conf文件，在其中任意一处加入如下代码

&lt;Directory "绝对路径"> 

   Order allow,deny
   
   Deny from all
   
&lt;/Directory>

然后 service httpd restart 重启Apache服务即可，此后若有外部用户试图 通过URL直接访问文件，就会得到权限不足的提示：

![](http://cl.ly/170Z0Z1o2f0z/forbidden.tiff)