# DeepSeek Harness 架构总览

> 面向非程序员的分层架构图。每一层是一个"区域"，箭头表示"谁调用谁 / 谁依赖谁"。

```mermaid
flowchart TB

    %% ================= 图例 =================
    subgraph LEGEND["图例 Legend"]
        direction LR
        L1["实线箭头 = 调用 / 请求"]
        L2["虚线箭头 = 承载 / 装载"]
        L3["🟦 = 入口　🟧 = 启动　🟥 = 大脑　🟩 = 能力　🟪 = 协作　🟫 = 数据"]
    end

    %% ================= 第 0 层：外部世界 =================
    subgraph WORLD["🌍 外部世界（人 / 其他程序 / 大模型）"]
        direction LR
        USER["🧍 人类用户"]
        EXT["🤖 其他程序"]
        DEEPSEEK["🧠 DeepSeek 等大模型<br/>(云端 API)"]
    end

    %% ================= 第 1 层：入口 =================
    subgraph ENTRY["🟦 ① 入口 —— 你怎么连进来"]
        direction LR
        UI["🖥️ 网页界面<br/>(Web GUI)"]
        CLI["⌨️ 命令行<br/>(CLI)"]
        SDK["🔌 程序接入<br/>(JSON-RPC SDK)"]
        ACP["⚙️ 自动化接入<br/>(ACP)"]
        HOOKS["🔗 外部工具桥接<br/>(Claude Code/Codex Hooks)"]
    end

    %% ================= 第 2 层：启动与组装 =================
    subgraph BOOT["🟧 ② 启动与组装 —— 决定装哪些零件"]
        DSH["🚀 dsh 启动器"]
        PROFILE["🧩 配置组合 Profile / Bundle<br/>(一份清单，指定用哪些插件)"]
        CORDIS["🏗️ Cordis 插件框架<br/>【核心】所有部件都是可插拔的插件"]
    end

    %% ================= 第 3 层：核心大脑 =================
    subgraph CORE["🟥 ③ 核心大脑 —— 思考和执行的地方"]
        LOOP["🔄 代理思考循环<br/>(Agent Loop)：想→动手→看结果"]
        SESS["🗒️ 会话记忆<br/>(Session 日志：聊天记录)"]
        PROMPT["📝 提示词组装<br/>(把指令+工具说明写给模型)"]
        TOOLS["🧰 工具注册表<br/>(登记所有能干的事)"]
        LLM_SEAM["🔌 大模型接口<br/>(LLM 适配器)"]
    end

    %% ================= 第 4 层：能力层 =================
    subgraph CAP["🟩 ④ 能力层 —— 代理的手和眼（可自由替换）"]
        direction LR
        FS["📁 文件读写"]
        SHELL["💻 命令行 / 终端"]
        SUBP["🔀 子进程"]
        WEB["🌐 联网搜索与抓取"]
        LSP["🔍 代码理解"]
        SKILL["📚 技能库"]
        SUBAGENT["👥 子代理<br/>(派小助手干活)"]
        SANDBOX["🛡️ 沙箱隔离<br/>(限制程序越界)"]
        JOB["⏳ 后台任务 / 工作流"]
    end

    %% ================= 第 5 层：人机协作 =================
    subgraph COLLAB["🟪 ⑤ 人机协作 —— 需要你拍板时"]
        ASK["🙋 审批 / 提问<br/>(需要你确认才能继续)"]
        PLAN["🗺️ 计划模式"]
        TODO["✅ 待办清单"]
        GOAL["🎯 目标管理"]
    end

    %% ================= 第 6 层：数据 =================
    subgraph DATA["🟫 ⑥ 数据与持久化 —— 记住一切"]
        PERSIST["💾 会话存储<br/>(JSONL / SQLite 文件)"]
        QUERY["🔎 会话检索"]
        SETTINGS["⚙️ 用户设置"]
        CRED["🔑 密钥凭据"]
    end

    %% ================= 底层支撑 =================
    subgraph FOUND["⬜ ⑦ 底层支撑 —— 大家都依赖的地基"]
        UTIL["🧱 公共工具库 (util)"]
        TYPERT["🧬 类型系统 (typert)"]
        VENDOR["📦 第三方框架依赖<br/>(vendor 目录)"]
    end

    %% ================= 连接关系 =================
    %% 外部 → 入口
    USER --- UI
    USER --- CLI
    EXT --- SDK
    EXT --- ACP

    %% 入口 → 启动器
    UI -->|启动| DSH
    CLI -->|启动| DSH
    SDK -->|启动| DSH
    ACP -->|启动| DSH
    HOOKS -->|启动| DSH

    %% 启动 → 组装
    DSH -->|读取| PROFILE
    PROFILE -.->|装载插件| CORDIS

    %% 组装 → 大脑
    CORDIS -.->|承载所有插件| LOOP
    CORDIS -.->|承载所有插件| CAP

    %% 大脑内部循环
    LOOP -->|读写记忆| SESS
    LOOP -->|组装| PROMPT
    LOOP -->|申请工具| TOOLS
    LOOP -->|请求模型| LLM_SEAM

    %% 大脑 → 模型
    LLM_SEAM -->|调用| DEEPSEEK

    %% 工具 → 能力层
    TOOLS -->|调用| FS
    TOOLS -->|调用| SHELL
    TOOLS -->|调用| WEB
    TOOLS -->|调用| LSP
    TOOLS -->|调用| SKILL
    TOOLS -->|调用| SUBAGENT
    TOOLS -->|调用| JOB

    %% 能力内部关系
    SHELL -->|底层用| SUBP
    FS -->|在沙箱里| SANDBOX
    SHELL -->|在沙箱里| SANDBOX
    SUBP -->|在沙箱里| SANDBOX

    %% 子代理复用大脑
    SUBAGENT -->|派生出新的| LOOP

    %% 协作
    LOOP -->|需要确认时| ASK
    ASK -->|弹出提问| UI
    LOOP -->|使用| PLAN
    LOOP -->|使用| TODO
    LOOP -->|使用| GOAL

    %% 数据
    SESS -->|保存到| PERSIST
    QUERY -->|查询| PERSIST
    LOOP -->|读取| SETTINGS
    LOOP -->|读取| CRED

    %% 底层支撑
    CORDIS -.->|基于| VENDOR
    LOOP -.->|依赖| UTIL
    CORDIS -.->|依赖| UTIL
    TYPERT -.->|支撑| CORDIS
```

## 一分钟读懂这张图

1. **入口（①）**：人类通过网页界面或命令行进来，其他程序通过 SDK/ACP 进来——入口只是"门"，不重要。
2. **启动组装（②）**：`dsh` 启动器按一份"插件清单"(Profile/Bundle) 把需要的零件拼起来。这套框架叫 **Cordis**，它的设计哲学是"**一切都是插件**"——连大脑本身也是插件，随时可换。
3. **核心大脑（③）**：真正的循环是"**想→动手→看结果**"（Agent Loop）。它一边读聊天记忆、一边向大模型提问，然后把模型想做的动作交给"工具注册表"。
4. **能力层（④）**：代理的"手和眼"——文件、命令行、联网、代码理解、技能库、子代理、沙箱。**这些能力都可以整体替换**，这正是插件架构的威力。
5. **人机协作（⑤）**：当代理想执行有风险的操作时，会停下来问你、展示计划，而不是擅自行动。
6. **数据（⑥）**：所有对话记录都存成文件（JSONL/SQLite），可以检索、续聊。
7. **地基（⑦）**：公共工具库、类型系统、第三方框架依赖，是上面所有层的共同地基。
