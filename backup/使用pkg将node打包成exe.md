# 需求描述
将node打包成exe

# 技术方案
使用`pkg`。通过此命令行界面，您可以将 Node.js 项目打包成可执行文件，甚至可以在未安装 Node.js 的设备上运行。

## 使用案例
* 制作一个没有来源的应用程序的商业版本
* 制作没有来源的应用程序的演示/评估/试用版本
* 立即为其他平台制作可执行文件（交叉编译）
* 制作某种自解压存档或安装程序
* 无需安装 Node.js 和 npm 即可运行打包的应用程序
* 无需通过 npm install 下载数百个文件来部署您的应用程序。将其部署为单个文件
* 将您的资源放入可执行文件中，使其更加便携
* 针对新的 Node.js 版本测试您的应用程序，而无需安装它

## 使用方法

+ `npm install -g pkg`
+ package.json 加入bin 和pkg入口配置，详细配置请查看[https://github.com/vercel/pkg](https://github.com/vercel/pkg)

  `
      "bin": "index.js",
      "pkg": {
        "scripts": "build/**/*.js",
        "assets": "public/**/*",
        "targets": [
          "node18-macos-x64",
          "node18-win-x64"
        ],
        "outputPath": "dist"
      },
  `
+ 执行命令`pkg .`

+ 打包成功输出到 dist 目录
<img width="894" alt="image" src="https://github.com/user-attachments/assets/aa1fe5fc-ca74-4bd6-a2f4-1aa2268f0e03">

+ 双击运行
![page1](https://github.com/user-attachments/assets/b269546a-10c1-4531-b3d8-8dddb0e198aa)
<img width="1195" alt="page" src="https://github.com/user-attachments/assets/c6a2e5ae-7480-460d-a376-82d4a2fcc733">

## 思考🤔

Further, we’re excited about Node.js 21’s support for [single executable applications](https://nodejs.org/api/single-executable-applications.html).




