# 博客内容分类管理规则


我想我应该给博客设置一下分类规则，现在看着太乱了，以免以后文章多起来就难找了。

问题本质是：**分类（Category）应该稳定地回答“这篇文章是什么形式”，标签（Tag）应该灵活地回答“这篇文章关于什么”。**  
现在按学科分类，遇到的最大问题是很多内容跨学科、或者根本不是学科（比如个人思考、踩坑记录、艺术创作），所以会越来越乱。

应该改成：**分类按“内容形式/用途”分，标签按“领域+主题+工具/语言”分**。下面给你一套可以直接落地的规范。

---

## 一、推荐原则

1. **分类少而稳定**：控制在 5–8 个，每篇文章只放 1 个主分类。
2. **标签多而灵活**：每篇 2–5 个标签，用前缀命名空间区分维度。
3. **分类管“形式”，标签管“内容”**。
4. **统一命名**：标签建议用英文小写 + 连字符，避免 URL 编码问题；显示中文可以用 Hexo 的 `tag_map` 映射。

---

## 二、分类方案：按内容形式，不按学科

| 分类标识       | 显示名   | 适用范围                     | 你现有内容举例                                 |
| -------------- | -------- | ---------------------------- | ---------------------------------------------- |
| `notes`        | 学习笔记 | 学科知识、概念理解、读书笔记 | 高数笔记、电磁学、状态机、巴科斯范式           |
| `tutorials`    | 教程     | 系统教学、系列文章、手把手   | 《Re0：用C++入门深度学习》《CMake基础指南》    |
| `howto`        | 踩坑记录 | 问题解决、配置、报错         | 《如何解决C/C++乱码》《如何配置C/C++第三方库》 |
| `thoughts`     | 思考随笔 | 个人思考、人生、价值观       | 《我与死亡教育》《个人目标与人生价值》         |
| `creations`    | 创作     | 艺术、设计、写作、游戏       | 编曲、板绘、游戏设计                           |
| `reviews`      | 复习备考 | 考研、考试复习资料           | 《数据结构考研复习》                           |
| `explorations` | 奇思考据 | 脑洞、跨学科设想、考据       | 《三进制下的计算机》《ACG狐娘起源考究》        |

如果以后有完整的硬件项目记录，可以再加一个 `projects`，但现阶段这 7 个已经够用。

**判断分类的决策树：**

- 记录知识/概念 → `notes`
- 教别人怎么做 → `tutorials`
- 记录问题和解决过程 → `howto`
- 表达个人观点/感受 → `thoughts`
- 展示自己的创作作品 → `creations`
- 备考/复习材料 → `reviews`
- 脑洞/考据/跨学科猜想 → `explorations`

---

## 三、标签方案：前缀命名空间

标签用前缀区分维度，这样既能按学科聚合，又能按语言、工具、主题检索。

| 前缀       | 含义                  | 示例                                               |
| ---------- | --------------------- | -------------------------------------------------- |
| `field-*`  | 学科/领域             | `field-math`、`field-ai`、`field-neuroscience`     |
| `lang-*`   | 编程语言/硬件描述语言 | `lang-cpp`、`lang-python`、`lang-verilog`          |
| `tool-*`   | 工具/框架/平台        | `tool-cmake`、`tool-git`、`tool-fpga`              |
| `topic-*`  | 具体主题/概念/项目    | `topic-state-machine`、`topic-deep-learning`       |
| `status-*` | 知识状态（维护用）    | `status-draft`、`status-review`、`status-outdated` |

**标签词典种子：**

```text
field-math
field-physics
field-biology
field-neuroscience
field-cs
field-ai
field-philosophy
field-psychology
field-art
field-hardware

lang-c
lang-cpp
lang-python
lang-rust
lang-verilog
lang-vhdl

tool-cmake
tool-git
tool-hexo
tool-fpga
tool-embedded

topic-advanced-math
topic-mathematical-analysis
topic-field-theory
topic-discrete-math
topic-electromagnetism
topic-dynamics
topic-information-theory
topic-computation-theory
topic-deep-learning
topic-computer-vision
topic-llm
topic-state-machine
topic-bnf
topic-ternary-computer
topic-neural-network
topic-brain-science
topic-death-education
topic-music-composition
topic-digital-painting
topic-game-design
topic-data-structure

status-draft
status-review
status-outdated
status-evergreen
```

