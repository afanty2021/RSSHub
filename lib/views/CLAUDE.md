[根目录](../../CLAUDE.md) > [lib](../) > **views**

# 视图层模块

## 模块职责

views 目录包含所有视图模板和渲染逻辑，负责：
- RSS/Atom XML 生成
- HTML 页面渲染
- 错误页面展示
- 响应模板处理

## 模板列表

### 1. index.tsx - 首页模板
渲染 RSSHub 首页，包含：
- 项目介绍
- 使用说明
- 热门路由展示
- 调试信息（开发模式）

### 2. rss.tsx - RSS 模板
生成标准的 RSS 2.0 格式 XML：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
    <channel>
        <title>频道标题</title>
        <link>频道链接</link>
        <description>频道描述</description>
        <item>...</item>
    </channel>
</rss>
```

### 3. atom.tsx - Atom 模板
生成标准的 Atom 格式 XML：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">
    <title>Feed 标题</title>
    <link href="Feed 链接" />
    <updated>更新时间</updated>
    <entry>...</entry>
</feed>
```

### 4. error.tsx - 错误页面
渲染错误信息页面：
- 错误代码
- 错误消息
- 帮助信息
- 返回链接

### 5. layout.tsx - 布局模板
提供基础布局组件：
- XML 声明
- 响应头设置
- 通用样式

## 渲染配置

### JSX 渲染器配置
```typescript
app.use(
    jsxRenderer(({ children }) => <>{children}</>, {
        docType: '<?xml version="1.0" encoding="UTF-8"?>',
        stream: {},
    })
);
```

### 响应类型设置
```typescript
// RSS/Atom 响应
ctx.header('Content-Type', 'application/xml; charset=utf-8');

// HTML 响应
ctx.header('Content-Type', 'text/html; charset=utf-8');
```

## RSS/Atom 数据模型

### 频道信息
```typescript
interface Channel {
    title: string;          // 频道标题
    link: string;          // 频道链接
    description: string;    // 频道描述
    language?: string;     // 语言代码
    pubDate?: Date;        // 发布日期
    lastBuildDate?: Date;  // 最后更新时间
    generator?: string;    // 生成器信息
    ttl?: number;         // 缓存时间（分钟）
    image?: {             // 频道图标
        url: string;
        title: string;
        link: string;
    };
}
```

### 项目信息
```typescript
interface Item {
    title: string;          // 标题
    description: string;    // 描述
    link: string;          // 链接
    pubDate: Date;         // 发布日期
    guid?: string;         // 唯一标识
    author?: string;       // 作者
    category?: string[];   // 分类
    comments?: string;     // 评论链接
    enclosure?: {          // 媒体附件
        url: string;
        type: string;
        length?: number;
    };
    source?: {            // 来源
        url: string;
        value?: string;
    };
}
```

## 使用示例

### RSS 生成示例
```typescript
import { RSSItem } from './rss';

const items: RSSItem[] = [
    {
        title: '文章标题',
        description: '文章描述',
        link: 'https://example.com/article/1',
        pubDate: new Date(),
        author: '作者名',
        category: ['技术', '前端'],
    }
];

return c.rss({
    title: '我的 RSS',
    link: 'https://example.com',
    description: 'RSS 描述',
    items,
    language: 'zh-cn',
});
```

### 错误页面示例
```typescript
import ErrorPage from './error';

return c.html(
    <ErrorPage
        code={404}
        message="页面未找到"
        description="请检查 URL 是否正确"
    />
);
```

## 模板扩展

### 自定义 RSS 模板
```typescript
// 创建自定义模板
const customRSS = (data: CustomRSSData) => (
    <rss version="2.0">
        <channel>
            <title>{data.title}</title>
            {/* 自定义字段 */}
            <customField>{data.custom}</customField>
            {data.items.map(item => (
                <item>
                    <title>{item.title}</title>
                    {/* 自定义渲染 */}
                </item>
            ))}
        </channel>
    </rss>
);
```

### 响应处理
```typescript
// 设置响应头
ctx.header('Cache-Control', `max-age=${cacheTime}`);
ctx.header('Content-Type', 'application/xml; charset=utf-8');

// 返回渲染结果
return ctx.body(rssXml);
```

## 性能优化

### 优化建议
1. 使用流式渲染
2. 合理设置缓存头
3. 压缩响应内容
4. 避免重复渲染

### 监控指标
- 渲染时间
- 响应大小
- 缓存命中率
- 错误率

## 测试策略

### 单元测试
- 模板渲染测试
- 数据格式验证
- 边界条件测试

### 集成测试
- 完整响应测试
- 浏览器兼容性
- RSS 验证器测试

## 相关文件清单

### 核心模板
- `index.tsx` - 首页模板
- `rss.tsx` - RSS 2.0 模板
- `atom.tsx` - Atom 1.0 模板
- `error.tsx` - 错误页面
- `layout.tsx` - 布局组件

### 资源文件
- `../assets/` - 静态资源
- `../templates/` - Art 模板文件

## 变更记录 (Changelog)

### 2025-12-07 14:11:44
- ✨ 创建视图层文档
- 📝 记录所有模板和渲染逻辑
- 🔗 提供使用示例和扩展指南