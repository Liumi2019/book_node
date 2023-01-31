## repo 下载源码

### repo 下载
curl -sSL  'https://gerrit-googlesource.proxy.ustclug.org/git-repo/+/master/repo?format=TEXT' |base64 -d > repo

清华源
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo

下载 repo 并复制到 /usr/bin/ 下 
cp repo /usr/bin/
chmod a+x /usr/bin/repo

repo 代理设置 清华源
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'

### 初始化
#### 1. 使用 repo 初始化 代码太大 190G 建议使用第二种
清华源:
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest

#### 2. 使用初始化包
下载初始化包:
wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar 
tar xf aosp-latest.tar
解压得到的 AOSP 工程目录
cd AOSP
这时 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录
正常同步一遍即可得到完整目录
repo sync -j4
仅checkout代码
repo sync -l -j4


### 同步代码库
repo sync -j4

### 可能的错误
error 403 forbidden
在wget前加上参数 -U，代表设置User Agent 

## 代码编译


