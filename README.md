# FPGA 开发 Agent：基于大语言模型的端到端 FPGA 开发自动化

**学术项目文档** | 创建：2026-03-18 | 版本：v0.7 | 更新：2026-03-19

---

## 摘要

本项目探索利用大语言模型（LLM）实现端到端的 FPGA 开发自动化。通过从高层抽象（自然语言/Spec）到低层实现（比特流）的全流程 AI 辅助，旨在降低 FPGA 开发门槛、加速设计迭代、优化设计质量。

**核心贡献**：
1. **C++ 模型优先的开发流程**：将功能验证前置，降低返工成本
2. **多 Agent 并行的设计空间探索**：自动探索架构方案，寻找最优 PPA
3. **Vivado 原生报告驱动的迭代**：充分利用 Vivado/Vitis 原生能力，自定义脚本仅负责解析报告和决策支持
4. **文本输出驱动的仿真验证**：Agent 不读波形，只读文本日志/CSV/覆盖率报告
5. **面向长上下文的压缩与分层管理**：解决 FPGA 开发流程长、上下文累积过多的问题
6. **基于反馈的提示词自进化系统**：持续优化 Agent 表现，无需微调模型

**关键设计原则**：
- ✅ 依赖 Vivado/Vitis 原生仿真、综合、时序分析
- ✅ 自定义脚本仅解析报告、提取指标、支持决策
- ✅ Agent 读文本（log/csv/rpt），不读波形（.wdb）
- ✅ Testbench 内置断言和打印，输出结构化日志
- ❌ 不重复造轮子（不自己实现仿真器/综合工具）

**关键词**: FPGA、大语言模型、高层次综合、设计空间探索、自动化、Vivado 原生流程

---

## 目录

