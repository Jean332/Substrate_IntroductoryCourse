# substrate入门_01

<a name="UBv3x"></a>
# 在M1上编译Substrate
<a name="UL4PS"></a>
## 一、环境配置三部曲

1. 安装homebrew
1. 安装依赖库
1. 安装rust编译链



<a name="SNRDO"></a>
### 1. 安装homebrew
参考：[Apple MacBookPro M1 安装 brew（国内源）](https://blog.csdn.net/tanshizhen119/article/details/111244245)
<a name="v1yFB"></a>
#### 1）准备目录并授权
M1之后，不能放到 /usr/local/bin目录下了。要放到/opt目录下 所以新建/opt目录
```
sudo mkdir -p /opt/homebrew
```


```
sudo chown -R $(whoami):staff /opt/homebrew  //将homebrew文件授权给当前用户
```

<br />也可以用下面这条直接授权所有<br />`cd /opt && sudo chmod -R 777 ./ ` 将文件夹授权给所有用户所有权限<br />

<a name="9cwcB"></a>
#### 2）下载安装脚本（国内源）
`cd /opt && curl -fsSL `[`https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh`](https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)` > Homebrew.sh`<br />

<a name="A1rqr"></a>
#### 3）打开脚本修改安装目录
`vim /opt/Homebrew.sh `<br />将 HOMEBREW_PREFIX的值修改成 HOMEBREW_PREFIX="/opt" 并保存

<a name="AvZGk"></a>
#### 4）执行安装命令
`cd /opt && /bin/zsh Homebrew.sh `<br />

<a name="OZRW2"></a>
#### 5） 配置环境变量
`sudo vim /etc/profile` <br />在最后一行添加 `export PATH=$PATH:/opt/Homebrew/bin`，保存、退出<br />
<br />`source /etc/profile `  让环境变量生效。<br />

<a name="bYL2J"></a>
#### 6)  执行brew -v
`% brew -v`<br />Homebrew 2.7.0<br />Homebrew/homebrew-core (git revision 2d1707; last commit 2020-12-26)<br />Homebrew/homebrew-cask (git revision 8cc958; last commit 2020-12-26)<br />

<a name="s5tQn"></a>
### 2. 安装依赖库
参考：[M1编译substrate随笔-知乎](https://zhuanlan.zhihu.com/p/337224781)
<a name="pvVdb"></a>
#### 1） 安装各依赖库
```
brew install -s cmake
brew install -s gcc
brew install -s llvm
brew install protobuf
```


<a name="PV81s"></a>
#### 2） 配置llvm config
`sudo vim /etc/profile` <br />在最后一行添加 `export PATH="/opt/homebrew/opt/llvm/bin`，保存、退出<br />`source /etc/profile `  让环境变量生效。<br />

<a name="TsAEy"></a>
#### 3） 直接编译会报错，修改依赖库
直接编译Substrate会有三个相关的依赖出现问题，分别是`ring`, `fs-swap`和`rust-rocksdb`

- `ring`和`fs-swap`目前都已对M1支持了，只需要`cargo update`就可以了：
```
cargo update -p fs-swap
cargo update -p ring
```


- 在`rust-rocksdb`对M1支持的pull request被merge之前，通过patch的方法来解决。
```
cd opt/homebrew/opt   
git clone https://github.com/hdevalence/rust-rocksdb.git
cd rust-rocksdb
git submodule update --init --recursive
```


修改cargo config
```
vim ~/.cargo/config
```
添加：
```
paths = ["/opt/homebrew/opt/rust-rocksdb/"]
```
保存，退出后，`source ~/.cargo/env`<br />

<a name="adCm1"></a>
### 3. 安装rust编译链
1) 安装rust环境，确保有`rustup`和`cargo`命令 （以下选择nightly版本）
```
RUSTUP_UPDATE_ROOT=https://dev-static.rust-lang.org/rustup sh <(curl --proto '=https' --tlsv1.2 -sSf https://dev-static.rust-lang.org/rustup/rustup-init.sh) --default-toolchain nightly
```

<br />2）安装 nightly 编译链<br />`rustup update nightly `<br />
<br />3） 对 nightly 编译链添加 wasm 编译target<br />`rustup target add wasm32-unknown-unknown --toolchain nightly`
> 实际上自rust stable 1.38之后 wasm 的这个target就已经可以提供给stable了，但是由于substrate改变了其编译wasm的方式，是在substrate的代码中指定了使用nightly编译wasm，因此这里如果只给stable添加`wasm32-unknown-unknown`是没有用的，必须先提供nightly编译链，再对nightly添加wasm的target。
> 参考：[Substrate 入门 - 环境配置与编译 -（一）](https://zhuanlan.zhihu.com/p/94624311)



<a name="6jJjv"></a>
## 二、编译
参考：[使用Substrate搭建你的第一条区块链](https://zhuanlan.zhihu.com/p/67580341)
<a name="SrvWe"></a>
### 编译相关指令
<a name="94lef"></a>
#### 1）拉取源码
`git clone https://github.com/paritytech/substrate.git`<br />

<a name="yWcSu"></a>
#### 2）初始化WASM的构建环境
`./scripts/init.sh`<br />

<a name="cNYeq"></a>
#### 3）编译rust代码为WASM
`./scripts/build.sh`<br />

<a name="J0qTR"></a>
#### 4）生成可执行程序
`cargo build --release`<br />

<a name="62fqk"></a>
#### 5）清空节点数据库
`./target/release/node-template purge-chain --dev`<br />

<a name="pEhQ8"></a>
#### 6）启动节点程序
`./target/release/node-template --dev`<br />
<br />`--dev`表示我们准备启动一个本地开发链，会自动初始化开发环境所需的数据和配置。


---


<br />

<a name="2rRxM"></a>
### 其他知识点
<a name="OUX13"></a>
#### [1]. chown 命令使用
`sudo chown -R $(whoami) /usr/local/lib/node_modules/`<br />作用：<br />chown将指定文件的拥有者改为指定的用户或组。系统管理员经常使用chown命令，在将文件拷贝到另一个用户的名录下之后，让用户拥有使用该文件的权限。<br />
<br />语法：<br />chown  [选项]  [所有者][:用户组]  文件或者文件夹<br />chown：change owner 的缩写<br />-R ：--recursive 的缩写，递归处理，将指定目录下的所有文件及子目录一并处理<br />whoami ：who am i的缩写，打印出当前的用户<br />-v ：--verbose 显示指令执行过程

示例：<br />`cd /opt && sudo chmod -R 777 ./   将当前目录授权给所有用户所有权限`<br />chmod能改变权限，-R是目录下所有文件，777就是高权限（读、写、执行）；<br />
<br />
<br />

<a name="JYvMU"></a>
#### 
