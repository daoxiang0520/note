# daoxiang 笔记仓库

基于 Obsidian 的个人笔记系统，使用 PARA 方法论组织。

## 目录结构

| 目录 | 用途 |
|------|------|
| `0-Inbox/` | 快速捕获的笔记，待后续整理分发 |
| `1-Projects/` | 有明确目标和截止日期的项目笔记 |
| `2-Areas/` | 持续维护的责任领域（健康、财务、工作等） |
| `3-Resources/` | 参考资料、知识卡片、感兴趣的话题 |
| `4-Archives/` | 已完成或不再活跃的内容归档 |
| `5-Journals/` | 日记、周记、月记等时间线记录 |
| `Templates/` | Obsidian 模板文件 |
| `Attachments/` | 图片、文件等附件（含知识库） |

## 知识库（Attachments 内）

`Attachments/` 下已建立两个课程知识库，均采用以下结构：

```
知识库文件夹/
├── 知识库概览.md       ← 入口索引（wikilink 可直接跳转笔记）
├── notes/              ← 已转换的 Markdown 笔记（可读）
└── originals/          ← 原始课件归档（PDF/PPTX）
```

### 已有知识库

| 知识库 | 路径 | 笔记数 | 内容 |
|:-------|:-----|:------:|:-----|
| **CSAPP** | `Attachments/csapp/CSAPP 知识库概览.md` | 29 篇 | CMU 15-213 计算机系统（CH2~CH9 全套） |
| **数字逻辑与数字电路** | `Attachments/数字逻辑与数字电路/数字逻辑与数字电路 知识库概览.md` | 17 篇 | 武汉大学数字逻辑课程（含 Verilog） |

回答相关问题时优先查阅 `notes/` 下的 Markdown 笔记，原始文件在 `originals/` 中。

## 知识点总结（3-Resources 内）

`3-Resources/` 下有两个合并的知识点总结文件，每篇均为单文件全课程总结，YAML frontmatter 标注 `source: AI 生成`。

| 文件 | 说明 |
|:-----|:-----|
| `3-Resources/CSAPP/CSAPP.md`（6425 行，29 节） | CMU 15-213 计算机系统 CH2~CH9 + 活动 + 工具 |
| `3-Resources/数字逻辑与数字电路/数字逻辑与数字电路.md`（2564 行，16 节） | 武汉大学数字逻辑 Chap1~Chap5 + 复习 |

在 Obsidian 中打开这两个文件，大纲侧栏可显示完整的章节层级结构，方便快速跳转。

## 笔记约定

- 文件格式为 Markdown（`.md`）
- 内部链接使用 `[[双链语法]]`
- 笔记开头建议包含 YAML frontmatter，字段：`title`、`date`、`tags`
- 标签使用扁平中文格式，如 `#阅读` `#想法` `#项目`
- 文件名尽量用描述性中文

## AI 行为规则

- 回答和交流默认使用中文
- 新建笔记默认放入 `0-Inbox/`，等待后续整理
- 搜索内容时优先覆盖 `1-Projects/`、`2-Areas/`、`3-Resources/`
- 归档或删除操作前先确认
- 修改已有笔记时保持原文风格和结构
- 读取大量笔记时优先用 Glob/Grep 搜索，避免盲目遍历