1. [引言](#1-引言)
   - [1.1 研究背景](#11-研究背景)
   - [1.2 研究目标](#12-研究目标)
   - [1.3 主要贡献](#13-主要贡献)
2. [相关工作](#2-相关工作)
   - [2.1 LLM 在代码生成中的应用](#21-llm-在代码生成中的应用)
   - [2.2 高层次综合（HLS）](#22-高层次综合 hls)
   - [2.3 AI 辅助 EDA](#23-ai-辅助 eda)
   - [2.4 与现有工作的区别](#24-与现有工作的区别)
3. [方法论](#3-方法论)
   - [3.1 整体架构](#31-整体架构)
   - [3.2 C++/HLS CoSim 流程](#32-chls-cosim-流程)
   - [3.3 Vivado 原生报告驱动的迭代流程](#33-vivado-原生报告驱动的迭代流程)
   - [3.4 子模块集成与系统组装](#34-子模块集成与系统组装)
     - [3.4.1 模块划分策略](#341-模块划分策略)
     - [3.4.2 接口定义与验证](#342-接口定义与验证)
     - [3.4.3 顶层集成](#343-顶层集成)
     - [3.4.4 系统级仿真](#344-系统级仿真)
     - [3.4.5 Sub-Agent 协调机制](#345-sub-agent-协调机制)
     - [3.4.6 系统级测试方法](#346-系统级测试方法)
   - [3.5 多 Sub-Agent 并行的设计空间探索](#35-多 sub-agent 并行的设计空间探索)
   - [3.6 上下文压缩与分层管理](#36-上下文压缩与分层管理)
   - [3.7 提示词自进化系统](#37-提示词自进化系统)
4. [技术实现](#4-技术实现)
   - [4.1 核心功能模块详解](#41-核心功能模块详解)
   - [4.2 工具链集成](#42-工具链集成)
     - [4.2.1 核心原则](#421-核心原则)
     - [4.2.2 Vivado/Vitis 原生流程集成](#422-vivadovitis-原生流程集成)
     - [4.2.3 报告解析脚本](#423-报告解析脚本)
     - [4.2.4 仿真工具集成](#424-仿真工具集成)
   - [4.3 错误处理与自动修复](#43-错误处理与自动修复)
   - [4.4 用户交互设计](#44-用户交互设计)
   - [4.5 版本管理与协作](#45-版本管理与协作)
5. [评估计划](#5-评估计划)
   - [5.1 评估指标](#51-评估指标)
   - [5.2 基准测试](#52-基准测试)
   - [5.3 消融实验](#53-消融实验)
6. [当前状态与计划](#6-当前状态与计划)
7. [讨论](#7-讨论)
8. [讨论与开放问题](#8-讨论与开放问题)
9. [研究问题（Research Questions）](#9-研究问题 research-questions)
10. [资源需求](#10-资源需求)
11. [风险管理](#11-风险管理)
12. [结论](#12-结论)
13. [参考文献](#参考文献)

---

## 1. 引言

### 1.1 研究背景

FPGA 开发是一个复杂的多阶段流程，涉及 RTL 设计、仿真验证、综合实现、布局布线、时序收敛和调试验证。传统开发模式存在以下挑战：

1. **学习曲线陡峭**：工程师需要掌握硬件描述语言、时序约束、工具链等，培养周期通常需 2-3 年
2. **迭代周期长**：综合和布局布线耗时数小时，设计迭代缓慢
3. **优化依赖经验**：布局布线参数调优、时序收敛等高度依赖工程师经验
4. **调试成本高**：波形分析和问题定位消耗大量时间

近年来，大语言模型在代码生成领域取得了显著进展，为 FPGA 开发自动化提供了新的可能性。

### 1.2 研究目标

本项目的核心目标是实现**端到端的 FPGA 开发自动化**：

```
输入：自然语言需求 或 形式化 Spec
输出：满足需求的比特流文件
```

### 1.3 主要贡献

1. 提出 C++ 模型优先的开发流程，将功能验证前置
2. 设计多 Sub-Agent 并行的架构探索框架
3. 开发面向 FPGA 开发的上下文压缩与分层管理系统
4. 实现基于反馈的提示词自进化机制

---

## 2. 相关工作

### 2.1 LLM 在代码生成中的应用

- **CodeX** (Chen et al., 2021): 基于 GPT 的代码生成模型
- **StarCoder** (Li et al., 2023): 面向代码的开源大模型
- **CodeLlama** (Rozière et al., 2023): Llama 的代码专用版本

### 2.2 高层次综合（HLS）

- **Xilinx Vitis HLS**: 商业 HLS 工具，支持 C/C++ 到 RTL 转换
- **Intel HLS Compiler**: Intel FPGA 的 HLS 解决方案
- **LegUp** (Canis et al., 2011): 学术界开源 HLS 工具

### 2.3 AI 辅助 EDA

- **AI 用于布局布线**: 近期研究表明强化学习可用于宏模块布局
- **AI 用于逻辑综合**: 探索利用 ML 预测综合结果
- **LLM 用于 RTL 生成**: 初步探索，但质量仍有待提升

### 2.4 与现有工作的区别

| 工作 | 范围 | 自动化程度 |
|------|------|-----------|
| HLS 工具 | C++→RTL | 半自动（需人工写 C++） |
| AI for EDA | 单点优化 | 辅助决策 |
| **本工作** | **需求→比特流** | **端到端自动** |

---

## 3. 方法论

### 3.1 整体架构

```
需求 → Spec → C++/HLS → RTL → 布局布线 → 比特流 → 调试
 ↑    │      │         │       │         │        │
 │    └──────┴─────────┴───────┴─────────┴────────┘
 │                    AI Agent 辅助
 └─────────────────── 闭环反馈
```

**C++/HLS 阶段说明**：
- C++ 模型与 HLS 在本项目中视为同一阶段
- 只需运行到 **CoSim（C 仿真）** 阶段，无需完整 HLS 综合
- CoSim 提供功能验证 + 架构信息，速度更快

**RTL 生成详细流程**：
```
C++ 模型 → HLS CoSim → CoSim 报告/架构信息 ─┐
                                           ├→ LLM → RTL
Spec（形式化）──────────────────────────────┘
```

**系统级集成流程**：
```
子模块 1 (Spec→C++/HLS→RTL) ─┐
子模块 2 (Spec→C++/HLS→RTL) ─┼→ 顶层集成 → 系统仿真 → 整体 P&R
子模块 N (Spec→C++/HLS→RTL) ─┘
```

系统采用分层 Agent 架构：
- **Main Agent**: 负责任务分解和最终决策
- **Sub-Agent**: 负责各阶段具体实现（Spec、C++/HLS、RTL、P&R、调试）
- **RTL Agent**: 特别地，接收 CoSim 输出 + Spec，生成可综合 RTL
- **集成 Agent**: 负责子模块组装和顶层集成（新增）

### 3.2 C++/HLS CoSim 流程

**动机**：直接在 RTL 级别进行功能验证成本高、效率低。完整的 HLS 综合耗时长，但本项目只需 CoSim 阶段。

**方法**：
```
需求 → C++ 功能模型 → Vitis HLS CoSim → LLM → RTL
       ↑ 功能验证在此完成    │
                    CoSim 报告 + Spec
```

**CoSim 阶段说明**：
- **输入**：可综合 C++ 代码（含 HLS pragma）
- **输出**：功能验证结果 + 架构信息（流水线、并行度、资源估算）
- **耗时**：数秒至数分钟（vs 完整 HLS 综合的数十分钟至数小时）

**优势**：
1. C++ 模型易于编写和仿真
2. 功能错误在早期发现，降低返工成本
3. CoSim 提供足够的架构信息给 LLM 参考
4. 避免完整 HLS 综合的时间开销
5. CoSim 输出为 LLM 提供架构参考，提升 RTL 生成质量

---

### 3.3 Vivado 原生报告驱动的迭代流程

**核心思想**：利用 Vivado 原生报告（时序、资源、DRC）驱动自动迭代，自定义脚本仅负责解析和决策支持

**迭代流程**：
```
┌─────────────────────────────────────────────────────────────┐
│  Iteration Loop (Vivado 原生报告驱动)                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Vivado 运行 → 生成报告 → 脚本解析 → 指标结构化            │
│     │                                        │              │
│     │                                        ↓              │
│     │                              ┌───────────────┐       │
│     │                              │ 决策矩阵      │       │
│     │                              │ - WNS < 0?    │       │
│     │                              │ - LUT > 90%?  │       │
│     │                              │ - DRC > 0?    │       │
│     │                              └───────────────┘       │
│     │                                        │              │
│     │              ┌─────────────────────────┼──────────┐  │
│     │              │                         │          │  │
│     │              ↓                         ↓          ↓  │
│     │        时序违例                   资源超限    DRC 违例│
│     │              │                         │          │  │
│     │              ↓                         ↓          ↓  │
│     │        改架构/约束              优化代码    修复代码│  │
│     │              │                         │          │  │
│     └──────────────┴─────────────────────────┴──────────┘  │
│                            ↑                                │
│                            │                                │
│                            └──── LLM 生成修改               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Vivado 原生报告类型**：

| 报告 | 生成命令 | 关键指标 | 迭代触发条件 |
|------|---------|---------|-------------|
| `timing.rpt` | `report_timing` | WNS, TNS, 违例路径数 | WNS < 0 |
| `utilization.rpt` | `report_utilization` | LUT/FF/BRAM/DSP 使用率% | 任何 > 90% |
| `drc.rpt` | `report_drc` | 违例数、警告数 | 违例 > 0 |
| `power.rpt` | `report_power` | 静态/动态功耗 | 超过预算 |

**决策矩阵示例**：
```python
def decide_action(timing, util, drc):
    if timing.wns < -0.5:
        return "MAJOR_ARCH_CHANGE"   # 大量时序违例，改架构
    elif timing.wns < 0:
        return "PIPELINE_INSERT"     # 少量违例，插流水线
    elif util.lut_pct > 95:
        return "RESOURCE_SHARE"      # 资源超限，资源共享
    elif drc.violations > 0:
        return "DRC_FIX"             # DRC 违例，修复
    else:
        return "DONE"                # 全部通过
```

**脚本职责边界**：
- ✅ **解析** Vivado 原生报告（.rpt, .log）
- ✅ **提取** 关键指标（WNS、使用率、违例数）
- ✅ **输出** 结构化 JSON 供 Agent 使用
- ❌ **不修改** Vivado 报告格式
- ❌ **不替代** Vivado 仿真/综合/时序分析

### 3.4 子模块集成与系统组装

**动机**：实际 FPGA 项目通常由多个子模块组成，需要解决模块间接口、时钟域、数据流等问题。

**方法**：

#### 3.4.1 模块划分策略

```
系统需求 → 模块划分 → 子模块 Spec1, Spec2, ..., SpecN
                        │        │           │
                        ↓        ↓           ↓
                    C++/HLS  C++/HLS    C++/HLS
                        │        │           │
                        ↓        ↓           ↓
                       RTL1     RTL2       RTLN
                        │        │           │
                        └────────┴───────────┘
                                 │
                                 ↓
                          顶层集成 → 系统仿真
```

**划分原则**：
- **功能独立性**：每个子模块功能相对独立
- **接口清晰**：模块间通过定义良好的接口通信
- **时钟域明确**：每个模块的时钟域在 Spec 阶段定义
- **可复用性**：通用模块（如 FIFO、DMA）可复用

#### 3.4.2 接口定义与验证

**接口形式化**：
```yaml
interface:
  name: axi_stream_master
  type: AXI4-Stream
  signals:
    - tdata: output [31:0]
    - tvalid: output
    - tready: input
    - tlast: output
  clock: clk
  reset: rst_n
```

**接口验证流程**：
1. 子模块 Spec 阶段定义接口
2. 生成接口检查断言（SystemVerilog Assertions）
3. 系统仿真时自动验证接口协议
4. 接口不匹配时自动报错并提示修复

#### 3.4.3 顶层集成

**集成 Agent 职责**：
- 收集所有子模块的 RTL 和接口定义
- 生成顶层模块框架
- 实例化子模块并连接信号
- 生成顶层约束文件（时钟、IO 等）

**顶层模块示例**：
```verilog
module top_module (
    input  clk,
    input  rst_n,
    // AXI4-Stream 输入
    input  [31:0] s_axis_tdata,
    input         s_axis_tvalid,
    output        s_axis_tready,
    // AXI4-Stream 输出
    output [31:0] m_axis_tdata,
    output        m_axis_tvalid,
    input         m_axis_tready
);
    // 子模块实例化
    module_a u_a (/* 端口连接 */);
    module_b u_b (/* 端口连接 */);
    // ...
endmodule
```

#### 3.4.4 系统级仿真（文本输出驱动）

**核心原则**：Agent 不读波形，只读文本输出（log/csv/rpt）

**仿真策略**：
- **模块级仿真**：每个子模块独立验证（Vivado XSIM）
- **系统级仿真**：顶层集成后整体验证（Vivado XSIM）
- **回归测试**：每次修改后自动运行全部测试

**Agent 可读的仿真输出**：

| 输出类型 | 文件格式 | 内容 | Agent 用途 |
|---------|---------|------|-----------|
| **仿真日志** | `simulation.log` | 断言结果、错误信息、时间戳 | 判断通过/失败 |
| **信号 CSV** | `signals.csv` | 关键信号值随时间变化 | 对比输入输出 |
| **覆盖率报告** | `coverage.rpt` | 行/分支/FSM 覆盖率 | 评估测试完整性 |
| **断言汇总** | `assertions.log` | 所有断言通过/失败统计 | 快速定位问题 |

**仿真流程**：
```
Testbench → Vivado XSIM → 生成文本输出 → 脚本解析 → Agent 决策
                              │
                              ↓
                    ┌─────────────────┐
                    │ simulation.log  │
                    │ assertions.log  │
                    │ signals.csv     │
                    │ coverage.rpt    │
                    └─────────────────┘
```

**Testbench 设计规范**（Agent 生成）：
```systemverilog
module tb_top;
    // 信号定义
    reg clk, rst_n;
    wire [31:0] data_in, data_out;
    
    // 被测模块
    dut_top u_dut (.clk(clk), .rst_n(rst_n), .data_in(data_in), .data_out(data_out));
    
    // 断言定义
    initial begin
        $display("[CHECK] @%0t ns: Simulation started", $time);
        
        // 复位检查
        @(posedge rst_n);
        $display("[CHECK] @%0t ns: Reset released", $time);
        
        // 功能检查
        assert (data_out === expected_value)
            $display("[PASS] @%0t ns: data match, got %h", $time, data_out);
        else
            $error("[FAIL] @%0t ns: got %h, expected %h", $time, data_out, expected_value);
        
        // 覆盖率检查
        if (coverage < 90%)
            $warning("[WARN] Coverage low: %0.1f%%", coverage);
    end
    
    // 自动结束
    initial begin
        #100000;
        $display("[INFO] Simulation timeout");
        $finish;
    end
endmodule
```

**仿真日志示例**（`simulation.log`）：
```
[0ns] INFO: Simulation started
[10ns] CHECK: Reset released
[1500ns] PASS: data match, got 0x42
[2300ns] FAIL: got 0x43, expected 0x42
[2300ns] ERROR: Assertion failure at tb_top.v:45
[5000ns] INFO: Simulation completed with 1 error
```

**脚本解析示例**（`parse_sim.py`）：
```python
def parse_simulation_log(log_path: str) -> dict:
    with open(log_path) as f:
        lines = f.readlines()
    
    result = {
        'status': 'PASS',
        'errors': [],
        'warnings': [],
        'checks': []
    }
    
    for line in lines:
        if '[FAIL]' in line or '[ERROR]' in line:
            result['status'] = 'FAIL'
            result['errors'].append(line.strip())
        elif '[WARN]' in line:
            result['warnings'].append(line.strip())
        elif '[PASS]' in line or '[CHECK]' in line:
            result['checks'].append(line.strip())
    
    return result
```

**Agent 决策逻辑**：
```python
def decide_sim_action(sim_result: dict, coverage: dict) -> str:
    if sim_result['status'] == 'FAIL':
        # 仿真失败 → 修复功能
        return "FIX_FUNCTIONAL"
    elif coverage['line'] < 90:
        # 覆盖率不足 → 补充测试
        return "ADD_TEST_CASES"
    elif len(sim_result['warnings']) > 5:
        # 警告过多 → 检查边界条件
        return "REVIEW_EDGE_CASES"
    else:
        return "SIM_PASS"
```

**覆盖率报告示例**（`coverage.rpt`）：
```
Module: fifo_controller
  Line Coverage: 95.2% (120/126)
  Branch Coverage: 88.9% (32/36)
  FSM Coverage: 100% (8/8 states)
  Uncovered Lines: 45, 67, 89
  Uncovered Branches: 12, 34
```

**仿真加速**：
- 使用 Verilator 进行快速语法检查（可选）
- 关键路径使用 FPGA 原型验证（可选）
- 并行仿真：多个测试用例同时运行

---

#### 3.4.5 Sub-Agent 协调机制

**问题**：多个子模块的 Sub-Agent 并行工作时，需要解决任务分配、依赖管理、冲突协调等问题。

**协调架构**：
```
                    ┌─────────────────┐
                    │   Main Agent    │
                    │  (协调器/决策)   │
                    └────────┬────────┘
                             │
           ┌─────────────────┼─────────────────┐
           │                 │                 │
    ┌──────▼──────┐   ┌──────▼──────┐   ┌──────▼──────┐
    │ Sub-Agent 1 │   │ Sub-Agent 2 │   │ Sub-Agent N │
    │  (子模块 A)  │   │  (子模块 B)  │   │  (子模块 N)  │
    └──────┬──────┘   └──────┬──────┘   └──────┬──────┘
           │                 │                 │
           └─────────────────┴─────────────────┘
                             │
                    ┌────────▼────────┐
                    │  集成 Agent     │
                    │  (组装/验证)    │
                    └─────────────────┘
```

**Main Agent 职责**：
1. **任务分解**：将系统需求分解为子模块任务
2. **依赖管理**：识别子模块间的依赖关系，确定执行顺序
3. **资源分配**：分配计算资源（GPU/CPU）给各 Sub-Agent
4. **进度跟踪**：监控各 Sub-Agent 的进度
5. **冲突解决**：当子模块接口不匹配时，协调修改方案

**Sub-Agent 间通信**：
- **共享上下文**：通过向量数据库共享 Spec、接口定义
- **消息队列**：任务状态、完成通知通过消息队列传递
- **接口注册表**：每个 Sub-Agent 完成后注册接口信息

**依赖管理示例**：
```yaml
task_decomposition:
  - id: module_a
    depends_on: []
    parallel: true
  - id: module_b
    depends_on: [module_a]  # 需要 module_a 的接口定义
    parallel: false
  - id: module_c
    depends_on: [module_a]
    parallel: true
  - id: top_integration
    depends_on: [module_a, module_b, module_c]
    parallel: false
```

**冲突解决策略**：
| 冲突类型 | 检测方法 | 解决策略 |
|---------|---------|---------|
| 接口位宽不匹配 | 接口注册表比对 | Main Agent 协调修改 |
| 时钟域不一致 | CDC 检查 | 插入跨时钟域模块 |
| 协议不兼容 | 协议检查器 | 生成适配层 |
| 资源超限 | 资源估算汇总 | 重新分配或优化 |

---

#### 3.4.6 系统级测试方法

**测试层次**：
```
┌─────────────────────────────────────────┐
│ Level 4: 系统级测试 (完整系统验证)      │
├─────────────────────────────────────────┤
│ Level 3: 集成测试 (模块间交互验证)      │
├─────────────────────────────────────────┤
│ Level 2: 模块级测试 (单模块功能验证)    │
├─────────────────────────────────────────┤
│ Level 1: 单元测试 (C++ 模型 CoSim 验证)  │
└─────────────────────────────────────────┘
```

**各级测试说明**：

| 层级 | 测试对象 | 测试方法 | 通过标准 |
|------|---------|---------|---------|
| L1 | C++ 模型 | HLS CoSim | 功能正确，无编译错误 |
| L2 | 单模块 RTL | Cocotb 仿真 | 与 C++ 模型行为一致 |
| L3 | 模块间交互 | 系统仿真 | 接口协议正确，数据流无误 |
| L4 | 完整系统 | FPGA 原型/仿真 | 满足系统需求 |

**测试用例生成**：
- **基于 Spec 生成**：从形式化 Spec 自动生成测试向量
- **边界值测试**：自动生成边界条件测试用例
- **随机测试**：随机输入验证鲁棒性
- **回归测试库**：历史测试用例自动积累

**自动化测试流程**：
```
代码生成 → 自动触发 L1 测试 → 通过？→ 触发 L2 测试 → 通过？→ 触发 L3/L4
              │                      │                      │
              ↓                      ↓                      ↓
           失败修复               失败修复               失败修复
```

**测试覆盖率指标**：
- **代码覆盖率**：行覆盖率、分支覆盖率、条件覆盖率
- **功能覆盖率**：Spec 中定义的功能点覆盖情况
- **接口覆盖率**：所有接口场景的测试覆盖

**持续集成**：
- 每次代码修改自动触发回归测试
- 测试报告自动生成并存档
- 失败用例自动分析并提示修复建议

---

### 3.5 多 Sub-Agent 并行的设计空间探索

**动机**：传统 DSE 需要手动改写多个版本，耗时费力。

**方法**：
```
                    ┌→ Sub-Agent A: 方案 1 ─┐
用户 Spec → Main Agent ─┼→ Sub-Agent B: 方案 2 ─┼→ 评估 → 最优
                    └→ Sub-Agent C: 方案 3 ─┘
```

**探索维度**：
- 并行度：串行 vs 流水线 vs 全并行
- 数据位宽：8/16/32 bit
- 缓存策略：无缓存 vs 行缓冲 vs 块缓存
- 计算单元数量：单 vs 多 vs 阵列

**输出**：每个方案提供资源估算（LUT/FF/BRAM/DSP）和性能估算（延迟/吞吐量）

### 3.6 上下文压缩与分层管理

**问题**：FPGA 开发流程长，累积上下文可达 100K+ token，超出 LLM 最优处理范围，导致推理质量下降。

#### 3.6.1 可行性论证

**必要性分析**：

1. **LLM 上下文窗口限制**：
   - 即使支持 128K 上下文的模型，实际有效注意力窗口通常<32K
   - 过长上下文导致"中间丢失"现象（Lost in the Middle）
   - 推理延迟随上下文长度线性增长

2. **FPGA 开发上下文特点**：
   - 多阶段累积：Spec→C++→HLS→RTL→P&R，每阶段产生 10K-50K token
   - 工具报告冗长：Vivado 综合报告可达 20K+ token
   - 代码迭代频繁：每次修改需保留历史对比

**技术可行性**：

| 压缩技术 | 压缩比 | 信息损失 | 适用场景 |
|---------|-------|---------|---------|
| 摘要生成 | 10:1 | 中 | Spec、对话历史 |
| 关键指标提取 | 50:1 | 低 | 工具报告 |
| 代码分块 | 可定制 | 无 | 代码/模型 |
| 向量检索 | N/A | 低 | 历史检索 |

**预期效果**：
- 上下文总量：100K+ → ~7.5K token（压缩比~13:1）
- 推理延迟：降低 60-80%
- 任务完成率：保持>90%（相比完整上下文）

#### 3.6.2 上下文组成分析

| 类型 | Token 估算 | 压缩策略 | 压缩后 |
|------|-----------|---------|-------|
| System Prompt | 500-2000 | 精简 + 延迟加载 | 500 |
| 项目 Spec | 2000-10000+ | 摘要 | 2000 |
| 代码/模型 | 5000-50000+ | 分层加载 | 3000 |
| 工具报告 | 1000-10000+ | 关键指标提取 | 500 |
| 对话历史 | 1000-20000+ | 摘要 + 剪枝 | 1500 |
| **总计** | **~100K** | - | **~7.5K** |

#### 3.6.3 分层管理策略

```
┌─────────────────────────────────────────┐
│ Layer 1: 核心上下文 (500 token, 始终)   │
│ - 任务目标、关键约束、最近对话          │
├─────────────────────────────────────────┤
│ Layer 2: 任务相关 (5K token, 按需)      │
│ - 当前阶段 Spec、代码、报告             │
├─────────────────────────────────────────┤
│ Layer 3: 历史 (2K token, 检索)          │
│ - 早期决策摘要、长期记忆                │
└─────────────────────────────────────────┘
       总计 ~7.5K token
```

**加载策略**：
- Layer 1：始终加载（每次请求）
- Layer 2：任务触发加载（进入特定阶段时）
- Layer 3：检索式加载（向量数据库查询 Top-K）

#### 3.6.4 与相关工作对比

| 方法 | 压缩比 | 任务保持率 | FPGA 适用性 |
|------|-------|-----------|-----------|
| StreamingLLM | 20:1 | 85% | 中（针对对话优化） |
| H2O | 15:1 | 88% | 中（通用代码） |
| RAG | 可定制 | 90%+ | 高（可检索历史） |
| **本方案** | **13:1** | **>90%** | **高（FPGA 专用）** |

### 3.7 提示词自进化系统

**核心思想**：在不修改基础模型的前提下，通过软件层面的持续优化提升 Agent 表现。

**机制**：

1. **经验库**：记录 (输入，输出，结果，反馈)，新任务检索相似经验注入
2. **用户反馈**：1-5 星评价 + 文字反馈，驱动优化
3. **反思模块**：LLM 自我反思，提取教训存入经验库
4. **Few-shot 示例库**：高质量示例动态注入
5. **自动优化**：A/B 测试、基于反馈迭代

```
任务执行 → 反思 → 经验库
    ↓                      ↑
用户反馈 → 优化器 ─────────┘
```

---

## 4. 技术实现

### 4.1 核心功能模块详解

#### 4.1.1 Spec 解析与形式化

**输入**：自然语言需求 或 非形式化 Spec  
**输出**：形式化 Spec（JSON/YAML）

```yaml
# 形式化 Spec 示例
module:
  name: async_fifo
  interface:
    clk_wr: input clock
    clk_rd: input clock
    rst_n: input reset
    wr_en: input
    rd_en: input
    din: input [31:0]
    dout: output [31:0]
  parameters:
    DEPTH: 256
    WIDTH: 32
  constraints:
    clock_freq: 200 MHz
    latency: < 10 cycles
    area: < 5000 LUT
```

**关键技术**：
- LLM 提取接口定义、参数、约束
- 语法检查（完整性、一致性）
- 生成可执行的验证环境框架

---

#### 4.1.2 C++ 模型生成与仿真

**输入**：形式化 Spec  
**输出**：可综合 C++ 模型 + Testbench

```cpp
// 生成的 C++ 模型示例
void fifo_top(
    bool clk_wr, bool clk_rd, bool rst_n,
    bool wr_en, bool rd_en,
    ap_uint<32> din, ap_uint<32>& dout
) {
    #pragma HLS INTERFACE ap_ctrl_none port=return
    #pragma HLS PIPELINE II=1
    
    static ap_uint<32> mem[256];
    static ap_uint<8> wr_ptr = 0;
    static ap_uint<8> rd_ptr = 0;
    
    if (wr_en) {
        mem[wr_ptr++] = din;
    }
    if (rd_en) {
        dout = mem[rd_ptr++];
    }
}
```

**关键技术**：
- HLS 兼容的 C++ 代码生成
- 自动添加 `#pragma` 优化指令
- 生成 SystemC/Cocotb Testbench
- 自动仿真验证

---

#### 4.1.3 RTL 生成与验证

**输入**：C++ 模型 + HLS 中间代码/报告 + Spec  
**输出**：可综合 Verilog + 约束文件

**生成方式**：

本项目采用**HLS 辅助的 LLM RTL 生成**流程：

```
C++ 模型 → HLS 工具 → HLS 中间代码/报告 ─┐
                                        ├→ LLM → RTL
Spec（形式化）───────────────────────────┘
```

**流程说明**：
1. C++ 模型经过 HLS 工具（Vitis HLS）处理，生成中间代码和资源/时序报告
2. HLS 输出提供架构信息（流水线、并行度、资源分配）
3. Spec 提供功能约束（接口定义、时序要求、面积约束）
4. LLM 综合两者信息，生成更符合预期的 RTL 代码

**相比纯 HLS 或纯 LLM 方式的优势**：

| 方式 | 优点 | 缺点 |
|------|------|------|
| 纯 HLS 工具 | 自动处理时序/并行 | 灵活性低，优化空间有限 |
| 纯 LLM 生成 | 灵活，可定制 | 质量不稳定，可能不可综合 |
| **HLS+LLM（本方案）** | 结合架构信息 + 灵活性 | 需要精心设计 Prompt |

**验证流程（依赖 Vivado 原生能力）**：
```
RTL 生成 → Vivado XSIM 仿真 → Vivado 综合 → Vivado 时序分析
    │            │                │              │
    │            ↓                ↓              ↓
    │      simulation.log     synth.log   timing.rpt
    │            │                │              │
    └────────────┴────────────────┴──────────────┘
                         │
                         ↓
                  脚本解析报告 → 提取关键指标 → 决策迭代
```

**关键技术**：
- **Vivado XSIM**：原生仿真，生成波形和覆盖率报告
- **Vivado 综合**：生成综合报告（资源利用率、逻辑层级）
- **Vivado 时序分析**：生成时序报告（WNS、TNS、违例路径）
- **自定义解析脚本**：提取关键指标供 Agent 决策
- **错误自动修复**：LLM + 工具反馈闭环

---

#### 4.1.4 布局布线优化（Vivado 原生流程）

**输入**：网表 + 约束  
**输出**：优化后的布局布线结果 + 原生报告

**核心原则**：充分利用 Vivado 原生能力，自定义脚本仅负责解析报告和决策支持

**Vivado 原生报告类型**：

| 报告类型 | 原生文件 | 关键指标 | 用途 |
|---------|---------|---------|------|
| 资源利用率 | `utilization.rpt` | LUT/FF/BRAM/DSP 使用率 | 判断是否超限 |
| 时序分析 | `timing.rpt` | WNS/TNS、违例路径数、最坏路径 | 时序收敛判断 |
| 功耗估算 | `power.rpt` | 静态/动态功耗、热点模块 | 功耗优化 |
| 拥塞分析 | `route_design.log` | 拥塞区域、DRC 违例 | 布局优化 |

**优化流程**：
```
Vivado 实现 → 生成原生报告 → 脚本解析 → 指标结构化 → Agent 决策
                                      │
                                      ↓
                              ┌───────────────┐
                              │ 决策矩阵      │
                              │ - 时序违例？  │
                              │ - 资源超限？  │
                              │ - 功耗过高？  │
                              └───────────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    ↓                 ↓                 ↓
              修改约束          修改架构          修改代码
              (Tcl 脚本)        (Prompt)          (LLM 生成)
```

**优化策略（基于 Vivado 报告）**：

| 问题 | Vivado 检测 | 脚本提取指标 | Agent 决策 |
|------|------------|-------------|-----------|
| 时序违例 | `timing.rpt` WNS<0 | WNS 值、违例路径数、关键路径层级 | 流水线/复制寄存器/调整约束 |
| 资源超限 | `utilization.rpt` | LUT/FF/BRAM/DSP 使用率% | 资源共享/位宽优化/模块拆分 |
| 拥塞 | `route_design.log` | 拥塞区域坐标、DRC 违例数 | 模块重定位/逻辑复制 |
| 功耗过高 | `power.rpt` | 动态功耗占比、热点模块 | 时钟门控/操作数隔离 |

**AI 辅助点**：
- **报告解析**：自定义脚本提取关键指标（非 AI）
- **决策支持**：Agent 根据指标选择优化策略
- **Tcl 生成**：LLM 生成 Vivado 优化 Tcl 脚本
- **参数调优**：Agent 自动调整实现策略（Performance_Explore/Resource_Explore 等）

---

#### 4.1.5 调试辅助（Vivado 原生调试）

**输入**：Vivado 波形文件 (.wdb)、日志文件 (.log)、报告文件 (.rpt)  
**输出**：问题定位报告 + 修复建议

**核心原则**：使用 Vivado 原生调试工具，脚本仅负责解析和结构化

**Vivado 原生调试能力**：

| 调试场景 | Vivado 工具 | 输出文件 | 脚本解析内容 |
|---------|------------|---------|-------------|
| 功能调试 | XSIM 仿真 | `simulation.log`, `.wdb` | 断言失败、信号跳变异常 |
| 时序调试 | Timing Analyzer | `timing.rpt` | 违例路径、slack 值、逻辑层级 |
| 综合调试 | Synthesis | `synth.log`, `utilization.rpt` | 推断失败、资源超限 |
| 实现调试 | Implementation | `route_design.log` | DRC 违例、拥塞区域 |

**调试流程**：
```
Vivado 运行 → 生成日志/波形 → 脚本解析 → 结构化错误 → Agent 分析
                                              │
                                              ↓
                                    ┌─────────────────┐
                                    │ 错误分类        │
                                    │ - 语法错误     │
                                    │ - 时序违例     │
                                    │ - 功能错误     │
                                    │ - 资源问题     │
                                    └─────────────────┘
                                              │
                                              ↓
                                      LLM 生成修复建议
```

**错误示例（基于 Vivado 报告）**：
```
[Vivado Timing] Setup time violation on path:
  From: clk/data_gen_fsm_reg
  To:   data_out_reg
  
  Slack: -0.523ns (required: 5.000ns, actual: 5.523ns)
  Logic levels: 8 (LUT6→LUT5→LUT6→LUT4→LUT6→LUT5→LUT6→LUT6)
  
脚本解析结果:
{
  "error_type": "timing_violation",
  "slack_ns": -0.523,
  "logic_levels": 8,
  "from_cell": "data_gen_fsm_reg",
  "to_cell": "data_out_reg"
}

Agent 决策:
- 问题：组合逻辑级数过多（8 级）
- 策略：插入流水线寄存器
- 修复：在 data_path 第 4 级后插入 2 级寄存器
```

**AI 辅助点**：
- **报告解析**：脚本提取关键指标（非 AI）
- **问题分类**：基于规则的自动分类（非 AI）
- **根因分析**：LLM 分析错误链，定位根本原因
- **修复生成**：LLM 生成具体修改方案（代码/约束）

---

### 4.2 工具链集成

#### 4.2.1 核心原则

**依赖 Vivado/Vitis 原生能力**：
- ✅ 使用 Vivado 原生仿真 (XSIM)、综合、时序分析、布局布线
- ✅ 使用 Vitis HLS 进行 C++ 模型验证和架构探索
- ✅ 自定义脚本仅负责：解析报告、提取指标、决策支持

**不重复造轮子**：
- ❌ 不自己实现仿真器
- ❌ 不自己实现综合/时序分析
- ❌ 不解析二进制波形文件（使用 Vivado 导出的文本报告）

---

#### 4.2.2 Vivado/Vitis 原生流程集成

```python
class VivadoFlow:
    """Vivado 原生流程封装"""
    
    def run_simulation(self, rtl_files, tb_files):
        """运行 XSIM 仿真"""
        tcl = self._gen_sim_tcl(rtl_files, tb_files)
        result = self._run_vivado(tcl)  # 调用 vivado -mode batch
        # 解析 simulation.log
        return self._parse_sim_log(result.log)
    
    def run_synthesis(self, rtl_files, constraints):
        """运行综合"""
        tcl = self._gen_synth_tcl(rtl_files, constraints)
        result = self._run_vivado(tcl)
        # 解析 synth.log + utilization.rpt
        return {
            'status': self._parse_synth_status(result.log),
            'utilization': self._parse_utilization_rpt(result.rpt)
        }
    
    def run_implementation(self, checkpoint, strategy):
        """运行布局布线"""
        tcl = self._gen_impl_tcl(checkpoint, strategy)
        result = self._run_vivado(tcl)
        # 解析 route_design.log + timing.rpt
        return {
            'status': self._parse_impl_status(result.log),
            'timing': self._parse_timing_rpt(result.rpt),
            'drc': self._parse_drc_rpt(result.drc)
        }
    
    def _run_vivado(self, tcl_script):
        """执行 Vivado Tcl 脚本"""
        cmd = f"vivado -mode batch -source {tcl_script} -log vivado.log"
        return subprocess.run(cmd, shell=True, capture_output=True, text=True)
```

**集成点**：

| 阶段 | Vivado 命令 | 输出文件 | 解析内容 |
|------|------------|---------|---------|
| 仿真 | `run -all` | `simulation.log` | 断言结果、覆盖率 |
| 综合 | `synth_design` | `synth.log`, `utilization.rpt` | 推断警告、资源使用 |
| 实现 | `place_design`, `route_design` | `route_design.log` | DRC 违例、拥塞 |
| 时序 | `report_timing` | `timing.rpt` | WNS/TNS、违例路径 |
| 比特流 | `write_bitstream` | `design.bit` | 生成状态 |

---

#### 4.2.3 报告解析脚本（核心）

**设计原则**：
- 脚本只负责**解析 Vivado 原生报告**，提取结构化指标
- Agent 根据结构化指标做**决策**（是否需要迭代、如何迭代）
- 不修改 Vivado 报告格式，不生成中间格式

**脚本架构**：
```
scripts/report_parser/
├── parse_timing.py      # 解析 timing.rpt → WNS/TNS/违例路径
├── parse_utilization.py # 解析 utilization.rpt → LUT/FF/BRAM/DSP
├── parse_synth.py       # 解析 synth.log → 综合状态/警告
├── parse_sim.py         # 解析 simulation.log → 断言结果/覆盖率
└── __init__.py
```

**parse_timing.py 示例**：
```python
#!/usr/bin/env python3
"""解析 Vivado timing.rpt，提取关键时序指标"""

import re
from dataclasses import dataclass

@dataclass
class TimingMetrics:
    wns_ns: float           # Worst Negative Slack
    tns_ns: float           # Total Negative Slack
    violation_paths: int    # 违例路径数
    worst_path: str         # 最坏路径详情
    logic_levels: int       # 最大逻辑层级
    fmax_mhz: float         # 最高频率

def parse_timing_rpt(rpt_path: str) -> TimingMetrics:
    with open(rpt_path) as f:
        content = f.read()
    
    # 提取 WNS/TNS
    wns = float(re.search(r'WSN\s*:\s*(-?\d+\.\d+)\s*ns', content).group(1))
    tns = float(re.search(r'TNS\s*:\s*(-?\d+\.\d+)\s*ns', content).group(1))
    
    # 提取违例路径数
    violations = int(re.search(r'Number of paths:\s*(\d+)', content).group(1))
    
    # 提取最坏路径详情
    worst_path = re.search(r'Path Type:.*?(?=Path Type:|$)', content, re.DOTALL).group(0)
    
    return TimingMetrics(wns, tns, violations, worst_path, ...)

# 输出 JSON 供 Agent 使用
if __name__ == '__main__':
    metrics = parse_timing_rpt('timing.rpt')
    print(metrics.to_json())
```

**parse_utilization.py 示例**：
```python
#!/usr/bin/env python3
"""解析 Vivado utilization.rpt，提取资源使用率"""

@dataclass
class UtilizationMetrics:
    lut_used: int
    lut_available: int
    ff_used: int
    ff_available: int
    bram_used: float  # 按 36Kb 单元计
    dsp_used: int
    lut_util_pct: float
    ff_util_pct: float

def parse_utilization_rpt(rpt_path: str) -> UtilizationMetrics:
    with open(rpt_path) as f:
        content = f.read()
    
    # 提取各类资源使用量
    lut_used = int(re.search(r'Slice LUTs.*?\|\s*(\d+)', content).group(1))
    lut_avail = int(re.search(r'Slice LUTs.*?\|\s*\d+\s*\|\s*(\d+)', content).group(1))
    # ... 类似提取 FF/BRAM/DSP
    
    return UtilizationMetrics(lut_used, lut_avail, ...)
```

**输出格式（JSON）**：
```json
{
  "timing": {
    "wns_ns": -0.523,
    "tns_ns": -12.456,
    "violation_paths": 23,
    "fmax_mhz": 180.5
  },
  "utilization": {
    "lut_util_pct": 65.2,
    "ff_util_pct": 42.1,
    "bram_util_pct": 30.0,
    "dsp_util_pct": 15.5
  },
  "drc": {
    "violations": 0,
    "warnings": 3
  }
}
```

**Agent 决策逻辑**：
```python
def decide_next_action(metrics: dict) -> str:
    """根据报告指标决定下一步行动"""
    
    # 时序违例 → 优化时序
    if metrics['timing']['wns_ns'] < 0:
        if metrics['timing']['violation_paths'] > 50:
            return "ARCHITECTURE_CHANGE"  # 大量违例，需要改架构
        else:
            return "CONSTRAINT_TWEAK"     # 少量违例，调整约束
    
    # 资源超限 → 优化资源
    if metrics['utilization']['lut_util_pct'] > 90:
        return "RESOURCE_OPTIMIZE"
    
    # DRC 违例 → 修复 DRC
    if metrics['drc']['violations'] > 0:
        return "DRC_FIX"
    
    # 全部通过 → 完成
    return "DONE"
```

---

#### 4.2.4 仿真工具集成（文本输出优先）

**核心原则**：
- ✅ Agent 读文本（log/csv/rpt），不读波形（.wdb）
- ✅ Testbench 内置断言和打印，输出结构化日志
- ✅ 覆盖率报告自动生成
- ✅ 波形留给人工调试

**Vivado XSIM 集成**：

```python
class VivadoSimulator:
    """Vivado XSIM 仿真封装"""
    
    def run_simulation(self, rtl_files, tb_files, output_dir='sim_output'):
        """运行仿真并生成文本输出"""
        
        # 生成 Tcl 脚本
        tcl = f"""
            set_sim_target XSIM
            add_files {rtl_files}
            add_files {tb_files}
            update_compile_order -fileset sim_1
            
            # 运行仿真
            launch_simulation
            
            # 导出关键信号到 CSV
            log_wave -r /tb_top/dut/*
            write_log -format csv -output {output_dir}/signals.csv
            
            # 导出仿真日志
            write_simulation_log -output {output_dir}/simulation.log
            
            # 导出覆盖率
            coverage save -file {output_dir}/coverage.ucdb
            report_coverage -all -file {output_dir}/coverage.rpt
            
            close_simulation
        """
        
        # 执行仿真
        result = self._run_vivado(tcl)
        
        # 解析输出
        return {
            'log': self._parse_sim_log(f'{output_dir}/simulation.log'),
            'signals': self._parse_signals_csv(f'{output_dir}/signals.csv'),
            'coverage': self._parse_coverage_rpt(f'{output_dir}/coverage.rpt')
        }
    
    def _parse_sim_log(self, log_path):
        """解析仿真日志，提取断言结果"""
        # 实现 parse_simulation_log() 逻辑
        pass
    
    def _parse_signals_csv(self, csv_path):
        """解析信号 CSV，提取关键时序"""
        # 实现信号对比逻辑
        pass
    
    def _parse_coverage_rpt(self, rpt_path):
        """解析覆盖率报告"""
        # 实现覆盖率解析逻辑
        pass
```

**Vitis HLS CoSim 集成**：

```python
class VitisHLSimulator:
    """Vitis HLS CoSim 仿真封装"""
    
    def run_cosim(self, cpp_file, tb_file, output_dir='cosim_output'):
        """运行 CoSim 并解析结果"""
        
        # 生成 Tcl 脚本
        tcl = f"""
            open_project my_project
            add_files {cpp_file}
            add_files -tb {tb_file}
            set_top top_function
            open_solution -reset solution1
            
            # 运行 CoSim
            csim_design -O
            
            # 导出结果
            """
        
        # 执行 CoSim
        result = self._run_vitis(tcl)
        
        # CoSim 输出已经是文本格式
        return {
            'status': self._parse_cosim_status(result.log),
            'rtl': self._parse_cosim_rtl(result.rtl),
            'report': self._parse_cosim_report(result.rpt)
        }
```

**工具对比**：

| 工具 | 阶段 | 输出形式 | Agent 读取内容 |
|------|------|---------|---------------|
| **Vitis HLS CoSim** | C++ 模型验证 | 文本日志 + RTL | 仿真状态 + 架构报告 |
| **Vivado XSIM** | RTL 验证 | simulation.log + CSV + coverage.rpt | 断言结果 + 信号对比 + 覆盖率 |
| **Verilator** | 快速检查（可选） | 文本日志 | 语法错误 + 警告 |
| **Cocotb** | 测试生成（可选） | Python 日志 | 测试结果 |

**推荐仿真流程**：

```
┌─────────────────────────────────────────────────────────────┐
│ 仿真验证流程（文本输出驱动）                                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  C++ 模型 → Vitis HLS CoSim → cosim.log                    │
│     │                        │                              │
│     │                        ↓                              │
│     │                  Agent 读取状态                       │
│     │                                                        │
│     ↓                                                        │
│  RTL 生成 → Vivado XSIM → simulation.log                   │
│                           signals.csv                       │
│                           coverage.rpt                      │
│                                 │                           │
│                                 ↓                           │
│                           脚本解析                          │
│                                 │                           │
│                                 ↓                           │
│                           Agent 决策                        │
│                                 │                           │
│              ┌──────────────────┼──────────────────┐       │
│              ↓                  ↓                  ↓       │
│        仿真失败           覆盖率不足          全部通过     │
│              │                  │                  │       │
│              ↓                  ↓                  ↓       │
│        修复功能          补充测试用例         进入下一步   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**波形处理说明**：
- **Agent**：不读取 .wdb 波形文件
- **人工调试**：保留 .wdb 文件供工程师使用 Vivado GUI 查看
- **自动化**：通过断言和信号 CSV 实现自动化判断

---

### 4.3 错误处理与自动修复

#### 4.3.1 错误分类

| 类型 | 检测工具 | 修复方式 |
|------|---------|---------|
| 语法错误 | Verilator | LLM 直接修复 |
| Lint 警告 | SpyGlass/VC Spy | LLM + 规则库 |
| CDC 问题 | SpyGlass CDC | 插入同步器 |
| 时序违例 | Vivado Timing | 流水线/优化 |
| 资源超限 | Vivado Report | 资源共享/优化 |

#### 4.3.2 自动修复流程

```
错误检测 → 错误分类 → 修复策略选择 → 生成修复 → 验证修复
    │                                              │
    └─────────────────── 失败重试 ─────────────────┘
```

**修复策略库**：
```yaml
timing_violation:
  strategies:
    - name: pipeline
      condition: combinational_levels > 4
      action: insert_pipeline_register
    - name: replicate
      condition: high_fanout
      action: duplicate_register
    
cdc_issue:
  strategies:
    - name: synchronizer
      condition: single_bit
      action: insert_2ff_sync
    - name: gray_fifo
      condition: multi_bit
      action: insert_gray_fifo
```

---

### 4.4 用户交互设计

#### 4.4.1 交互模式

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **全自动** | 用户输入需求，Agent 完成全部流程 | 简单模块 |
| **半自动** | 用户确认关键决策点 | 复杂设计 |
| **辅助模式** | 用户主导，Agent 提供建议 | 探索性设计 |

#### 4.4.2 关键决策点

```
需求 → [Spec 确认] → C++ 模型 → [架构选择] → RTL → [约束确认] → 比特流
              ↑                    ↑                    ↑
         用户确认接口        用户选择方案          用户确认定时
```

#### 4.4.3 可视化界面

**Web UI 功能**：
- 项目概览（进度、状态）
- Spec 编辑器（形式化 Spec）
- 代码查看器（C++/RTL 对比）
- 报告仪表板（资源、时序、功耗）
- 波形查看器（集成仿真）
- 聊天界面（与 Agent 对话）

---

### 4.5 版本管理与协作

#### 4.5.1 设计版本管理

```
design_v1.0/
├── spec.yaml          # 形式化 Spec
├── model/             # C++ 模型
├── rtl/               # RTL 代码
├── constraints/       # 约束文件
├── reports/           # 综合/时序报告
└── metadata.json      # 版本元数据
```

**元数据**：
```json
{
  "version": "1.0",
  "parent": null,
  "changes": ["初始版本"],
  "metrics": {
    "lut": 4500,
    "ff": 3200,
    "bram": 12,
    "wns": -0.2,
    "fmax": 180
  },
  "timestamp": "2026-03-18T23:00:00"
}
```

#### 4.5.2 协作功能

- 多人编辑 Spec（类似 Google Docs）
- 代码审查（Comment + 建议）
- 变更追踪（Diff 对比）
- 评论与讨论（Thread）

---

### 4.1 技术栈

| 组件 | 选型 |
|------|------|
| LLM | Qwen3.5-Plus / CodeLlama |
| FPGA 工具链 | Vivado / Vitis HLS |
| 后端 | Python / Tcl |
| 向量数据库 | Chroma / FAISS |

### 4.2 系统架构

```
┌─────────────────────────────────────────┐
│              用户界面                    │
│      CLI / Web UI / IDE 插件            │
└─────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│            Main Agent                    │
└─────────────────────────────────────────┘
                  │
    ┌─────────────┼─────────────┐
    ▼             ▼             ▼
┌──────┐   ┌──────────┐   ┌──────────┐
│ Spec │   │ C++/HLS  │   │   RTL    │
│Agent │   │  Agent   │   │  Agent   │
└──────┘   └──────────┘   └──────────┘
    │             │             │
    └─────────────┴─────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│           支持系统                       │
│  经验库 │ 知识库 │ 上下文管理 │ 优化器 │
└─────────────────────────────────────────┘
```

---

## 5. 评估计划

### 5.1 评估指标

| 指标 | 定义 | 基线 | 目标 |
|------|------|------|------|
| 开发周期 | 需求到比特流的时间 | 数周 | 数天 |
| 迭代速度 | 单次修改到验证的时间 | 数小时 | 数分钟 |
| 代码质量 | 综合通过率、时序满足率 | - | >90% |
| 用户满意度 | 主观评价（1-5 分） | - | >4.0 |

### 5.2 基准测试

计划使用以下基准进行评估：
- 经典数字电路模块（FIFO、FFT、卷积等）
- FPL 竞赛题目
- 实际科研项目需求（如 Linear Attention 加速器）

### 5.3 消融实验

- C++ 模型优先 vs 直接 RTL 生成
- 多 Agent DSE vs 单方案
- 上下文压缩 vs 完整上下文
- 自进化系统 vs 静态提示词

---

## 6. 当前状态与计划

### 6.1 当前状态（2026-03-18）

- ✅ 项目启动
- ✅ 完成系统设计
- ✅ 完成技术调研
- 📋 待开发 MVP

### 6.2 实施计划

| 阶段 | 时间 | 任务 |
|------|------|------|
| Phase 1 | 1-3 月 | 经验库、用户反馈、C++ 模型生成 |
| Phase 2 | 3-6 月 | 反思模块、DSE、上下文管理 |
| Phase 3 | 6-12 月 | 自动优化、知识库、布局布线 |

### 6.3 相关应用

- **FPL 2026 竞赛**：布局布线优化方向
- **科研项目**：Linear Attention 硬件加速器

---

## 7. 讨论

### 7.1 潜在挑战

1. **RTL 生成质量**：LLM 生成的 RTL 可能功能正确但不可综合
2. **工具链集成**：与 Vivado/Quartus 的深度集成存在技术难度
3. **长上下文管理**：复杂项目的上下文管理仍是开放问题

### 7.2 未来方向

1. **模型微调**：收集 FPGA 专用数据微调开源模型
2. **形式验证**：集成形式化验证工具确保功能正确
3. **多 FPGA 支持**：扩展到多 FPGA 系统开发

---

## 8. 讨论与开放问题

### 8.1 潜在挑战

1. **RTL 生成质量**：LLM 生成的 RTL 可能功能正确但不可综合
2. **工具链集成**：与 Vivado/Quartus 的深度集成存在技术难度
3. **长上下文管理**：复杂项目的上下文管理仍是开放问题

### 8.2 研究局限

1. **模型依赖**：系统性能受限于基础 LLM 能力
2. **领域特定性**：当前设计主要针对 FPGA，泛化能力待验证
3. **评估难度**：缺乏标准基准测试集

### 8.3 伦理考量

1. **知识产权**：AI 生成的 RTL 代码版权归属
2. **安全性**：硬件木马检测与防范
3. **可解释性**：AI 决策过程需要可追溯

---

## 9. 研究问题（Research Questions）

为明确研究方向，我们提出以下核心研究问题：

| 编号 | 研究问题 | 验证方法 |
|------|---------|---------|
| **RQ1** | LLM 能否生成可综合的 RTL 代码？ | 综合通过率、人工评估 |
| **RQ2** | C++ 模型优先流程是否降低返工率？ | 对比实验（有/无 C++ 阶段） |
| **RQ3** | 多 Agent DSE 能否找到更优设计点？ | PPA 对比（Power-Performance-Area） |
| **RQ4** | 上下文压缩是否保持推理质量？ | 任务完成率、用户满意度 |
| **RQ5** | 自进化系统能否持续提升性能？ | 纵向追踪（任务数 vs 成功率） |

---

## 10. 资源需求

### 10.1 计算资源

| 资源 | 需求 | 用途 |
|------|------|------|
| GPU | 1-2×A100/4090 | LLM 推理、微调 |
| CPU | 16+ 核心 | 工具链运行 |
| 内存 | 64GB+ | 大型设计处理 |
| 存储 | 1TB+ SSD | 项目数据、缓存 |

### 10.2 软件资源

| 软件 | 许可 | 用途 |
|------|------|------|
| Vivado/Vitis | 学术许可 | 综合、布局布线 |
| LLM API | 付费/开源 | 代码生成 |
| 向量数据库 | 开源 | 经验库存储 |

### 10.3 数据资源

| 数据 | 来源 | 用途 |
|------|------|------|
| RTL 代码 | OpenCores、GitHub | 微调数据 |
| Spec-RTL 配对 | 人工构建 | 监督训练 |
| 错误模式 | 工具报告分析 | 纠错训练 |

---

## 11. 风险管理

| 风险 | 概率 | 影响 | 缓解措施 |
|------|------|------|---------|
| LLM 能力不足 | 中 | 高 | 降级方案（模板+AI） |
| 工具链集成困难 | 中 | 中 | 分阶段集成 |
| 评估数据不足 | 高 | 中 | 提前收集基准 |
| 时间超支 | 中 | 中 | 敏捷开发、MVP 优先 |

---

## 12. 结论

本项目提出了一种基于 LLM 的端到端 FPGA 开发自动化框架。通过 C++ 模型优先、多 Agent 并行 DSE、Vivado 原生报告驱动的迭代、上下文压缩、提示词自进化等技术创新，旨在显著降低 FPGA 开发门槛、加速设计迭代。

**核心设计原则**：
- 充分利用 Vivado/Vitis 原生能力（仿真、综合、时序分析）
- 自定义脚本仅负责解析报告、提取指标、支持决策
- 不重复造轮子，专注于 Agent 决策逻辑和流程自动化

目前项目处于启动阶段，计划分三个阶段实施。

**预期贡献**：
1. **方法论**：C++ 优先的 FPGA 开发流程 + Vivado 原生报告驱动的迭代
2. **系统**：端到端自动化原型（含报告解析脚本库）
3. **数据**：FPGA 专用代码数据集 + 错误模式库
4. **洞见**：LLM 在硬件开发中的能力边界 + Agent 决策有效性

---

## 参考文献

（待补充）

**建议补充的文献方向**：
- LLM 代码生成（CodeX、StarCoder、CodeLlama）
- 高层次综合（Vitis HLS、LegUp）
- AI for EDA（强化学习布局布线、ML 预测综合结果）
- 上下文压缩（StreamingLLM、H2O、RAG）
- Agent 系统（AutoGen、LangChain）

---

**作者**: 博士  
**单位**: 香港中文大学（深圳）  
**日期**: 2026-03-19 (v0.7 更新 Vivado 仿真文本输出方案)
