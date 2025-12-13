### App 在macOS 下提示已损坏无法打开解决办法：

1、打开终端；
2、输入以下命令，回车；
`sudo xattr -d com.apple.quarantine /Applications/xxxx.app`
注意：/Applications/xxxx.app 换成你的App路径（推荐直接将.app文件拖入终端中自动生成路径，以防空格等转义字符手动复制或输入出现错误）

    这是核心命令，分解如下：
    sudo: 表示“Super User Do”。它让你以管理员（root）权限执行后面的命令。因为修改系统级别的文件属性需要管理员权限。输入这个命令后，系统会要求你输入你的管理员账户密码。 输入密码时，终端不会显示任何字符（星号也没有），这是正常的安全措施，输完直接按回车即可。
    xattr: 这是 macOS 上用于管理文件“扩展属性”的命令。扩展属性是附加在文件或文件夹上的额外元数据。
    -d: 这是 xattr 命令的一个选项，意思是“删除”。
    com.apple.quarantine: 这是你要删除的特定扩展属性的名称。这个属性就是 Gatekeeper 用来标记文件来源“不安全”（如从网上下载）并对其进行“隔离”的关键标识。移除它，系统就不再认为这个应用是“被隔离”的。
    /Applications/xxxx.app: 这是目标应用程序的路径。你需要将 xxxx.app 替换成你实际遇到问题的那个应用程序的名称（包括 .app 后缀）。这个应用通常安装在 /Applications 文件夹下。

3、重启App即可。

### 总结：

这条命令的作用是手动移除 macOS 附加在某个下载应用程序上的“隔离”属性标记 (com.apple.quarantine)。通过管理员权限 (sudo) 使用 xattr 工具删除这个属性后，Gatekeeper 就不再阻止该应用运行。使用拖拽 .app 文件到终端的方法来填充路径是最不容易出错的操作方式。


### 验证成功

我装的Navicat Premium

`sudo xattr -d com.apple.quarantine /Applications/Navicat\ Premium.app`