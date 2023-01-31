# Ubuntu 搭建 CPP 编译环境

## 一、安装必要的环境包

1. 环境设置代理

``` shell
# 在 ~ 路径下创建一个 run_proxy.sh 文件 执行一次 或添加加到 ~/.bashrc 自动执行
vim run_proxy.sh
#!/bin/bash
export hostip=$(cat /etc/resolv.conf | grep -oP '(?<=nameserver\ ).*')
export https_proxy="http://${hostip}:14424"
export http_proxy="http://${hostip}:14424"
export all_proxy="socks5://${hostip}:14424"
export ALL_PROXY="socks5://${hostip}:14424"
echo ${hostip}
echo ${ALL_PROXY}

```


2. 安装 g++、gcc

``` shell

sudo apt update
sudo apt upgrade -y
sudo apt install build-essential -y

# gcc g++ 安装路径：
# 1. include 路径: 

```

3. 安装 CMake 

``` shell
# 1. 直接安装，不推荐，版本太低
sudo apt install cmake 

# 2. 官网下载后安装
# 解包 源代码安装
tar -zxv -f cmake-3.17.1.tar.gz
# 进入 [cmake] 文件夹 执行安装即可
./bootstrap
make
sudo make install

# 二进制安装 下载 cmake-3.25.1-linux-x86_64.sh

# 路径映射 ubuntu 安装位置一般在 /usr/local/bin 下
# 创建映射
ln -s /mnt/b/Learn/UbuntuCPP/softAll/cmake-3.25.1-linux-x86_64 /usr/local/bin/cmake_ln

# 环境比变量设置 添加 /usr/local/bin/cmake_ln/bin
cd /etc/profile.d
vim cmakePaht.sh
# 添加如下文本，保存
#!/bin/bash
export PATH=/usr/local/bin/cmake_ln/bin:$PATH

# 立即生效
source profile
# 验证、查看
cmake --version

```

## 二、第三方包安装

gtest、glog、gflags 和 absl 库安装

``` shell
# gtest、glog和gflags 直接安装即可
apt install libgtest-dev
apt install libgoogle-glog-dev

# absl 库安装
# 拉取源码 创建 build 和 ablsInstall 文件夹
cd build 
cmake .. -DCMAKE_INSTALL_PREFIX=../ablsInstall
make && make install

# 复制头文件
cp -R ../ablsInstall/include/absl/ /usr/include/

# 汇集库文件并复制
find ../ablsInstall -name "*.o" | xargs ar cr libabsl.a
cp ../ablsInstall/libabsl.a /usr/lib

```

