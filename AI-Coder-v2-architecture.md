# AI-Coder v2.0 — Technical Architecture Blueprint

## 1. Goal

AI-Coder should become a true cross-platform local AI coding assistant, not just a GUI wrapper around one LLM.

The project must provide:

- local-first AI coding workflow
- automatic dependency installation
- installer for Windows and Linux
- mobile companion support (Android/remote control)
- model auto-download and management
- secure execution of tasks
- update and rollback support

## 2. Core concept

AI-Coder should be split into three major layers:

1. Core runtime
   - project analysis
   - task planning
   - file editing
   - git integration
   - command execution
   - sandboxed tool execution

2. UI layer
   - desktop UI for Windows/Linux/macOS
   - mobile companion UI for Android

3. Runtime/dependency layer
   - Python runtime
   - model runtime (Ollama / llama.cpp)
   - toolchain install
   - updater
   - version manifests

This separation is critical: the “brain” of the system should not be coupled to a single platform UI.

---

## 3. Target platform support

### Desktop
- Windows 10/11
- Linux (Ubuntu/Debian/Fedora-based)
- macOS (later milestone)

### Mobile companion
- Android (primary companion app)
- iOS only after the desktop system is stable

### Local AI runtime
- Ollama (preferred)
- llama.cpp (fallback)
- model registry with hardware detection

---

## 4. Recommended project structure

```text
AI-Coder/
├── apps/
│   ├── desktop/
│   │   ├── gui/
│   │   ├── core-shell/
│   │   └── launcher/
│   ├── mobile/
│   │   └── app/
│   └── installer/
│       ├── windows/
│       ├── linux/
│       └── updater/
├── packages/
│   ├── core/
│   │   ├── agent/
│   │   ├── planner/
│   │   ├── project_manager/
│   │   ├── task_runner/
│   │   ├── git_tools/
│   │   └── sandbox/
│   ├── llm/
│   │   ├── model_manager/
│   │   ├── ollama_adapter/
│   │   ├── llama_cpp_adapter/
│   │   └── model_registry/
│   ├── ui/
│   │   ├── desktop_ui/
│   │   └── mobile_ui/
│   └── common/
│       ├── config/
│       ├── events/
│       └── schemas/
├── services/
│   ├── runtime/
│   ├── sandbox/
│   ├── updater/
│   ├── config_service/
│   └── telemetry/
├── scripts/
│   ├── bootstrap.py
│   ├── install_windows.ps1
│   ├── install_linux.sh
│   ├── setup_models.py
│   └── update.py
├── config/
│   ├── app.json
│   ├── models.json
│   └── release_manifest.json
├── docs/
│   ├── architecture.md
│   ├── install.md
│   ├── security.md
│   └── model_guide.md
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── requirements.txt
├── pyproject.toml
├── README.md
├── LICENSE
└── .github/
    └── workflows/
```

---

## 5. Component breakdown

### 5.1 Installer / bootstrap system

Purpose:
- detect OS and architecture
- install missing runtime dependencies
- install Python if needed
- install Git if missing
- install Ollama or llama.cpp
- download default model(s)
- verify checksums
- create config and profiles

Flow:

```text
Start installer
  -> Detect OS + CPU + RAM + GPU
  -> Check required runtime files
  -> Download missing runtime packages
  -> Validate SHA256 signatures
  -> Install dependencies
  -> Download recommended default model
  -> Create app config
  -> Launch app
```

Requirements:
- one-click setup
- safe reinstall support
- update capability
- rollback support

---

### 5.2 Model Manager

Purpose:
- maintain supported model list
- choose the best model by available hardware
- auto-download model weights
- manage multiple model versions
- toggle local vs remote inference

Recommended model tiers:

- Tiny: 1B–3B
- Balanced: 7B–8B
- Advanced: 14B–32B

Hardware-based selection:

- low-end CPU: tiny model only
- modern laptop with 16GB RAM: 7B–8B model
- GPU with sufficient VRAM: 14B+/32B if supported

Model registry format:

```json
{
  "models": [
    {
      "id": "qwen2.5-coder-7b",
      "provider": "ollama",
      "size": "7B",
      "min_ram_gb": 8,
      "min_vram_gb": 6,
      "recommended": true,
      "download_url": "https://example.com/model",
      "sha256": "..."
    }
  ]
}
```

---

### 5.3 Core agent runtime

This is the main logic layer. It should be able to:

- read project tree
- inspect files
- detect build system
- create task plan
- patch files
- run commands
- validate builds/tests
- handle git status, commits, diffs
- report progress to UI

Main modules:

- project_manager
  - scans repo structure
  - reads config files
  - builds workspace index

- task_planner
  - turns user request into executable steps
  - can split tasks into sub-tasks

- code_editor
  - applies code patches safely
  - supports function-level or file-level patching

- task_runner
  - executes build/test commands
  - handles command output
  - captures errors

- git_tools
  - status, diff, commit history, branch control

- sandbox
  - executes scripts in limited environment
  - blocks dangerous operations by default

---

### 5.4 Sandbox and security

This is mandatory. AI coding assistants writing code can perform dangerous operations.

Security rules:

- run commands inside isolated temp workspace
- allow only explicit project directories
- ask for confirmation before destructive actions
- block network access unless explicitly allowed
- block system-level writes outside project folder
- keep logs of executed commands
- create rollback points before major edits

Recommended sandbox model:

- per-task temp environment
- read-only project root by default
- editable path only within selected workspace
- logs stored locally in a secure directory

---

### 5.5 Update system

AI-Coder should check for updates automatically.

Requirements:

- version manifest per release
- SHA256 verification
- incremental patch support
- rollback on failed update
- pre-update backup
- no silent breakage across OS updates

Manifest example:

```json
{
  "version": "2.0.0",
  "platform": "windows-x64",
  "download_url": "https://github.com/den7447/AI-Coder/releases/download/v2.0.0/AI-Coder-windows-x64.zip",
  "sha256": "...",
  "required_runtime": "python3.12",
  "notes": "Fixes model manager + Linux launcher"
}
```

---

### 5.6 Mobile companion

The mobile app should not be a full local coding environment.

Instead, it should be a companion:

- manage tasks remotely
- monitor background jobs
- review code changes
- trigger builds
- receive notifications
- control local desktop AI agent

Best approach:

- desktop runs as local agent
- phone connects via local network or cloud
- phone acts as UI / remote controller

This is far more robust than trying to run full local models on phone hardware.

---

## 6. Recommended tech stack

### Desktop backend
- Python 3.12
- FastAPI or gRPC for internal service layer
- SQLite for project metadata and task history
- PySide6 for GUI
- GitPython for git interaction
- watchdog for file change monitoring
- PyInstaller or Nuitka for packaging

### Mobile
- Flutter
- API communication via REST/WebSocket

### Model runtime
- Ollama
- llama.cpp fallback

### Installer
- Windows: Inno Setup / NSIS / MSI
- Linux: AppImage / DEB / RPM

---

## 7. MVP sequence

### Stage 1 — foundation
- project structure
- desktop launcher
- Python runtime bootstrap
- installer script
- config manager
- local model manager

### Stage 2 — productivity core
- repo scanning
- planning layer
- file editing engine
- git integration
- task queue

### Stage 3 — verification
- build/test execution
- sandbox protection
- model error handling
- logs

### Stage 4 — distribution
- Windows installer
- Linux installer
- update manager
- release pipeline

### Stage 5 — mobile companion
- Android companion UI
- remote control mode
- background task monitoring

---

## 8. What should NOT be done

Avoid these mistakes:

- building everything in one Python file
- mixing UI, logic, and installer code in one layer
- shipping an app that assumes a user already knows how to install Ollama manually
- allowing arbitrary shell execution without sandboxing
- creating a Windows/Linux build without versioning and update support
- trying to run heavy LLMs on mobile without remote control model

---

## 9. Product requirements summary

AI-Coder should provide:

- local AI programming assistant
- one-click installation for Windows and Linux
- automatic model download and verification
- secure sandboxed task execution
- git-aware coding workflow
- project-aware planning
- desktop-first operation with mobile companion
- auto-updates and rollback safety

---

## 10. Final recommendation

The right path is:

1. Build desktop core first
2. Add installers for Windows/Linux
3. Add automatic model installation
4. Add update system
5. Add Android companion later

This gives a realistic and maintainable roadmap.

The project should evolve from “GUI wrapper around local LLM” into a proper local coding agent system.

---

## 11. Short strategic statement

AI-Coder is not just a coding app. It is a local software development agent platform for desktop and mobile companion workflows.

That is the direction that makes it look serious, scalable, and useful.

## 12. Suggested next milestone

The next milestone should be:

- “AI-Coder v2.0 — Desktop core + installer + model auto-download”

This is the first real version that feels production-ready.
