[根目录](../../CLAUDE.md) > [lib](../) > **middleware**

# 中间件系统模块

## 模块职责

middleware 目录包含所有请求处理中间件，负责：
- 请求预处理和后处理
- 访问控制和权限管理
- 缓存管理和性能优化
- 日志记录和监控追踪

## 中间件列表

### 1. access-control - 访问控制
处理跨域请求和访问权限控制
- CORS 配置
- 请求头验证
- IP 白名单/黑名单

### 2. anti-hotlink - 热链保护
防止外部网站直接盗链资源
- Referer 检查
- 路径白名单/黑名单
- 模板替换支持

### 3. cache - 缓存管理
提供多级缓存支持
- 内存缓存
- Redis 缓存
- 缓存策略配置
- ETag 支持

### 4. debug - 调试支持
开发环境调试功能
- 请求/响应日志
- 性能计时
- 调试信息注入

### 5. header - 请求头处理
统一处理 HTTP 请求头
- User-Agent 设置
- 自定义请求头
- 头部信息标准化

### 6. logger - 日志记录
统一的请求日志记录
- 请求信息记录
- 响应时间统计
- 错误日志捕获

### 7. parameter - 参数处理
请求参数预处理和验证
- 参数解析
- 默认值设置
- 参数验证

### 8. sentry - 错误追踪
Sentry 错误监控集成
- 错误自动上报
- 性能监控
- 用户追踪

### 9. template - 模板处理
响应模板处理
- 内容类型设置
- 模板渲染
- 压缩处理

### 10. trace - 链路追踪
OpenTelemetry 集成
- 请求追踪
- 性能指标
- 分布式追踪

## 执行顺序

中间件按以下顺序执行：
1. trimTrailingSlash - 去除尾部斜杠
2. compress - 响应压缩
3. jsxRenderer - JSX 渲染器
4. mLogger - 日志记录
5. trace - 链路追踪
6. sentry - 错误追踪
7. access-control - 访问控制
8. debug - 调试支持
9. template - 模板处理
10. header - 请求头处理
11. anti-hotlink - 热链保护
12. parameter - 参数处理
13. cache - 缓存管理

## 配置选项

### 缓存配置
```typescript
interface CacheConfig {
    type: 'memory' | 'redis';
    expire: number;
    requestTimeout: number;
    contentExpire: number;
    memoryMax?: number;
    redisUrl?: string;
}
```

### 访问控制配置
```typescript
interface AccessControlConfig {
    allowOrigin?: string;
    allowMethods?: string[];
    allowHeaders?: string[];
    maxAge?: number;
    credentials?: boolean;
}
```

### 热链保护配置
```typescript
interface AntiHotlinkConfig {
    template?: string;
    includePaths?: string[];
    excludePaths?: string[];
    allowUserTemplate?: boolean;
}
```

## 使用示例

### 自定义中间件
```typescript
import { type Middleware } from 'hono';

const customMiddleware: Middleware = async (c, next) => {
    // 前置处理
    const start = Date.now();

    await next();

    // 后置处理
    const duration = Date.now() - start;
    c.header('X-Response-Time', `${duration}ms`);
};
```

### 条件中间件
```typescript
app.use('*', async (c, next) => {
    if (c.req.path.startsWith('/api/')) {
        await apiMiddleware(c, next);
    } else {
        await next();
    }
});
```

## 性能考虑

### 优化建议
1. 缓存中间件尽可能靠前
2. 压缩中间件在最后执行
3. 日志记录使用异步方式
4. 避免阻塞 I/O 操作

### 监控指标
- 响应时间分布
- 缓存命中率
- 错误率统计
- 并发请求数

## 测试策略

### 单元测试
- 中间件功能测试
- 配置参数测试
- 边界条件测试

### 集成测试
- 中间件链测试
- 请求流程验证
- 错误处理测试

## 扩展指南

### 添加新中间件
1. 创建中间件文件
2. 实现中间件逻辑
3. 添加配置支持
4. 编写测试用例
5. 更新文档

### 最佳实践
- 保持中间件独立性
- 提供配置选项
- 处理错误情况
- 添加类型定义

## 相关文件清单

### 核心中间件
- `access-control.ts` - 访问控制
- `anti-hotlink.ts` - 热链保护
- `cache.ts` - 缓存管理
- `debug.ts` - 调试支持
- `header.ts` - 请求头处理
- `logger.ts` - 日志记录
- `parameter.ts` - 参数处理
- `sentry.ts` - 错误追踪
- `template.ts` - 模板处理
- `trace.ts` - 链路追踪

### 测试文件
- 每个中间件都有对应的 `.test.ts` 文件

## 变更记录 (Changelog)

### 2025-12-07 14:11:44
- ✨ 创建中间件系统文档
- 📝 记录所有中间件和执行顺序
- 🔗 提供使用示例和扩展指南