---
title: Hexo + GitHub 搭建个人博客
date: 2019-03-22 18:15:39
tags:
- windows
- git
- github
- hexo
- node.js
- 网站
categories:
- 教程
---

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-blog1-1-7f8236.jpg" alt="blog1-1" width="50%" />

对于程序员来说，搭建和维护一个个人博客十分必要​，写博客既是对所学知识的整理和总结，同时也能向他人展现自己的学习成果。这篇教程就是基于 Hexo 和 GitHub 来搭建属于自己的个人博客，简单快捷，对小白也十分友好。

<!-- more -->

***

# 前期准备

本次Hexo博客搭建环境

> Windows 10  1803
> node-v10.14.1
> git version 2.21.0.windows.1
> hexo-v3.8.0

安装各种依赖环境还是比较简单的，基本上就是从各自的官网下载进行安装，十分方便。

## 安装 Node.js

Hexo 的运行需要 Node.js 的支持，所以我们需要首先安装好 Node.js 。打开[Node.js官网](https://nodejs.org/en/)就能很明显地看到下载提示，点击左边的按钮进行下载即可。

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-blog1-2-a794b6.jpg" alt="blog1-2" width="70%" />

**下载完点击安装程序进行安装，无需修改安装路径的话无脑点击下一步即可。**
当安装完成后打开命令行工具（cmd/powershell)，输入`node -v`，如果输出如下信息，安装即为成功。

```bash
$ node -v
v10.14.1
```

## 安装 Git

我们需要从 GitHub 上下载主题文件，最重要的是我们需要将本地的博客部署到可供外部访问的网页上去，我们借助的是 GitHub ，这些都离不开 Git 的支持。同样的我们到 [Git官网下载页](https://git-scm.com/downloads)下载即可。这里我们选择 Windows 64 位最新版本的 Git for Windows 进行安装。

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-blog1-3-e1fa43.jpg" alt="blog1-3" width="70%" />

**安装和 Node.js 差不多，不做修改的话一直点下一步即可。**
安装完成后同样打开命令行工具（cmd/powershell)，输入`git --version`，如果输出如下信息，安装即为成功。

```bash
$ git --version
git version 2.21.0.windows.1
```

## 安装 Hexo

安装完 Git 后，我们的操作就可以在 Git Bash 中进行**(当然在其他命令行工具中也行)**，在资源管理的任意目录下右键鼠标可以看到 Git Bash Here 选项。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-blog1-4-4e20bf.jpg)

单击该选项，我们便进入了 Git 的命令行工具界面如下(同样打开Windows自带的cmd或是其他命令行工具都可以)，之后 Hexo 的安装、博客的部署等操作都在这个界面进行。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-blog1-5-85707f.jpg)

由于国内的 npm 访问外网下载速度较慢，我们可以将 npm 源更换为淘宝的镜像（当然如果你觉得你的下载速度较快的话，也可以选择不进行更换），在 Git Bash 中输入以下指令

