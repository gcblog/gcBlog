---
title: github ssh key 问题
date: 2016-02-27 17:02:21
tags: [github, ssh key]
categories: 
- 随笔
---
   
   当我部署hexo在github上的时候去看了一下ssh key，发现github上早就有ssh key了，但是我从来没有手动去生成过，并且这个key的名字与手动生成的不一样，文件名是这样github_rsa，当然还有另外一个pub格式的，而如果我们手动生成的话，必然是id_rsa,除非我们自己改名。这个key应该是我安装github客户端的时候生成的，不过我可以在github提交代码，hexo也部署成功了，这些东西也从来没去了解过，然后这次出问题了，折腾了很久。
   <!-- more -->
   我打算把博客迁移到gitcafe上，然后我申请了一个gitcafe账号，然后也是配置相关ssh key，当我生成了gitcafe 的ssh key 后，我的github 就不能提交代码了。可恶的是我在gitcafe上添加ssh key后，也是提交不了代码，现在我两个地方都不能用了。现在我的ssh是这样的，github是github_rsa，gitcafe是id_rsa，然后我又在网上搜索管理多个ssh key方法，通过config配置，但是还是没有用。最后搞了很久，实在没有办法，我就先把gitcafe 的 key 给删了，看看github能不能用了，结果发现github还是不能用，这回蛋疼了。最后终于决定，我把ssh目录给清空了，然后重新手动生成github ssh key，这回github能用了，有点劫后余生的感觉，毕竟公司的代码就托管在github上，要是不能用了，工作都做不了。然后我又有些不甘心，再次尝试 gitcafe的 ssh key，然后在通过config管理，这回能用了。
   原因就在于第一个github的key不是我自己生成的，如果你也遇到此类问题，可以试试清空哪些不是自己生成的key。

