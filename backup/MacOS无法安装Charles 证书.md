### **问题**
在新的mac上无法安装Charles的证书，无论是直接在Charles中直接安装和将证书保存下来导入都无法安装。

> to change whether a root certificate is trusted, open it in keychain access and modify its trust settings. new root certificates should be added to the login keychain for the current user, or to the system keychain if they are to be shared by all users of this machine.



### 解决方案

我的解决方案是：Help->SSL Proxying -> Reset Charles Root Certificate。重置后再重新安装，就可以了。证书过期后也可以进行重置。

<img width="2382" height="832" alt="Image" src="https://github.com/user-attachments/assets/ef01e3d1-4517-4837-ba44-56dcbf7dc303" />



### 后续设置使用

https://segmentfault.com/a/1190000017957302