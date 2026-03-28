# Vivado FPGA Development Skills

7 个 skill 覆盖 AMD Vivado 2025.2 FPGA 开发全流程（RTL → Bitstream），基于官方 UG 文档构建。

## FPGA 开发流程对应

```
RTL设计 ──→ 综合 ──→ 约束 ──→ 实现 ──→ 时序分析 ──→ 编程调试
             synth   constraints  impl    analysis     debug

仿真验证（贯穿全流程）sim
TCL自动化（贯穿全流程）tcl
```

## Skill 一览

| Skill | 文档来源 | 定位 | 核心内容 |
|-------|---------|------|---------|
| **vivado-synth** | UG901 | 综合决策 | 综合策略/指令选择、属性配置(RAM_STYLE/USE_DSP/SHREG等)、资源推断控制、FSM编码、OOC综合、增量综合、RTL Linting |
| **vivado-constraints** | UG903 | 约束编写 | 时钟定义、I/O延迟、时序例外(false_path/multicycle)、CDC约束、物理约束、XDC优先级规则、约束调试 |
| **vivado-impl** | UG904 | 实现优化 | opt/place/phys_opt/route_design指令选择、拥塞分析、增量实现、ECO流程、运行策略(Performance/Congestion/Area) |
| **vivado-analysis** | UG906 | 报告解读 | report_timing解读、Slack计算、QoR评分(1-5)、QoR建议工作流、design_analysis(Rent/拥塞)、时序收敛决策树、Waiver |
| **vivado-debug** | UG908 | 调试策略 | ILA/VIO/JTAG-to-AXI选型、mark_debug流程、ILA配置(深度/触发/Cross-Trigger)、Versal调试架构、硬件编程、错误排查 |
| **vivado-sim** | UG900 | 仿真验证 | 行为/后综合/后实现仿真、xsim三步流、第三方仿真器集成、SAIF/VCD功耗仿真、SDF时序仿真 |
| **vivado-tcl** | UG835+UG892 | TCL执行 | Project/Non-Project模式、完整命令速查、IP Integrator BD、Debug Core插入、Hardware编程模板 |

## 架构设计

每个 skill 采用三层信息结构：

```
SKILL.md      → 决策指南（何时用什么、策略选择表、决策树）
REFERENCE.md  → 命令语法（完整参数、属性表、值域）
examples/     → 代码示例（按需读取，不污染上下文）
```

- **SKILL.md** 在 skill 触发时自动加载，聚焦决策知识
- **REFERENCE.md** 在 skill 触发时自动加载，提供精确语法
- **examples/** 不自动加载，通过索引表按需 Read

## Skill 间交叉引用

```
vivado-synth ──→ vivado-tcl (执行)
vivado-constraints ──→ vivado-tcl (执行), vivado-analysis (报告解读)
vivado-impl ──→ vivado-tcl (执行), vivado-synth (综合), vivado-constraints (约束), vivado-analysis (分析)
vivado-analysis ──→ vivado-tcl (执行), vivado-constraints (约束修改), vivado-impl (策略调整)
vivado-debug ──→ vivado-tcl (执行), vivado-impl (策略), vivado-analysis (时序)
vivado-sim ──→ vivado-tcl (执行)
vivado-tcl ──→ vivado-debug (调试决策), vivado-analysis (分析)
```

## Examples 目录

| Skill | 目录 | 内容 |
|-------|------|------|
| vivado-synth | `examples/` | 64个 Verilog/SV 文件 — UG901 HDL编码模板(RAM/DSP/ROM/SRL/FSM等) |
| vivado-impl | `examples/ug906/` | 3组 before/after RTL — report_qor_suggestions 优化示例 |

## 尚未覆盖

| 缺口 | 文档 | 内容 |
|------|------|------|
| vivado-ip | UG994+UG896 | BD架构决策、AXI协议选型、IP配置、PS-PL接口 |
| vitis-hls | UG1399 | HLS pragma、接口综合、循环优化、C仿真/协同仿真 |

## 命令验证

所有 skill 中引用的 TCL 命令均通过 Vivado 2025.2 实际调用验证（`-help` 测试），确认命令存在且可用。
