[vue2-zoom-image:GitHub仓库地址](https://github.com/IVANLEE99/vue2-zoom-image#readme)

# [vue2-zoom-image](https://www.npmjs.com/package/vue2-zoom-image) — 开发 / 调试 / 打包 / 发布 / 使用 全流程指南

> 本文档基于真实开发与踩坑过程整理，可直接作为 Vue2 组件库开发模板。

---

## 🚀 1. 整体流程

```
开发组件 → Rollup 打包 → 本地 npm link 调试 → Nuxt2 测试 → 修复问题 → npm publish → 正式安装测试 → npm unlink 清理
```

---

## 📁 2. 项目推荐结构

```
vue2-zoom-image/
├── package.json
├── rollup.config.js
├── src/
│   ├── ZoomImage.vue
│   └── index.js
└── dist/
```

### `src/ZoomImage.vue`

```vue
<template>
  <div class="ZoomImage">
    <img :src="src" />
  </div>
</template>

<script>
export default {
  name: "ZoomImage",
  props: {
    src: {
      type: String,
      required: true
    }
  }
};
</script>

<style scoped>
.ZoomImage {
  width: 100%;
}
</style>
```

### `src/index.js`（入口）

```js
import ZoomImage from "./ZoomImage.vue";

export default {
  install(Vue) {
    Vue.component("ZoomImage", ZoomImage);
  }
};

export { ZoomImage };
```

---

## 📦 3. Rollup 打包配置（自动内联 CSS）

### 安装依赖

```bash
npm install -D rollup rollup-plugin-vue rollup-plugin-commonjs rollup-plugin-node-resolve rollup-plugin-postcss
```

### `rollup.config.js`

```js
import vue from "rollup-plugin-vue";
import postcss from "rollup-plugin-postcss";
import resolve from "rollup-plugin-node-resolve";
import commonjs from "rollup-plugin-commonjs";

export default {
  input: "src/index.js",
  output: [
    {
      file: "dist/vue2-zoom-image.esm.js",
      format: "esm"
    },
    {
      file: "dist/vue2-zoom-image.umd.js",
      format: "umd",
      name: "Vue2ZoomImage",
      globals: {
        vue: "Vue"
      }
    }
  ],
  external: ["vue"],
  plugins: [
    vue({ css: false }),
    postcss({ extract: false, minimize: true }), // ✅ CSS 直接打进 JS
    resolve(),
    commonjs()
  ]
};
```

### 打包

```bash
npm run build
```

产物：

```
dist/
  ├── vue2-zoom-image.esm.js
  └── vue2-zoom-image.umd.js
```

---

## 🔗 4. 本地调试（npm link）

### 在组件库项目

```bash
cd vue2-zoom-image
npm link
```

### 在 Nuxt2 / Vue2 项目

```bash
cd nuxt-mall-pc
npm link vue2-zoom-image
```

---

## 🧪 5. 在 Nuxt2 中使用

### 创建插件

`plugins/vue2-zoom-image.js`

```js
import Vue from "vue";
import Vue2ZoomImage from "vue2-zoom-image";

Vue.use(Vue2ZoomImage);
```

### 注册插件（nuxt.config.js）

```js
export default {
  plugins: [
    { src: "~/plugins/vue2-zoom-image.js", mode: "client" }
  ]
};
```

### 页面中使用

```vue
<template>
  <div>
    <ZoomImage src="https://via.placeholder.com/300" />
  </div>
</template>
```

---

## 🐛 6. 常见问题与解决

| 问题 | 原因 | 解决办法 |
|------|------|---------|
| `Unexpected token (<)` | 在 JS 里写 Vue 模板 | 所有模板必须写在 `.vue` 文件 |
| CSS 未加载 | 未内联 | 使用 `postcss({ extract:false })` |
| Vue 版本不一致 | vue vs vue-template-compiler | 保持同一 2.x 版本 |
| `btoa is not defined` | Node SSR 环境缺失 | 仅在客户端使用或添加 polyfill |
| UMD 被当成 ESM | `package.json` 有 `"type": "module"` | 发布前删除该字段 |

---

## 🚀 7. 发布到 npm

### 登录（需 2FA）

```bash
npm login
```

### 发布

```bash
npm publish
```

---

## 📦 8. 正式环境测试（不使用 link）

```bash
cd nuxt-mall-pc
npm unlink vue2-zoom-image
npm install vue2-zoom-image
npm run dev
```

---

## 🧹 9. 清理本地 link

### 在测试项目

```bash
cd nuxt-mall-pc
npm unlink vue2-zoom-image
```

### 在组件库项目

```bash
cd vue2-zoom-image
npm unlink
```

### 校验

```bash
npm ls vue2-zoom-image
```

若显示**版本号而非本地路径**，说明清理完成。

---

## 🎯 10. 一页速查版

```bash
# 开发 → 打包
npm run build

# 本地调试
cd vue2-zoom-image && npm link
cd nuxt-mall-pc && npm link vue2-zoom-image

# 发布
cd vue2-zoom-image && npm publish

# 正式测试
cd nuxt-mall-pc
npm unlink vue2-zoom-image
npm install vue2-zoom-image

# 清理
cd vue2-zoom-image && npm unlink
```

