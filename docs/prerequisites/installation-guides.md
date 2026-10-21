# Installation Guides

Install the tools you need before starting the workshop.

The setup comes in two parts:

- **[Common tools](#common-tools)**: everyone needs these. They're used both when a coding agent builds
  automations for you and when you build a coded agent yourself.
- **[Coding agents](#coding-agents)**: the extra tools for the lessons where a coding agent, such as
  **Claude Code** or **Codex**, builds Studio workflows, coded agents and other automations for you.

!!! tip "Where to run these commands"
    Run every command in a terminal window. On **Windows**, open **PowerShell** (press the Windows key,
    type `PowerShell` and press Enter). On **Mac**, open **Terminal** (press Cmd + Space, type
    `Terminal` and press Enter). The install commands work from any folder. After installing a tool,
    close the terminal and open a new one, so it can find the tool.

## Common tools

### 1. Node.js

**Node.js** (LTS) — required for the UiPath CLI and the React app build.

1. Download the LTS version from [nodejs.org](https://nodejs.org)
2. Verify with `node --version` and `npm --version`

!!! tip "npm version must be 11.x.x"
    If `npm --version` returns a different version, upgrade it:

    === "Windows"

        ```bash
        npm install --global npm@11
        ```

    === "Mac"

        ```bash
        npm install -g npm@11
        ```

### 2. UiPath CLI

The `uip` CLI signs you in to UiPath and runs, publishes and manages your automations from the terminal.

```bash
npm install -g @uipath/cli@latest
```

Verify: `uip --version` — you should see `1.195.1` or later.

### 3. Python and uv

**Python** — required for coded agents, including the ones a coding agent generates for you. Coded
agents need **Python 3.11 to 3.13**.

The easiest way to get the right version is **uv**, a fast Python package manager. Coded agent
projects use it to create their Python environment, and it downloads Python 3.13 by itself when a
project needs it.

Install uv:

=== "Windows"

    ```powershell
    winget install astral-sh.uv
    ```

=== "Mac"

    ```bash
    curl -LsSf https://astral.sh/uv/install.sh | sh
    ```

Open a new terminal and verify with `uv --version`.

!!! note "Prefer a regular Python install?"
    Download Python 3.11, 3.12 or 3.13 from [python.org/downloads](https://www.python.org/downloads/) —
    on Windows, check **Add Python to PATH** during install. Verify with `python --version` or
    `python3 --version`.

### 4. Authenticate with Your Tenant

```bash
uip login --authority https://cloud.uipath.com --organization tpenlabs -t AgenticWorkshop
```

A browser window opens. Complete the sign-in there. Check the result with `uip login status`: it
should show **Logged in** for tenant **AgenticWorkshop**.

## Coding agents

The tools below are for the lessons where a coding agent builds automations for you, for example
Studio workflows (XAML), coded agents, React apps and Maestro processes.

### 1. .NET SDK 8.0

**.NET SDK 8.0** — required to build and run the UiPath Studio project.

1. Download from [dotnet.microsoft.com/download/dotnet/8.0](https://dotnet.microsoft.com/download/dotnet/8.0) — or on Mac, install via Homebrew:

    ```bash
    brew install --cask dotnet-sdk
    ```

2. Verify with `dotnet --list-sdks` — you should see `8.x.x` in the output

!!! tip "Windows — make sure .NET is on your PATH"
    If `dotnet` is not recognised after install, add it to the PATH manually in PowerShell:

    ```powershell
    $env:Path = "C:\Program Files\dotnet;$env:Path"
    dotnet --list-sdks
    ```

### 2. UiPath Studio

**UiPath Studio** — required to open, run, and debug the generated XAML workflow. Currently supported on Windows only; Mac support is coming soon.

1. Download from [uipath.com/start-trial](https://www.uipath.com/start-trial)
2. Sign in with your UiPath account and verify Studio launches

### 3. Git

**Git** — required for the UiPath skills marketplace. Try Method 1 first; if it doesn't work, use Method 2.

=== "Method 1: winget (quickest)"

    1. Open a terminal and run:

        ```bash
        winget install --id Git.Git -e --source winget
        ```

    2. If prompted to accept terms, type `Y` and press Enter.
    3. Wait for "Successfully installed", then close the window.

=== "Method 2: Download the installer"

    1. Go to [git-scm.com/download/win](https://git-scm.com/download/win) — the 64-bit installer should download automatically.
    2. Run the downloaded file (e.g. `Git-2.xx.x-64-bit.exe`). Click Yes if Windows asks for admin permission.
    3. Click through the installer accepting all defaults. On the **Adjusting your PATH environment** screen, make sure the middle option is selected: *"Git from the command line and also from 3rd-party software"* — this is the default.
    4. Click Finish when done.

!!! warning "Required for both methods"
    PATH changes only take effect in new windows. Close every open terminal, then open a fresh Command Prompt and verify:

    ```bash
    git --version
    ```

    You should see something like `git version 2.45.1.windows.1`.

### 4. Install a Coding Agent

Pick one — or install both:

```bash
# Claude Code (Anthropic)
npm install -g @anthropic-ai/claude-code

# Codex CLI (OpenAI)
npm install -g @openai/codex
```

Verify: `claude --version` or `codex --version`

!!! note "Claude Code installation error?"
    If the install fails with a scripts permission error, run:

    ```bash
    npm config set allow-scripts=@anthropic-ai/claude-code --location=user
    ```

    Then run the install command again:

    ```bash
    npm install -g @anthropic-ai/claude-code
    ```

### 5. Install UiPath Skills into Your Coding Agent

Install the skills that give the agent the knowledge it needs to generate Studio workflows, coded agents, React apps, and Maestro processes. They use the UiPath CLI you installed in [Common tools](#2-uipath-cli).

=== "Claude"

    ```bash
    # 1. Add the marketplace (once)
    claude plugin marketplace add https://github.com/UiPath/skills
    # 2. Install the plugin from it
    claude plugin install uipath@uipath-marketplace
    uip skills install
    ```

=== "Codex"

    ```bash
    # 1. Add the marketplace (once)
    codex plugin marketplace add UiPath/skills --ref main
    uip skills install
    ```

You should see output similar to this:

```json
{
  "Result": "Success",
  "Code": "SkillsInstall",
  "Data": {
    "RootDir": "/Users/YourUser",
    "Skills": [
      "uipath-admin",
      "uipath-agents",
      "uipath-api-workflow",
      "uipath-automation-discovery",
      "uipath-coded-apps",
      "uipath-data-fabric",
      "uipath-feedback",
      "uipath-governance",
      "uipath-human-in-the-loop",
      "uipath-maestro-bpmn",
      "uipath-maestro-case",
      "uipath-maestro-flow",
      "uipath-mcp-servers",
      "uipath-planner",
      "uipath-platform",
      "uipath-review",
      "uipath-rpa",
      "uipath-solution",
      "uipath-tasks",
      "uipath-test",
      "uipath-troubleshoot"
    ],
    "Agents": [
      "claude",
      "codex"
    ],
    "Installed": 42
  }
}
```

Confirm the following skills appear in the output:

- `uipath-rpa` — for Studio XAML workflows
- `uipath-agents` — for Python LangGraph coded agents

Each skill gives the agent deep knowledge of one part of the UiPath platform:

| Skill | What it knows |
| :--- | :--- |
| **uipath-rpa** | Studio XAML structure, activity libraries, selectors, error handling, Orchestrator Assets, packaging |
| **uipath-agents** | Python agent project layout, LangGraph patterns, tool definitions, UiPath AI Trust Layer |

Skills are auto-discovered on session start. Ask the agent to build a coded agent and it will load the right skill automatically.
