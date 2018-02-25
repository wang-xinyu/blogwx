Title: Modify the user name of Raspbian
Date: 2016/4/19
Category: Raspberry Pi
Tags: Linux, Raspi
Author: WangXinyu
# 修改Raspbian的用户名
####版权声明：本文为作者原创，欢迎批评指正，谢绝转载。

本文将介绍通过SSH修改Raspbian默认用户名pi的详细过程。
## 1. 启用root用户
输入两遍密码解锁root用户，最后通过su验证root用户是否已经启用。

	pi@raspberrypi:~ $ sudo passwd root
	Enter new UNIX password: 
	Retype new UNIX password: 
	passwd: password updated successfully
	
	pi@raspberrypi:~ $ sudo passwd --unlock root
	passwd: password expiry information changed.
	
	pi@raspberrypi:~ $ su
	Password: 
	root@raspberrypi:/home/pi#

## 2. 允许SSH登录root用户
SSH出于安全考虑，默认不允许登录root用户，需修改配置文件/etc/ssh/sshd_config，将PermitRootLogin的属性改为yes。

	#PermitRootLogin without-password
	PermitRootLogin yes
## 3. 修改Boot选项
利用raspi-config工具修改Boot选项，将自动以pi用户登录，改为requiring user to login。修改后重启。
## 4. SSH登录root用户

	wxy@wxy:~ $ ssh root@192.168.1.135
	password: 
	root@raspberrypi:~# 
	
## 5. 修改名字
修改user name，group name，home路径名，以及/etc/passwd中的home路径。

	root@raspberrypi:~# usermod -l urname pi
	root@raspberrypi:~# groupmod -n urname pi
	root@raspberrypi:~# mv /home/pi/ /home/urname
	root@raspberrypi:~# usermod -d /home/urname urname

## 6. 配置新用户的sudo权限
修改/etc/sudoers文件，将具有sudo权限的pi改为urname。

	pi ALL=(ALL) NOPASSWD: ALL
	改为
	urname ALL=(ALL) NOPASSWD: ALL
	
## 7. 收尾
修改完之后，可用新用户名登录，最后不要忘了锁定root用户，并禁止SSH登录root用户。


####版权声明：本文为作者原创，欢迎批评指正，谢绝转载。
