https://nacos.io/docs/latest/quickstart/quick-start/?spm=5238cd80.c984973.0.0.6be14023Z3kwkz

https://github.com/nacos-group/nacos-sdk-nodejs?spm=5238cd80.c984973.0.0.6be14023Z3kwkz

<img width="1280" height="280" alt="Image" src="https://github.com/user-attachments/assets/aac34ac4-a29e-4680-b17f-708d70b3cefb" />

1、前置工作
- 设置必要的环境变量
NACOS_SERVER_URL：nacos服务地址
NACOS_CONFIG_DIR：写入nacos配置的文件目录

export NACOS_SERVER_URL=http://your_nacos_server:8848
export NACOS_CONFIG_DIR=./config
node nacosClient.js --dataid=test --namespaceid=your_namespace_id


2、客户端

```
#!/usr/bin/env node
const fs = require('fs');
const path = require('path');
const http = require('http');
const https = require('https');
const crypto = require('crypto');
const { URL } = require('url');
const minimist = require('minimist');

// 解析命令行参数
const args = minimist(process.argv.slice(2), {
  alias: {
    d: 'dataid',
    n: 'namespaceid',
    g: 'group'
  },
  string: ['d', 'n', 'g'],
  default: {
    group: 'DEFAULT_GROUP'
  }
});

// 环境变量检查
const nacosServerUrl = process.env.NACOS_SERVER_URL;
const nacosConfigDir = process.env.NACOS_CONFIG_DIR;

if (!nacosServerUrl) {
  console.error('配置服务地址无效,无法读取环境变量: NACOS_SERVER_URL');
  process.exit(1);
}

if (!nacosConfigDir) {
  console.error('配置目录无效,无法读取环境变量: NACOS_CONFIG_DIR');
  process.exit(1);
}

const { dataid, namespaceid, group } = args;
if (!dataid || !namespaceid) {
  console.error('dataid和namespaceid不能为空');
  process.exit(1);
}

// 创建配置目录
if (!fs.existsSync(nacosConfigDir)) {
  fs.mkdirSync(nacosConfigDir, { recursive: true });
}

console.log(`启动监听进程 [PID: ${process.pid}]`);

class NacosClient {
  constructor(configServer, dataId, namespaceId, group = 'DEFAULT_GROUP') {
    this.configServer = configServer.replace(/\/$/, '');
    this.dataId = dataId;
    this.namespaceId = namespaceId;
    this.group = group;
    this.saveDir = nacosConfigDir;
    this.longPullingTimeout = 90000; // 90秒长轮询超时
    this.lastContentMd5 = '';
    this.isActive = true;
    this.retryDelay = 5000; // 5秒重试延迟
  }

  async start() {
    console.log('开始监听配置变更...');
    while (this.isActive) {
      try {
        await this.fetchConfig();
      } catch (error) {
        console.error(`请求失败: ${error.message}`);
        await new Promise(resolve => setTimeout(resolve, this.retryDelay));
      }
    }
  }

  async fetchConfig() {
    const url = new URL(`${this.configServer}/nacos/v1/cs/configs`);
    url.searchParams.append('dataId', this.dataId);
    url.searchParams.append('tenant', this.namespaceId);
    url.searchParams.append('group', this.group);

    const options = {
      timeout: this.longPullingTimeout + 10000
    };

    return new Promise((resolve, reject) => {
      const protocol = url.protocol === 'https:' ? https : http;
      const req = protocol.get(url.toString(), options, (res) => {
        let data = '';
        
        res.on('data', (chunk) => data += chunk);
        
        res.on('end', () => {
          if (res.statusCode === 200) {
            this.handleConfigUpdate(data);
            resolve();
          } else if (res.statusCode === 304) {
            console.log('配置无变更，继续监听...');
            resolve();
          } else {
            reject(new Error(`HTTP ${res.statusCode}: ${data || 'Unknown error'}`));
          }
        });
      });

      req.on('error', reject);
      req.on('timeout', () => {
        req.destroy();
        console.log('长轮询超时，重新请求...');
        resolve();
      });
    });
  }

  handleConfigUpdate(content) {
    const currentMd5 = crypto.createHash('md5').update(content).digest('hex');
    
    if (currentMd5 !== this.lastContentMd5) {
      const filePath = path.join(this.saveDir, `${this.dataId}.ini`);
      
      try {
        fs.writeFileSync(filePath, content, { flag: 'w', encoding: 'utf8' });
        this.lastContentMd5 = currentMd5;
        console.log(`配置已更新: ${filePath} (${new Date().toISOString()})`);
      } catch (error) {
        console.error(`文件写入失败: ${error.message}`);
      }
    }
  }

  shutdown() {
    this.isActive = false;
    console.log('监听器已停止');
  }
}

// 创建并启动客户端
const client = new NacosClient(nacosServerUrl, dataid, namespaceid, group);
client.start();

// 处理进程退出信号
process.on('SIGINT', () => client.shutdown());
process.on('SIGTERM', () => client.shutdown());
```


## 使用说明
1、安装依赖:
`npm install minimist`
2、设置环境变量:
`export NACOS_SERVER_URL=http://your-nacos-server:8848
export NACOS_CONFIG_DIR=./config`
3\运行命令:
`node nacos-client.js \
  --dataid=your_data_id \
  --namespaceid=your_namespace \
  --group=YOUR_GROUP`

提示：生产环境建议结合Supervisor等工具实现进程守护，确保中断后自动重启。

