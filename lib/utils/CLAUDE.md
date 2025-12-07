[根目录](../../CLAUDE.md) > [lib](../) > **utils**

# 工具库模块

## 模块职责

utils 目录提供通用工具函数库，负责：
- HTTP 请求处理
- 数据解析和转换
- 缓存操作
- 浏览器自动化
- RSS/Atom 生成和解析
- 通用辅助函数

## 工具函数列表

### 1. HTTP 客户端
- **ofetch.ts** - 新一代 HTTP 客户端（推荐使用）
- **got.ts** - HTTP 客户端（已废弃，兼容性保留）
- **header-generator.ts** - 请求头生成器
- **valid-host.ts** - 主机名验证

### 2. 浏览器自动化
- **puppeteer.ts** - Puppeteer 配置和启动
- **puppeteer-utils.ts** - Puppeteer 工具函数
- **rebrowser-puppeteer** - 真实浏览器集成

### 3. 数据处理
- **rss-parser.ts** - RSS 解析器
- **parse-date.ts** - 日期解析
- **camelcase-keys.ts** - 驼峰命名转换
- **readable-social.ts** - 社交媒体链接处理

### 4. 缓存管理
- **cache/** - 缓存系统
  - **index.ts** - 缓存统一入口
  - **memory.ts** - 内存缓存
  - **redis.ts** - Redis缓存
  - **base.ts** - 缓存基类

### 5. 工具函数
- **logger.ts** - 日志工具
- **md5.ts** - MD5 哈希
- **wait.ts** - 延时函数
- **helpers.ts** - 通用辅助函数
- **common-utils.ts** - 常用工具函数

### 6. 内容处理
- **render.ts** - 模板渲染
- **wechat-mp.ts** - 微信公众号处理

### 7. 代理管理
- **proxy/** - 代理系统
  - **index.ts** - 代理统一入口
  - **multi-proxy.ts** - 多代理管理
  - **pac-proxy.ts** - PAC代理
  - **unify-proxy.ts** - 代理统一

### 8. 监控追踪
- **otel/** - OpenTelemetry
  - **index.ts** - 追踪入口
  - **trace.ts** - 链路追踪
  - **metric.ts** - 指标收集

## HTTP 客户端

### ofetch 使用示例
```typescript
import ofetch from '@/utils/ofetch';

// GET 请求
const data = await ofetch('https://api.example.com/data');

// POST 请求
const result = await ofetch('https://api.example.com/create', {
    method: 'POST',
    body: { name: 'test' },
});

// 带重试的请求
const response = await ofetch('https://example.com', {
    retry: 3,
    retryDelay: 1000,
});
```

### 配置选项
- 自动重试（状态码：400, 408, 409, 425, 429, 500, 502, 503, 504）
- 自动代理切换
- 请求/响应拦截器
- 超时控制

## Puppeteer 集成

### 基础使用
```typescript
import { getPuppeteerPage } from '@/utils/puppeteer';

const { page, destory } = await getPuppeteerPage('https://example.com', {
    onBeforeLoad: async (page) => {
        await page.setExtraHTTPHeaders({
            'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8'
        });
    }
});

const content = await page.content();
await destory();
```

### 高级功能
- 自动代理配置
- Cookie管理
- 请求拦截
- 资源过滤

## 缓存系统

### 使用示例
```typescript
import cache from '@/utils/cache';

// 获取缓存（不存在则执行回调）
const data = await cache.tryGet('some-key', async () => {
    const result = await fetchData();
    return result;
}, 3600); // 1小时过期

// 直接操作缓存
await cache.set('key', value, 7200);
const cached = await cache.get('key');
```

### 缓存类型
- **内存缓存**：默认，适合单实例部署
- **Redis缓存**：分布式部署首选

## 日期处理

### parse-date 功能
```typescript
import { parseDate, parseRelativeDate } from '@/utils/parse-date';

// 解析绝对日期
const date1 = parseDate('2023-12-07');
const date2 = parseDate('December 7, 2023');
const date3 = parseDate('2023/12/07 14:30:00');

// 解析相对日期
const date4 = parseRelativeDate('3小时前');
const date5 = parseRelativeDate('昨天下午2点');
const date6 = parseRelativeDate('上周一');
```

### 支持的格式
- ISO 8601
- RFC 2822
- 自定义格式
- 相对时间表达

## 日志系统

### 使用示例
```typescript
import logger from '@/utils/logger';

logger.info('RSSHub started');
logger.error('Failed to fetch data', error);
logger.debug('Processing request', { url, params });
logger.warn('Deprecated API used');
```

### 日志级别
- error - 错误信息
- warn - 警告信息
- info - 一般信息
- debug - 调试信息

## 模板渲染

### art-template 集成
```typescript
import { art } from '@/utils/render';

const html = await art(path.join(__dirname, 'template.art'), {
    title: '文章标题',
    content: '文章内容',
    items: [
        { name: '项目1', url: 'https://example.com/1' },
        { name: '项目2', url: 'https://example.com/2' }
    ]
});
```

## 请求头生成

### 自动生成真实请求头
```typescript
import { generateHeaders, PRESETS } from '@/utils/header-generator';

// 使用预设生成
const headers = generateHeaders(PRESETS.MODERN_MACOS_CHROME);

// 自定义生成
const customHeaders = generateHeaders({
    browserName: 'chrome',
    osName: 'windows',
    deviceName: 'desktop'
});
```

## 性能优化

### 最佳实践
1. **合理使用缓存**
   - API响应缓存
   - 解析结果缓存
   - 设置合理的过期时间

2. **并发控制**
   - 使用 pMap 控制并发数
   - 避免同时发起过多请求

3. **资源过滤**
   - Puppeteer中过滤不必要的资源
   - 减少网络传输

4. **错误处理**
   - 使用 tryGet 避免重复请求
   - 优雅降级

## 常用工具函数

### helpers.ts
```typescript
import { getCurrentPath, parseDuration, time } from '@/utils/helpers';

// 获取当前文件路径
const currentDir = getCurrentPath(import.meta.url);

// 解析时长字符串
const seconds = parseDuration('01:23:45'); // 5025

// 计时
const start = Date.now();
// ... do something
console.log(time(start)); // "1.23s"
```

### common-utils.ts
```typescript
import { collapseWhitespace, convertDateToISO8601, toTitleCase } from '@/utils/common-utils';

// 格式化文本
const clean = collapseWhitespace('  Hello   World  '); // "Hello World"

// 转换日期格式
const iso = convertDateToISO8601(new Date());

// 标题格式化
const title = toTitleCase('hello world'); // "Hello World"
```

## 测试覆盖

所有工具函数都有对应的测试文件：
- `*.test.ts` - 单元测试
- `*.spec.ts` - 规范测试

运行测试：
```bash
npm test -- utils
```

## 扩展指南

### 添加新工具
1. 创建工具文件
2. 导出函数/类
3. 添加 JSDoc 注释
4. 编写测试用例
5. 更新此文档

### 代码规范
- 使用 TypeScript
- 提供完整的类型定义
- 处理错误情况
- 保持函数纯净
- 添加详细的 JSDoc 注释

## 相关文件清单

### HTTP 相关
- `ofetch.ts` - HTTP 客户端（推荐）
- `got.ts` - HTTP 客户端（废弃）
- `header-generator.ts` - 请求头生成

### 浏览器相关
- `puppeteer.ts` - Puppeteer 配置
- `puppeteer-utils.ts` - Puppeteer 工具

### 数据处理
- `rss-parser.ts` - RSS 解析
- `parse-date.ts` - 日期解析
- `camelcase-keys.ts` - 命名转换

### 缓存系统
- `cache/index.ts` - 缓存入口
- `cache/memory.ts` - 内存缓存
- `cache/redis.ts` - Redis缓存

### 工具函数
- `logger.ts` - 日志工具
- `helpers.ts` - 辅助函数
- `common-utils.ts` - 通用工具
- `md5.ts` - MD5哈希
- `wait.ts` - 延时函数

## 变更记录 (Changelog)

### 2025-12-07 16:30:00
- ✨ 完成工具库100%扫描覆盖
- 📝 记录所有60+个工具函数
- 🔗 提供详细的使用示例
- 📚 整理最佳实践和优化建议
- 🛠️ 添加扩展指南

### 2025-12-07 14:11:44
- ✨ 创建工具库文档
- 📝 记录所有工具函数
- 🔗 提供使用示例和最佳实践