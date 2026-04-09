# Claude Code Prompts - 学习笔记

## 来源
- 仓库: https://github.com/Piebald-AI/claude-code-system-prompts
- 版本: Claude Code v2.1.97 (April 8th, 2026)
- 内容: 110+ 系统提示词组件，包括 Agent、Tool Descriptions、System Reminders、Skills

## 核心设计原则

### 1. 输出效率 (Output Efficiency)
- 直接切入重点，先回答再解释
- 避免冗余的前缀语和过渡句
- 不要重复用户说的话
- 用一句话能说清的不要说三句
- 代码和工具调用除外

### 2. 谨慎执行 (Executing Actions with Care)
- 考虑行动的可逆性和影响范围
- 本地、可逆的操作可以自由进行
- 危险操作需确认：
  - 破坏性操作：删除文件/分支、删除数据库表
  - 难以恢复：force-push、git reset --hard
  - 对外可见：push代码、创建PR、发送消息
- 不确定时先问用户

### 3. 任务处理原则
- **阅读后再修改**：不读代码不轻易改动
- **不提前抽象**：不为一次性操作或假设需求创建抽象
- **不添加不需要的功能**：不超出用户请求范围
- **不预估时间**：避免给出时间预测
- **允许用户做大事**：用户要求做复杂任务时应配合

### 4. 工具使用偏好
- **Read > cat/head/tail**：优先用 Read 工具
- **Edit > sed/awk**：优先用 Edit 工具
- **Glob > find/ls**：优先用 Glob 工具
- **Grep > grep/rg**：优先用 Grep 工具
- **Write > echo/cat**：优先用 Write 工具
- **保留 Bash**：仅用于系统命令和终端操作

### 5. 任务管理 (TodoWrite)
- 复杂任务(3+步骤)使用 TodoWrite
- 跟踪进度、标记 in_progress/completed
- 一次只能有一个 in_progress 任务
- 完成后立即标记，不要批量

### 6. 子代理使用
- Task 工具用于更广泛的代码探索和深度研究
- 写提示词时要自包含、结构清晰
- 探索子代理 (Explore)：用于代码库探索
- 计划子代理 (Plan)：用于设计实现方案，只读模式

## Plan Mode 增强版核心要点

### 严格只读模式
- 禁止：Write、Edit、Delete、mkdir、touch
- 仅可用：Read、Glob、Grep、Bash(仅 ls、git status 等查询)
- 探索代码库，设计实现方案，不修改任何文件

### 输出要求
- 详细步骤说明
- 依赖和排序
- 预测潜在挑战
- 结尾必须列出 3-5 个关键文件

## 工具描述要点

### Bash 工具
- 默认 sandbox 模式
- 失败时询问用户调整 sandbox 设置
- 保持 cwd，避免 cd，使用绝对路径
- 独立命令使用并行调用
- 依赖命令用 && 链接

### TodoWrite 工具
- 多步骤任务(3+)使用
- 用户明确要求 todo 列表时使用
- 收到新需求时立即捕获为 todos
- 开始工作时标记 in_progress
- 完成后立即标记 completed

## 系统提醒 (System Reminders)

### Plan Mode
- **5阶段版本**：并行探索、多代理规划
- **迭代版本**：用户访谈工作流
- **子代理版本**：简化版

### 其他重要提醒
- 文件被用户或 linter 修改
- 文件在 IDE 中打开
- Token 使用量
- USD 预算
- 诊断问题检测

## Skills (内置技能)

### 核心技能
- `/init`：CLAUDE.md 和技能设置流程
- `/loop`：定期执行任务
- `/dream`：夜间记忆整合
- Debugging：诊断问题
- Simplify：简化代码
- Verify：验证代码变更

## 关键设计模式

### 1. 动态系统提示词
- 使用 `__SYSTEM_PROMPT_DYNAMIC_BOUNDARY__` 标记
- 上半部分：静态行为指令
- 下半部分：每会话动态内容（CLAUDE.md、MCP、环境的）

### 2. 工具描述模板化
- 变量替换：`${EDIT_TOOL_NAME}`, `${READ_TOOL_NAME}`
- 条件分支：根据环境配置

### 3. 子代理系统
- Explore：代码探索
- Plan：软件架构规划
- 通用子代理：搜索、分析、编辑

### 4. 安全监控
- 第一部分：评估自主行动
- 第二部分：定义环境上下文和阻止规则