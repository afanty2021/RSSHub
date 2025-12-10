# 错误处理API

<cite>
**本文档引用的文件**  
- [errorHandler.ts](file://lib/errors/index.tsx)
- [not-found.ts](file://lib/errors/types/not-found.ts)
- [invalid-parameter.ts](file://lib/errors/types/invalid-parameter.ts)
- [captcha.ts](file://lib/errors/types/captcha.ts)
- [reject.ts](file://lib/errors/types/reject.ts)
- [request-in-progress.ts](file://lib/errors/types/request-in-progress.ts)
- [config-not-found.ts](file://lib/errors/types/config-not-found.ts)
- [sentry.ts](file://lib/middleware/sentry.ts)
- [logger.ts](file://lib/middleware/logger.ts)
- [trace.ts](file://lib/middleware/trace.ts)
</cite>

## 目录
1. [简介](#简介)
2. [错误分类体系](#错误分类体系)
3. [错误响应格式](#错误响应格式)
4. [自定义错误处理中间件](#自定义错误处理中间件)
5. [错误日志记录最佳实践](#错误日志记录最佳实践)
6. [错误监控集成方案](#错误监控集成方案)
7. [结论](#结论)

## 简介
RSSHub 是一个开源的 RSS 生成器，其错误处理机制设计用于捕获、分类和响应各种运行时异常。本文档全面介绍 RSSHub 的错误处理 API，涵盖错误分类、标准化响应格式、自定义中间件开发、日志记录和监控集成等核心方面。通过本指南，开发者可以深入理解系统如何处理客户端请求错误、服务器内部错误以及第三方服务调用失败，并学习如何扩展默认行为以满足特定需求。

**Section sources**
- [errorHandler.ts](file://lib/errors/index.tsx#L1-L82)

## 错误分类体系
RSSHub 将错误分为多个类别，每类对应不同的异常场景和 HTTP 状态码。主要错误类型包括：

- **客户端错误**：由用户输入或请求参数不当引起，如无效参数、未找到资源等。
- **服务器错误**：系统内部异常，如网络请求失败、正在进行中的请求冲突等。
- **第三方服务错误**：与外部 API 通信时发生的错误，如超时、认证失败等。

这些错误通过继承 JavaScript 的 `Error` 类实现，并在 `lib/errors/types/` 目录下定义。例如：
- `InvalidParameterError` 表示请求参数不合法
- `NotFoundError` 表示路由不存在
- `CaptchaError` 表示需要验证码验证
- `RejectError` 表示访问被拒绝（如鉴权失败）
- `RequestInProgressError` 表示同一路径的请求正在处理中
- `ConfigNotFoundError` 表示必要配置缺失

错误处理器根据构造函数名称判断错误类型并设置相应的 HTTP 状态码。

**Section sources**
- [invalid-parameter.ts](file://lib/errors/types/invalid-parameter.ts#L1-L6)
- [not-found.ts](file://lib/errors/types/not-found.ts#L1-L6)
- [captcha.ts](file://lib/errors/types/captcha.ts#L1-L6)
- [reject.ts](file://lib/errors/types/reject.ts#L1-L6)
- [request-in-progress.ts](file://lib/errors/types/request-in-progress.ts#L1-L6)
- [config-not-found.ts](file://lib/errors/types/config-not-found.ts#L1-L6)

## 错误响应格式
RSSHub 的错误响应遵循标准化格式，确保前后端交互的一致性。响应内容根据请求参数 `format=json` 或打包模式决定输出 JSON 还是 HTML 页面。

### JSON 响应格式
当请求包含 `format=json` 或处于打包模式时，返回如下 JSON 结构：
```json
{
  "error": {
    "message": "错误信息"
  }
}
```

### HTML 响应格式
在普通模式下，返回包含详细错误信息的 HTML 页面，展示请求路径、错误消息、匹配路由及 Node.js 版本等调试信息。

HTTP 状态码映射规则如下：
- `HTTPError`, `RequestError`, `FetchError` → 503 Service Unavailable
- `RequestInProgressError` → 503 并设置缓存头
- `RejectError` → 403 Forbidden
- `NotFoundError` → 404 Not Found
- 其他未识别错误 → 503

此外，错误消息在生产环境中仅显示 `error.message`，而在开发环境中会包含完整的堆栈信息。

**Section sources**
- [errorHandler.ts](file://lib/errors/index.tsx#L46-L82)

## 自定义错误处理中间件
开发者可以通过创建自定义中间件来扩展默认的错误处理行为。RSSHub 使用 Hono 框架，支持标准的中间件模式。

### 中间件开发步骤
1. 创建一个新的中间件文件，导出类型为 `MiddlewareHandler` 的函数
2. 在函数体内调用 `next()` 执行后续逻辑
3. 使用 `try-catch` 捕获异常并进行自定义处理
4. 可结合 `ctx` 对象修改响应头、状态码或返回内容

### 示例：超时监控中间件
```ts
const timeoutMiddleware: MiddlewareHandler = async (ctx, next) => {
    const start = Date.now();
    await next();
    const duration = Date.now() - start;
    if (duration > 5000) {
        console.warn(`Slow request: ${ctx.req.path} took ${duration}ms`);
    }
};
```

此类中间件可插入到应用流水线中，实现性能监控、请求重试、错误转换等功能。

**Section sources**
- [sentry.ts](file://lib/middleware/sentry.ts#L17-L28)
- [logger.ts](file://lib/middleware/logger.ts#L29-L45)
- [trace.ts](file://lib/middleware/trace.ts#L7-L26)

## 错误日志记录最佳实践
RSSHub 集成了结构化日志记录机制，确保所有错误都被有效追踪。

### 日志级别与格式
- 使用 `logger.error()` 记录错误事件，包含请求路径和完整错误信息
- 使用 `logger.info()` 输出请求进出日志，包含方法、路径、状态码和响应时间
- 状态码通过颜色编码区分：2xx 绿色、3xx 青色、4xx 黄色、5xx 红色

### 调试信息收集
系统维护一个调试对象，统计以下指标：
- 总错误数 (`debug.error`)
- 各路径错误次数 (`debug.errorPaths`)
- 各路由错误次数 (`debug.errorRoutes`)
- 缓存命中情况 (`debug.hitCache`)

这些数据可用于分析高频错误路径和优化系统稳定性。

**Section sources**
- [errorHandler.ts](file://lib/errors/index.tsx#L70-L71)
- [logger.ts](file://lib/middleware/logger.ts#L33-L42)

## 错误监控集成方案
RSSHub 支持与 Sentry 等第三方监控平台集成，实现错误的集中管理和实时告警。

### Sentry 集成配置
通过环境变量 `SENTRY_DSN` 启用 Sentry 监控。系统在启动时初始化 Sentry 客户端，并设置节点名称标签。

### 错误上报机制
- 所有捕获的异常通过 `Sentry.captureException()` 上报
- 设置 `name` 标签为请求路径的第一级（如 `/github/trending` 的 `github`）
- 路由执行时间超过阈值时，自动上报 `Route Timeout` 异常

### 监控中间件
`sentry.ts` 中间件监控每个请求的执行时间，若超过 `config.sentry.routeTimeout` 配置值，则触发超时报错上报，帮助识别性能瓶颈。

此集成方案使得开发者能够在生产环境中快速定位问题根源，提升系统可观测性。

**Section sources**
- [sentry.ts](file://lib/middleware/sentry.ts#L8-L28)
- [errorHandler.ts](file://lib/errors/index.tsx#L39-L43)

## 结论
RSSHub 的错误处理机制设计严谨，具备良好的可扩展性和可观测性。通过清晰的错误分类、标准化的响应格式、灵活的中间件架构以及与 Sentry 的深度集成，系统能够有效地应对各类异常情况。开发者可基于现有框架轻松实现自定义错误处理逻辑，同时利用日志和监控工具快速诊断和修复问题，保障服务的稳定运行。