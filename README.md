# Codex Thread Supervisor (跨对话修复监视与协同技能)

[![Codex Skill](https://img.shields.io/badge/Codex-Skill-blue.svg)](https://github.com/hieght123/codex-thread-supervisor)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **专为 OpenAI Codex 桌面端设计的双对话协同监考技能**。
> 使用当前对话作为“监督方（考官）”，已有目标对话作为“执行方（工兵）”。在双方保留各自已选模型与推理档位的前提下，监督方自动定期巡检、独立阅读源码差分、打假虚假完成，并按授权自动将漏洞打回重修。

---

## 核心痛点

在长周期的工程修复、大模型编程或复杂系统迭代中，单对话自主执行往往面临以下困境：
1. **闭门造车与越修越偏**：执行模型在修复过程中容易遗忘最初的人类要求，把正常功能或既有测试断言悄悄删改以迎合单测通过；
2. **“假通过”与汇报泡沫**：模型容易只凭感觉宣布“已修复”，而实际代码并未处理异常边界或触及问题根因；
3. **人肉传话筒内耗**：用户在另一个思考对话中产生了新的约束或思路，必须反复在两个窗口之间人肉复制、转述和催促。

**Thread Supervisor** 建立了一套原生的“考官-工兵”协作机制，让监督方基于真实物理证据持续站岗，彻底释放用户的看守精力。

---

## 运行机制

```text
[ 人类用户 ]
     │
     │ 1. 授权口令：“监视这个对话：codex://threads/<ID>”
     ▼
[ 监督对话 (Supervisor) ]
     │
     ├── 2. 原生 Heartbeat 调度（默认每 3 分钟自动检查一次）
     ├── 3. wait_threads 紧凑快照监听（无实质变化时静默零打扰）
     ├── 4. 独立代码审查（物理读取目标工作区 git diff 与真实源码切片）
     │
     ▼ 5. 发现破损/漏洞时，调用 send_message_to_thread
[ 执行对话 (Worker) ] ──> 接收精准问题行号与修复建议 ──> 现场重修并重跑测试
```

### 核心设计原则
- **零额外并发与模型自主**：不另外派生多层代理套娃，不强制切换双方已配置的模型（例如监督方可用高推理模型，执行方用轻快模型）；
- **物理证据高于模型解释**：执行方的自述、日志或道歉只作线索；监督方只认真实的 git diff、函数输入输出与测试退出码；
- **静默零打扰**：目标对话未产出实质代码变动时，不汇报例行套话；只在出现有价值进展、确认漏洞或需要人工决策时打扰用户；
- **最小必要修复**：反对为了代码行数好看而隐藏关联缺陷，也反对无关代码大重构；严格聚焦于根因修复与回归防守。

---

## 快速安装与使用

### 1. 安装到本地 Codex Skills
将本项目克隆或拷贝至本机 Codex Skills 目录：

```bash
# Windows
git clone https://github.com/hieght123/codex-thread-supervisor.git %USERPROFILE%/.codex/skills/thread-supervisor

# macOS / Linux
git clone https://github.com/hieght123/codex-thread-supervisor.git ~/.codex/skills/thread-supervisor
```

安装后结构如下：
```text
~/.codex/skills/thread-supervisor/
├── SKILL.md              # 完整的监督协议、审计准则与 FSM 状态机
├── agents/
│   └── openai.yaml       # Codex 技能描述与隐式调用元数据
└── README.md
```

### 2. 日常调用口令
在任意新的或空闲的 Codex 对话中，直接发送：

- **标准监督（自动反馈并复查）**：
  > `监视这个对话：codex://threads/<目标ID>`
- **只读旁听（只汇报不发消息）**：
  > `监视这个对话：codex://threads/<目标ID>，只读汇报，别发消息。`
- **动态需求追加**：
  > `把刚才的新要求同步给修复对话，继续监督。`
- **生命周期控制**：
  > `汇报进度` / `暂停监督` / `停止监督`

---

## 协议与规范支持

- 完整遵循 Codex 原生自动化任务调度（`automation_update` 与 `wait_threads` 接口契约）；
- 严格遵循 `AGENTS.md` 人机边界与学习所有权规范；
- 兼容 Windows 与 Unix 换行符与字符集标准。

---

## License

[MIT License](LICENSE) © 2026 hieght123

