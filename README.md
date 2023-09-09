# hops_singularity


## 一些建议

推荐不要直接在主机终端上进行操作，而是使用MobaXterm软件，通过ssh连接到服务器，因为MobaXterm不需要再配置x11图形转发功能了。即使是虚拟机或者windows下的linux子系统，也推荐在其上安装好ssh服务后，再用MobaXterm进行连接。

注：以下操作是在centos7系统上进行的

## 构建新的Singularity容器（可忽略）

如果你打算在Singularity容器上安装该软件，方便移植到其他服务器上使用，可以参考Singularity的文档：https://docs.sylabs.io/guides/3.7/user-guide/index.html

按照下面的安装流程打包的的Singularity镜像链接如下，关于使用可参考本文最下面的部分——“在其他主机上使用该镜像”。也建议使用MobaXterm连接远程服务器，再进入该镜像进行使用。远程服务器本身仍需要开启x11图形转发功能，该功能的开启见于本文的“安装ssh和开启x11功能”部分。

Google driver下载链接：https://drive.google.com/file/d/1fBp3TmriXfn-WQ-QcB1p2ueYeeIN_aZk/view?usp=sharing

百度云盘下载链接：链接：https://pan.baidu.com/s/1QOuegr9dD_thrwEdTJWt3w  提取码：vm2w


```
sudo singularity -d build --sandbox centos/ docker://centos:7 # 构建Centos7
sudo singularity shell --writable centos/ # 进入Centos7
```

## 更新和安装一些软件

```
yum update -y
yum install vim wget make
yum install gcc gcc-c++ gcc-gfortran kernel-devel
```

## 创建并切换到安装的基目录

```
export ASTROSOFT=/HOPS_SOFT # 定义环境变量ASTROSOFT为基目录

mkdir $ASTROSOFT && cd $ASTROSOFT # 创建并切换到基目录

export PATH=$PATH:$ASTROSOFT/bin # 把$ASTROSOFT/bin加入到PATH搜索路径
export C_INCLUDE_PATH=$ASTROSOFT/include:$C_INCLUDE_PATH  # 加入gcc的include搜索路径
export CPLUS_INCLUDE_PATH=$ASTROSOFT/include:$CPLUS_INCLUDE_PATH # 加入g++的include搜索路径

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ASTROSOFT/lib # 把$ASTROSOFT/lib加入到LD_LIBRARY_PATH路径
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib:/usr/lib64 # 把/usr/lib和/usr/lib64加入到LD_LIBRARY_PATH路径
```

## 安装fftw

```
wget http://www.fftw.org/fftw-3.3.8.tar.gz
tar -xvf fftw-3.3.8.tar.gz && cd fftw-3.3.8
./configure --prefix=$ASTROSOFT --enable-shared
make
make install
make clean

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ASTROSOFT/fftw-3.3.8/lib # 把fftw的库目录加入LD_LIBRARY_PATH这个搜索路径
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:$ASTROSOFT/lib/pkgconfig # 把pkgconfig目录加入PKG_CONFIG_PATH搜索路径，使得目录下的fftw.pc文件能够被搜索到
```

## 安装ssh和开启x11功能

```
yum install openssh-server -y
yum install -y xorg-x11-xauth libXt-devel libXext-devel

vim /etc/ssh/sshd_config # 打开配置文件，修改一些参数如下

#AllowAgentForwarding yes
AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost no   #网上很多说明这里保持默认不需要修改
#PermitTTY yes
#PrintMotd yes
#PrintLastLog yes
#TCPKeepAlive yes

PasswordAuthentication  yes  #启用口令认证方式，默认是yes，如果是no的话可以改成yes
```

## 安装pgplot的功能

