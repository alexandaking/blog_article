---
title: Hexo通过Github的Webhooks实现本地发布服务器自动部署
date: 2020-02-17 20:51:03
tags: hexo
categories: 杂技
---

# 1. 需求

本地写完markdown文件，自动部署到服务器，不需要在手动上传到服务器，然后执行：

```
hexo clean
hexo g
hexo d
```



# 2. 思路

这时候我们需要借助到github的**Webhooks**，当监听到有上传的`git push`的动作时触发webhook请求预设的地址，我们在服务器里写好了脚本，当收到github的webhooks发来的请求时，触发上述hexo的命令。

![](http://cdn.wdaking.com/2020-02-18-blog-1-1.jpg)

# 3. 开始

### Prepare

* 在github上建立一个空仓库，名字随意，这里叫blog.source, 点击Create repository

<img src="http://cdn.wdaking.com/2020-02-18-blog-1-2.jpg" style="zoom:50%;" />

* 一台已经安装好hexo的服务器，网上有大把的教程，跟着安装，这里不做讲解，准备就绪后开始下面的步骤

1. 登录服务器，找个位置新建文件夹blog.source，这里我在/home路径下建立:

   ```
   cd /home
   mkdir -p /hexo/blog.source
   ```

2. 将该文件夹作为github上blog.source的仓库：

   ```
   echo "# blog.post" >> README.md
   git init
   git add README.md
   git commit -m "first commit"
   git remote add origin https://github.com/yourname/blog.post.git
   git push -u origin master
   ```

   这样我们就将 **README.md**  push到github仓库了，如果你的服务器上没有装git环境， 或者ssh，自行搜索。

3. 下面我们回到github找到仓库blog.source -> Settings ->Webhooks，然后我们创建一个webhook如下图：

   

   Payload URL：为你需要请求的地址，稍后会在nginx里配置

   Content type：选择json

   然后点击创建，这样我们就创建好了一个webhook，github会监听该仓库的push事件，当这个仓库收到push请求时，都会向你预设的url发送一条post请求，请求体为json格式，如下图：

   ![](http://cdn.wdaking.com/2020-02-18-blog-1-4.jpg)

   Payload为请求体包含本次提交的所有内容

4. 接着我们回到服务器，在/home/hexo下写关于hexo部署的脚本如下：

   ```
   cd /home/hexo
   vim deploy.sh
   ```

   ```shell
   #!/bin/bash
   cd /home/hexo/blog.source
   git pull
   rm -rf /home/your_hexo_init_path/source/_posts/*
   cp -r /home/hexo/blog.source/* /home/your_hexo_init_path/source/_posts/
   cd /home/your_hexo_init_path
   hexo clean
   hexo generate
   echo "部署成功 !"
   ```

   执行改脚本的时候，会进入到/blog.source目录下，执行`git pull`

   并删除hexo目录_post下生成的虽有Markdown文件

   将我们新拉取的/blog.source下所有文件拷贝到hexo的/_post文件夹下

   然后在执行我们熟悉的```hexo clean```, ``` hexo g``` , ```hexo d```

5. 接着我们需要接收webhooks发送的请求，这里我使用Python，其他语言皆可，代码如下：

   ```python
   from flask import Flask, request, jsonify
   import json
   import jwt
   import os
   
   app = Flask(__name__)
   
   @app.route('/', methods = ["POST"])
   def get_post_data():
   
           recv = request.get_json()
           if recv["ref"] == "refs/heads/master":
                   os.system('sh deploy.sh')
                   os.system('deploy success')
   
           resp = jsonify(recv)
           resp.headers['Access-Control-Allow-Origin'] = '*'
           return resp
   
   if __name__ == "__main__":
           app.run(host = "0.0.0.0",port = 5500, debug = True)
   ```

   代码大概意思为该服务运行在5500端口，收到post请求时，如果请求体中包含了`"ref"`字段，并且该字段的值为`"refs/heads/master"`，那么执行deploy.sh脚本

   接着我们启动该服务：

   ```shell
   nohup python -u /home/hexo/deploy.py > /home/hexo/webhooks.log 2>&1 &
   ```

   可以在/hexo/wenhooks.log查看日志

6. 注意需要将上述url在你的域名中添加A记录值

7. 我们利用nginx将请求转发到`http://localhost:5500`上, 配置如下：

   ```
   server {
           server_name hooks.wdaking.com;
   
           location /{
                   proxy_redirect off;
                   proxy_set_header Host $host;
                   proxy_set_header X:-Real-IP $remote_addr;
                   proxy_set_header REMOTE-HOST $remote_addr;
                   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                   proxy_pass http://127.0.0.1:5500;
       }
   }
   ```

   

OK ! 接下来我们就可以在本地写完Markdown直接推送到github仓库blog.source下，不出意外服务器也会同步成功。

ENJOY IT！