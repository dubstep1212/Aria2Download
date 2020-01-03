
## 目录
* [前言](#前言)
* [环境要求](#环境要求)
* [安装必要组件](#安装必要组件)
* [安装Nginx](#安装Nginx)
* [安装Aria2](#安装Aria2)
* [安装AriaNg](#安装AriaNg)
* [配置Nginx虚拟主机](#配置Nginx虚拟主机)
* [安装Rclone并挂载GoogleDrive](#安装Rclone并挂载GoogleDrive)
    * [安装Rclone](#安装Rclone)
    * [配置Rclone](#配置Rclone)
    * [修改上传脚本](#修改上传脚本)
    * [修改Aria2配置文件](#修改Aria2配置文件)
* [总结](#总结)
    * [Aria2使用说明](#Aria2使用说明)
    * [Nginx使用说明](#Nginx使用说明)


来源[https://www.yuque.com/docs/share/4c02f96e-5799-45ec-b208-709190fef793?#](https://www.yuque.com/docs/share/4c02f96e-5799-45ec-b208-709190fef793?#)

## 前言
随着Aria2的大火，[**Aria2**](https://aria2.github.io/)+[**AriaNg**](http://ariang.mayswind.net/zh_Hans/)+[**Rclone**](https://rclone.org/)+[**Google Drive**](https://drive.google.com/)模式的离线下载更成为各大博主的主流下载方式，博主把优化过的教程发出来。本教程主要通过AriaNg+Nginx提供Web界面，Aria2下载资源，下载后通过Rclone把资源自动上传到GoogleDrive，以达到离线下载资源的目的。

_注：网上的大部分教程有严重的_**_BUG_**_，下载到VPS的文件并不能自动删除，当下载到硬盘满了之后就无法再继续运行，而且会上传.Aria2后缀文件碎片，本教程经过二次修改后脚本采用移动方式上传到网盘。_<br />

## 环境要求

- 系统支持：CentOS6+ / Debian6+ / Ubuntu14+
- 一台KVM/OpenVZ架构的VPS
- 谷歌网盘(其他网盘也可以，但需要[**Rclone**](https://rclone.org/)支持)

**推荐使用CentOS 7，本教程也以CentOS 7+谷歌无限团队盘来演示，若使用其他系统，以下某些命令会出现不兼容，请自行修改为对应系统的命令。**<br />**<br />**以下命令行建议一条一条输入**<br />**
<a name="elMe2"></a>
## 安装必要组件
首先更新系统并安装必要组件，此处为安装Nginx Web服务器
```bash
yum -y update
yum -y install epel-release
yum -y install wget git unzip gcc gcc-c++ openssl-devel nginx
```

<a name="XYmam"></a>
## 安装Nginx
```bash
yum -y install nginx
```
启动nginx并设置开机自启动
```bash
systemctl start nginx #启动Nginx
systemctl enable nginx.service #设置开机启动Nginx
```
关闭防火墙并关闭开机启动
```bash
systemctl stop firewalld #关闭防火墙
systemctl disable firewalld.service #静止开机启动
```

<a name="pNb7C"></a>
## 安装Aria2
下载Aria2脚本并运行脚本。
```bash
cd ~ #回到根目录
wget -N https://git.io/aria2.sh && chmod +x aria2.sh && bash aria2.sh #下载Aria2脚本并运行脚本
```
![sp200103_141148.png](https://cdn.nlark.com/yuque/0/2020/png/393161/1578032772414-aa7f1cc7-79b2-4064-a579-071b6df26785.png#align=left&display=inline&height=400&name=sp200103_141148.png&originHeight=400&originWidth=183&size=10208&status=done&style=none&width=183)<br />运行脚本后会出现脚本操作菜单，选择并输入**1**回车开始安装<br />安装成功后得到以下Aria2配置信息，并用记事本记录<br />![sp200103_142105.png](https://cdn.nlark.com/yuque/0/2020/png/393161/1578032755710-6ec161a8-3d14-4d26-ac5c-30118f57e2b7.png#align=left&display=inline&height=154&name=sp200103_142105.png&originHeight=154&originWidth=222&size=7576&status=done&style=none&width=222)<br />
<br />设置自动更新**BT-Tracker**，后台会定期更新[**trackerslist**](https://github.com/ngosang/trackerslist)，一般情况下会加强BT下载效果(**若不需要，请跳过次步骤**)<br />再次执行菜单脚本
```bash
bash aria2.sh
```
输入**12**回车


## 安装AriaNg
AriaNg是一个前端(HTML+JS静态)控制面板，不需要和Aria2(后端/服务端)放在一个服务器或者设备中，你甚至可以只在服务器上面搭建Aria2后端，然后访问别人建好的AriaNg前端面板，也可以远程操作Aria2后端！<br />**AriaNg官方演示页面：[http://ariang.mayswind.net/latest](http://ariang.mayswind.net/latest)**<br />**<br />创建Ariang目录**
```bash
mkdir -p /data/www/ariang #创建一个Ariang目录
cd /data/www/ariang #进入这个目录
```
下载并解压AriaNg文件，这段代码会自动检测并下载最新版本(**代码较长，请复制完整**)
```bash
Ver=$(wget --no-check-certificate -qO- https://api.github.com/repos/mayswind/AriaNg/releases/latest | grep -o '"tag_name": ".*"' | sed 's/"//g;s/tag_name: //g') && echo ${Ver}
```
如果上面自动检测最新版本的代码返回空白或者错误，那么请访问下面网址来查看最新版本号<br />[**https://github.com/mayswind/AriaNg/releases/latest**](https://github.com/mayswind/AriaNg/releases/latest)** **<br />例如手动获取的版本号是 1.1.4，那么手动执行命令： **Ver=1.1.4**，然后继续下面步骤即可<br />下载并解压AriaNg文件(**代码较长，请复制完整**)
```bash
wget -N --no-check-certificate "https://github.com/mayswind/AriaNg/releases/download/${Ver}/AriaNg-${Ver}.zip" && unzip AriaNg-${Ver}.zip && rm -rf AriaNg-${Ver}.zip
```
 
<a name="51ENf"></a>
## 配置Nginx虚拟主机
创建虚拟主机配置文件
```bash
yum install vim #安装vim编辑器
cd /etc/nginx/conf.d 
touch ariang.conf #创建配置文件
vim ariang.conf #编辑配置文件
```

按键盘**I**键或**Insert**键进入修改模式，复制粘贴以下配置(**复制粘贴时请仔细对照一下，有时候可能粘贴不全**)
```nginx
server {
    listen 80;
    server_name IP;
    location / {
        root   /data/www/ariang;
        index  index.html index.htm;
    }
}
```
把以上配置中**server_name**右边的**IP**地址改成你自己的服务器IP或域名<br />保存并退出
```shell
:wq
```

现在可以使用**nginx -t**测试一下配置文件是否修改正确<br />更新nginx配置后，需要重启nginx
```bash
systemctl reload nginx
```

关闭**SELINUX**
```bash
vim /etc/selinux/config
```
按键盘**I**键或**Insert**键进入修改模式<br />找到**SELINUX=enforcing**这行，并用#注释掉，并在下一行加上**SELINUX=disabled**
```shell
#SELINUX=enforcing
SELINUX=disabled
```
保存并退出
```shell
:wq
```

重启系统
```bash
reboot
```

重启好后访问IP地址或域名即可看到AriaNg Web界面，并按照下图1-4步骤填写Aria2 RPC密钥(**该密钥为前面安装Aria2时生成配置信息中的密码**)![sp200103_151356.png](https://cdn.nlark.com/yuque/0/2020/png/393161/1578035707646-7fe49690-3c47-4124-b5a4-1518a5b39cef.png#align=left&display=inline&height=716&name=sp200103_151356.png&originHeight=716&originWidth=1232&size=78161&status=done&style=none&width=1232)


## 安装Rclone并挂载GoogleDrive

#### 安装Rclone
```bash
cd ~ #回到根目录
```
下载脚本并执行
```bash
curl https://rclone.org/install.sh | sudo bash
```
如果系统为**OpenVZ**系统，则使用以下脚本安装
```bash
curl https://rclone.org/install.sh | bash
```

<a name="r9HOT"></a>
#### 配置Rclone
```bash
rclone config
```
[root@localhost[ ]()~]# **rclone config**<br />2020/01/03 02:41:17 NOTICE: Config file "/root/.config/rclone/rclone.conf" not found - using defaults<br />No remotes found - make a new one<br />n) New remote<br />s) Set configuration password<br />q) Quit config<br />n/s/q>**n**<br />name> **GD**(该名称为Rclone挂载的名称可以随便填，但要记住，后面会用到)<br />Type of storage to configure.<br />Enter a string value. Press Enter for the default ("").<br />Choose a number from below, or type in your own value<br />1 / 1Fichier<br />   \ "fichier"<br />2 / Alias for an existing remote<br />   \ "alias"<br />3 / Amazon Drive<br />   \ "amazon cloud drive"<br />4 / Amazon S3 Compliant Storage Provider (AWS, Alibaba, Ceph, Digital Ocean, Dreamhost, IBM COS, Minio, etc)<br />   \ "s3"<br />5 / Backblaze B2<br />   \ "b2"<br />6 / Box<br />   \ "box"<br />7 / Cache a remote<br />   \ "cache"<br />8 / Citrix Sharefile<br />   \ "sharefile"<br />9 / Dropbox<br />   \ "dropbox"<br />10 / Encrypt/Decrypt a remote<br />    \ "crypt"<br />11 / FTP Connection<br />    \ "ftp"<br />12 / Google Cloud Storage (this is not Google Drive)<br />    \ "google cloud storage"<br />**13 / Google Drive<br />    \ "drive"**<br />14 / Google Photos<br />    \ "google photos"<br />15 / Hubic<br />    \ "hubic"<br />16 / JottaCloud<br />    \ "jottacloud"<br />17 / Koofr<br />    \ "koofr"<br />18 / Local Disk<br />    \ "local"<br />19 / Mail.ru Cloud<br />    \ "mailru"<br />20 / Mega<br />    \ "mega"<br />21 / Microsoft Azure Blob Storage<br />    \ "azureblob"<br />22 / Microsoft OneDrive<br />    \ "onedrive"<br />23 / OpenDrive<br />    \ "opendrive"<br />24 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)<br />    \ "swift"<br />25 / Pcloud<br />    \ "pcloud"<br />26 / Put.io<br />    \ "putio"<br />27 / QingCloud Object Storage<br />    \ "qingstor"<br />28 / SSH/SFTP Connection<br />    \ "sftp"<br />29 / Transparently chunk/split large files<br />    \ "chunker"<br />30 / Union merges the contents of several remotes<br />    \ "union"<br />31 / Webdav<br />    \ "webdav"<br />32 / Yandex Disk<br />    \ "yandex"<br />33 / http Connection<br />    \ "http"<br />34 / premiumize.me<br />    \ "premiumizeme"<br />Storage> **13(由于Rclone官方会更新，13并不一定是Google Drive，所以填入需要配置的网盘对应的序号即可)**<br /> See help for drive backend at: https://rclone.org/drive/

Google Application Client Id<br />Setting your own is recommended.<br />See https://rclone.org/drive/#making-your-own-client-id for how to create your own.<br />If you leave this blank, it will use an internal key which is low performance.<br />Enter a string value. Press Enter for the default ("").<br />client_id>**直接回车**<br />Google Application Client Secret<br />Setting your own is recommended.<br />Enter a string value. Press Enter for the default ("").<br />client_secret>**直接回车**<br />Scope that rclone should use when requesting access from drive.<br />Enter a string value. Press Enter for the default ("").<br />Choose a number from below, or type in your own value<br />1 / Full access all files, excluding Application Data Folder.<br />   \ "drive"<br />2 / Read-only access to file metadata and file contents.<br />   \ "drive.readonly"<br />   / Access to files created by rclone only.<br />3 | These are visible in the drive website.<br />   | File authorization is revoked when the user deauthorizes the app.<br />   \ "drive.file"<br />   / Allows read and write access to the Application Data folder.<br />4 | This is not visible in the drive website.<br />   \ "drive.appfolder"<br />   / Allows read-only access to file metadata but<br />5 | does not allow any access to read or download file content.<br />   \ "drive.metadata.readonly"<br />scope>**1**<br />ID of the root folder<br />Leave blank normally.

Fill in to access "Computers" folders (see docs), or for rclone to use<br />a non root folder as its starting point.

Note that if this is blank, the first time rclone runs it will fill it<br />in with the ID of the root folder.

Enter a string value. Press Enter for the default ("").<br />root_folder_id>**直接回车**<br />Service Account Credentials JSON file path<br />Leave blank normally.<br />Needed only if you want use SA instead of interactive login.<br />Enter a string value. Press Enter for the default ("").<br />service_account_file>**直接回车**<br />Edit advanced config? (y/n)<br />y) Yes<br />n) No<br />y/n> **n**<br />Remote config<br />Use auto config?

Say Y if not sure<br />Say N if you are working on a remote or headless machine<br />
y) Yes<br />
n) No<br />
y/n> **n**<br />
If your browser doesn't open automatically go to the following link: **https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=2024815644.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive&state=eSMtROkBwv7nTw36Wg(****复制这段网址到浏览器中，最后会得到一段密钥并复制粘贴到下面****)**<br />
Log in and authorize rclone for access<br />
Enter verification code> **4/vAGEnm1Vm2O7Ba**************************<br />
Configure this as a team drive?<br />
y) Yes<br />
n) No<br />
y/n>**y**<br />
Fetching team drive list...<br />
Choose a number from below, or type in your own value<br />1 / dd<br />   \ "0AAk7UDUUk9PVA"<br />
Enter a Team Drive ID> **1**<br />----------------------------<br />[GD]<br />type = drive<br />scope = drive<br />token = {"access_token":"ya29.Il4B97vj2p_*****************","token_type":"Bearer","refresh_token":"1//0eINZ8S2xdmkPdzFk7-*************","expiry":"2020-0103T03:44:19.994344966-05:00"}
team_drive = 0AAk7UoZt************<br />-----------------------------<br />y) Yes this is OK<br />e) Edit this remote<br />d) Delete this remote<br />y/e/d> **y**<br />Current remotes:

Name                 Type<br />====                 ====<br />GD                     drive

e) Edit existing remote<br />n) New remote<br />d) Delete remote<br />r) Rename remote<br />c) Copy remote<br />s) Set configuration password<br />q) Quit config<br />e/n/d/r/c/s/q>**q<br />**

<a name="xS0Pb"></a>
#### 修改上传脚本
修改我们的上传脚本中的Rclone名字和需要上传到网盘的文件夹
```bash
vim /root/.aria2/autoupload.sh #编辑上传脚本
```
按键盘**I**键或**Insert**键进入修改模式<br />修改上传脚本的内容，博主修改为Google Drive的文件夹Download，**这个文件夹必须存在，如果不存在请先在Google Drive中自行创建，不然会出错**。
```shell
name='GD' #配置Rclone时的name
folder='/Download' #网盘里的文件夹，留空为网盘根目录
```
保存并退出
```shell
:wq
```

<a name="U5ETE"></a>
#### 修改Aria2配置文件
打开配置文件
```bash
vim /root/.aria2/aria2.conf
```
按键盘**I**键或**Insert**键进入修改模式<br />修改为以下内容

- 在on-download-complete=/root/.aria2/delete.aria2.sh前面加上#号
- 把#on-download-complete=/root/.aria2/autoupload.sh前面#号去掉
```shell
# 下载完成后执行的命令
# 删除.aria2文件
#on-download-complete=/root/.aria2/delete.aria2.sh
# 调用 rclone 上传(move)到网盘
on-download-complete=/root/.aria2/autoupload.sh
```
保存并退出
```shell
:wq
```
然后重启Aria2
```bash
service aria2 restart
```

**到此为止离线下载已经完美搞定，输入服务器网址或域名到AriaNg Web页面进行下载资源，下载完成后直接到Google Drive中进行查看。**<br />![QQ截图20200103231317.jpg](https://cdn.nlark.com/yuque/0/2020/jpeg/393161/1578064452665-13d0ea62-5f70-4563-af00-7bae5b291f42.jpeg#align=left&display=inline&height=718&name=QQ%E6%88%AA%E5%9B%BE20200103231317.jpg&originHeight=718&originWidth=1232&size=40317&status=done&style=none&width=1232)<br />
<a name="ovG4y"></a>
## 总结<br />
<a name="m9Dwo"></a>
#### Aria2使用说明

- 菜单：bash /root/aria2.sh
- 启动：/etc/init.d/aria2 start
- 停止：/etc/init.d/aria2 stop
- 重启：/etc/init.d/aria2 restart
- 查看状态：/etc/init.d/aria2 status
- 配置文件：/root/.aria2/aria2.conf 
- 下载目录：/root/Download
<a name="LEaJC"></a>
#### Nginx使用说明

- 查看服务状态：systemctl status nginx
- 启动服务：systemctl start nginx
- 停止服务：systemctl stop nginx
- 重新启动服务：systemctl restart nginx 或 systemctl reload nginx
- 设置开机自动启动：systemctl enable nginx
- 关闭开机自动启动：systemctl disable nginx
- 查看所有已启动的服务：systemctl list-units --type=service
- 默认配置文件：/etc/nginx/nginx.conf
- AriaNg主机配置文件：/etc/nginx/conf.d/ariang.conf
- 默认目录：/usr/share/nginx/html
- 访问日志：/var/log/nginx/access.log
- 错误日志：/var/log/nginx/error.log
