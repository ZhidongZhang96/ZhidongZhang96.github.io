# Zhidong Zhang - Personal Academic Website

这是一个基于 Hugo 构建的学术个人主页。
本指南将帮助你快速定位并修改网站的各个部分。

## 🚀 快速开始

在终端运行：

```bash
hugo server
```

打开浏览器访问 `http://localhost:1313/` 即可预览。修改文件后通常会自动刷新。

---

## 📝 内容修改指南

网站的内容由 **配置文件** (`.toml`)、**内容文件** (`.md`) 和 **数据文件** (`.yml`) 共同驱动。

### 1. 个人基本信息 (Sidebar)
**文件**: [`hugo.toml`](hugo.toml)
这是网站的核心配置文件。

*   **姓名与头衔**: 修改 `[params]` 下的 `name`, `role`, `organization`, `location`。
*   **头像**:
    1.  将你的照片放入 [`static/images/`](static/images/) 文件夹。
    2.  修改 `hugo.toml` 中的 `avatar` 路径 (例如: `avatar = "images/真人头像.jpg"`).
*   **社交链接**: 修改 `[[params.social]]` 部分。你可以添加 GitHub, LinkedIn, Twitter, Email 等。

### 2. "About Me" 简介
**文件**: [`content/_index.md`](content/_index.md)
*   这是首页中间的自我介绍部分。
*   使用标准 Markdown 语法编写。

### 3. 最新动态 (Highlights)
**文件**: [`data/highlights.yml`](data/highlights.yml)
*   以列表形式存储新闻动态。
*   **格式**:
    ```yaml
    - date: "2025-10-01"
      description: "新闻描述..."
      url: "可选的外部链接"
    ```

### 4. 论文发表 (Publications)
**文件**: [`data/publications.yml`](data/publications.yml)
*   展示你的学术成果。每篇论文是一个列表项。
*   **格式**:
    ```yaml
    - title: "论文标题"
      authors: "**Zhidong Zhang**, Co-author Name"  # 使用 ** ** 加粗你的名字
      venue: "会议/期刊名称 (SWC 2024)"
      year: 2024
      links:
        - name: "DOI"
          url: "https://doi.org/..."
        - name: "PDF"
          url: "files/paper.pdf" # 将PDF文件放在 static/files/ 目录下
    ```

### 5. 项目经历 (Projects)
**文件**: [`data/projects.yml`](data/projects.yml)
*   展示科研或工程项目。
*   支持 `badges` (徽章) 和 `links` (项目链接/Slides)。

### 6. 博客文章 (Blogs)
**目录**: [`content/blogs/`](content/blogs/)
*   **新建文章**: 建议复制该目录下现有的 `.md` 文件作为模板。
*   **Front Matter (头部元数据)**:
    ```yaml
    ---
    title: "文章标题"
    date: 2026-01-25
    tags: ["Category1", "Category2"]
    slug: "可选的URL别名, 默认为title的小写连字符形式"
    ---
    ```
    正文在 `---` 之后。

## 📂 文件结构速查

```text
├── hugo.toml            # 核心配置 (头像、侧边栏、菜单)
├── content/
│   ├── _index.md        # 首页介绍主文本
│   └── blogs/           # 博客文章内容
├── data/
│   ├── highlights.yml   # 动态数据
│   ├── publications.yml # 论文数据
│   └── projects.yml     # 项目数据
├── static/
│   ├── images/          # 图片静态资源
│   └── files/           # 附件 (如 PDF)
└── layouts/             # HTML 结构模版 (高阶修改)
```
