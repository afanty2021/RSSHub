[根目录](../../CLAUDE.md) > [lib](../) > **api**

# API 系统模块

## 模块职责

api 目录提供 RESTful API 接口，负责：
- 提供路由信息查询服务
- 支持 RSSHub Radar 浏览器扩展
- 提供分类和命名空间管理
- 生成 OpenAPI 文档

## 入口与启动

### 主要入口
- **index.ts** - API 路由聚合和 OpenAPI 配置
- 使用 **@hono/zod-openapi** 进行 API 路由定义
- 集成 **@scalar/hono-api-reference** 生成 API 文档

### API 前缀
所有 API 路由使用 `/api` 前缀，与 RSS 路由区分。

## 对外接口

### 核心 API 端点

#### 1. 命名空间管理
```
GET /api/namespace/all    # 获取所有命名空间
GET /api/namespace/:id    # 获取指定命名空间信息
```

#### 2. 分类管理
```
GET /api/category/:id     # 获取指定分类的路由列表
```

#### 3. RSSHub Radar
```
GET /api/radar/rules/all  # 获取所有 Radar 规则
GET /api/radar/rules/:id  # 获取指定 Radar 规则
```

#### 4. 关注功能
```
GET /api/follow/config    # 获取关注配置信息
```

#### 5. API 文档
```
GET /api/openapi.json     # OpenAPI 3.1.0 规范
GET /api/reference        # Swagger UI 界面
```

## 关键依赖与配置

### 核心依赖
- **@hono/zod-openapi** - OpenAPI 3.1 支持
- **@scalar/hono-api-reference** - API 文档界面
- **zod** - 运行时类型验证

### OpenAPI 配置
```typescript
const docs = app.getOpenAPI31Document({
    openapi: '3.1.0',
    info: {
        version: '0.0.1',
        title: 'RSSHub API',
    },
});
```

## 数据模型

### API 响应格式
统一使用 JSON 格式响应，支持标准 HTTP 状态码。

### 命名空间模型
```typescript
interface Namespace {
    name: string;          // 显示名称
    url: string;          // 官方网址
    description: string;   // 描述
    lang: string;         // 语言代码
    categories?: string[]; // 分类列表
}
```

### 路由信息模型
```typescript
interface RouteInfo {
    path: string;         // 路由路径
    name: string;         // 路由名称
    categories: string[]; // 分类
    example: string;      // 示例 URL
}
```

### Radar 规则模型
```typescript
interface RadarRule {
    title: string;        // 规则标题
    docs: string;         // 文档链接
    source: string;       // 源码位置
    target: string;       // 目标 URL
    example: string;      // 示例
}
```

## 测试与质量

### 测试策略
- API 接口单元测试
- 集成测试验证
- OpenAPI 规范验证
- 文档自动生成验证

### 质量保证
- TypeScript 类型检查
- Zod 运行时验证
- HTTP 状态码规范
- 响应格式一致性

## 使用示例

### 获取所有命名空间
```bash
curl https://rsshub.app/api/namespace/all
```

### 获取指定分类路由
```bash
curl https://rsshub.app/api/category/multimedia
```

### 查询 API 文档
访问: https://rsshub.app/api/reference

## 扩展指南

### 添加新 API
1. 在对应目录创建路由文件
2. 使用 OpenAPI 装饰器定义路由
3. 实现处理函数
4. 更新文档

### API 设计原则
- RESTful 设计风格
- 清晰的资源命名
- 一致的响应格式
- 完善的错误处理

## 相关文件清单

### 核心 API
- `index.ts` - API 路由聚合
- `namespace/` - 命名空间 API
- `category/` - 分类 API
- `radar/` - Radar 相关 API
- `follow/` - 关注功能 API

### 配置文件
- OpenAPI 配置
- Swagger UI 配置
- 类型定义文件

## 变更记录 (Changelog)

### 2025-12-07 14:11:44
- ✨ 创建 API 系统文档
- 📝 记录所有 API 端点
- 🔗 建立使用示例和扩展指南