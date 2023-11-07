---
title: 【教程向】半小时搭建自己的个人博客
date: 2023-01-01 5:03:26
tags: [教程]
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## 配置云服务器

这里以腾讯云为例，系统是Ubuntu 20.04 LTS，首先设置root远程访问（虽然不建议）

``` bash
sudo vi /etc/ssh/sshd_config
```

将`PermitRootLogin`设置为yes

然后重启ssh

``` bash
service sshd restart
```

## 安装Hexo

这里没什么难的，可以看Hexo官方文档，我踩了个坑：node.js要安装它建议的版本！

然后换一个你喜欢的主题，我的是[Fluid](https://github.com/fluid-dev/hexo-theme-fluid).

继续，读[文档](https://hexo.fluid-dev.com/docs/guide/).

## 发表新Post

``` bash
$ hexo new "title"
```

``` bash
$ hexo g -d
```

## 配置Nginx添加SSL证书

``` nginx
server {
     #SSL 默认访问端口号为 443
     listen 443 ssl;
     #请填写绑定证书的域名
     server_name bz2021.cn; 
     #请填写证书文件的相对路径或绝对路径
     ssl_certificate bz2021.cn_nginx/bz2021.cn_bundle.crt; 
     #请填写私钥文件的相对路径或绝对路径
     ssl_certificate_key bz2021.cn_nginx/bz2021.cn.key; 
     ssl_session_timeout 5m;
     #请按照以下协议配置
     ssl_protocols TLSv1.2 TLSv1.3; 
     #请按照以下套件配置，配置加密套件，写法遵循 openssl 标准。
     ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
     ssl_prefer_server_ciphers on;

     location / {
         #网站主页路径。此路径仅供参考，具体请您按照实际目录操作。
         #例如，您的网站主页在 Nginx 服务器的 /etc/www 目录下，则请修改为 /etc/www。
         root /etc/www;
         index  index.html index.htm;
     }
}
```

## 将HTTP重定向到HTTPS

``` nginx
server {
    listen 80;
    server_name bz2021.cn;
    rewrite ^(.*) https://$server_name$1 permanent;
}
```

