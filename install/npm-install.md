# npm install

ubuntu 环境



## 方法一

#### 1）以 sudo 用户身份运行下面的命令，下载并执行 NodeSource 安装脚本：

```shell
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
```

这个脚本将会添加 NodeSource 的签名 key 到你的系统，创建一个 apt 源文件，安装必备的软件包，并且刷新 apt 缓存。
如果你需要另外的 Node.js 版本，例如`12.x`，将`setup_14.x`修改为`setup_12.x`。

#### 2）NodeSource 源启用成功后，安装 Node.js 和 npm:

```shell
sudo apt install -y nodejs
```

nodejs 软件包同时包含`node`和`npm`二进制包。

#### 3）验证 Node.js 和 npm 是否正确安装。打印它们的版本号：

```shell
node --version
v14.2.1
```

```shell
npm --version
6.14.12
```



## 方法二



ubuntu使用apt install nodejs 安装后发现nodejs版本是4.*，需要更新。

#### 1.先安装npm

```shell
apt install npm
npm -g install npm //升级npm
```

#### 2.安装n

```shell
npm -g install n
```

#### 3.通过n来管理node的版本

```shell
n stable	//安装最新的稳定版
ln -s /usr/local/n/versions/node/10.16.0/bin/node /usr/bin/node -f //强制将最新的版本软连到usr/bin下
```

> n是一个npm的包，专门用来管理node的版本