```bash
 $ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

**请注意如果进行了上述操作，那么之后的下载等操作需要使用到 npm 的地方你都应该更换为 cnpm 指令，这样才能起到加速作用，如果未进行更换，则使用 npm 即可。**

接下来就是重头戏**安装 Hexo **了。输入如下指令即可安装完成 Hexo 的安装，十分方便。

```bash
$ npm install hexo-cli -g
```

在安装过程中可能会出现`WARN`提示，这是因为 Hexo 的某些内容不支持 Windows 系统，我们无需理会 ~~(程序员只关心 Error，不关心 Warning🙃)~~。

安装完成后，我们输入 `hexo -v` 命令，如果显示如下信息，则安装成功。

```bash
hexo-cli: 1.1.0
os: Windows_NT 10.0.17134 win32 x64
http_parser: 2.8.0
node: 10.14.1
v8: 6.8.275.32-node.36
uv: 1.23.2
...
```

底下还有许多包的版本信息，可以看到 Hexo 的依赖项还是比较多的。

***

# 初始化博客

在我们想要存放博客文件的盘下进入 Git Bash ，首先我们要新建一个文件夹用来存放我们写的博客和其它相关文件，在命令行中输入`mkdir Blog` 命令，便可新建一个名称为 Blog 的文件夹（文件夹名可自取）。接下来进入刚创建的文件夹，可使用 `cd Blog` 命令，或是进入该文件夹后在空白处单击右键，再点击 Git Bash Here （还是多练习以下命令行的简单命令为好）。

进入 Blog 文件夹后，输入 `hexo init` 命令，即可完成该文件夹的初始化，在此期间，Hexo 会自动克隆和创建一系列相关文件，在结束后为得到如下反馈：

```bash
$ hexo init
INFO  Cloning hexo-starter to F:\desktop\Blog
Cloning into 'F:\desktop\Blog'...
...
Unpacking objects: 100% (71/71), done.
Submodule 'themes/landscape' (https://github.com/hexojs/hexo-theme-landscape.git) registered for path 'themes/landscape'
Cloning into 'F:/desktop/Blog/themes/landscape'...
...
Receiving objects: 100% (890/890), 2.56 MiB | 362.00 KiB/s, done.
Resolving deltas: 100% (465/465), done.
Submodule path 'themes/landscape': checked out '73a23c51f8487cfcd7c6deec96ccc7543960d350'
INFO  Install dependencies
...
INFO  Start blogging with Hexo!
```
当看到 `Start blogging with Hexo` 就知道我们已经初始化成功了，Blog 文件夹中会创建如下文件：

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-blog1-6-1d6269.jpg)

***

# 生成博客页面

在 \source\\_posts\ 文件夹中就保存着我们写的所有博客，当前我们并没有写任何博客，但是 Hexo 为自动生成一个 hello-world.md 文件，我们可以以此为例，看看我们的博客网页是啥样啦。

在 Git Bash 中输入 `hexo g` 命令（是 `hexo generate` 的简写），等待 Hexo 自动生成网页，得到如下反馈则生成成功：

```bash
$ hexo g
INFO  Start processing
INFO  Files loaded in 192 ms
INFO  Generated: index.html
...
INFO  28 files generated in 459 ms
```

接下来我们需要开启开启本地服务器，输入 `hexo s` 命令（是 `hexo server` 的简写），输出

```bash
$ hexo s
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
```

我们就知道了网页运行在 http://localhost:4000 上，我们在浏览器中输入该地址便能进入我们创建的博客网页啦。

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-blog1-7-d44e55.jpg" alt="blog1-7" width="70%" />

***

# 美化博客页面

## 下载主题

可以看到打开的博客页面就是 Hexo 默认的页面，所以并不是非常得好看，我们可以自行选择更换，在 GitHub 上搜索 Hexo 主题还是有非常多的项目的。我在这里选择了目前使用人数比较多的 Next 主题进行演示。Next 主题的 Github 地址是 [theme-next/hexotheme-next](https://github.com/theme-next/hexo-theme-next)。回到我们存放博客文件的根目录，输入如下指令，将该仓库克隆到本地。

```bash
$ cd Blog
$ git clone https://github.com/theme-next/hexo-theme-next themes/next
```

这样我们就将 Next 主题克隆到 themes/next 文件夹下啦。

## 更改配置文件

使用编辑器打开根目录下的 `_config.yml` 文件，这个文件保存的就是 Hexo 的诸多配置项，我们可以对其进行个性化修改。在文件中找到 `theme` 配置选项，如下所示：

```yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: landscape
```

可以看到现在的主题是默认的 `landscape` ，我们将其改为 `next` 即可。

```yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

再在 Git Bash 中依次输入下面三条指令（发布三连），**请注意所有的 hexo 指令都是在根目录下进行的，如果当前目录不是根目录，必须先切换到根目录再执行命令**：

```bash
$ hexo clean   #清除之前生成的网站文件
$ hexo g       #生成当前网站文件
$ hexo s       #开启服务器
```

再次打开 http://localhost:4000 ，我们就能看到更换主题后的网站了，非常便捷。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-blog1-8-14675b.jpg)

这就是我们新生成的页面了，是不是看上去精致多了呢。

***

# 部署博客

在完成了上述步骤之后，我们就可以在自己的电脑上打开博客网页了，但是怎么才能让别人也能访问到我们的网页呢？这就需要我们部署我们的博客网站了。幸运的是，GitHub 能为我们免费提供这一服务，那就是 GitHub Page ，我们需要做的就是在 GitHub 上新建一个名为 `<username>.github.io` (在`<username>`处填入你的用户名） 的仓库即可。

## 修改配置文件

打开根目录下的 `_config.yml` 文件，找到 `deploy` 选项，如下所示：

```bash
deploy:
  type:
```

将其修改为

```bash
deploy:
  type: git
  repo: https://github.com/<username>/<username>.github.io.git
  branch: master
```

在其中的 `<username>` 处填入你的 GitHub 用户名即可。保存配置文件并退出。

接下来在 Git Bash 中输入下面三条指令（部署三连）：

```bash
$ hexo clean
$ hexo g
$ hexo d
```

至此，我们就已经完成了个人博客网站的部署，在浏览器中输入你的地址 ：`<username>.github.io` ，就能看到我们的个人网站啦！
