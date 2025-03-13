### 需求点击tab关闭按钮，将对应的tab的缓存关掉

![Image](https://github.com/user-attachments/assets/ed7a5e56-b685-440a-be2e-70ff5d1de495)


### 解决办法

![Image](https://github.com/user-attachments/assets/02176cc2-259e-4317-b243-e9c059fbf818)

```
  mounted() {
    let that = this
    setTimeout(() => {
      console.log("that.$refs.myKeepAliveRef", that.$refs.myKeepAliveRef)
      let keepAliveRef = that.$refs?.myKeepAliveRef?.$vnode?.parent?.componentInstance
      console.log("keepAliveRef", keepAliveRef)
      if (keepAliveRef) {
        that.$store.commit("tagsView/SET_KEEP_ALIVE_REF", keepAliveRef)
      }
    }, 1000)
  }
```

```
  clearCache({ state }, view) {
    const cache = state.keepAliveRef.cache;
    const keys = state.keepAliveRef.keys;
    let index = -1
    for (let i = 0; i < keys.length; i++) {
      if (keys[i] && keys[i].endsWith(view.fullPath)) {
        index = i
        break
      }
    }
    let name = ''
    if (index > -1) {
      name = keys[index]
    }
    console.log('cache, keys', cache, keys)
    if (cache[name]) {
      // 销毁组件实例
      cache[name].componentInstance.$destroy();
      // 删除缓存
      delete cache[name];
      // 移除对应的 key
      if (index > -1) {
        keys.splice(index, 1);
      }
    }
  }
```

![Image](https://github.com/user-attachments/assets/4991eb80-2576-47d4-b0e5-9ee3c9da58c8)


### 坑

```
<keep-alive ref="keepAliveRef">
  </keep-alive>
```
`this.$refs.keepAliveRef` 取不到值～


### 参考链接

https://blog.csdn.net/qq_19945487/article/details/133903323
vue2 keepalive多层缓存bug以及清除缓存
