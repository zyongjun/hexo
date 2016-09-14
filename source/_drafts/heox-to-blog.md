---
title: heox-to-blog
categories: 其他
---

### 草稿
> $ hexo new draft "new draft"

source/_drafts目录下生成一个new-draft.md

#### 预览草稿，更改配置文件：

render_drafts: true

或者
>$ hexo server --drafts

发布草稿
>$ hexo publish [layout] filename

### 图片
2 解决方案

CodeFalling/hexo-asset-image

2.1 使用
_config.yml 中有post_asset_folder:true
npm install https://github.com/CodeFalling/hexo-asset-image --save
![logo](MacGesture2-Publish/logo.jpg)
