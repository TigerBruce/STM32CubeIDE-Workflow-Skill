# STM32CubeIDE Safe Flash

Codex skill for safely building and flashing STM32CubeIDE firmware projects.

中文名：STM32CubeIDE 安全编译烧录。

## What It Does

- Reads project-local memory and rules before touching firmware.
- Builds STM32CubeIDE generated projects with explicit make targets.
- Uses STM32 Programmer CLI for ST-LINK/SWD flashing.
- Avoids common mistakes such as bare `make -C Debug`, mass erase, stale snapshot paths, and unverified flashing tools.
- Keeps private project paths, ST-LINK serial numbers, COM ports, and calibration Flash addresses in project-local memory instead of the public skill.

## Install

Copy the skill folder into your Codex skills directory:

```powershell
Copy-Item -Recurse .\stm32-cubeide-safe-flash "$env:USERPROFILE\.codex\skills\"
```

Restart Codex or open a new session so the skill can be discovered automatically.

## Editor Compatibility

This is not a Zed, Cursor, or VS Code plugin. It is a Codex skill.

The workflow works with any editor because building and flashing are terminal/toolchain actions. Zed, Cursor, VS Code, STM32CubeIDE, and a plain terminal are only frontends.

For large STM32 projects, avoid broad editor indexing over generated folders such as `Debug`, `Release`, `Drivers`, `.git`, and large PDFs when possible.

## Skill Folder

The installable skill is:

```text
stm32-cubeide-safe-flash/
├── SKILL.md
└── agents/
    └── openai.yaml
```