**使用规则：**

- 每篇标签 2–5 个。
- 领域标签 `field-*` 一般 1–2 个，不要堆太多。
- 分类已经表示“类型”了，标签里不必再写 `type-tutorial` 这类重复信息。
- 如果某个标签只有 1 篇文章，先考虑合并到更宽泛的标签里。

---

## 四、Front-matter 示例

```yaml
---
title: 《Re0：用C++入门深度学习》
date: 2024-03-01
categories: tutorials
tags:
  - field-ai
  - lang-cpp
  - topic-deep-learning
  - topic-neural-network
---
```

```yaml
---
title: 《如何解决C/C++乱码》
date: 2024-03-05
categories: howto
tags:
  - lang-c
  - lang-cpp
  - topic-encoding
---
```

```yaml
---
title: 《我与死亡教育》
date: 2024-04-10
categories: thoughts
tags:
  - field-philosophy
  - field-psychology
  - topic-death-education
---
```

```yaml
---
title: 《三进制下的计算机》
date: 2024-05-15
categories: explorations
tags:
  - field-cs
  - field-math
  - topic-ternary-computer
---
```

```yaml
---
title: 《数据结构考研复习》
date: 2024-06-01
categories: reviews
tags:
  - field-cs
  - topic-data-structure
---
```

---

## 五、Hexo 配置

在 `_config.yml` 中配置默认分类和名称映射，这样 URL 是英文，页面显示中文。

```yaml
default_category: notes

category_map:
  notes: 学习笔记
  tutorials: 教程
  howto: 踩坑记录
  thoughts: 思考随笔
  creations: 创作
  reviews: 复习备考
  explorations: 奇思考据

tag_map:
  field-math: 数学
  field-physics: 物理
  field-biology: 生物
  field-neuroscience: 神经科学
  field-cs: 计算机科学
  field-ai: 人工智能
  field-philosophy: 哲学
  field-psychology: 心理学
  field-art: 艺术
  field-hardware: 硬件
  lang-c: C
  lang-cpp: C++
  lang-python: Python
  lang-rust: Rust
  topic-state-machine: 状态机
  topic-bnf: 巴科斯范式
  topic-deep-learning: 深度学习
  # 其余按需添加
```

这样标签页 URL 是 `/tags/field-math/`，但页面标题显示“数学”。

---

## 六、旧文章迁移建议

1. **备份**：先把 `source/_posts` 整个备份。
2. **建立映射表**：把旧分类/标签对应到新系统。  
   例如旧分类“数学”“物理”“编程语言”→ 新分类 `notes`；旧标签“C++”→“lang-cpp + field-cs”。
3. **写脚本批量改 front-matter**：用 Python 的 `python-frontmatter` 库遍历 `.md` 文件，按映射表修改。
4. **检查标签页**：迁移后打开标签云，合并那些只有一篇的标签。
5. **以后写新文章用模板**：在 `scaffolds/post.md` 里预设：

```yaml
---
title: {{ title }}
date: {{ date }}
categories: notes
tags:
  - field-
  - topic-
---
```

---

## 七、

- **不再纠结“信息论属于物理还是数学”**：分类都是 `notes`，标签可以同时打 `field-math` 和 `field-physics`。
- **个人思考、艺术创作不再无处安放**：它们有独立的分类 `thoughts`、`creations`。
- **学科浏览仍然保留**：通过 `field-*` 标签页，依然可以点进“数学”“人工智能”看所有相关文章。
- **知识库更清晰**：`status-*` 标签可以帮你标记哪些文章已经过时、需要修订、属于常青内容。

如果想进一步做成“知识库”，可以手动维护一个索引页，按 `field-*` 标签链接到各个领域入口，这样导航会非常舒服。

---

> 作者: [SynJadeX](https://www.synjadex.top)  
> URL: https://www.synjadex.top/posts/2026-08-14-blog-content-classify-rule/  

