# GitHub

## 创建图片库

+ 在自己的 GitHub 上创建一个库，当做图床，专门用来存储图片。具体操作流程与建仓库的流程一样

## 获取令牌

GitHub 的令牌，其实就是 token，自我感觉就像自己的 GitHub 对外的一个公钥一样，可以让拥有此 token 的软件访问 GitHub 的 API 接口，生成过程，参考经验即可，大致步骤如下

- 点击自己的 GitHub 头像
- Settings
- Developer settings
- Personal access tokens
- Generate new token

![image-20230124142445913](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301241424010.png)

+ 点击 Generate new token 创建一个新 token，选择 repo

![image-20230124142615473](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301241426561.png)

# Picgo安装及配置

## 安装配置PicGo

+ 直接安装下载下来的 EXE 文件即可

- 接下来配置 GitHub 作为图床，在左侧找到**图床设置**，找到**GitHub图床**
- 前边有星号的为必填项，依次填入之前创建的仓库名，注意是：账户名/仓库名
- 然后填入设定的分支名（创建仓库时如果没有创建其他分支，默认就是 master 分支）
- 最后填入之前生成的 token 令牌，点击确定
- 指定存储路径一定要写
- 配置完成后设为默认图床

![image-20230124142920869](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301241429920.png)

+ 找到 PicGo 设置，打开里边的 ***时间戳重命名***，这样可以避免图床在上传文件时，由于文件名相同造成的错误
+ 剩下的配置项可以不用管，等会配置好 typora 后，typora 在上传图片时会自动打开 PicGo 软件

## 安装配置node.js

+ 整个安装过程也很简单，一路 next ，全部使用默认配置即可

## 测试PicGo

+ 到这一步，可以打开软件，直接拖动到首页上传区，测试是否上传成功；或者直接利用截图，然后点击右下角的**剪贴板图片上传**，即可快速实现上传

## 图片上传常见失败原因

+ github设置错误，仓库名称、token、存储路径等

+ 在`PicGo设置`此选项下，可以找到对应的日志文件，查看相关错误信息，进而辅助我们排查问题

![image-20230124143719189](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301241437242.png)

# Typora安装及配置

## 软件破解

+ 将winmm.dll文件放置于typora安装路径下

![image-20230124144155584](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301241441635.png)

![image-20230124144025998](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301241440042.png)

## 设置上传图片并关联图床工具

+ 设置如下图

![image-20230124144336066](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301241443131.png)

+ 在配置完成的时候，可以点击**验证图片上传选项**，进行测试，查看配置的 PicGo 插件是否有用。成功时会出现如下界面

![image-20230124144450802](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301241444866.png)

# 文件上传及下载

## 上传

```bash
git add .
git commit -m "版本名称"
git push
```

## 下载

```bash
git clone URL
```

# 软件下载地址

- [Typora](https://link.juejin.cn/?target=http%3A%2F%2Ftypora.io%2F)
- PicGo 在 GitHub 上的地址：[GitHub - PicGo](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FMolunerfinn%2FPicGo%2Freleases)
- node.js 插件下载地址：[node.js下载](https://link.juejin.cn/?target=http%3A%2F%2Fnodejs.cn%2Fdownload%2F)