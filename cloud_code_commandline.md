# 云代码命令行工具使用详解

## 介绍

云代码命令行工具是用来管理、部署云代码项目的命令行工具，称之为`avoscloud-code`项目。通过它，你可以部署、发布、回滚云代码，并且可以对同一个云代码项目做多应用管理，还可以查看云代码日志，批量上传文件到 LeanCloud 平台上等。

## 安装和使用

首先，你需要安装 [Node.js](http://nodejs.org/) 环境以及 [npm](https://npmjs.org/) 包管理工具，在 ubuntu 上的 terminal（终端）可以简单地执行

```
sudo apt-get install nodejs
```

(apt 提供的版本的 nodejs 版本可能太老旧，推荐从[官方网站](http://nodejs.org/)下载安装)。

在 Mac OSX 上，可以通过 [MacPort](http://www.macports.org/) 或者 [Homebrew](http://brew.sh/) 安装，terminal（终端）执行下列命令：

```
sudo port install nodejs
```

或者

```
brew install nodejs
```

在其他操作系统上，你也可以从 nodejs 官方网站下载源码或者二进制安装，自己安装。

接下来，通过执行下列命令安装 avoscloud 命令行工具：

```
sudo npm install -g avoscloud-code
```

加上 `-g` 表示全局安装。**以后升级也请执行此命令**。

我们更推荐从 [cnpm](http://cnpmjs.org/) 仓库安装，速度理论上更快，请执行下列两个命令：

```
sudo npm install -g cnpm --registry=http://r.cnpmjs.org
sudo cnpm install -g avoscloud-code
```

如果从 npm 安装失败，也可以直接从 Github 源码安装：

```
sudo npm install -g  git+https://github.com/avoscloud/avoscloud-code-command
```

安装成功之后，直接在 terminal（终端）运行 `avoscloud -h`，输出帮助信息：

```
$ avoscloud -h
Usage: avoscloud [选项] <命令>

  有效的命令列表包括:
    deploy: 部署云代码到 LeanCloud 平台开发环境
    undeploy: 从 LeanCloud 平台清除云代码部署，包括生产环境和开发环境
    status: 查询当前部署状态
    search <keyword>: 根据关键字查询开发文档
    publish: 发布开发环境代码到生产环境
    new: 创建云代码项目
    logs: 查看云代码日志
    clear: 清除本地状态，在输入 app id 或者 master key 错误的情况下使用
    upload <file-or-directory>: 导入文件到 LeanCloud 平台，如果是目录，则会将该目录下的文件递归导入。
    app [list]:  显示当前应用，deploy、status 等命令运行在当前应用上，如果加上 list ，则显示所有的应用信息。
    checkout <app>: 切换到一个应用，deploy、status 等命令将运行在该应用上。
    add <app>: 添加一个应用。
    rm <app>: 移除一个应用。

  Options:

    -h, --help                 output usage information
    -V, --version              output the version number
    -f, --filepath <path>      本地云代码项目根路径，默认是当前目录。
    -g, --git                  使用定义在管理平台的 git 仓库或者 -u 指定的 git 仓库部署云代码，默认使用本地代码部署。
    -p, --project <app>        命令运行在指定应用上，默认运行在当前应用或者 origin 应用上。
    -l, --local                使用本地代码部署云代码，该选项是默认选中。
    -o, --log <log>            本次部署的提交日志，仅对从本地部署有效。
    -n, --lines <lines>        查看多少行最新的云代码日志，默认 10 行。
    -t, --tailf                自动刷新云代码日志，结合 logs 命令使用。
    -r, --revision <revision>  git 的版本号，仅对从 git 仓库部署有效。
```

可以通过 `-V` 选项查看版本：


```
$ avoscloud -V
0.5.0
```

后面我们都假定 `$ avoscloud` 开始的都表示在终端里执行这个命令。

## Bash Completion

下载 [leancloud.cnpletion.sh](https://github.com/avoscloud/avoscloud-code-command/blob/master/leancloud.cnpletion.sh) 保存到某个目录，例如通常保存为 `~/.leancloud.cnpletion.sh`，然后在 `.bashrc` 或者 `.bash_profile` 文件中添加：

```
source ~/.leancloud.cnpletion.sh
```

重启终端 bash，或者重新加载 profile 文件，就可以让 avoscloud 命令拥有自动提示和完成功能(tab 按键提示)。

**Mac 上建议通过 homebrew 安装 bash-completion**


## 创建项目

在安装命令行工具后，除了从网站的云代码菜单下载新应用项目框架之外，你也可以通过 `new` 命令来创建一个新的云代码项目：

```
$ avoscloud new
```

它会要求你输入应用的 Id 和 Master Key（都可以从开发者平台的应用设置里的应用 key 菜单里找到这些信息），

第三步选择应用类型包括两类：

* Standard 标准版：输入字符 's' 或者字符串 "standard" 即可创建，标准的云代码项目，不包括 web 主机功能（网站托管）。
* Web Hosting 版：输入字符 'w' 或者字符串 'web' 即可创建，包含 web 主机功能的云代码项目。

输入正确后，会自动创建一个新的云代码项目框架：

```
Creating project...
  new_app/
  new_app/README.md
  new_app/config/
  new_app/config/global.json
  new_app/public/
  new_app/public/index.html
  new_app/cloud/
  new_app/cloud/main.js
Project created!
```

进入 `new_app` 目录就可以看到新建立的项目。

## 本地调试

`cd new_app` 进入该目录，并执行 `avoscloud` 命令，可以在本机调试云代码。

* 通过浏览器打开 [http://localhost:3000/avos](http://localhost:3000/avos)，进入云代码函数和 Class Hook 函数调试界面。
* 通过浏览器打开 [http://localhost:3000](http://localhost:3000)，可以看到 public 目录的 index.html 页面的内容。

avoscloud 命令还支持代码的热加载，修改代码后无需重新启动，就可以看到代码的最新结果（包括 web 功能）。

更多关于云代码开发，请参考 [云代码开发指南](https://leancloud.cn/docs/cloud_code_guide.html) 。


## 部署

### 本地推送部署

在你开发和本地测试云代码项目通过后，你可以直接将本地源码推送到 LeanCloud 云代码平台运行，只要执行 `deploy` 命令：

```
$ avoscloud deploy
```

请注意，这个命令将部署本地源码到远程平台的开发环境，无条件覆盖原来开发环境的版本（无论是从 Git 仓库部署或者还是本地部署）。

如果部署成功，会打印部署后的状态：

```
[INFO]: Cloud Code Project Home Directory: /Users/dennis/programming/avos/new_app/
Compress cloud code files...
Wrote compressed file /var/folders/90/xwqqy61d6lg6v8ztpbfhwb5c0000gp/T/1412920453227.tar.gz ...
Begin to upload cloud code files...
Upload cloud code files successfully. Begin to deploy...
Congrats! Deploy cloud code successfully.
Cloud code status is:

Development version    : 'local:adcce5dfb740a0e80c261dea038798b5'
Development commit log : 'Uploaded at 2014-10-10 13:54:26'
Production version     : 'local:3684975ddc5dc7abbfb06ed72ebcb371'
Production commit log  : 'Uploaded at 2014-10-09 16:56:06'

```

默认部署日志是 `Updated at YYYY-MM-DD HH:mm:ss` 的时间戳日志，你可以通过 `-o` 选项来提供更详细的部署日志：

```
$ avoscloud deploy -o '测试本地推送部署'
```

部署之后，你可以通过 curl 命令，或者访问你设置的 dev.xxxx.avosapps.com 的二级域名测试你的云代码。

### Git仓库部署

如果你的代码是保存在某个 Git 仓库，例如 [Github](https://github.com) 上，你也可以请求 LeanCloud 平台从 Git 仓库获取源码并自动部署，这个操作可以在云代码的部署菜单里完成，也可以在本地执行 `deploy` 命令和 `-g` 选项配合完成：

```
$ avoscloud -g deploy
```

* `-g` 选项指定要求从 Git 仓库部署，Git 仓库地址必须已经在云代码菜单里保存。

默认部署都将是 master 分支的最新代码，你可以通过 `-r <revision>` 来指定部署特定的 commit 或者 branch。

## 发布

开发环境如果测试没有问题，你希望将开发环境的云代码切换到生产环境，你可以使用开发者平台的云代码部署菜单做发布，也可以直接运行 `publish` 命令：

```
$ avoscloud publish
```

就会将开发环境的云代码发布到生产环境。

```
[INFO]: Cloud Code Project Home Directory: /Users/dennis/programming/avos/new_app/
[INFO]: Current App: origin <app id>
Publishing cloud code to production...
Published cloud code successfully. Current status is:

Development version    : 'local:adcce5dfb740a0e80c261dea038798b5'
Development commit log : 'Uploaded at 2014-10-10 13:54:26'
Production version     : 'local:adcce5dfb740a0e80c261dea038798b5'
Production commit log  : 'Uploaded at 2014-10-10 13:54:26'

```

## 查看部署状态

可以通过 `status` 命令查询当前生产环境和开发环境的部署状态：

```
$ avoscloud status
[INFO]: Cloud Code Project Home Directory: /Users/dennis/programming/avos/new_app/
[INFO]: Current App: origin <app id>
Cloud code status is:

Development version    : 'local:adcce5dfb740a0e80c261dea038798b5'
Development commit log : 'Uploaded at 2014-10-10 13:54:26'
Production version     : 'local:adcce5dfb740a0e80c261dea038798b5'
Production commit log  : 'Uploaded at 2014-10-10 13:54:26'

```

通过 `undeploy` 命令，可以将云代码彻底从 LeanCloud 平台移除（包括代码、版本信息、提交日志等）：

```
$ avoscloud undeploy
```

**请慎重执行此操作**。

## 查看日志

使用 `logs` 命令可以查询云代码最新日志：

```
$ avoscloud logs
[INFO]: Cloud Code Project Home Directory: /Users/dennis/programming/avos/new_app/
[INFO]: Current App: origin 7104en0u071tcb5d1tr2juxa499ouvdn1gm5szq47nqzt06q
[2014-10-09T16:56:32.279Z] [production] -- info:  undefined

[2014-10-09T17:03:33.672Z] [development] -- info:  Deploying cloud code to development environment, the commit is 'local:3684975ddc5dc7abbfb06ed72ebcb371', and log is 'Uploaded at 2014-10-09 17:03:33'.

[2014-10-10T13:54:26.450Z] [development] -- info:  Deploying cloud code to development environment, the commit is 'local:adcce5dfb740a0e80c261dea038798b5', and log is 'Uploaded at 2014-10-10 13:54:26'.

[2014-10-10T14:09:11.744Z] [production] -- info:  Deploying cloud code to production environment, the commit is 'local:adcce5dfb740a0e80c261dea038798b5', and log is 'Uploaded at 2014-10-10 13:54:26'.
```

默认返回最新的 10 条，最新的在最下面。

可以通过 `-n` 选项设定返回的日志数目，例如返回最近的 100 条

```
$ avoscloud -n 100 logs
```

也可以加上 `-t` 选项来自动滚动更新日志，类似`tailf`命令的效果：

```
$ avoscloud -t logs
```

当有新的云代码日志产生，都会自动填充到屏幕下方。


## 多应用管理

从 0.5.0 版本开始，我们为 avoscloud 添加了多应用管理功能，类似 git 的多分支功能。使用这个功能，允许你将同一个云代码项目部署到多个 LeanCloud 应用上，**但是仅限于云代码 2.0 项目使用**。

### 查看应用状态

使用 `avoscloud app list` 可以查看当前应用列表，默认情况下应该显示 `config/global.json` 里设定的应用：

```
$ avoscloud app list
  origin <config/global.json 里的 applicationId>
```

执行 `avoscloud app` 查看当前应用，因为目前没有明确指定，会告诉你：

```
$ avoscloud app
You are not in a app.Please checkout <app>
```

我们明确切换到 `origin` 应用试试：

```
$ avoscloud checkout origin
Switced to app origin
$ avoscloud app
* origin <config/global.json 里的 applicationId>
```

现在确认我们处于默认的初始应用。此时，执行 `deploy`、`publish`、`status`、`logs`等命令都将是针对当前应用。

### 添加应用

如果你想将 new_app 发布到其他 LeanCloud 应用，你可以通过 `add` 命令来添加一个应用：

```
$ avoscloud add other_app <other app 的应用 id>
```

`add` 接收两个参数，第一个是应用的名称，用于后续的显示和切换，第二个是新应用的 id，可以在应用设置的应用 Key 信息里找到。

添加成功将打印：

```
Added a new app: other_app -- <应用 id>
```

通过 `app list` 命令将看到两个应用：

```
$ avoscloud app list
* origin    7104en0u071tcb5d1tr2juxa499ouvdn1gm5szq47nqzt06q
  other_app 1qdney6b5qg2i69t79yq941krrwdu3glt0ot69re6w7xv6lf
```

前面有星号的应用，表示是当前应用。切换应用，需要使用 `checkout` 命令。

### 切换应用

从当前应用切换到另一个应用，可以使用 `checkout <应用名称>`:

```
$ avoscloud checkout other_app
Switced to app other_app
```

切换成功后，执行 `deploy`、`publish`、`status`、`logs`等命令都将运行在 `other_app` 上。如果你过去没有部署过，第一次部署的时候会要求你输入新应用的 master key。

通过 `app` 命令可以看到当前应用已经是 `other_app`:

```
$ avoscloud app
* other_app <应用 id>
```

如果你想在不切换应用的情况下去部署云代码到其他应用，也可以通过 `-p` 选项来指定，例如:

```
$ avoscloud deploy -p other_app
```

这样就无需通过 checkout 切换应用，就可以部署项目到其他应用。`status`、`publish` 等应用相关的命令也同样支持 `-p` 选项。

### 移除应用

同样，你可以删除一个应用，使用  `rm` 命令：

```
$ avoscloud rm other_app
Removed app: other_app
```

通过 `app list` 确认已经删除：

```
$ avoscloud app list
  origin 7104en0u071tcb5d1tr2juxa499ouvdn1gm5szq47nqzt06q
```

## 批量上传文件

如果你有一些文件希望上传到 LeanCloud 平台上，可以通过 `upload` 命令，既可以上传单个文件，也可以批量上传一个目录下（包括子目录）下的所有文件。

```
$ avoscloud upload public/index.html
Uploads /Users/dennis/programming/avos/new_app/public/index.html successfully at: http://ac-7104en0u.qiniudn.com/f9e13e69-10a2-1742-5e5a-8e71de75b9fc.html
```

上传成功后会显示文件在 LeanCloud 平台上的 URL。

上传 images 目录下的所有文件：

```
avoscloud upload images/
```

## 其他命令

为了方便开发阶段查询资料或者文档，可以使用 `search` 命令：

```
$ avoscloud search AVObject
```

这将打开浏览器，显示[搜索结果](https://leancloud.cn/search.html?q=AVObject)。

也可以查询多个关键字，空格隔开即可：

```
$ avoscloud search 云代码 命令行
```


## 贡献

`avoscloud-code` 本身是开源，基于 [GNU LGPL](https://www.gnu.org/licenses/lgpl.html) 协议，源码托管在 Github: [https://github.com/avoscloud/avoscloud-code-command](https://github.com/avoscloud/avoscloud-code-command)

欢迎大家贡献。

