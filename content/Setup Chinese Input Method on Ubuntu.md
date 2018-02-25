Title: Setup Chinese Input Method on Ubuntu
Date: 2017/5/9
Category: Ubuntu
Tags: Linux
Author: WangXinyu
# Ubuntu设置中文输入法IBus

## 1. 安装语言包
打开System Settings–>Language Support–>Install/Remove Languages，安装Chinese(simplified)。
 
## 2. 安裝IBus框架

	$ sudo apt-get install ibus ibus-clutter ibus-gtk ibus-gtk3 ibus-qt4


## 3. 启用IBus框架
运行以下命令，重启电脑。

	$ im-switch -s ibus		// Ubuntu 14.04
	$ im-config -s ibus		// Ubuntu 16.04

## 4. 安装拼音引擎

	$ sudo apt-get install ibus-pinyin

## 5. 设置IBus框架
运行如下命令，在弹出的窗口中选择Add->Chinese->Pinyin

	$ ibus-setup

## 6. 添加输入法
打开System Settings–>Text Entry->Add(+)->Chinese(Pinyin)(IBus)

## 7. 如果拼音打字不正常
尝试在运行如下命令：

	$ ibus-daemon -drx