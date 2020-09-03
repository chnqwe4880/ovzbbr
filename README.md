# ovzbbr

# 使用方法
已测试通过的系统：Ubuntu 14.04 x64、Ubuntu 16.04 x64、CentOS 6 x64、CentOS 7 x64只支持64位系统，要求glibc版本2.14以上。


# 注意：
安装失败的话，可能后台没有开启TUN/TAP。
#需要配置的有如下几个选项：
#1、需要加速的端口，即的55端口。加速开启之后，流量会先经过BBR处理，之后再发送给后端的55。
#2、可能需要配置 “公网接口名称”，即你服务器上具有公网IP的接口名称。搬瓦工OpenVZ上默认都是venet0，但是有朋友可能需要安装在其他服务器上，所以我加入了此选项。


# 注意：
在有firewalld的服务器上安装的时候，firewalld会干扰iptables的规则，造成网络不通（现在具体原因未知，谁有解决方案可以提示一下）。所以在装有firewalld的服务器上需要先退出firewalld：

systemctl stop firewalld
systemctl stop firewalld


# 如需卸载，请使用：

#./ovz-bbr-installer.sh uninstall
#错误说明
#有些机器一切正常，但是加速失败。从网友的反馈来看，可能需要将55的监听地址从vps IP改到127.0.0.1或者 0.0.0.0，具体未测试，加速失败的朋友可以试一试。


# 多端口加速
安装的时候只配置了一个加速端口，但是你可以配置多端口加速，配置方法非常简单。修改文件
 vi /usr/local/haproxy-lkl/etc/port-rules
在文件里添加需要加速的端口，每行一条，可以配置单个端口或者端口范围，以#开头的行将被忽略。 例如：8800或者8800-8810配置完成之后，只需要重启haproxy-lkl即可。


使用systemctl或者service命令来启动、停止和重启HAporxy-lkl：

systemctl {start|stop|restart} haproxy-lkl
service haproxy-lkl {start|stop|restart}

#

/usr/local/haproxy-lkl/etc/haproxy.cfg这个文件是通过port-rules自动生成的，每次启动都会重新生成，所以直接修改它的配置没用。 如果想要自定义配置，请修改启动文件：

/usr/local/haproxy-lkl/sbin/haproxy-lkl

# 更新glibc
# 1、CentOS 6更新glibc，首先下载如下几个文件：

wget http://ftp.redsleeve.org/pub/steam/glibc-2.15-60.el6.x86_64.rpm \
http://ftp.redsleeve.org/pub/steam/glibc-common-2.15-60.el6.x86_64.rpm \
http://ftp.redsleeve.org/pub/steam/glibc-devel-2.15-60.el6.x86_64.rpm \
http://ftp.redsleeve.org/pub/steam/glibc-headers-2.15-60.el6.x86_64.rpm \
http://ftp.redsleeve.org/pub/steam/nscd-2.15-60.el6.x86_64.rpm

# 2、然后安装：

 rpm -Uvh glibc-2.15-60.el6.x86_64.rpm \
glibc-common-2.15-60.el6.x86_64.rpm \
glibc-devel-2.15-60.el6.x86_64.rpm \
glibc-headers-2.15-60.el6.x86_64.rpm \
nscd-2.15-60.el6.x86_64.rpm

# 3、如果以上步骤无法更新，可以手动编译更新

wget http://ftp.gnu.org/gnu/glibc/glibc-2.15.tar.gz
wget http://ftp.gnu.org/gnu/glibc/glibc-ports-2.15.tar.gz
tar -zxf glibc-2.15.tar.gz
tar -zxf glibc-ports-2.15.tar.gz
mv glibc-ports-2.15 glibc-2.15/ports
mkdir glibc-build-2.15
cd glibc-build-2.15
../glibc-2.15/configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
make all && make install

# 4、检查一下：

 # ldd --version
ldd (GNU libc) 2.15
Copyright (C) 2012 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
Written by Roland McGrath and Ulrich Drepper.
已经升级到glibc 2.15了。

判断BBR已正常工作
判断bbr是否正常启动可以尝试ping 10.0.0.2，如果能通，说明bbr已经启动。


















