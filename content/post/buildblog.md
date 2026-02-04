+++
title = 'Buildblog'
date = 2025-02-27T17:31:27+08:00
draft = true
+++
# 使用hugo和stack主题构建博客页面

说实话，很多同学都是从博客页面起步开始，走上web开发的不归之路。但我大一的时候还没有这样先进的技术视阈，直接打开csdn开写。虽然也没有写几篇就是了。但坏处就是我真的没有给自己构建一个在线博客，直到这几天痛定思痛，觉得有点丢人了，才算想起来弄一下。本来准备jekyll构建一下得了，但被铺天盖地的软文劝退，最终还是准备用之前学弟推荐的hugo，至于主题，那是theme上找的，挑了个顺眼的而已。但拿来做博客还是足够了

# 项目注册

Github上注册项目，项目名对应用户名和github.io的域名。

# 本地预览博客（Hugo）

## 进入项目目录

```bash
cd /Users/linzheyuan/MartyLinZY.github.io
```

## 安装 Hugo（如未安装）

使用 Homebrew：

```bash
brew install hugo
```

验证安装：

```bash
hugo version
```

## 启动本地预览服务器

- **正常预览（不包含草稿 `draft = true` 的文章）**：

```bash
hugo server
```

- **包含草稿文章的预览**：

```bash
hugo server -D
```

启动成功后，终端会输出一个地址，一般是：

```text
http://localhost:1313
```

在浏览器中打开该地址，即可本地预览博客。修改任意 `content/` 下的 Markdown 文件并保存后，页面会自动刷新显示最新内容。
