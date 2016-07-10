---
layout: post
title:  "node-sass 安装卡在 node scripts/install.js 解决办法"
tags: node js sass
comments: true
share: true
---

<p class="lead">一个 node 项目里用到了 `node-sass@3.8.0` ，安装的时候在这一步：

```bash
> node-sass@3.8.0 install path/to/project/node_modules/node-sass
> node scripts/install.js
```

一直卡住，至少有半个小时没反应，自己的 Mac 和腾讯云的服务器上都是这样

</p>

去看 `node_modules/node-sass/scripts/install.js` 的[代码](https://github.com/sass/node-sass/blob/master/scripts/install.js#L101)，发现是要在 GitHub 上下载编译好的 `node-sass` 二进制包 ，去看 node-sass 的 [Release](https://github.com/sass/node-sass/releases/tag/v3.8.0)，平均在 2.5 MB 左右

于是明了了，GitHub 在国内访问本来就不稳定，然后还是用 [request](https://github.com/request/request) 去访问，就更慢了。看了一下，半个小时左右才下了 500 K

正好又在 [这里](https://github.com/sass/node-sass/blob/master/lib/extensions.js#L229) 的 `getBinaryPath()` 可以设置二进制的位置。在这之前还要先知道自己的系统需要的版本。

用这行命令：

```bash
node -p "[process.platform, process.arch, process.versions.modules].join('-')"
```

复制输出的结果，去 [Release 列表](https://github.com/sass/node-sass/releases) 找到对应的版本，Ctrl+F 粘贴，找到那个文件，下载（必要的时候挂代理，浏览器下载通常都比 node 下载更快更稳定），然后文件存到一个稳定的路径，并复制路径（比如 `~/.node/.npm/node-sass/darwin-x64-48_binding.node`）

在 `~/.npmrc` 下面新增一行，新增 `sass_binary_path` 项并填入刚才的路径，比如

```
sass_binary_path=/home/ubuntu/.npm/node-sass/darwin-x64-48_binding.node
```

最后再去项目目录下：

```
rm -rf node_modules/ && npm i
```

即可
