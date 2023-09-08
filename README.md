# hops_singularity

## 构建新的Singularity容器

singularity -d build --sandbox centos/ docker://centos:7 # 构建Centos7

singularity shell --writable centos/ # 进入Centos7

## 更新和安装一些软件

yum update
yum install vim wget make
yum install gcc gcc-c++ gcc-gfortran kernel-devel


## 切换到home目录创建apps文件夹

mkdir ~/apps && cd ~/apps

## 安装fftw

mkdir fftw
cd fftw/
wget http://www.fftw.org/fftw-3.3.8.tar.gz
tar -xvf fftw-3.3.8.tar.gz
cd fftw-3.3.8
./configure --enable-shared
make
make install
make clean


vim ~/.bashrc # 打开.bashrc文件
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig/ # 在.bashrc文件末尾写入该行
source ~/.bashrc # 使得环境变量生效

## 安装x11功能

yum install -y xorg-x11-xauth libXt-devel libXext-devel


vim /etc/ssh/sshd_config # 打开配置文件，修改如下

#AllowAgentForwarding yes
AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost no   //网上很多说明这里保持默认不需要修改
#PermitTTY yes
#PrintMotd yes
#PrintLastLog yes
#TCPKeepAlive yes

## 安装pgplot的功能


cd /usr/local/src/
wget ftp://ftp.astro.caltech.edu/pub/pgplot/pgplot5.2.tar.gz
gunzip -c pgplot5.2.tar.gz | tar xvof -
mkdir /usr/local/pgplot
cd /usr/local/pgplot
cp /usr/local/src/pgplot/drivers.list .
vim drivers.list

cd /usr/local/pgplot
/usr/local/src/pgplot/makemake /usr/local/src/pgplot linux g77_gcc

将其中的FCOMPL=g77修改为FCOMPL=gfortran
make 对pgplot编译
make cpg 编译安装


## 安装gs

cd ~/apps
mkdir gs
cd gs
wget  https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs925/ghostscript-9.25.tar.gz
(下载不了可以尝试下其他的)
tar zxvf ghostscript-9.25.tar.gz
cd ghostscript-9.25
 ./configure
make
make install


## 安装hops

cd ~/apps
mkdir hops
tar zxf hops-3.24-3753.tar.gz
mkdir bld-3.24
cd bld-3.24
../hops-3.24/configure
make install


## 构建容器

singularity build centos-vim.simg centos/
singularity shell centos-vim.simg
Singularity> vim
