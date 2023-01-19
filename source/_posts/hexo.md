---
title: Hexo的使用
categories:
  - 开始
tags:
  - Hexo
abbrlink: e15610f1
date: 2022-07-05 20:39:47
---

摘录自 [知乎@Unity晚阳](https://zhuanlan.zhihu.com/p/156915260)

<!--more-->

### 文件的创建

```text
$ hexo new <title>
$ hexo new "我的第一篇文章"
```

### Front-matter

当我们创建一个md文件后，打开后会看到一些内容，这些称为`Front-matter`，它是文件最上方以 `---` 分隔的区域，用于指定个别文件的变量，举例来说：

```
  layout  布局  默认为true，如果你不想你的文章被处理，可以设置为false
  title  标题  标题会显示在最上方居中位置     
  date  建立日期    如果不指定则为默认值-文件创建日期，可以自定义。
  update  更新日期  如果不指定则为默认值-文件修改后重新生成静态文件的日期。
  comments  是否开启文章的评论功能 默认值为true
  tags  标签（不适用于页面page布局）
  categoreies  分类（不适用于页面page布局）
  permalink  覆盖文章网址
  keywords  仅用于 meta 标签和 Open Graph 的关键词（不推荐使用）
```

### 为文章添加分类与标签

只有文章（post布局）支持分类和标签，需要在`Front-matter`中设置。分类有层级关系，标签没有。

举个例子：
1）下面文章它的标签是：Hexo、博客
2）分类是： 个人博客 > Hexo博客
3）“Hexo博客” 是 “个人博客” 的子分类

```yaml
categories:
- 个人博客（第一层级）
- Hexo博客（第二层级）
tags:
- Hexo
- 博客
```

### 为文章添加多个分类

1）下面文章属于三个分类：日常 > 生活，日常 > 随想，日记
2）其中生活、随想为日常的子分类，日常和日记为同级分类

```yaml
categories:
- [日常, 生活]
- [日常, 随想]
- [日记]
```

## 基本操作

$ hexo clean

$ hexo g

$ hexo d

```csharp
hexo clean && hexo g && hexo d
```

