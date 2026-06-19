# CLAUDE.md — ML 刷题软件项目指引

## 项目简介

为一个编程新手构建的 Windows 机器学习基础知识刷题软件。单 HTML 文件，双击浏览器打开，零依赖。包含概念选择题和代码拼图题（代码块排序）两种题型，题库约 200 道。

---

## 关键文件索引

### 规范文档（开发前必读）
| 文件 | 内容 | 何时读 |
|------|------|--------|
| [docs/requirements.md](docs/requirements.md) | 功能需求、非功能需求、排除项 | 了解"要做什么" |
| [docs/tech-spec.md](docs/tech-spec.md) | 数据格式、模块架构、判分算法 | 写代码前 |
| [docs/design-spec.md](docs/design-spec.md) | 配色、布局、交互规范 | 写 UI 前 |
| [docs/dev-plan.md](docs/dev-plan.md) | 分阶段执行步骤、验收标准 | 规划每日任务 |
| [docs/changelog.md](docs/changelog.md) | 变更记录 | 了解历史 |

### 开发日志
| 目录 | 说明 |
|------|------|
| [dev-log/](dev-log/) | 每日开发记录（格式：YYYY-MM-DD.md） |

### 代码文件
| 文件 | 说明 |
|------|------|
| [index.html](index.html) | 完整应用（HTML + CSS + JS） |

### 用户文档
| 文件 | 说明 |
|------|------|
| README.txt | 用户使用说明（阶段 6 创建） |
| [custom-questions.json](custom-questions.json) | 自定义题库模板（阶段 5 创建） |

---

## AI 工作原则

### 每次开发的标准流程

1. **读取规范** → 先读 `docs/tech-spec.md` 和 `docs/design-spec.md`，了解当前阶段的上下文
2. **执行单阶段** → 按 `docs/dev-plan.md` 中定义的阶段，一次只做一个阶段
3. **写完即验证** → 每阶段完成立刻验证，确认无误再进入下一阶段
4. **记录日志** → 在 `dev-log/` 中更新当日日志，记录已完成 + 待办
5. **更新 changelog** → 如有重要变更，更新 `docs/changelog.md`

### 代码风格
- HTML/CSS/JS 全部内嵌在一个文件中
- JS 使用 IIFE 命名空间 `MLQuiz`
- 中文注释 + 中文 UI 文字
- 代码块中的特殊字符使用 `escapeHTML()` 处理
- 不使用任何外部依赖（CDN、npm、字体文件等）

### 用户交互
- 用户是编程新手，解释时用通俗语言
- 每次完成一个阶段后，告知用户完成了什么、效果如何、下一步是什么
- 不问"是否继续"，而是告知进度后主动推进（除非有需要用户决策的事项）

---

## 当前状态

- **已完成**：阶段 0（项目初始化）+ HTML/CSS 骨架
- **下一步**：阶段 1（JS 数据层）
- **参考**：[dev-plan.md](docs/dev-plan.md) 查看完整路线图
