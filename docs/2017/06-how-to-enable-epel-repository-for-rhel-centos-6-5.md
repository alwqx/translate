# RHEL/CentOS 7.x/6.x/5.x开启EPEL仓库

## 说明
- [原文链接](http://www.tecmint.com/how-to-enable-epel-repository-for-rhel-centos-6-5/)
- [翻译：@adolphlwq](https://github.com/adolphlwq)
- [项目地址](https://github.com/adolphlwq/translate)
- <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/80x15.png" /></a>

>这篇指南文章教你如何在`RHEL/CentOS 7.x/6.x/5.x`系统中开启EPEL仓库支持，以便你可以使用`yum`命令
安装额外的标准开源软件包。

![](http://www.tecmint.com/wp-content/uploads/2012/06/Install-Epel-in-Linux1.jpg)

您还可以参考：[Install and Enable RPMForge Repository in RHEL/CentOS 7/6/5/4](http://www.tecmint.com/install-and-enable-rpmforge-repository-in-rhel-centos-6-5-4/)

## EPEL是什么？
EPEL (Extra Packages for Enterprise Linux)是来自于Fedora team的开源、免费的社区软件仓库项目，
旨为像RHEL(Red Hat Enterprise Linux)、CentOS和Scientific Linux这样的Linux发行版提供100%高质量
附加软件包。Epel项目不是RHEL或CentOS的一部分，它是为主要的Linux发行版设计的，因为它提供networking、sysadmin、programming、monitoring
等工具。Epel中的大部分软件包由Fedora repo维护。

## 为什么使用EPEL仓库？
- 为Yum提供大量开源软件包
- Epel repo 100%开源并且免费使用
- 它不复制Linux核心软件，也不存在兼容性问题
- 所有epel软件包都有Fedora team维护

## 如何在RHEL/CentOS 7/6/5上开启EPEL
首先你需要使用**wget**这样的工具下载所需文件，然后在系统上使用**RPM**开启EPEL仓库。根据你的Linux系统版本使用下面
提供的链接（确保你是root用户）：

### RHEL/CentOS 7 64bit
```
## RHEL/CentOS 7 64-Bit ##
# wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-9.noarch.rpm
# rpm -ivh epel-release-7-9.noarch.rpm
```

### RHEL/CentOS 6 32-64 Bit
```
## RHEL/CentOS 6 32-Bit ##
# wget http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
# rpm -ivh epel-release-6-8.noarch.rpm
## RHEL/CentOS 6 64-Bit ##
# wget http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
# rpm -ivh epel-release-6-8.noarch.rpm
```

### RHEL/CentOS 5 32-64 Bit
```
## RHEL/CentOS 5 32-Bit ##
# wget http://download.fedoraproject.org/pub/epel/5/i386/epel-release-5-4.noarch.rpm
# rpm -ivh epel-release-5-4.noarch.rpm
## RHEL/CentOS 5 64-Bit ##
# wget http://download.fedoraproject.org/pub/epel/5/x86_64/epel-release-5-4.noarch.rpm
# rpm -ivh epel-release-5-4.noarch.rpm
```

### RHEL/CentOS 4 32-64 Bit
```
## RHEL/CentOS 4 32-Bit ##
# wget http://download.fedoraproject.org/pub/epel/4/i386/epel-release-4-10.noarch.rpm
# rpm -ivh epel-release-4-10.noarch.rpm
## RHEL/CentOS 4 64-Bit ##
# wget http://download.fedoraproject.org/pub/epel/4/x86_64/epel-release-4-10.noarch.rpm
# rpm -ivh epel-release-4-10.noarch.rpm
```

## 如何验证EPEL仓库
只需运行下面的命令验证EPEL repo是否开启，如果你已经开启，你可能看到下面的内容。
```
yum repolist
```

**Sample Output**
```
➜  ~ yum repolist
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirrors.cn99.com
 * epel: mirror.lzu.edu.cn
 * extras: mirror.lzu.edu.cn
 * updates: mirror.lzu.edu.cn
repo id                                                repo name                                                                            status
!base/7/x86_64                                         CentOS-7 - Base                                                                       9,363
!dockerrepo                                            Docker Repository                                                                        90
!epel/x86_64                                           Extra Packages for Enterprise Linux 7 - x86_64                                       11,179
!extras/7/x86_64                                       CentOS-7 - Extras                                                                       263
!kubernetes                                            Kubernetes                                                                                5
!updates/7/x86_64                                      CentOS-7 - Updates                                                                      834
repolist: 21,734
```

## 我如何使用EPEL？
你需要使用yum命令来搜索和安装软件包。比如我们使用epel repo来搜索`Zabbix`：
```
yum --enablerepo=epel info zabbix
```

**Sample Output**
```
Available Packages
Name       : zabbix
Arch       : i386
Version    : 1.4.7
Release    : 1.el5
Size       : 1.7 M
Repo : epel
Summary    : Open-source monitoring solution for your IT infrastructure
URL        : http://www.zabbix.com/
License    : GPL
Description: ZABBIX is software that monitors numerous parameters of a network.
```

我们也可以使用选项`–enablerepo=epel`安装zabbix
```
# yum --enablerepo=epel install zabbix
```

Note：epel配置文件默认在`/etc/yum.repos.d/epel.repo`等工具。Epel中的大部分软件包由Fedora

## 译者观点
本文介绍了`EPEL`的基础知识，对个人很有帮助，了解了EPEL的起源和维护组织。
用户在安装EPEL时**要安装和系统对应的正确版本**。
