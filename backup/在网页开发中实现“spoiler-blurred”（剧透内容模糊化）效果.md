### 在网页开发中实现“spoiler-blurred”（剧透内容模糊化）效果

### 1、代码     filter: ### blur(0.5em);



```
.spoiler-blurred {
    cursor: default;
    -webkit-user-select: none;
    user-select: none;
    cursor: pointer;
    **filter: blur(0.5em);**
}
```
### 2、点击后移除poiler-blurred类名即可

![Image](https://github.com/user-attachments/assets/0e4ad262-6dd3-4cc3-a4e3-289a8b987450)

### 点击后移除poiler-blurred类名即可

![Image](https://github.com/user-attachments/assets/2fa395d5-4b66-48c5-a1c5-b1002716068f)