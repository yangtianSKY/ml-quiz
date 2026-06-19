# 技术规范 — ML 刷题软件

## 运行环境

- **平台**：Windows / macOS / Linux 桌面浏览器
- **浏览器**：Edge 90+、Chrome 90+、Firefox 90+
- **架构**：单文件 HTML（内嵌 CSS + JS），零依赖
- **字符编码**：UTF-8

---

## 题目数据格式（JSON Schema）

### 概念选择题

```json
{
  "id": "c001",
  "type": "concept",
  "difficulty": "easy",
  "topic": "基础概念",
  "question": "题目文字...",
  "options": ["A. 选项一", "B. 选项二", "C. 选项三", "D. 选项四"],
  "correct": 0,
  "explanation": "解释文字..."
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | string | 是 | 唯一标识，如 `c001` |
| type | string | 是 | 固定值 `"concept"` |
| difficulty | string | 是 | `"easy"` / `"medium"` / `"hard"` |
| topic | string | 是 | `"基础概念"` / `"经典算法"` / `"神经网络基础"` |
| question | string | 是 | 题目文字 |
| options | string[4] | 是 | 恰好 4 个选项 |
| correct | number | 是 | 正确选项索引（0-3） |
| explanation | string | 是 | 答案解析 |

### 代码拼图题

```json
{
  "id": "p001",
  "type": "codepuzzle",
  "difficulty": "medium",
  "topic": "神经网络基础",
  "question": "请排列代码块，完成...",
  "context": "可选的上下文说明（如已给的导入语句）",
  "correct_blocks": ["正确块1", "正确块2", "正确块3"],
  "distractor_blocks": ["迷惑块1", "迷惑块2"],
  "explanation": "解释文字..."
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | string | 是 | 唯一标识，如 `p001` |
| type | string | 是 | 固定值 `"codepuzzle"` |
| difficulty | string | 是 | `"easy"` / `"medium"` / `"hard"` |
| topic | string | 是 | 同上 |
| question | string | 是 | 题目说明 |
| context | string | 否 | 可选的上下文/前置信息 |
| correct_blocks | string[] | 是 | 有序的正确代码块数组 |
| distractor_blocks | string[] | 是 | 干扰代码块数组（可为空数组） |
| explanation | string | 是 | 答案解析 |

---

## JS 模块架构

```
MLQuiz (IIFE 命名空间，挂载到 window)
│
├── DataStore          # 会话状态容器
│   ├── questions[]    # 全部题目（内置 + 导入）
│   ├── filtered[]     # 筛选后的题目
│   ├── quizSet[]      # 本次答题的题目集合
│   ├── currentIndex   # 当前题目索引 (0-based)
│   ├── answers[]      # 答题记录 [{id, userAnswer, correct, score}]
│   ├── score          # 累计得分
│   ├── settings       # {difficulty, topics[], questionCount}
│   └── puzzleState    # 当前拼图题的状态
│
├── QuestionBank       # 数据管理
│   ├── loadDefaults()         # 加载内置题库
│   ├── importFromJSON(json)   # 导入外部题库
│   ├── validateQuestion(q)    # 校验单题
│   ├── filter(diff, topics)   # 筛选
│   ├── selectRandom(n)        # 随机抽题
│   └── shuffleArray(arr)      # Fisher-Yates 洗牌
│
├── UIRenderer         # DOM 渲染
│   ├── showScreen(name)       # 切换屏幕
│   ├── renderHome()           # 首页
│   ├── renderQuiz(question)   # 答题页（分发到具体题型）
│   ├── renderConcept(q)       # 概念选择题
│   ├── renderCodePuzzle(q)    # 代码拼图
│   ├── renderPuzzleAreas()    # 刷新拼图区域
│   ├── renderResults()        # 结果页
│   ├── showFeedback(...)      # 显示答题反馈
│   └── updateProgress()       # 更新进度条
│
├── QuizEngine         # 业务逻辑
│   ├── startQuiz()            # 开始答题
│   ├── selectOption(idx)      # 选择概念题选项
│   ├── selectBlock(idx)       # 选择代码块
│   ├── deselectBlock(idx)     # 退回代码块
│   ├── submitAnswer()         # 提交答案
│   ├── checkConcept(q, ans)   # 判分：概念题
│   ├── checkPuzzle(q, blocks) # 判分：代码拼图（部分给分）
│   ├── nextQuestion()         # 下一题
│   ├── skipQuestion()         # 跳过
│   └── calculateResults()     # 计算结果统计
│
└── EventHandler       # 事件管理
    ├── init()                 # 绑定所有事件
    ├── onDifficultyClick(e)   # 难度筛选点击
    ├── onTopicClick(e)        # 知识点筛选点击
    ├── onStart()              # 开始按钮
    ├── onImport()             # 导入题库
    ├── onOptionClick(e)       # 选项点击
    ├── onPoolBlockClick(e)    # 代码块池点击
    ├── onAnswerBlockClick(e)  # 答案区代码块点击
    ├── onSubmit()             # 提交
    ├── onSkip()               # 跳过
    ├── onNext()               # 下一题
    ├── onPlayAgain()          # 再来一次
    └── onGoHome()             # 返回首页
```

---

## 判分算法

### 概念题
- 选对：得分 = 1
- 选错：得分 = 0

### 代码拼图题
- 比较用户选中的块序列与 `correct_blocks`
- `score = 正确位置上的块数 / 总正确块数`
- 正确位置定义：索引 i 处，用户选的块内容 === correct_blocks[i]
- 如果用户多选了块（比正确块多），超过长度的部分不计分
- 得分为 0~1 之间的浮点数（精确到两位小数）

### 总得分
- 概念题每题满分 1 分
- 代码拼图题每题满分 1 分（部分得分按比例）
- 总得分 = 所有题得分之和
- 正确率 = 总得分 / 总题数 × 100%

### 等第评级
| 正确率 | 等第 |
|--------|------|
| ≥ 90% | A |
| ≥ 80% | B |
| ≥ 70% | C |
| ≥ 60% | D |
| < 60% | F |

---

## 数据流

```
页面加载
  → QuestionBank.loadDefaults()
  → UIRenderer.renderHome()
  → EventHandler.init()

用户点"开始刷题"
  → 读取筛选条件 → QuestionBank.filter() → QuestionBank.selectRandom()
  → DataStore.quizSet 就绪
  → UIRenderer.showScreen('quiz')
  → UIRenderer.renderQuiz(quizSet[0])

用户答题
  → 选择/操作 → DataStore 暂存
  → 提交 → QuizEngine.submitAnswer()
  → DataStore.answers 记录
  → UIRenderer.showFeedback()

用户点"下一题"
  → QuizEngine.nextQuestion()
  → 如果还有题 → renderQuiz(下一题)
  → 如果没题 → calculateResults() → renderResults()
```
