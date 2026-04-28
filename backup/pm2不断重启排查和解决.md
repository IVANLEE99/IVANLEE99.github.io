# pm2不断重启排查

【p0】pc端进程重启

## 一、问题：pm2 list 显示进程不断重启
![qcLTdZsBo-KX6H6AGs3q03Xw6YysoQsBo86gLFit1xI](https://github.com/user-attachments/assets/c597e0aa-cb1d-47bd-927f-78139f4e0fa9)
![aMQgFYFN2LPXrumFoX47qJe6RK5O3XO6lYt8J2DrYsQ](https://github.com/user-attachments/assets/bdf11f94-93e1-4481-bc2d-ed4137a1a0ba)
![FHkbMpCXktNXOmqTS2Wj4I07igCwIiJStuy6OuVac2g](https://github.com/user-attachments/assets/4049b95d-943e-4c04-8839-0d4d57bb3232)



## 二、解决办法

本地打包查看内存情况

1、新增.env.development.debug文件

```plain
# just a flag
NUXT_APP_ENV = 'development.debug'
```
 
2、package.json 新增打包命令

```plain
		"build:debug": "node ./generate-version.js && cross-env NODE_ENV=development.debug nuxt build",
		"debug": "cross-env NODE_ENV=development.debug node --inspect  node_modules/.bin/nuxt start",
		"start:debug": "rm -rf node_modules && yarn install && npm run build:debug && npm run debug"
```
 
3、启动

```plain
npm run start:debug
```
 
![GM9Y_Sol16fJv3zDbeMg4yUbGCky_2amLXIdd3pBqHg](https://github.com/user-attachments/assets/c5069739-e147-4c76-9972-6adf96a50297)



4、检测内存

chrome://inspect/#devices

<img width="778" alt="xAjQj-ESG3p3_LEwMEb8fpb3PAezwC--R9jXU_IF2oI" src="https://github.com/user-attachments/assets/2a3b665c-448b-48cc-b125-9ea9e4d5eaf6">


<img width="1440" alt="OPN3BlYPUJL5Ck8jJi0idNRNLtilt-utioJPBOs66qM" src="https://github.com/user-attachments/assets/075f9c0d-f9e9-41f2-a87a-92229a1bcfff">


未访问localhost:3000前打1次内存快照

访问localhost:3000后打第2次快照

关闭localhost:3000标签后打第三次快照

访问localhost:3000后打第4次快照

关闭localhost:3000标签后打第5次快照

访问localhost:3000后打第6次快照

关闭localhost:3000标签后打第7次快照


<img width="1440" alt="WaMxganUC8h_8uXh7CufDioYDDfsiZJIdghHw0VkQb0" src="https://github.com/user-attachments/assets/c85a7014-3bea-454a-9a8c-582d23eaa106">




内存不断增加，查看第5和弟6次之间分配的内存

点击具体实例，点击对应的文件

查看对应代码的情况

<img width="1440" alt="TrRLtJyTzfoeEfX5MAItHoNbPUn_eo3t9Iilx2P3niI" src="https://github.com/user-attachments/assets/3fdd98da-9f40-434a-bb01-e65a039ec327">


<img width="1440" alt="Dn_qcZ-SlSMP1GTRYl8NXwopGDang393TDkhiRu8Rmo" src="https://github.com/user-attachments/assets/fa757604-b5d7-4db8-9ef7-3e94ef0f7ac6">


![4giPWSgINKpwiChJrJ7VAY8PEXnupSBB3PYXIkvmB8U](https://github.com/user-attachments/assets/5228b2f3-8bb1-4c8d-b517-a78187b27f67)



![ogtZbpZjy4fjXie5WcgeMAN8H0CuZYbYDha9SFhCg_4](https://github.com/user-attachments/assets/639fd5bf-4ea2-4ff8-8350-1745a0f410d6)


发现是layouts/error.vue的computed中调用了setInterval 没清除

## 三、修复

computed 中注释`handleCountDown方法`

改为mounted中调用

![Gc-lsa0Jp4PQRCrHxElSThInc54ZWPVQMkWWNm46mcs](https://github.com/user-attachments/assets/67e4b41b-a54e-455d-8ab8-26c7947efe28)




## 四、修复成功后pm2 list 显示重启为0，内存没有飙升

![H0FWTAsJV1Tzhnx9yCDSiFZj2-sZXqgDql92RKiHTmU](https://github.com/user-attachments/assets/9bc69438-5ab4-4e87-a99d-cce52fb33ab1)



![HARrZcRYkhGNhlZNXiAiM44BiH2tVwh3F-MovXUBV5k](https://github.com/user-attachments/assets/746415a7-20ba-44cf-9f81-01a3a86b03db)



![eI5mu3_t-ORlpLh4QEI7jCz5ZHa0a1O5KU6cJWkdF6c](https://github.com/user-attachments/assets/ec20823f-4e05-4b24-8727-ae4b6ca1f9c6)



## 五、总结

内存泄漏代码上就那些点～

1、事件监听器：如果在组件中添加了事件监听器，但在组件销毁时没有移除它们，可能会导致内存泄漏。

2\. 定时器和异步操作：使用 setTimeout 或 setInterval 时，如果没有在组件销毁时清除这些定时器，也会导致内存泄漏。

3\. 未清理的引用：在 Vue 组件中，如果有对 DOM 元素或其他对象的引用，而这些引用在组件销毁时没有被清理，可能会导致内存泄漏。

4\. Vuex 状态管理：如果在 Vuex 中存储了大量数据，而这些数据在不再需要时没有被清理，也可能导致内存泄漏



## 六、参考链接

 [排查 Nuxt.js服务器端内存泄漏问题（Nuxt 3同理）](https://juejin.cn/post/7228834021152473145)

 [https://juejin.cn/post/7228834021152473145](https://juejin.cn/post/7228834021152473145) 

 [https://developer.chrome.com/docs/devtools/memory?hl=zh-cn](https://developer.chrome.com/docs/devtools/memory?hl=zh-cn) 

 [https://www.ruanyifeng.com/blog/2018/12/git-bisect.html](https://www.ruanyifeng.com/blog/2018/12/git-bisect.html) 




