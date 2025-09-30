nuxt.config.js

```
const dotenv = require('dotenv')
const envConfig = dotenv.config({ path: `.env.${process.env.NODE_ENV}` }).parsed
const isPro = !!(process.env.NODE_ENV !== 'development') //判断是否开发环境
const isBuildTest = process.env.NUXT_APP_ENV === 'production.dev'

    env: envConfig,

```

.env.production.dev

```
# just a flag
NUXT_APP_ENV = 'production.dev'
```

### 执行

NODE_ENV=production.dev pm2 start ecosystem.config.js

### 需求

需要判断是否是测试环境

### 尝试解决
const isBuildTest = process.env.NODE_ENV === 'production.dev'//process.env.NODE_ENV->production->false

### 解决办法
const isBuildTest = process.env.NUXT_APP_ENV === 'production.dev'//process.env.NUXT_APP_ENV->production.dev->true


### 坑坑坑
process.env.NODE_ENV 只有development/production
process.env.NODE_ENV 只有development/production
process.env.NODE_ENV 只有development/production
