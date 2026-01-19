# Zotero Web API 模式使用指南

## 快速开始

### 前置要求

1. **Zotero 7** 已安装并运行
2. **思源笔记** 已安装此插件
3. （可选）**Better BibTeX** 插件用于生成 Citation Key

### 配置步骤

#### 1. 打开插件设置

在思源笔记中：
1. 点击顶部工具栏的"插件"图标
2. 找到"Citation"插件
3. 点击设置图标

#### 2. 配置数据库类型

在"基本设置"标签页：
1. **选择笔记本**: 选择存储文献笔记的笔记本
2. **设置文献库路径**: 例如 `/References/`
3. **数据库类型**: 选择 **"Zotero (Web API)"**
4. **使用 itemKey 作为索引**: ✓ 勾选（推荐）

#### 3. 重新载入数据库

点击"重新载入数据库"按钮，等待加载完成。

## 使用功能

### 1. 插入引用链接

**方法一：使用快捷键**
1. 在文档中输入 `/cite` 或使用快捷键
2. 在弹出的搜索框中输入关键词（标题、作者、年份）
3. 选择文献条目
4. 引用链接将自动插入

**方法二：从 Zotero 插入**
1. 在思源笔记中使用"插入引用"命令
2. 搜索并选择文献
3. 选择引用类型（如果配置了多种类型）

### 2. 创建文献笔记

1. 搜索并选择文献
2. 系统会根据模板自动生成文献笔记
3. 包含以下内容：
   - 标题、作者、年份
   - 摘要
   - DOI、URL等元数据
   - 附件链接
   - Zotero定位链接

### 3. 插入笔记内容

如果文献在 Zotero 中有笔记：
1. 使用"插入笔记"功能
2. 选择文献
3. 所有笔记内容将被插入到当前文档

## 配置模板

### 引用链接模板

在"思源内容模板 → 引用链接"中配置：

**默认模板**: `({{shortAuthor}} {{year}})`

**示例输出**: `(Smith et al. 2024)`

**可用变量**:
- `{{shortAuthor}}`: 简化作者名
- `{{year}}`: 年份
- `{{title}}`: 标题
- `{{citekey}}`: Citation Key
- `{{itemKey}}`: Zotero Item Key

### 文献内容模板

在"思源内容模板 → 文献内容"中配置：

```markdown
---
**Title**: {{title}}
**Author**: {{authorString}}
**Year**: {{year}}
**DOI**: {{DOI}}
---

# Abstract

{{abstract}}

# Files

{{files}}

# Select on Zotero

[Open in Zotero]({{zoteroSelectURI}})

# Notes

{{note}}
```

## 常见问题

### Q1: "Zotero 未运行"错误

**解决方法**:
1. 确保 Zotero 7 已打开
2. 检查 Zotero 是否在 23119 端口运行
3. 尝试在浏览器访问: `http://localhost:23119/api/users/0/items?limit=1`

### Q2: 找不到 Citation Key

**解决方法**:
1. 安装 Better BibTeX 插件
2. 在 Better BibTeX 设置中配置 Citation Key 格式
3. 右键点击文献 → Better BibTeX → Refresh BibTeX key
4. 或者使用 itemKey 模式（勾选"使用 itemKey 作为索引"）

### Q3: 附件链接无法打开

**解决方法**:
1. 检查附件是否存在于 Zotero 存储目录
2. 使用"在 Zotero 中定位"链接跳转到 Zotero 查看附件
3. 附件链接格式: `file:///path/to/file.pdf`

### Q4: 搜索速度慢

**解决方法**:
1. Web API 模式首次加载会获取所有文献，可能需要一些时间
2. 如果文献库很大（>1000条），考虑：
   - 使用"Zotero"搜索对话框类型（在debug-bridge设置中）
   - 或等待首次加载完成后，后续搜索会更快

## 技术细节

### itemKey vs citekey

| 特性 | itemKey | citekey |
|------|---------|---------|
| **来源** | Zotero 内置 | Better BibTeX 生成 |
| **格式** | 8位随机字符 (如 `BNQ6EZIN`) | 可自定义 (如 `smith2024`) |
| **稳定性** | 永不改变 | 可能改变 |
| **可读性** | 低 | 高 |
| **依赖** | 无 | 需要 Better BibTeX |

**推荐**: 使用 itemKey 模式更稳定可靠

### API 请求示例

```bash
# 获取所有文献
curl "http://localhost:23119/api/users/0/items?limit=100" \
  -H "Zotero-API-Version: 3"

# 获取单个文献
curl "http://localhost:23119/api/users/0/items/BNQ6EZIN" \
  -H "Zotero-API-Version: 3"

# 获取附件和笔记
curl "http://localhost:23119/api/users/0/items/BNQ6EZIN/children" \
  -H "Zotero-API-Version: 3"
```

### 数据索引格式

使用 itemKey 模式时，文献的唯一标识符格式为：
```
{libraryID}_{itemKey}
```

例如: `6690314_BNQ6EZIN`

这确保了即使在多个文库中也能正确识别文献。

## 高级配置

### 多种引用类型

在"引用链接"设置中，可以配置多种引用类型：

1. 点击"添加"按钮创建新的引用类型
2. 为每种类型配置：
   - 名称（如"脚注"、"正文"）
   - 引用模板
   - shortAuthor 限制
   - 多文献引用的前缀、连接符、后缀

3. 如果不勾选"直接使用第一种引用类型"，插入引用时会弹出选择菜单

### 自定义锚文本

- **静态锚文本**: 引用文本固定，不随文献信息更新
- **动态锚文本**: 引用文本会随文献信息更新（需要刷新）

## 性能优化建议

1. **首次加载**: 文献库较大时，首次加载需要时间，请耐心等待
2. **搜索优化**: 使用具体的关键词可以更快找到文献
3. **缓存**: 已加载的文献数据会缓存，重复使用不需要重新请求

## 支持与反馈

如有问题或建议，请访问：
- GitHub Issues: https://github.com/WingDr/siyuan-plugin-citation/issues
- 邮件: siyuan_citation@126.com
