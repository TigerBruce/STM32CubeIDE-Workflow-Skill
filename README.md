# STM32CubeIDE Workflow Skill

中文名：**STM32CubeIDE 工作流 Skill**

这是一个面向 Codex / Agent 的 STM32CubeIDE 工程工作流 Skill，用于让 Agent 在处理 STM32 固件工程时，先读项目本地记忆和规则，再执行编译、下载/烧录和调试排查。

它不是 Zed、Cursor 或 VS Code 插件。编辑器只是前端；真正的编译、下载和调试依赖 STM32CubeIDE 生成的 Makefile、ST-LINK、STM32 Programmer CLI、串口工具和项目本地规则。

## 适用场景

- STM32CubeIDE 工程编译
- STM32 固件下载 / 烧录 / flash
- ST-LINK / SWD 连接排查
- STM32 Programmer CLI 使用
- Debug / Release 目录和 ELF 文件检查
- 串口调试、上位机调试、协议联调
- ST-LINK 热读 RAM、查变量地址、定位固件运行状态
- 需要保护校准参数页、Flash 参数区、Option Bytes 的工程

## 核心原则

1. **先读项目记忆，再执行命令**

   优先读取项目里的 `RULES/memory/MEMORY.md`、`docs/` 下载/烧录记录、`CLAUDE.md`、`.cursor/rules/` 等文件。项目事实永远不能写死到开源 Skill 里。

2. **编译必须显式指定目标**

   对 STM32CubeIDE 生成的 Makefile，默认使用显式目标：

   ```powershell
   & $make -C Debug all -j8
   ```

   不要裸跑 `make -C Debug`，避免生成的 Makefile include 顺序导致误触发非预期目标。

3. **下载/烧录默认只写 ELF 并校验**

   默认使用 STM32 Programmer CLI + ST-LINK/SWD，按项目已验证命令执行。不要随意全片擦除、写 Option Bytes、改读保护或擦掉校准页。

4. **调试先确认工具占用和项目事实**

   烧录失败时先查 STM32CubeIDE Debug 会话、STM32CubeProgrammer、OpenOCD、pyOCD、串口上位机等是否占用设备，再按项目记忆确认 reset/connect 模式。

5. **开源 Skill 不包含私有工程信息**

   私有工程路径、ST-LINK SN、COM 口、芯片板卡状态、Flash 参数页地址、历史验证结果都应保存在项目本地 memory 或 docs 中。

## 安装

复制 Skill 目录到 Codex skills 目录：

```powershell
Copy-Item -Recurse .\stm32-cubeide-workflow "$env:USERPROFILE\.codex\skills\"
```

然后重启 Codex 或打开新会话，让 Skill 自动发现。

## 目录结构

```text
stm32-cubeide-workflow/
├── SKILL.md
└── agents/
    └── openai.yaml
```

## 编辑器兼容性

这个 Skill 与编辑器无关：

- Zed 可以用
- Cursor 可以用
- VS Code 可以用
- STM32CubeIDE 可以用
- 纯终端也可以用

但对大型 STM32 工程，建议避免让编辑器或 AI 工具对 `Debug/`、`Release/`、`Drivers/`、`.git/`、大型 PDF 做无差别索引。终端命令更适合精确检查构建产物、ELF、Makefile 和工具链路径。

---

# English

This is a Codex / Agent skill for STM32CubeIDE firmware workflows. It guides the agent to read project-local memory and rules first, then handle build, flash/download, and debugging tasks with explicit, safe steps.

It is not a Zed, Cursor, or VS Code extension. Editors are only frontends. The real workflow depends on STM32CubeIDE generated Makefiles, ST-LINK, STM32 Programmer CLI, serial tools, and project-local documentation.

## Use Cases

- Build STM32CubeIDE firmware projects.
- Flash/download STM32 firmware through ST-LINK/SWD.
- Use STM32 Programmer CLI safely.
- Inspect Debug/Release folders and ELF files.
- Debug ST-LINK connection issues and tool conflicts.
- Support serial debugging, protocol bring-up, and upper-computer tools.
- Read RAM through ST-LINK, inspect symbol addresses, and verify runtime state.
- Preserve calibration Flash pages, parameter sectors, and option-byte safety.

## Key Rules

- Read project-local memory before acting.
- Use explicit make targets such as `make -C Debug all -j8`.
- Do not run bare `make -C Debug`.
- Do not mass erase, write option bytes, change readout protection, or erase calibration sectors unless the user explicitly asks and the project memory confirms the safe procedure.
- Keep private project paths, ST-LINK serial numbers, COM ports, and board-specific facts in project-local memory, not in this public skill.
