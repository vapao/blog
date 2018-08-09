---
title: Nextcloud docker nginx
tags:
- nextcloud
- nginx
---

[Nextcloud](https://nextcloud.com/)是一个开源的私有云盘项目。由于之前老婆的手机进水差点丢掉了女儿的好多照片视频之类的资料，不过好在最后手机修好了这些数据完好无损，由此萌生了要把这些数据备份到云盘的需要，现在可选的云盘也就百度网盘了，但必须会员才能上传视频。我在阿里云上开的有ECS之前也了解过NextCloud就自告奋勇自己为老婆搭建个云盘。

好了，下边就介绍安装过程，首先说明下我这边的环境，操作系统`ubuntu 16.04` ，当前稳定版的`nextcloud 13.0.5`，存储用到了阿里云的`OSS`，毕竟`OSS`便宜又可靠。

## 安装docker

`docker`在不同的linux发行版下名字可能不一样，ubuntu叫`docker.io`具体请参考docker的官方安装文档。

```bash
apt update
apt install docker.io
```

## 获取nextcloud官方镜像

[官方镜像](https://hub.docker.com/_/nextcloud/) 上有多个版本可选，这里选择13版本带apache的。

```bash
docker pull nextcloud:13
```

## 运行镜像

官方推荐的数据库使用`mysql`或`postgresql`，因为我的ECS只有1G内存，且目前对性能没有要求，为了方便就使用默认的`sqlite`数据库了。我这里准备使用nginx代理通过域名访问，所以映射的ip是`127.0.0.1`。镜像内的`/var/www/html`就是数据的根目录了，包括nextcloud的项目代码和云盘的存储数据`/var/www/html/data`。

```bash
docker run -d \
--name nextcloud \
-v /data/nextcloud:/var/www/html \
-p 127.0.0.1:9000:80 \
nextcloud:13
```

## 配置nginx

如果使用的域名访问，需要在nextcloud的配置文件(`/var/www/html/config/config.php`)的`trusted_domains`项中添加域名，否则在登录时可能会报400的错误。

```bash
server {
        listen 80;
        server_name example.domain.com;
        client_max_body_size 100m;		# 最大上传文件大小限制

        location / {
                proxy_pass http://127.0.0.1:9000;
                proxy_set_header Host $http_host;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header X-Real-IP $remote_addr;
        }
}
```

##创建管理员

如果一切正常的话，现在就可以通过访问域名看到创建管理员账户的页面了，创建后进入了网盘的首页，默认会从模板中复制一些样例文件。至此已经算是搭建完成了，如果你想把网盘数据存放至阿里云的OSS中，可参考以下步骤。

## 数据存至OSS

阿里的OSS是支持挂在到系统中的，可参考阿里云的[官方文档](https://help.aliyun.com/document_detail/32196.html?spm=a2c4g.11186623.6.1107.51hbCi) 安装`ossfs`工具进行挂载操作，这里不再给出安装`ossfs`的步骤。我这边把容器内的`/var/www/html`映射到了`/data/nextcloud`目录，所以`/data/nextcloud/data/home`即代表用户`home`的网盘数据目录。以下是挂载命令

```bash
ossfs bucket_name /data/nextcloud/data/home -ourl=oss-cn-hangzhou-internal.aliyuncs.com -ouid=33 -ogid=33 -oumask=002 -o allow_other
```

强烈推荐OSS云ECS处于同一地域使用内网地址进行挂载，`uid`和`gid`请参照原有的目录属主和属组设置，避免出现权限问题。

## 客户端下载

官方提供了各平台的客户端，移步[官方地址](https://nextcloud.com/install/#install-clients)进行选择下载。



> 好啦，终于为老婆干点程序员该干的事儿😄