---
title: Manjaro 安装配置记录
date: 2022-04-30 16:21:51
categories: [daily]
tags: [linux]
description: 把自己的台式机从 Hackintosh 转到 Manjaro，记录安装步骤。
---

# Manjaro 安装配置记录

## 安装步骤

以在 Virtual Box 虚拟机中安装为例，介绍如何安装 Manjaro。

### 下载 Manjaro 镜像

[下载地址](https://manjaro.org/download/)，目前 Manjaro 官方提供三种桌面：xface、kde 和 gnome，可以选择自己喜欢的版本下载。

### 创建虚拟机

在 Virtual Box 管理界面点击「新建」，填写虚拟机的基本信息，其中「版本」选择「Arch Linux（64-bit）」：

![Snipaste_2022-04-27_20-57-53](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/211559.png)

![Snipaste_2022-04-27_20-58-00](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/211630.png)

创建完成后，选中创建的虚拟机，右键点击「设置」进入设置页面，在「系统」栏调整「处理器数量」，在「显示」栏调整「显存大小」：

![image-20220507211848679](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/211848.png)

![Snipaste_2022-04-27_20-58-29](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/212004.png)

![Snipaste_2022-04-27_20-58-36](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/212018.png)

设置完成后，双击启动虚拟机，首次启动时，会提示选择虚拟光盘（即 Manjaro 的镜像文件），按提示选择已下载好的 Manjaro 镜像文件（忘记截图了，以 Ubuntu 的镜像为例），即可进入安装界面：

![Snipaste_2022-04-27_20-47-56](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/212230.png)

![Snipaste_2022-04-27_20-48-07](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/212447.png)

![Snipaste_2022-04-27_20-48-35](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/212459.png)

### 安装 Manjaro

可以在 Manjaro 的引导界面调整一些参数，比如时区、语言、键盘布局等，这里选择「Boot with open source drivers」进入 Manjaro 的 LiveCD 界面：

![Snipaste_2022-04-27_20-59-52](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213023.png)

点击「Luanch installer」进入安装界面：

![Snipaste_2022-04-27_21-00-33](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213047.png)

根据「Manjaro Linux 安装程序」的引导步骤完成对应到设置即可顺利安装：

![Snipaste_2022-04-27_21-00-48](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213316.png)

![Snipaste_2022-04-27_21-00-57](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213324.png)

![Snipaste_2022-04-27_21-01-03](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213337.png)

在「分区」这里，如果没有特殊需求，可以不做调整，如果想要「手动分区」或者设置「交换分区」，推荐阅读 Arch Linux 的安装教程：[建立硬盘分区](https://wiki.archlinux.org/title/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%BB%BA%E7%AB%8B%E7%A1%AC%E7%9B%98%E5%88%86%E5%8C%BA)。

![Snipaste_2022-04-27_21-01-10](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213349.png)

在「用户」栏推荐选中「为管理员帐号使用同样的密码」：

![Snipaste_2022-04-27_21-01-26](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213822.png)

在「摘要」栏会展示即将安装的概要信息，确认后点击「安装」即可：

![Snipaste_2022-04-27_21-01-32](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213838.png)

![Snipaste_2022-04-27_21-01-37](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213853.png)

![Snipaste_2022-04-27_21-01-42](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213908.png)

安装结束后即可重启虚拟机，进入新装的 Manjaro 系统中：

![Snipaste_2022-04-27_21-19-08](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/213921.png)

## 基础配置

### 修改源配置

参见：[Manjaro Linux 源使用帮助](https://mirrors.ustc.edu.cn/help/manjaro.html)。

选择国内镜像站提升体验，使用如下命令生成可用中国镜像站列表：

```bash
sudo pacman-mirrors -i -c China -m rank
```

![2022-04-29 21-52-29屏幕截图](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/221356.png)勾选最接近的镜像站，然后按两次 `OK` ，最后使用如下命令刷新缓存：

`sudo pacman -Syy`

### 添加 Arch Linux CN 源

参见：[Arch Linux CN 源使用帮助](https://mirrors.ustc.edu.cn/help/archlinuxcn.html)。

Arch Linux 中文社区仓库是由 Arch Linux 中文社区驱动的非官方用户仓库。包含中文用户常用软件、工具、字体/美化包等。

在 `/etc/pacman.conf` 文件末尾添加两行：

```properties
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

然后请安装 `archlinuxcn-keyring` 包以导入 GPG key：

```bash
sudo pacman -Sy archlinuxcn-keyring
```

[yay](https://github.com/Jguer/yay) 是一个用 `Go` 语言编码的 Arch Linux 帮助工具和包管理器，允许自动从 `PKGBUILD` 安装包，使用如下命令安装：

```bash
sudo pacman -S --needed git base-devel yay 
```

### 使用英文目录

默认情况下，Manjaro 会根据语言环境创建中文目录，比如：文档、图片等，可以通过以下两步调整为英文目录：

参见： [XDG user directories (简体中文)](https://wiki.archlinux.org/title/XDG_user_directories_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

* 在终端中输入如下命令

  ```bash
  LC_ALL=C xdg-user-dirs-update --force
  ```

  ![2022-04-30 16-06-51屏幕截图](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/161227.png)

* 重新登录系统，选择“保留旧的名称”

  ![2022-04-30 16-08-05屏幕截图](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/161322.png)

### 安装中文输入法

参见：[IBus_GNOME](https://wiki.archlinux.org/title/IBus_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#GNOME)

Gnome 默认使用 IBus，所以只需要安装输入法引擎，这里选择 `ibus-rime` ，一个强大的智能中文输入法，支持拼音、注音或者没有音调的拼音、双拼、粤拼、中州韵、仓颉和五笔 86：

```bash
sudo pacman -S ibus-rime
```

随后打开设置界面，通过“键盘”中的“输入源”添加：

![2022-04-29 21-46-43屏幕截图](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/222234.png)

### 安装 Microsoft Edge

使用 `yay` 安装所需的版本：

```bash
yay microsoft-edge
```

![2022-04-29 22-27-05屏幕截图](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/223139.png)

### 使用 OneDrive

参见：[OneDrive Client for Linux](https://github.com/abraunegg/onedrive)

#### 安装 OneDrive

使用如下命令安装 OneDrive：

```bash
pacman -Ss onedrive-abraunegg
sudo pacman -S onedrive-abraunegg
```

![image-20220430154629576](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/154629.png)

#### 配置 OneDrive

![image-20220430155147440](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/155147.png)

#### 同步文件

使用如下命令启动文件同步：

```bash
onedrive --synchronize
```

#### 开机自启动

使用如下命令将 `onedrive` 添加为系统服务，设置为开机自启动：

```bash
systemctl --user enable onedrive
systemctl --user start onedrive
```

### 科学上网

这里使用 clash 作为客户端，具体参见：[Clash](https://github.com/Dreamacro/clash)。

#### 安装

使用如下命令安装 clash 客户端：

```bash
sudo pacman -S clash
```

#### 配置

下载订阅配置到 `~/.config/clash/config.yaml`，配置如下：

```yaml
port: 7890
socks-port: 7891
allow-lan: false
mode: Rule
log-level: info
external-controller: 127.0.0.1:9090
```

随后打开 http://clash.razord.top/#/settings 进入配置页面：

![image-20220501202533766](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/202533.png)

#### 开机自启动

在 `/usr/share/applications/` 创建 `clash_start.desktop` 文件，内容如下：

```shell
[Desktop Entry]
Exec=/usr/bin/clash
Name=Clash
Type=Application
StartupNotify=false
Terminal=false
Hidden=false
```

然后在「优化」—「开机启动程序」中选择 clash，实际上是在 `~/.config/autostart` 中创建了相同的文件：

![image-20220501204218419](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/204218.png)

![image-20220501204406181](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/204406.png)

## 开发工具安装及配置

### Typora

#### 安装

Typora 是一款 Markdown 编辑器和阅读器，可以通过 `pacman` 直接安装：

```bash
sudo pacman -S typora
```

#### 自动上传图片

Typora 支持自动上传图片至图床，比如 GitHub，参见 [Upload Images](https://support.typora.io/Upload-Image/)，我这里选择使用 [Upgit](https://github.com/pluveto/upgit) 作为上传工具。首先从 GitHub 上下载 upgit 可执行文件，然后在可执行文件同级目录下增加配置文件`config.toml`，内容如下：

```toml
# =============================================================================
# UPGIT Config
# =============================================================================
# default uploader id
default_uploader = "github"
# The file name formatting template was uploaded
rename = "{year}{month}/{hour}{minute}{second}{ext}"
# -----------------------------------------------------------------------------
# Custom extra output formats
# -----------------------------------------------------------------------------
#   {url} direct URL of the file
[output_formats]
"bbcode" = "[img]{url}[/img]"
"html" = '<img src="{url}" />'
"markdown-simple" = "![]({url})"

# If your network access to Github is abnormal or sluggish, you can try the following CDN acceleration.
[replacements]
"raw.githubusercontent.com" = "cdn.jsdelivr.net/gh"
"/master" = "@master"

# Github uploader
[uploaders.github]
# Branch to save files, for example master or main
branch = "master"
# "pat" enter the Github token that has the "repo" permission
# Get token from https://github.com/settings/tokens
pat = "ghp_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
# The name of your public Github repository
repo = "gallery"
# your Github username  
username = "xianglin2020"
```

主要是填写 `repo` 、 `username` 和 `pat` 三项，其中 `replacements` 用于 CDN 加速。

最后在 Typora 中配置使用 upgit ，「上传服务」选择「Custom Command」，命令填写 upgit 执行路径，点击「验证图片上传选项」，如下图所示：

![2022-04-29 22-42-29屏幕截图](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/224343.png)

![2022-04-29 22-42-43屏幕截图](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/224349.png)

#### 生成 Lisence

参见：[Typora](https://www.seesharper.cn/archives/56.html)。

### Open JDK

参见：[Java (简体中文)](https://wiki.archlinux.org/title/Java_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))。

使用 `pacman` 查找和安装 Open JDK，如下所示：

```bash
# 查找 JDK
pacman -Ss openjdk | grep jdk
# 安装 JDK 17
sudo pacman -S jdk-openjdk
# 安装 JDK 11
sudo pacman -S jdk11-openjdk
```

![image-20220430143914269](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/143914.png)

使用 `archlinux-java` 命令管理已安装的 JDK：

![image-20220508114205717](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/114205.png)

查看已安装的 JDK：

```bash
archlinux-java status
```

![image-20220508114316372](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/114316.png)

修改默认 JDK：

```bash
sudo archlinux-java set java-11-openjdk
```

![image-20220508114421770](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/114421.png)

### Intellij IDEA

使用 `pacman` 查找和安装 Intellij IDEA，如下所示：

```bash
pacman -Ss intellij
```

![image-20220430144137003](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/144137.png)

IDEA 中文输入法候选框不跟随光标的解决办法参见：[JetBrainsRuntime-for-Linux-x64](https://github.com/RikudouPatrickstar/JetBrainsRuntime-for-Linux-x64)。

### Maven

IDEA 自带 Maven，但是为了在终端使用，可以单独安装。使用如下命令安装最新版 Maven：

```bash
sudo pacman -S maven
```

使用如下命令查看 Maven 安装路径：

```bash
ls -l `which mvn`
```

![image-20220501184151501](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202205/184151.png)

复制一份 `settings.xml` 到 `~/.m2` 目录下，并修改使用阿里云的 Maven 仓库：

```bash
cp /opt/maven/conf/settings.xml ~/.m2/
```

```xml
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

### Sbt

[sbt](https://www.scala-sbt.org/) 是一个类似于 maven 的构建工具，使用如下命令安装：

```bash
sudo pacman -S sbt
```

### Sublime Text

#### 安装

使用 `yay` 安装 Sublime Text，如下所示：

```bash
yay sublime
```

![image-20220430144414159](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/144414.png)

#### 激活码

一个可用的激活码（2022-04-30）:

—– BEGIN LICENSE —–

Mifeng User

Single User License

EA7E-1184812

C0DAA9CD 6BE825B5 FF935692 1750523A

EDF59D3F A3BD6C96 F8D33866 3F1CCCEA

1C25BE4D 25B1C4CC 5110C20E 5246CC42

D232C83B C99CCC42 0E32890C B6CBF018

B1D4C178 2F9DDB16 ABAA74E5 95304BEF

9D0CCFA9 8AF8F8E2 1E0A955E 4771A576

50737C65 325B6C32 817DCB83 A7394DFA

27B7E747 736A1198 B3865734 0B434AA5

—— END LICENSE ——

#### 一些插件

点击 `Tools` — `Install Package Control` 安装包管理工具；

点击 `Preferences` — `Package Control` 选择 `Install Package`；

![image-20220430145202846](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/145202.png)

输入 `ChineseLocalzations` 安装中文语言包：

![image-20220430145443835](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/145443.png)

输入 `PrettyJSON` 安装 `JSON` 格式化工具，使用 `Ctrl` + `Shift` + `P` 打开命令面板，输入并选择 `Format JSON` 即可格式化 `JSON`：

![image-20220430150037393](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/150037.png)

### Node.js

#### 安装

使用 `pacman` 安装 `Node.js` ，如下所示：

```bash
sudo pacman -S nodejs npm
```

使用 [cnpm](https://www.npmmirror.com/) 替代 `npm` 加速下载：

```bash
npm install -g cnpm --registry=https://registry.npmmirror.com
```

#### 解决 EACCES 问题

参考官方指导：[Resolving EACCES permissions errors when installing packages globally](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally)

* 安装 `nvm` 

  ```bash
  sudo pacman -S nvm
  echo 'source /usr/share/nvm/init-nvm.sh' >> ~/.zshrc
  ```

  ![image-20220430153304384](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/153304.png)

* 使用 `nvm` 安装 `node` 

  ```bash
  nvm install node
  ```

  ![image-20220430153514915](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/153514.png)

* 重新使用 `npm install` 

  ![image-20220430153712711](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/153712.png)

### 使用 Hexo

参见官方文档：[安装 Hexo](https://hexo.io/zh-cn/docs/)

```bash
npm install -g hexo-cli
npx hexo server
```

![image-20220430162721008](https://cdn.jsdelivr.net/gh/xianglin2020/gallery/202204/162721.png)

### 使用 Docker

#### 安装 Docker

使用如下命令安装和启动 `docker`：

```bash
# 安装 Docker
sudo pacman -S docker
# 启动 Docker
sudo systemctl start docker
```

#### 加入 docker 组

使用如下命令将当前用户加入 docker 组：

```bash
sudo usermod -aG docker $USER
reboot
```

#### 使用镜像加速服务

参见：[Docker 镜像加速器](https://developer.aliyun.com/article/29941)：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://rgotb8v4.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### MongoDB Compass

```bash
yay mongodb-compass
```

### Postman

```bash
sudo pacman -S postman-bin
```

### Wireshark

```bash
sudo pacman -S wireshark-qt

sudo usermod -aG wireshark $USER 
```

## 其他常用软件

### vlc

```bash
sudo pacman -S vlc
```

### Google Chrome

```bash
yay -S google-chrome
```

### 截图工具

参见：[Flameshot](https://github.com/flameshot-org/flameshot)。

```bash
sudo pacman -S flameshot
```

### VirtualBox

参见：[VirtualBox](https://wiki.archlinux.org/title/VirtualBox_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))。

```bash
sudo pacman -S virtualbox linux-headers virtualbox-ext-oracle
# 手动加载内核模块（或者重启）
sudo modprobe vboxdrv vboxnetadp vboxnetflt
```

### VSCode

```bash
sudo pacman -S vscodium
```

### 有道词典

```bash
yay youdao-dict
```

### uluancher

参见：[Application launcher for Linux](https://ulauncher.io/)，在 Wayland 下需要手动添加快捷键，详见：[Hotkey In Wayland](https://github.com/Ulauncher/Ulauncher/wiki/Hotkey-In-Wayland)。

```bash
yay ulauncher
```

### WPS

```bash
yay wps-office-cn
yay wps-office-mui-zh-cn
yay ttf-wps-fonts
yay ttf-ms-fonts
```

### Aria2

```bash
sudo pacman -S aria2 
```


### 连接 iPhone

#### 从 iPhone 中复制照片到 Manjaro

如果想将 iPhone 中的照片传输到 Manjaro 中，可以通过 `ifuse` 将 iPhone 作为磁盘挂在到 Manjaro 上。

首先需要安装 `ifuse` ，如图：

```bash
pacman -Ss ifuse
```

![image-20220515144838615](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/144838.png)

为 iPhone 创建挂载点：

```bash
mkdir -p ~/Documents/iPhone
```

使用如下命令将 iPhone 挂在到 Manjaro，（执行如下命令时，iPhone 会提示你“是否信任此设备”，然后输入密码，所以第一次挂载时需要多次执行如下命令）：

```bash
ifuse ~/Documents/iPhone
```

![image-20220515145438208](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202205/145519.png)

#### 从 Manjaro 中复制文件到 iPhone

iPhone 的「文件」APP 中能连接服务器，因此可以在 Manjaro 中启动文件共享服务，iPhone 作为客户端连接即可。这里选择使用 `samba`，一种Linux、UNIX系统上可用于共享文件和打印机等资源的协议，这种协议是基于 Client \ Server 型的协议，Client 端可以通过 SMB 访问到 Server（服务器）上的共享资源。

首先需要安装 `samba`：

```bash
sudo pacman -S samba manjaro-settings-samba
```

然后编辑 `samba` 的配置文件：

```bash
sudo vim /etc/samba/smb.conf
```

只需要编辑 `[homes]` 这部分即可，如下所示：

```properties
[homes]
   comment = Home Directories
   browseable = yes # no -> yes
   read only = yes
   create mask = 0700
   directory mask = 0700
   valid users = %S
   path = /run/media/xianglin # path to share
```

使用如下命令为 `samba` 添加用户：

```bash
sudo smbpasswd -a xianglin
```

完成后启动 `smb` 服务，即可在 iPhone 上连接到共享目录中：

```bash
sudo systemctl restart smb
```
