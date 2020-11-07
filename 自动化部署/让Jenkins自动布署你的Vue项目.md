## 让Jenkins自动布署你的Vue项目

zhou_web 前端瓶子君 *1周前*

关注 前端瓶子君，回复“交流”

加入我们一起学习，天天进步

![img](https://mmbiz.qpic.cn/mmbiz_jpg/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1epgK2q5UMVo5YwwHYcH0poDZc4uTE8MMibZW6v3P9fyA2qViaia6GM8KeQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

作者: zhou_web

https://segmentfault.com/a/1190000019212628 

### 实现目标 

本地push代码到GitHub，Webhook自动触发jenkins上的构建动作,完成安装node插件并且打包，然后通过Publish Over SSH插件，将打包出来的文件，部署到目标服务器上。

### 前期准备

1. github 账号和项目
2. centos 服务器;
3. 服务器安装 Java SDK;
4. 服务器安装 nginx + 启动；
5. 服务器安装jenkins + 启动；

### jenkins介绍

Jenkins是开源的,使用Java编写的持续集成的工具，在Centos上可以通过yum命令行直接安装。Jenkins只是一个平台，真正运作的都是插件。这就是jenkins流行的原因，因为jenkins什么插件都有。

#### 首先登录服务器更新系统软件

```
$ yum update
```

#### 安装Java和git

```
$ yum install java
$ yum install git
```

#### 安装nginx

```
$ yum install nginx //安装
$ service nginx start //启动
```

出现Redirecting to /bin/systemctl start nginx.service

说明nginx已经启动成功了，访问http://你的ip/，如果成功安装会出来nginx默认的欢迎界面

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1er4kPMWhbGibor8mFnKVbqrJkc6rJ5CAdYY5L87JiaRRvUdbLxzla9y8Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

#### 安装Jenkins

```
$ wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key 

$ yum install jenkins //完成之后直接使用 yum 命令安装 Jenkins

$ service jenkins restart  //启动 jenkins
```

jenkins启动成功后默认的是8080端口，浏览器输入你的服务器 ip 地址加8080 端口就可以访问了。

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1er4kPMWhbGibor8mFnKVbqrJkc6rJ5CAdYY5L87JiaRRvUdbLxzla9y8Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

输入 cat /var/lib/jenkins/secrets/initialAdminPassword 查看初始密码

这里我们选择推荐通用插件安装即可，选择后等待完成插件安装以及初始化账户

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1eOvUH1QgLtALDuqk9hER7YQD7OdVzoUtTl0A4qBq1yib0sQG5SiavuyWw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1eI5WVKADrNUmXTcuF3k0PZpBdVpKDEgozAjRKl7Sot2xSXsVWGYOzKA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1e0jDcdnYRxA7QhlibBz3R6WfNaFTRLe36fnpeM2fwqMVgc1z0gdFj1PA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

然后安装两个推荐的插件 Rebuilder
SafeRestart

### 在jenkins中安装nodeJs插件

因为我们的项目是要用到node打包的，所以先在jenkins中安装nodeJs插件，安装后进入全局工具配置，配置一个我们要用到的node版本。

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1eapkpA1DAPupE1DMGtnhkictiaEViaujiaiaP3RqovTSMkPJjEibK87ZaS8JA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1e0mA8PF1HbNP1ykfRiaViaxsqk9I2NjFOHro3mDdiaXMtxqw8K3rR75uFw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

### 创建任务

1. 点击创建一个新任务

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1eWialic3WXEHto7icjibrJibnPYlNkDIWmicRmlP8W2yFema94yRWZXJSpOlg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1eqAWVr2A0IaRWPZRt3ExwicO3CpeXAJX205te1VJFXr43uMsqzxHblYw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

1. jenkins关联 GitHub项目地址

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1e07HdkSCqyyWhDDWuIiaddicxvgqyxIRRQHzgInD6m36Uiaicr6DiaVEE6Yg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

1. 选择构建环境并编写shell 命令

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1eZUx3SasBdITpQkbENAq5icq830TXiccfIZQZ0p75fUib42n6W36r2O9uw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

配置完成后点击立即构建，等待构建完，点击工作空间，可以发现已经多出一个打包后的dist目录。点击控制台输出可以查看详细构建log

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1eVNe8Yx8eeCRNDopU8BYzo6icXOWOzkmkRMW6G75NfSyNQ30lPqviaEuQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1eDCvPQEQQKhI2FkNOSyK3pDII3XAOiaiciaia8y6eKr4jibKq13pBibQI0Wibw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1e9nkv0HAkzgj2Mr7wLeUNAicCiceRlvhgvsrDJqrKLh3fhpnEicv9pJ7xA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

到这里已经实现了本地代码提交到github，然后在jenkins上点击构建，可以拉取代码并且打包，下一步实现打包后的dist目录放到目标服务器上。

### 安装Publish Over SSH 插件，我们将通过这个工具实现服务器部署功能。

安装完成后在系统管理-> 系统设置->Publish over SSH
里设置服务器信息

```
Passphrase：密码（key的密码，没设置就是空）
Path to key：key文件（私钥）的路径
Key：将私钥复制到这个框中(path to key和key写一个即可)

SSH Servers的配置：
SSH Server Name：标识的名字（随便你取什么）
Hostname：需要连接ssh的主机名或ip地址（建议ip）
Username：用户名
Remote Directory：远程目录（上面第二步建的testjenkins文件夹的路径）

高级配置：
Use password authentication, or use a different key：勾选这个可以使用密码登录，不想配ssh的可以用这个先试试
Passphrase / Password：密码登录模式的密码
Port：端口（默认22）
Timeout (ms)：超时时间（毫秒）默认300000
```

这里配置的是账号密码登录，填写完后点击test，出现Success说明配置成功

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1ejficXADwRDyic0LP20kTs2qGdqJpCvSqIXWZHl1LxomyEh2zib3r7qAxQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

在刚才的testJenkins工程中配置**构建后操作**，选择send build artificial over SSH， 参数说明：

```
Name:选择一个你配好的ssh服务器
Source files ：写你要传输的文件路径
Remove prefix ：要去掉的前缀，不写远程服务器的目录结构将和Source files写的一致
Remote directory ：写你要部署在远程服务器的那个目录地址下，不写就是SSH Servers配置里默认远程目录
Exec command ：传输完了要执行的命令，我这里执行了进入test目录,解压缩,解压缩完成后删除压缩包三个命令
```

注意在构建中添加压缩dist目录命令

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1eCharJSiamWibibBgPUPPHPmeWvAOoFf2k2Zx51tccoJDMibtRzznYib6SdA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

填完后执行构建。成功后登录我们目标服务器发现test目录下有了要运行的文件

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1ewzjty88rJbn7r4v8cqarlAsZibZW4fjzDRRrYEI5hre4zpZvHsgMMbA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

访问域名发现项目可以访问了

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1eiccjjzNseTY6JqbD4IUu88dxdXYKTMK6V7feO1meSvtnhFeYFASjlaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

接下来实现开发本地push代码到github上后，触发Webhook，jenkins自动执行构建。

1. jenkins安装Generic Webhook Trigger 插件
2. github添加触发器

### 配置方法

1.在刚才的testJenkins工程中点击构建触发器中选择Generic Webhook Trigger，填写token

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1eRgygrqgPrhJW8nzYkeicnsql3qfKvyODYObyB1OYWEvktqH4jKR1icXg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

2.github配置Webhook
选择github项目中的Settings->Webhooks>add webhook
配置方式按上图红框中的格式，选择在push代码时触发webhook，成功后会在下方出现一个绿色的小勾勾

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1e1JfiaZZPicj6UEMVpsg3AD5SYGtyK5RfT1ANC4ASDRict9Uvrq7SHTP6A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

测试一下，把vue项目首页的9900去了，然后push代码去github，发现Jenkins中的构建已经自动执行，

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1e5CABEDm2lGeWW8317eLcKxjK7DznN6H9Gib4YeQdOpT6D2w8tGrztrw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

查看页面也是ok的

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRNQpor2DVnHiaa4bFUoYI1evH2ybp2rtbXeHKYXHxnoTLFvoe2WIPnw43I8svibu9R9W9tpicpvl3cA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)image

一套简单的前端自动化工作流就搭建完成，是选择代码push后在Jenkins中手动构建，还是push后自动构建，看公司情况使用。