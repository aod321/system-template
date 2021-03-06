[TOC]

# 服务器一键配置

## 介绍

提供快速安装配置服务器下个人账号的方案，包括：

* 更改HOME，使得HOME文件夹、同步文件夹、工作文件夹相互分离，避免工作文件夹下一堆点文件

* 配置shell（zsh、bash）、tmux、ipython、jupyter、neovim及其插件、一些终端下的实用脚本和二进制文件

* 按电脑分组，将共享配置文件同步
* 一键安装shadowsocks服务器

**子模块**

* https://github.com/hyliang96/shareENV
* https://github.com/hyliang96/serverENV
* https://github.com/hyliang96/shadowsocks_ips

## 安装方法

### 用户管理

不要在root下装，先开用户，给他root权限。给root/下用脚本装，会搞坏文件，很危险。

[教程](https://blog.51cto.com/zlfwmm/1764264)

#### 开用户

```bash
sudo adduser <用户名>
# 输入密码
```

#### 加管理员权限

```bash
sudo vim /etc/sudoers
```

```bash
# User privilege specification
root ALL=(ALL) ALL
<用户名> ALL=(ALL) ALL
```

退出vim按

```
:wq!
```

#### 时区修改

- 查看时间

```bash
date "+%Y-%m-%d-%H:%M:%S"
# 返回 2019-03-06-16:52:38
```

- 个人用户修改

获得修改指令

```bash
tzselect
```

一路选择地区，直到得到返回`export TZ='Asia/Shanghai'`，即为修改指令。

将修改指令加到`~/.zshenv`，则每次用户使用zshenv，时间为此种时间。

- 整个系统修改

```bash
date -R # 再次查看时间
sudo cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
date -R # 再次查看时间，确认已经修改为北京时间
sudo hwclock --systohc # 修改硬件CMOS的时间，不然后面时间还是不准
```

#### 安装软件

```bash
sudo apt install zsh
# YouCompleteMe 的依赖 如下
sudo apt install python # python2,3 都装了
sudo apt install cmake
sudo apt install python-dev
sudo apt-get install build-essential
# YouCompleteMe 的依赖 结束
```

### 备份

下文


把目录清空

```bash
rm /home/$USER/backup_before_install -rf
mkdir /home/$USER/backup_before_install
mv /home/$USER/*  /home/$USER/backup_before_install
mv /home/$USER/.*  /home/$USER/backup_before_install
```

或者全部清空【危险】

```bash
rm /home/$USER/* -rf
rm /home/$USER/.* -rf
```

重新登录，进入bash

### 安装

* 安装系统环境：
  * 文件组织、符号链接
  * shell的dotfiles、git/tmux/ipython/jupyter的配置、zsh、vim、
  * 翻墙服务端和客户端
  * 登录登出同步共享文件的脚本

```bash
cd /home/$USER
git clone https://github.com/hyliang96/system-template.git
cp /home/$USER/system-template/{*,.*} /home/$USER -r
rm /home/$USER/system-template -rf
```

```bash
# vi init_script/install_path.sh # 默认不是root用户，不用填写安装路径
# vi ENV/CONF/.env               # 默认不是root用户，不用修改my=《/home/$USER》
bash /home/$USER/init_script/install_sys.sh
```

* 安装科学计算软件：anaconda、pytorch、7z、tensoflow等等

```bash/home/$USER=/home/$USER
bash /home/$USER/init_script/cpu_software.sh
# 或
bash /home/$USER/init_script/gpu_software.sh
```

* 如果vim中的YouCompleteMe用不了，就得自己手动编译

```bash
$shareENV/app_config/vim/.vim/plugged/YouCompleteMe/install.py
```

### ssh设置

- 笔记本上

```bash
ssh-copy-id -i ~/.ssh/<私钥名> <服务器别名>
```

私钥名默认为`id_rsa`


- 服务器上

当确认服务器运行正常，将其`.ssh/id_ras.pub`公钥添加到github上，则可以同步shareENV、serverENV

### 安装shadowsocks服务器

```
s3 install
```

执行过程中需要输入sudo密码

## 注意事项

### 隐私文件

列出来，把它们单独写成一个repo，然后每次登录登出时同步？

### 时区修改

虽然上述配置为个人用户修改了时区。有root权限的话，记得修改系统时区

```bash
date -R # 再次查看时间
sudo cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
date -R # 再次查看时间，确认已经修改为北京时间
sudo hwclock --systohc # 修改硬件CMOS的时间，如果没有这一步的话，后面时间还是不准
```

## 组织逻辑

- 各种shell的dot 文件
- CONF、shareENV、serverENV、localENV分开

### 文件夹结构

```
/home/$USER/
	一堆软链接点文件，指向CONF下的文件，详见下[#软连接]
	init_script/
		系统环境安装脚本
	ENV/
		CONF/
			一堆软链接点文件，指向shareENV、serverENV、localENV下的文件，详见下[#软连接]
		shareENV/     ------- 一个repo，笔记本、服务器共用，每次登录登出时同步
			local/
				bin/
				man/
				...
			script/
			shell_config/
				zsh、bash、所有shell通用的点文件
			app_config/
				tmux、ipython、jupyter通用的点文件
		serverENV/    ------- 一个repo，服务器共用，每次登录登出时同步
			(结构同shareENV/)
			app_config/
				.shadowsocks  : https://github.com/hyliang96/shadowsocks_ips、个人ss账号
			nvim/             : neovim 的镜像及其解压文件夹
			...
		localENV/     ------- 一个文件夹，该服务器自己用的，不同步
			(结构同shareENV/)
		其他共享ENV ------- 一个repo，在我的一些电脑之间共用，每次登录登出时同步
			(结构同shareENV/)
```

### 我设计的点文件的加载逻辑

执行顺序从上到下

| 执行场景 | 自定义共用文件                                               | zsh                                      | bash                                                         |
| -------- | ------------------------------------------------------------ | ---------------------------------------- | ------------------------------------------------------------ |
| 总执行   | .env: 定义环境变量、定义登录登出的文件同步函数；非交互则加载.aliases | .zshenv: **总是触发**; 加载.env          | .bash_env: **登录但非交互 触发**; 加载.env                   |
|          | .aliases: 包括别名和shell函数                                |                                          |                                                              |
| 登录     | .login: git pull共享文件                                     | .zprofile: **登录触发**;加载.login       | .bash_profile: **登录触发**; 加载.bash_env, .login, .bashrc  |
| 交互     | 无                                                           | .zshrc: **交互触发**; 交互则加载.aliases | .bashrc: **非登录但交互 触发**; 若非登录则加载.bash_env, 交互则加载.aliases |
| 登出     | .logout: git push共享文件; 打扫卫生                          | .zlogout: **登出触发**;加载.logout       | .bash_logout: **登出触发**;加载.logout                       |

触发：即系统自动加载此文件的触发条件

加载：即在此文件内，source了别的文件

**特别设置**

bash的非交互需要额外设置

* 登录式的非交互：若bash是默认shell，`ssh <服务器> <命令>`，不执行`~/.bash_env`，**没有找到办法**；只能

  ```bash
  ssh <服务器> '. ~/.bash_env && <命令>’
  ```

* 非登录式的非交互：`bash -c '<命令>'`或`bash 文件`，在.env中添加一行：

  ```bash
  export BASH_ENV=~/.bash_env   # 可登录后，非交互bash加载之
  ```

zsh 在上述情况都会自动加载`~/.zshenv`，无需另外设置

### 软链接

#### `$HOME/`

`$HOME=/home/$USER/ENV/CONF`

`$shareENV=/home/$USER/ENV/shareENV`

`$serverENV=/home/$USER/ENV/serverENV`

| 源文件            | 目标文件                                  |
| ----------------- | ----------------------------------------- |
| `$HOME/`          | `$shareENV/app/`                          |
| .gitignore_global | ../shareENV/app_config/.gitignore_global  |
| .gitconfig        | ../shareENV/app_config/.gitconfig         |
| .ipython          | ../shareENV/app_config/.ipython/          |
| .jupyter          | ../shareENV/app_config/.jupyter/          |
| .tmux.conf        | ../shareENV/app_config/.tmux.conf         |
| .vimrc.bundles    | ../shareENV/app_config/vim/.vimrc.bundles |
| .vim              | ../shareENV/app_config/vim/.vim/          |
| .vimrc            | ../shareENV/app_config/vim/.vimrc         |
| `$HOME/`          | `$serverENV/app/`                         |
| .shadowsocks      | ../serverENV/app_config/.shadowsocks/     |
| `$HOME/`          | `$shareENV/shell_config/`                 |
| .autojump         | ../shareENV/shell_config/.autojump/       |
| `$HOME/`          | `$$ENV/.shell_config/`                    |
| .env              | ../CONF/.shell_config/.env                |

#### `/home/$USER/`
| 源文件          | 目标文件                    |
| --------------- | --------------------------- |
| `/home/$USER/` | `$HOME`                     |
| .bash_env       | ENV/CONF/.bash_env          |
| .bash_history   | ENV/CONF/.bash_history      |
| .cache          | ENV/CONF/.cache/            |
| .env            | ENV/CONF/.shell_config/.env |
| .bash_profile   | ENV/CONF/.bash_profile      |
| .ssh            | ENV/CONF/.ssh/              |
| .zcompdump      | ENV/CONF/.zcompdump         |
| .zshenv         | ENV/CONF/.zshenv            |

# 附录

## shell的状态类型

### 预备知识——shell的状态类型

|        | 交互                                | 非交互                                    |
| ------ | ----------------------------------- | ----------------------------------------- |
| 登录   | ssh登录服务器; 打开iTerm2登录笔记本 | `ssh <服务器> <命令>`                     |
| 非登录 | `zsh或bash`                         | `zsh或bash -c '<命令>'`或`zsh或bash 文件` |

### 判断方法

参考：[千万别混淆 Bash/Zsh 的四种运行模式](https://zhuanlan.zhihu.com/p/47819029)

|            | bash                                                       | zsh                                              |
| ---------- | ---------------------------------------------------------- | ------------------------------------------------ |
| 是否是登录 | `if [[ $0 = -* ]]` 或`if [[ $(shopt login_shell) = *on ]]` | `if [[ -o login ]] `                             |
| 是否交互   | `if [[ $- = *i* ]]`                                        | `if [[ -o interactive ]]` 或 `if [[ $- = *i* ]]` |

在任何bash环境下，以下两种的结果相同

- 文件中写\<判断命令>再`. 文件`
- 直接执行\<判断命令>

**示例**

| 场景 \  返回  \ \<命令>                   | `echo $-` | `echo $0`                | 是否是登录 | 是否交互 |
| ----------------------------------------- | --------- | ------------------------ | ---------- | -------- |
| bash 为默认shell，ssh登录后，执行命令     | himBH     | -bash                    | yes        | yes      |
| ssh登录后，`bash` 后，执行命令            | himBH     | bash                     | no         | yes      |
| `bash  -c '<命令>'`                       | hBc       | bash                     | no         | no       |
| 在当前位置执行`bash 文件`，文件中有<命令> | hB        | <文件相对当前位置的路径> | no         | no       |

| 场景 \  返回  \ \<命令>                  | `echo $-`        | `echo $0`                | 是否是登录 | 是否交互 |
| ---------------------------------------- | ---------------- | ------------------------ | ---------- | -------- |
| zsh为默认shell，ssh登录后，执行命令      | 3569JNRXZghiklms | -zsh                     | yes        | yes      |
| ssh登录后，`zsh` 后，执行命令            | 3569JNRXZghikms  | zsh                      | no         | yes      |
| `zsh  -c '<命令>'`                       | 569X             | zsh                      | no         | no       |
| 在当前位置执行`zsh 文件`，文件中有<命令> | 569X             | <文件相对当前位置的路径> | no         | no       |

