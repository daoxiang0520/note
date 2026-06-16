# daoxiang 笔记仓库 — Claude 指令

基于 Obsidian 的个人笔记系统，使用 PARA 方法论组织。

## 当前会话蒸馏（2026-06-16）

### 最新变更
- **CLAUDE.md → README.md**：本文件为新的 AI 指令，原内容已移至 README.md（仓库说明）
- **知识库镜像**：`3-Resources/CSAPP/CSAPP.md`（6425 行，29 节）和 `3-Resources/数字逻辑与数字电路/数字逻辑与数字电路.md`（2564 行，16 节）是完整的知识点合并总结，每节对应一个课件/章节，可在 Obsidian 大纲侧栏直接跳转
- **Git 推送修复**：`obsidian-git` 插件已安装，commit 信息配置为 `{{date}}`（仅日期）；已排除 `.obsidian/plugins/*/main.js`（含第三方 OAuth 凭据）
- **飞书图片修复**：`3-Resources/AI/AI相关.md` 中 43 张失效飞书链接已全部替换为本地图片 `Attachments/AI/img_*.png`
- **附件目录规范**：`Attachments/csapp/` 和 `Attachments/数字逻辑与数字电路/` 使用 `originals/`（原始 PDF/PPTX）+ `notes/`（可读 MD）+ `知识库概览.md`（索引）结构
- **历年试卷归档**：`Attachments/数字逻辑与数字电路/exam/` 新增 50+ 份历年期末试卷（06-07 ~ 24-25 学年），含试题与参考答案

### 知识库概览
| 位置 | 内容 | 来源 |
|:-----|:-----|:-----|
| `Attachments/csapp/notes/` | CSAPP 29 篇笔记（CH2~CH9 + 活动 + 工具） | CMU 15-213 原版课件 |
| `Attachments/数字逻辑与数字电路/notes/` | 数字逻辑 17 篇笔记（Chap1~Chap5 + 复习） | 武汉大学 2025-2026 课件 |
| `Attachments/数字逻辑与数字电路/exam/` | 50+ 份历年期末试卷（含答案） | 武汉大学 2006~2025 |
| `3-Resources/CSAPP/CSAPP.md` | 知识点总结合并版（29 节） | AI 生成 |
| `3-Resources/数字逻辑与数字电路/数字逻辑与数字电路.md` | 知识点总结合并版（16 节） | AI 生成 |
| `3-Resources/AI/AI相关.md` | AI/深度学习笔记（含本地图片） | 飞书导入 |
| `Attachments/AI/img_*.png` | 45 张本地化图片 | 飞书原文下载 |

### 文件分类
- `originals/` = 源课件归档（PDF/PPTX），不经修改
- `notes/` = 转换的 Markdown 笔记，按课件逐份对应，可读可搜
- `exam/` = 历年试卷归档（PDF），按学年命名，含试题与参考答案
- 合并版 MD（`3-Resources/` 下）= 知识点提炼，适合复习速查

## 目录结构

| 目录 | 用途 |
|------|------|
| `0-Inbox/` | 快速捕获的笔记，待后续整理分发 |
| `1-Projects/` | 有明确目标和截止日期的项目笔记 |
| `2-Areas/` | 持续维护的责任领域 |
| `3-Resources/` | 参考资料、知识卡片、知识点总结 |
| `4-Archives/` | 已完成或不再活跃的内容归档 |
| `5-Journals/` | 日记、周记、月记等时间线记录 |
| `Templates/` | Obsidian 模板文件 |
| `Attachments/` | 图片、文件、知识库（含 originals + notes） |

## 笔记约定

- 文件格式为 Markdown（`.md`）
- 内部链接使用 `[[双链语法]]`
- 笔记开头建议包含 YAML frontmatter：`title`、`date`、`tags`
- 标签使用扁平中文格式，如 `#阅读` `#想法` `#项目`
- 文件名尽量用描述性中文
- 知识点总结（`3-Resources/` 下）标注 `source: AI 生成`

## AI 行为规则

- 回答和交流默认使用中文
- 新建笔记默认放入 `0-Inbox/`，等待后续整理
- 搜索内容时优先覆盖 `1-Projects/`、`2-Areas/`、`3-Resources/`
- 归档或删除操作前先确认
- 修改已有笔记时保持原文风格和结构
- 读取大量笔记时优先用 Glob/Grep 搜索，避免盲目遍历
- 知识库提问优先查阅 `notes/` 下的 Markdown 笔记，或 `3-Resources/` 下的合并总结
