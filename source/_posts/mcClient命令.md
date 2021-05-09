---
title: mcClient命令
date: 2020-12-08 20:58:53
tags: minio
---
minio支持客户端使用命令来操作服务器的数据

# 1 安装mc客户端

去minio中国镜像下载linux mc客户端
`http://dl.minio.org.cn/client/mc/release/`

下载完成后,给予权限
`chmod +x /home/minio/mc`

添加到环境变量
`vi /etc/profile`

在文件末尾添加
`export PATH="/home/minio:$PATH"`

<!--more-->

![Alt text](/images/minio_env.png) 

然后保存,退出.

验证是否配置成功

在任意目录下执行
`mc --help`

有如下显示则代表配置成功

![Alt text](/images/minio_env_help.png) 

# 2 常用命令

命令 | 意义 
:-: | :-: 
config host list | 列出当前配置的服务器 
config host add | 添加服务器 
mb | 创建桶 
cp | 复制(上传)文件到服务器 
ls | 列出存储桶和对象 
cat | 查看文件 
rm | 删除文件或存储桶 
share | 共享 
mirror | 存储桶镜像 
policy | 设置文件可直接下载 

## 2.1列出当前配置的服务器
`mc config host list`

![Alt text](/images/minio_config_list.png) 


## 2.2添加一个别名为minio131的服务器
`mc config host add minio131 http://www.minio.muklee.com admin AAbb1234`


再次查看
![Alt text](/images/mc_config_list.png)

## 2.3 mb 创建存储桶

![Alt text](/images/minio_create_bucket.png)

## 2.4 cp 复制(上传)文件
创建1.txt文件,输入”hello world”
`echo "hello world ! " > 1.txt`

上传1.txt
`mc cp 1.txt minio131/mybucket`
![Alt text](/images/minio_upload_1txt.png)

在minio服务器界面可看到上传的文件

## 2.5 ls 列出存储的对象
`mc ls minio131/mybucket`

![Alt text](/images/minio_mc_ls.png)

## 2.6 cat 查看文件
`mc cat minio131/mybucket`
![Alt text](/images/minio_mc_cat.png)

## 2.7 share共享上传或下载文件
Share 授予的上传或下载权限是临时的,最长有效期为7天.如果想生成永久的权限,可以用mc policy设置.

### 2.7.1 共享下载
`mc share download --expire 4h minio131/mybucket/1.txt`

### 2.7.2 共享上传
`mc share upload --expire 4h minio131/mybucket/1.txt`

`
curl http://www.minio.muklee.com/mybucket/ -F x-amz-date=20201206T094754Z -F x-amz-signature=f1ec7d8f4f41408887859fb04001a2d20d653be9679b79790f89c6bdc41a1053 -F bucket=mybucket -F policy=eyJleHBpcmF0aW9uIjoiMjAyMC0xMi0wNlQxMzo0Nzo1NFoiLCJjb25kaXRpb25zIjpbWyJlcSIsIiRidWNrZXQiLCJteWJ1Y2tldCJdLFsiZXEiLCIka2V5IiwiMS50eHQiXSxbImVxIiwiJHgtYW16LWRhdGUiLCIyMDIwMTIwNlQwOTQ3NTRaIl0sWyJlcSIsIiR4LWFtei1hbGdvcml0aG0iLCJBV1M0LUhNQUMtU0hBMjU2Il0sWyJlcSIsIiR4LWFtei1jcmVkZW50aWFsIiwiYWRtaW4vMjAyMDEyMDYvdXMtZWFzdC0xL3MzL2F3czRfcmVxdWVzdCJdXX0= -F x-amz-algorithm=AWS4-HMAC-SHA256 -F x-amz-credential=admin/20201206/us-east-1/s3/aws4_request -F key=1.txt -F file=@<FILE>
`


这个url能更新 mybucket下1.txt文件的内容.

假如1.txt 的内容为 hello world !

我要用本地 /home/minio/nohup.out 的内容替换1.txt原来的内容


`curl http://www.minio.muklee.com/mybucket/ -F x-amz-date=20201206T094754Z -F x-amz-signature=f1ec7d8f4f41408887859fb04001a2d20d653be9679b79790f89c6bdc41a1053 -F bucket=mybucket -F policy=eyJleHBpcmF0aW9uIjoiMjAyMC0xMi0wNlQxMzo0Nzo1NFoiLCJjb25kaXRpb25zIjpbWyJlcSIsIiRidWNrZXQiLCJteWJ1Y2tldCJdLFsiZXEiLCIka2V5IiwiMS50eHQiXSxbImVxIiwiJHgtYW16LWRhdGUiLCIyMDIwMTIwNlQwOTQ3NTRaIl0sWyJlcSIsIiR4LWFtei1hbGdvcml0aG0iLCJBV1M0LUhNQUMtU0hBMjU2Il0sWyJlcSIsIiR4LWFtei1jcmVkZW50aWFsIiwiYWRtaW4vMjAyMDEyMDYvdXMtZWFzdC0xL3MzL2F3czRfcmVxdWVzdCJdXX0= -F x-amz-algorithm=AWS4-HMAC-SHA256 -F x-amz-credential=admin/20201206/us-east-1/s3/aws4_request -F key=1.txt -F file=`<font color='red'> @/home/minio/nohup.out</font>


### 2.7.3 查看共享的url
查看上传
`mc share list upload`

查看下载
`mc share list download`

## 2.8 mirror存储桶镜像
将本地的/home/minio/localdir 文件夹的内容镜像到mybucket中,加上 -w 参数后会持续监听本地文件夹的变化. 但如果退出终端,则不再监听
`mc mirror -w /home/minio/localdir minio131/mybucket`

![Alt text](/images/minio_mc_watch.png)

## 2.9 policy 设置bucket公开(静态资源服务器)
`mc policy set public minio131/mybucket`

设置mybucket的权限为公开后,浏览器可直接访问url下载

`wget http://www.minio.muklee.com/mybucket/1.txt`