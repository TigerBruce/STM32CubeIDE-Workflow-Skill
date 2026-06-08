---
name: stm32-cubeide-workflow
description: Build, flash/download, and debug STM32CubeIDE firmware safely using project-local memory, generated makefiles, STM32 Programmer CLI, ST-LINK/SWD, serial tools, and runtime verification. Use when working with STM32CubeIDE projects, .ioc/.cproject files, STM32 firmware compile/build/download/flash tasks, Debug/Release ELF files, ST-LINK, SWD, STM32_Programmer_CLI, serial protocol bring-up, RAM readback, or when the user mentions CubeIDE, make all, burning, flashing, downloading firmware, or debugging STM32 firmware.
---

# STM32CubeIDE Workflow

Use this skill for STM32CubeIDE firmware build, flash/download, and debugging workflows. It is editor-agnostic: Zed, Cursor, VS Code, STM32CubeIDE, or a terminal are only frontends. Do not rely on editor automation for correctness.

## First Read Project Facts

Before building, flashing, or debugging, inspect project-local memory and documentation if present:

- `RULES/memory/MEMORY.md`
- `RULES/memory/*flash*`, `RULES/memory/*build*`, `RULES/memory/*stm32*`, `RULES/memory/*debug*`
- `docs/*下载*`, `docs/*烧录*`, `docs/*编译*`, `docs/*调试*`, `docs/*flash*`, `docs/*build*`, `docs/*debug*`
- `CLAUDE.md`, `.cursor/rules/*`, or other project rules when they exist

Extract only project-specific facts from those files:

- Active project root, not archived snapshots.
- CubeIDE install root and exact plugin tool paths.
- Build configuration: usually `Debug` or `Release`.
- Expected ELF path.
- Target MCU, ST-LINK connection mode, SWD frequency, serial bridge, baud rate.
- Flash pages, parameter sectors, option bytes, or calibration regions that must be preserved.
- Known debug tools, serial pages, scripts, RAM-read commands, and validation outputs.

Project memory overrides this generic skill. Do not copy private paths, serial numbers, COM ports, or hardware IDs into this skill; keep those in project memory.

## Build Workflow

If the project has a build/flash script, read it first and prefer it:

```powershell
Set-ExecutionPolicy -Scope Process Bypass -Force
.\tools\flash_firmware.ps1
```

If a remembered script path is missing, say that clearly and fall back to explicit commands only when the project memory gives enough information.

For STM32CubeIDE generated makefiles, always request the target explicitly:

```powershell
& $make -C Debug all -j8
```

Do not run bare `make -C Debug`. Some generated makefiles can resolve the first included target unexpectedly and remove the ELF.

When invoking CubeIDE's bundled tools manually:

1. Locate `make.exe` under the CubeIDE `externaltools.make` plugin.
2. Locate the GNU Arm toolchain `bin` directory under the CubeIDE `gnu-tools-for-stm32` plugin.
3. Add the GNU Arm `bin` directory to `PATH` for the process if needed.
4. Run from the project root.
5. Verify the build output, exit code, and ELF timestamp.

## Flash / Download Workflow

Flash only after a successful build or when the user explicitly asks to reflash an existing ELF.

Prefer the project's remembered or scripted command. A typical safe pattern is:

```powershell
& $programmer -c port=SWD mode=UR freq=4000 -w Debug\ProjectName.elf -v -rst
```

Use the project's known-good connection mode and reset mode when documented. Common modes include `mode=UR`, `mode=HotPlug`, and hardware reset options, but do not guess if the board is sensitive.

Do not perform full chip erase, mass erase, option byte writes, readout-protection changes, or calibration-page erase unless the user explicitly asks and the project memory confirms the safe procedure.

Preserve parameter/calibration Flash pages. Check linker scripts, flash-parameter modules, and project memory before changing erase behavior.

## Debug Workflow

Use project memory to select the right debug route:

- Serial or upper-computer tools for protocol/state validation.
- STM32 Programmer CLI RAM reads for live state inspection.
- ELF symbol tools such as `arm-none-eabi-nm` or map files for variable addresses.
- STM32CubeIDE debug launch files only after confirming they match the active project and target.

When runtime state matters:

1. Build or locate the exact ELF currently flashed.
2. Find symbol addresses from the ELF or map file; do not reuse stale addresses after code changes.
3. Read RAM through the project's verified ST-LINK mode, often `HotPlug` for non-halting inspection.
4. Compare RAM, serial output, and project documentation before changing firmware.

If serial debugging is involved:

- Confirm the actual USB serial device, baud rate, and protocol mode from project memory.
- Check TX/RX counters, frame counters, CRC errors, and parser state separately.
- Do not assume a browser or upper-computer page selected the intended COM port.

## Validation

Report the important verification facts:

- Build command used.
- ELF path and whether it exists after build.
- Size summary when available, such as `text`, `data`, and `bss`.
- Flash command used, excluding unnecessary private serial details.
- Programmer result, especially `Download verified successfully` or the exact failure.
- Debug evidence used: serial response, RAM readback, symbol address source, or tool log.
- Target voltage/target MCU only when relevant to the user's task.

If flashing or debugging fails:

1. Check whether STM32CubeIDE debug, STM32CubeProgrammer, OpenOCD, pyOCD, serial terminals, browser serial pages, or another process holds the device.
2. Check USB/ST-LINK connection and target power.
3. Re-read project memory for required reset/connect-under-reset/HotPlug mode.
4. Do not switch tools blindly. `pyocd`, OpenOCD, Keil, and CubeProgrammer are not interchangeable unless the project has verified them.

## Editor Notes

This skill works regardless of whether the user edits with Zed, Cursor, VS Code, or STM32CubeIDE.

- Zed can stay open while creating or using the skill.
- Cursor or VS Code may index large generated folders; avoid relying on their search when the terminal can inspect exact files.
- Exclude or ignore generated folders such as `Debug`, `Release`, `Drivers`, `.git`, and large PDFs during broad searches when possible.
- Building, flashing, and low-level debugging are terminal/toolchain actions, not editor actions.
