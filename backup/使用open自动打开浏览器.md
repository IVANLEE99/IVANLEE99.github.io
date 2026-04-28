# 使用open自动打开浏览器
## 需求背景
我用pkg打包成exe后，双击exe自动打开浏览器
## 技术实现
使用[open](https://github.com/sindresorhus/open)

+ 安装`npm install open@8`
+ 使用
`
app.listen(3003, async () => {
    console.log('Server is running on http://localhost:3003');
    // const { default: open } = await import('open'); // 动态导入 open 模块
    await open('http://localhost:3003'); // 启动后自动打开浏览器
});

`
![page1](https://github.com/user-attachments/assets/9b107d1b-7b9d-488b-86bd-d368bdb3dc1c)
<img width="1195" alt="page" src="https://github.com/user-attachments/assets/7c59122f-1792-4cd3-b581-2779ba59cf0b">

## 注意事项

警告：此包是本机 ESM，不再提供 CommonJS 导出。如果您的项目使用 CommonJS，则必须转换为 ESM 或使用动态 import() 函数。请不要针对有关 CommonJS / ESM 的问题打开问题。

If you cannot switch to ESM, please use v2 which remains compatible with CommonJS. Critical bug fixes will continue to be published for v2.


