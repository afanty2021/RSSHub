# API参考

<cite>
**本文档中引用的文件**  
- [index.ts](file://lib/api/index.ts)
- [one.ts](file://lib/api/category/one.ts)
- [all.ts](file://lib/api/namespace/all.ts)
- [one.ts](file://lib/api/namespace/one.ts)
- [all.ts](file://lib/api/radar/rules/all.ts)
- [one.ts](file://lib/api/radar/rules/one.ts)
- [config.ts](file://lib/api/follow/config.ts)
- [config.ts](file://lib/config.ts)
- [registry.ts](file://lib/registry.ts)
- [router.js](file://lib/router.js)
</cite>

## 目录
1. [简介](#简介)
2. [API端点规格](#api端点规格)
   - [命名空间API](#命名空间api)
   - [分类API](#分类api)
   - [雷达规则API](#雷达规则api)
   - [关注配置API](#关注配置api)
3. [API版本控制与向后兼容性](#api版本控制与向后兼容性)
4. [API使用示例](#api使用示例)
5. [错误响应](#错误响应)
6. [速率限制与认证](#速率限制与认证)
7. [API探索工具](#api探索工具)

## 简介

RSSHub是一个开源的RSS聚合服务，提供了一个全面的API接口来访问其丰富的RSS路由。本API参考文档详细介绍了RSSHub提供的所有公开API端点，包括HTTP方法、URL模式、请求参数、响应格式和状态码等信息。通过这些API，开发者可以程序化地获取RSSHub的路由信息、分类数据、雷达规则等元数据，从而构建更智能的RSS订阅体验。

RSSHub的API设计遵循RESTful原则，使用JSON作为数据交换格式，使得API易于理解和使用。API端点主要分为四类：命名空间API、分类API、雷达规则API和关注配置API，每类API都提供了特定的功能来满足不同的使用场景。

**Section sources**
- [README.md](file://README.md#L1-L62)
- [router.js](file://lib/router.js#L1-L800)

## API端点规格

### 命名空间API

命名空间API提供了对RSSHub中所有命名空间的访问，允许开发者获取单个或全部命名空间的信息。

#### 获取所有命名空间
```http
GET /api/namespace
```

**请求参数**
- 无

**响应格式**
```json
{
  "namespace_name": {
    "name": "命名空间名称",
    "description": "命名空间描述",
    "routes": {
      "/path": {
        "name": "路由名称",
        "example": "示例URL",
        "parameters": ["参数列表"],
        "categories": ["分类列表"],
        "features": {
          "requireConfig": [
            {
              "name": "配置名称",
              "optional": false
            }
          ]
        }
      }
    }
  }
}
```

**状态码**
- `200 OK`: 成功获取命名空间列表
- `500 Internal Server Error`: 服务器内部错误

#### 获取单个命名空间
```http
GET /api/namespace/{namespace}
```

**路径参数**
- `namespace`: 要获取的命名空间名称

**响应格式**
```json
{
  "name": "命名空间名称",
  "description": "命名空间描述",
  "routes": {
    "/path": {
      "name": "路由名称",
      "example": "示例URL",
      "parameters": ["参数列表"],
      "categories": ["分类列表"],
      "features": {
        "requireConfig": [
          {
            "name": "配置名称",
            "optional": false
          }
        ]
      }
    }
  }
}
```

**状态码**
- `200 OK`: 成功获取命名空间信息
- `404 Not Found`: 指定的命名空间不存在
- `500 Internal Server Error`: 服务器内部错误

**Section sources**
- [all.ts](file://lib/api/namespace/all.ts#L1-L20)
- [one.ts](file://lib/api/namespace/one.ts#L1-L36)
- [registry.ts](file://lib/registry.ts#L1-L272)

### 分类API

分类API允许开发者根据分类获取相关的命名空间和路由信息。

#### 按分类获取命名空间
```http
GET /api/category/{category}
```

**路径参数**
- `category`: 要查询的分类名称

**查询参数**
- `categories`: 附加的分类列表，用逗号分隔
- `lang`: 语言过滤器

**响应格式**
```json
{
  "namespace_name": {
    "name": "命名空间名称",
    "description": "命名空间描述",
    "lang": "语言",
    "routes": {
      "/path": {
        "name": "路由名称",
        "example": "示例URL",
        "parameters": ["参数列表"],
        "categories": ["分类列表"],
        "features": {
          "requireConfig": [
            {
              "name": "配置名称",
              "optional": false
            }
          ]
        }
      }
    }
  }
}
```

**状态码**
- `200 OK`: 成功获取分类信息
- `404 Not Found`: 指定的分类不存在
- `500 Internal Server Error`: 服务器内部错误

**Section sources**
- [one.ts](file://lib/api/category/one.ts#L1-L84)
- [registry.ts](file://lib/registry.ts#L1-L272)

### 雷达规则API

雷达规则API提供了对RSSHub雷达规则的访问，这些规则用于自动发现和订阅RSS源。

#### 获取所有雷达规则
```http
GET /api/radar/rules
```

**请求参数**
- 无

**响应格式**
```json
{
  "domain.com": {
    "_name": "命名空间名称",
    "subdomain": [
      {
        "title": "规则标题",
        "docs": "文档链接",
        "source": ["源路径列表"],
        "target": "目标路径"
      }
    ]
  }
}
```

**状态码**
- `200 OK`: 成功获取雷达规则
- `500 Internal Server Error`: 服务器内部错误

#### 获取特定域名的雷达规则
```http
GET /api/radar/rules/{domain}
```

**路径参数**
- `domain`: 要查询的域名

**响应格式**
```json
{
  "_name": "命名空间名称",
  "subdomain": [
    {
      "title": "规则标题",
      "docs": "文档链接",
      "source": ["源路径列表"],
      "target": "目标路径"
    }
  ]
}
```

**状态码**
- `200 OK`: 成功获取域名的雷达规则
- `404 Not Found`: 指定的域名没有相关规则
- `500 Internal Server Error`: 服务器内部错误

**Section sources**
- [all.ts](file://lib/api/radar/rules/all.ts#L1-L59)
- [one.ts](file://lib/api/radar/rules/one.ts#L1-L75)
- [registry.ts](file://lib/registry.ts#L1-L272)

### 关注配置API

关注配置API提供了RSSHub实例的配置信息，包括所有者用户ID、描述、价格等。

#### 获取关注配置
```http
GET /api/follow/config
```

**请求参数**
- 无

**响应格式**
```json
{
  "ownerUserId": "所有者用户ID",
  "description": "实例描述",
  "price": "价格",
  "userLimit": "用户限制",
  "cacheTime": "缓存时间（秒）",
  "gitHash": "Git提交哈希",
  "gitDate": "Git提交时间戳"
}
```

**状态码**
- `200 OK`: 成功获取配置信息
- `500 Internal Server Error`: 服务器内部错误

**Section sources**
- [config.ts](file://lib/api/follow/config.ts#L1-L30)
- [config.ts](file://lib/config.ts#L1-L800)

## API版本控制与向后兼容性

RSSHub的API设计注重向后兼容性，确保现有客户端在API更新时不会中断。API版本控制主要通过以下方式实现：

1. **语义化版本控制**: RSSHub遵循语义化版本控制规范，确保重大变更不会影响现有功能。
2. **路径稳定性**: API路径一旦发布，将保持稳定，不会随意更改。
3. **字段兼容性**: 在响应中添加新字段时，不会移除或修改现有字段，确保客户端可以安全地忽略未知字段。
4. **弃用策略**: 当需要移除功能时，会先标记为弃用，并在多个版本中保持兼容，给予开发者充分的迁移时间。

API的向后兼容性考虑还包括：
- 保持HTTP状态码的一致性
- 维持错误响应格式的稳定性
- 确保查询参数的向后兼容
- 提供详细的变更日志和迁移指南

**Section sources**
- [index.ts](file://lib/api/index.ts#L1-L36)
- [config.ts](file://lib/config.ts#L1-L800)

## API使用示例

以下示例展示了如何使用不同编程语言调用RSSHub API。

### 使用curl
```bash
# 获取所有命名空间
curl https://rsshub.app/api/namespace

# 获取GitHub命名空间
curl https://rsshub.app/api/namespace/github

# 按分类获取命名空间
curl https://rsshub.app/api/category/popular

# 获取所有雷达规则
curl https://rsshub.app/api/radar/rules

# 获取特定域名的雷达规则
curl https://rsshub.app/api/radar/rules/github.com

# 获取关注配置
curl https://rsshub.app/api/follow/config
```

### 使用JavaScript (Fetch API)
```javascript
// 获取所有命名空间
fetch('https://rsshub.app/api/namespace')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error('Error:', error));

// 获取GitHub命名空间
fetch('https://rsshub.app/api/namespace/github')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error('Error:', error));

// 按分类获取命名空间
fetch('https://rsshub.app/api/category/popular')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error('Error:', error));
```

### 使用JavaScript (Axios)
```javascript
const axios = require('axios');

// 获取所有命名空间
axios.get('https://rsshub.app/api/namespace')
    .then(response => {
        console.log(response.data);
    })
    .catch(error => {
        console.error('Error:', error);
    });

// 获取GitHub命名空间
axios.get('https://rsshub.app/api/namespace/github')
    .then(response => {
        console.log(response.data);
    })
    .catch(error => {
        console.error('Error:', error);
    });
```

### 使用Python (requests)
```python
import requests

# 获取所有命名空间
response = requests.get('https://rsshub.app/api/namespace')
print(response.json())

# 获取GitHub命名空间
response = requests.get('https://rsshub.app/api/namespace/github')
print(response.json())

# 按分类获取命名空间
response = requests.get('https://rsshub.app/api/category/popular')
print(response.json())
```

**Section sources**
- [index.ts](file://lib/api/index.ts#L1-L36)
- [all.ts](file://lib/api/namespace/all.ts#L1-L20)
- [one.ts](file://lib/api/namespace/one.ts#L1-L36)

## 错误响应

RSSHub API使用标准的HTTP状态码来表示请求结果。以下是常见的错误响应：

### 4xx客户端错误
- `400 Bad Request`: 请求参数无效或缺失
- `404 Not Found`: 请求的资源不存在
- `429 Too Many Requests`: 请求频率超过限制

### 5xx服务器错误
- `500 Internal Server Error`: 服务器内部错误
- `502 Bad Gateway`: 网关错误
- `503 Service Unavailable`: 服务不可用
- `504 Gateway Timeout`: 网关超时

错误响应格式：
```json
{
  "error": "错误消息",
  "status": "HTTP状态码",
  "timestamp": "时间戳"
}
```

开发者在处理API调用时应妥善处理这些错误情况，实现适当的重试机制和错误恢复策略。

**Section sources**
- [errors/index.ts](file://lib/errors/index.ts)
- [errors/types/*.ts](file://lib/errors/types/)

## 速率限制与认证

### 速率限制策略

RSSHub实施了速率限制策略以保护服务器资源和确保服务质量。默认情况下，API请求受到以下限制：

- 每分钟请求数限制
- 每小时请求数限制
- 每天请求数限制

具体的速率限制值可能因部署实例而异。当请求超过限制时，服务器将返回`429 Too Many Requests`状态码，并在响应头中包含`Retry-After`字段，指示客户端应在多久后重试。

建议开发者实现以下最佳实践：
- 实现指数退避重试机制
- 缓存API响应以减少重复请求
- 批量处理请求以减少API调用次数
- 监控请求频率并调整使用模式

### 认证机制

目前，RSSHub的公共API端点不需要认证，可以匿名访问。然而，某些私有部署或特定功能可能需要认证。

对于需要认证的场景，RSSHub支持以下认证方式：
- API密钥认证：通过`Access-Key`请求头提供API密钥
- 令牌认证：通过`Authorization`请求头提供Bearer令牌

认证相关的配置可以通过环境变量进行设置，如`ACCESS_KEY`等。

**Section sources**
- [config.ts](file://lib/config.ts#L1-L800)
- [middleware/access-control.ts](file://lib/middleware/access-control.ts)

## API探索工具

RSSHub提供了多种API探索工具，帮助开发者更好地理解和使用API。

### 交互式API控制台

RSSHub集成了Scalar API文档生成器，提供了一个交互式的API控制台。开发者可以通过访问`/api/reference`路径来使用这个工具，它提供了以下功能：
- API端点的可视化浏览
- 在浏览器中直接测试API调用
- 自动生成的请求代码示例
- 响应格式的实时预览

### OpenAPI规范

RSSHub的API遵循OpenAPI 3.1规范，开发者可以通过访问`/api/openapi.json`获取完整的API规范文档。这个JSON文件包含了所有API端点的详细描述，可以用于：
- 生成客户端SDK
- 创建API文档
- 进行API测试
- 集成到各种开发工具中

### API参考文档

除了本参考文档外，RSSHub还提供了详细的API参考文档，包含：
- 每个API端点的详细说明
- 请求和响应示例
- 参数验证规则
- 错误代码解释

这些工具共同构成了一个完整的API生态系统，帮助开发者快速上手并有效使用RSSHub API。

**Section sources**
- [index.ts](file://lib/api/index.ts#L1-L36)
- [config.ts](file://lib/config.ts#L1-L800)