```
cd $ASTROSOFT //切回安装基目录
wget ftp://ftp.astro.caltech.edu/pub/pgplot/pgplot5.2.tar.gz
tar -zxvf pgplot5.2.tar.gz
mv pgplot pgplot_source
mkdir pgplot
cp pgplot_source/drivers.list pgplot/
cd pgplot

vim drivers.list
# 编辑drivers.list文档，把下面提到的/NULL,/PS等前面的!去掉，即取消注释：

the null device (/NULL)
PostScript printers (/PS, /VPS, /CPS, and /VCPS),
Tektronix terminals (/TEK, /XTERM, and possibly other variants),
and, if the X window system is available on the target, the X window drivers (/XWINDOW, /XSERVE).
You may also wish to include drivers for GIF files (/GIF, /VGIF) or some of the other printers.


../pgplot_source/makemake ../pgplot_source linux g77_gcc

vim makefile
# 编辑makefile文档，将其中第一页的FCOMPL=g77修改为FCOMPL=gfortran

make # 对pgplot编译
make clean
make cpg  #编译安装

export PGPLOT_DIR=$ASTROSOFT/pgplot
export PATH=$PATH:$PGPLOT_DIR # 把PGPLOT_DIR加入到搜索路径
```

## 安装ghostscript(gs)

```
cd $ASTROSOFT //切回安装基目录
wget  https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs925/ghostscript-9.25.tar.gz # 下载不了可以尝试下其他途径，比如先从浏览器下载，再复制到这里
tar zxvf ghostscript-9.25.tar.gz
cd ghostscript-9.25
 ./configure --prefix=$ASTROSOFT
make
make install
make clean
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ASTROSOFT/share/ghostscript/9.25/lib # 把gs的lib加入到LD_LIBRARY_PATH路径
```

## 安装hops

```
cd $ASTROSOFT # 切回安装基目录
yum install lftp
lftpget ftp://gemini.haystack.mit.edu/pub/hops/hops-3.24-3753.tar.gz
lftpget ftp://gemini.haystack.mit.edu/pub/hops/hops-3.24-README.txt
tar zxf hops-3.24-3753.tar.gz
mkdir bld-3.24 && cd bld-3.24
../hops-3.24/configure --prefix=$ASTROSOFT
make install
cp ./hops.bash $ASTROSOFT/bin/hops.bash
```

## 批量加载环境变量
```
用export加入的环境变量都是临时的，重新登录时会消失，所以可以建立.sh文件，重新登录后批量添加，可以命名为HOPS_PATH.sh，放到基目录中，内容如下：

#-----------------------------------------
# Base directory
export ASTROSOFT=/HOPS_SOFT


# PGPLOT
export PGPLOT_DIR=$ASTROSOFT/pgplot

# LD_LIBRARY_PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib:/usr/lib64
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ASTROSOFT/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ASTROSOFT/fftw-3.3.8/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ASTROSOFT/share/ghostscript/9.25/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$PGPLOT_DIR


# PKG_CONFIG_PATH
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:$ASTROSOFT/lib/pkgconfig

# PATH
export PATH=$PATH:$ASTROSOFT/bin:$PGPLOT_DIR


# INCLUDE_PATH
export C_INCLUDE_PATH=$ASTROSOFT/include:$C_INCLUDE_PATH
export CPLUS_INCLUDE_PATH=$ASTROSOFT/include:$CPLUS_INCLUDE_PATH 
#-----------------------------------------


sh /HOPS_SOFT/HOPS_PATH.sh  # 执行sh文件中的内容
```


## 测试hops

```
sh /HOPS_SOFT/HOPS_PATH.sh # 重新登录后再使用该命令，刚装测试时，不需要该命令
source /HOPS_SOFT/bin/hops.bash # 加载hops的配置
fourfit # 输出后会出现fourfit的帮助说明

注：hops的源码文件在/HOPS_SOFT/hops-3.24，编译后的bin文件在/HOPS_SOFT/bin中，include文件在/HOPS_SOFT/include/hops中。
```


## 打包成singularity镜像（可忽略）

```
sudo singularity build centos-singularity.simg centos/
```

## 在其他主机上使用该镜像（可忽略）
```
singularity shell centos-singularity.simg
sh /HOPS_SOFT/HOPS_PATH.sh
source /HOPS_SOFT/bin/hops.bash
fourfit # 输出后会出现fourfit的帮助说明
```