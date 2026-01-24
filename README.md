# 学术主页模板

这是一个基于 [Hugo](https://gohugo.io/) 的简约学术主页模板。
设计参考了简约风格，旨在提供清晰的信息展示和便捷的内容管理。

## 目录结构

*   `hugo.toml`: **站点配置文件**。在这里修改你的名字、头衔、社交链接、头像路径等。
*   `content/_index.md`: **关于我 (About)** 部分的文字内容。支持 Markdown。
*   `data/`: **数据文件夹**。用于管理列表项，格式为 YAML (简单易读)。
    *   `highlights.yml`: 重要动态/新闻。
    *   `publications.yml`: 发表的论文列表。
    *   `projects.yml`: 项目经历。
*   `static/images/`: 存放图片的文件夹。请将你的头像 (如 `avatar.jpg`) 放在这里。

## 如何使用

### 1. 本地预览

在终端中运行以下命令启动本地服务器：

```bash
hugo server
```

然后打开浏览器访问 `http://localhost:1313/`。
当你修改文件时，网页会自动刷新。

### 2. 修改内容

#### 基本信息 & 侧边栏
打开 `hugo.toml`，修改 `[params]` 下的内容：
```toml
name = "你的名字"
role = "博士生"
# ...
[[params.social]]
  name = "GitHub"
  url = "..."
```

#### 个人简介
打开 `content/_index.md`，像写文档一样编写你的简介。

#### 论文列表
打开 `data/publications.yml`，按照现有格式添加新的论文：
```yaml
- title: "论文标题"
  authors: "**你的名字**, 作者二, 作者三"
  venue: "会议名称 (CVPR 2024)"
  year: 2024
  links:
    - name: "PDF"
      url: "你的PDF链接"
      type: "primary"
```

#### 动态 & 项目
同样地，编辑 `data/highlights.yml` 和 `data/projects.yml`。

### 3. 发布 (Build)

运行命令生成静态文件：

```bash
hugo
```

生成的网页文件会在 `public/` 目录下。你可以将该目录的内容上传 to GitHub Pages 或其他托管服务。

## 自定义样式

如果你懂 CSS，可以在 `static/css/style.css` 中修改样式。
布局文件位于 `layouts/index.html`。
