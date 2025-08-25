# Basic Memory: The Comprehensive "Full+Full" Setup Guide (v4)

Welcome to the definitive guide for `basic-memory`. This guide provides a complete, code-level deep dive into every feature, from installation and configuration to the complete CLI reference and the inner workings of its advanced systems. This is the "ultradeep" guide you requested.

## Table of Contents
1.  [Introduction: What is Basic Memory?](#1-introduction-what-is-basic-memory)
2.  [Installation](#2-installation)
3.  [Core Concepts](#3-core-concepts)
4.  [Getting Started: Your First Project](#4-getting-started-your-first-project)
5.  [The `config.json` File: The Heart of the System](#5-the-configjson-file-the-heart-of-the-system)
6.  [Complete CLI Command Reference](#6-complete-cli-command-reference)
7.  [Client Configuration for AI Agents](#7-client-configuration-for-ai-agents)
8.  [Importing Your Data](#8-importing-your-data)
9.  [Advanced Topics: How It Works](#9-advanced-topics-how-it-works)
    *   [The `memory.json` File](#the-memoryjson-file)
    *   [The Legal & IP Inventory System](#the-legal--ip-inventory-system)
    *   [Understanding the Search Engine](#understanding-the-search-engine)
    *   [Understanding Prompts and Templates](#understanding-prompts-and-templates)

---

## 1. Introduction: What is Basic Memory?
`basic-memory` is a powerful tool that allows you to build a persistent, local-first knowledge base using simple Markdown files. It acts as a long-term memory for your interactions with Large Language Models (LLMs), enabling them to read from and write to your local notes.

## 2. Installation
The recommended way to install is using `uv`:
```bash
pip install uv
uv tool install basic-memory
```
Verify with: `basic-memory --version`.

## 3. Core Concepts
*   **Projects:** A directory on your filesystem that contains a collection of notes.
*   **Markdown as Source of Truth:** Your knowledge is stored in standard Markdown files.
*   **Semantic Markdown:** `basic-memory` understands special syntax (`- [category] ...`, `[[Note Title]]`) to build a knowledge graph.
*   **MCP Server:** A local server (`basic-memory mcp`) that exposes tools to AI clients.
*   **Sync Service:** A background service (`basic-memory sync --watch`) that keeps your knowledge graph and search index up-to-date with your files.

## 4. Getting Started: Your First Project
1.  **Create a Project:** `basic-memory init`
2.  **Start the MCP Server:** (In a new terminal) `basic-memory mcp`
3.  **Run the Real-time Sync Service:** (In another new terminal) `basic-memory sync --watch`

## 5. The `config.json` File: The Heart of the System
The entire `basic-memory` system is configured through a single file: `config.json`.

*   **Location:** This file is located in a hidden directory in your user home folder: `~/.basic-memory/config.json`.
*   **How it Works:** The logic in `src/basic_memory/config.py` defines and manages this file. If it doesn't exist on first run, a default one is created. The `ConfigManager` class handles all reads and writes to this file, which are then triggered by the `basic-memory project` CLI commands.

### All Configuration Keys Explained

| Key | Type | Default Value | Description |
|:----|:-----|:--------------|:------------|
| `env` | string | `"dev"` | The operating environment ("dev", "test", or "user"). |
| `projects` | object | `{"main": "~/basic-memory"}` | A dictionary mapping your project names to their absolute paths on your filesystem. This is the central registry of all your knowledge bases. |
| `default_project` | string | `"main"` | The name of the project to use when you don't specify one with the `--project` flag. |
| `log_level` | string | `"INFO"` | The logging level for the application. Can be "DEBUG", "INFO", "WARNING", "ERROR". |
| `sync_delay` | integer | `1000` | The time in milliseconds that the watch service waits after detecting a file change before it starts a sync operation. |
| `update_permalinks_on_move` | boolean | `false` | If `true`, the system will update the `permalink:` in a note's frontmatter when you move it. |
| `sync_changes` | boolean | `true` | Globally enables or disables the real-time sync service. |
| `api_url` | string | `null` | For advanced use: if you are running a remote `basic-memory` server, you can put its URL here to have your local client connect to it. |

## 6. Complete CLI Command Reference
This is the complete command tree for the `basic-memory` CLI, based on the source code in `src/basic_memory/cli/`.

*   `basic-memory [OPTIONS]`
    *   `--project, -p TEXT`: Use a specific project for this command.
    *   `--version, -v`: Show version and exit.

*   `basic-memory reset [OPTIONS]`
    *   **Description:** Deletes the database and resets `config.json` to default.
    *   `--reindex`: If specified, also runs `basic-memory sync` to rebuild the database from your files.

*   `basic-memory status [OPTIONS]`
    *   **Description:** Shows which files have changed and are out of sync with the database.
    *   `--verbose, -v`: Shows a detailed list of every changed file instead of a summary.

*   `basic-memory sync [OPTIONS]`
    *   **Description:** Performs a one-time sync, updating the database to match the state of your files.
    *   `--verbose, -v`: Shows a detailed list of every file that was synced.

*   `basic-memory mcp [OPTIONS]`
    *   **Description:** Starts the MCP server for AI clients to connect to. Also starts the background file sync service.
    *   `--transport TEXT`: `stdio`, `streamable-http`, or `sse`. Default: `stdio`.
    *   `--host TEXT`: Host IP for HTTP. Default: `0.0.0.0`.
    *   `--port INTEGER`: Port for HTTP. Default: `8000`.
    *   `--path TEXT`: URL path for HTTP. Default: `/mcp`.

*   `basic-memory project`
    *   `list`: Lists all configured projects.
    *   `add <NAME> <PATH> [--default]`: Adds a new project.
    *   `remove <NAME>`: Removes a project from the configuration.
    *   `default <NAME>`: Sets a project as the default.
    *   `sync-config`: Synchronizes project configurations between `config.json` and the database.
    *   `move <NAME> <NEW_PATH>`: Updates the configured path for a project.
    *   `info [--json]`: Displays a detailed dashboard of statistics for the current project.

*   `basic-memory tool`
    *   **Description:** Provides direct command-line access to the MCP tools.
    *   `write-note`, `read-note`, `build-context`, `recent-activity`, `search-notes`, `continue-conversation`. Each of these subcommands is a direct wrapper around the corresponding MCP tool.

*   `basic-memory import`
    *   `chatgpt [OPTIONS] [CONVERSATIONS_JSON]`: Imports a ChatGPT `conversations.json` file.
    *   `claude conversations [OPTIONS] [CONVERSATIONS_JSON]`: Imports a Claude `conversations.json` file.
    *   `claude projects [OPTIONS] [PROJECTS_JSON]`: Imports a Claude `projects.json` file.
    *   `memory-json`: Imports a `basic-memory` backup file.

## 7. Client Configuration for AI Agents
(This section remains the same as v3, detailing setup for VS Code, Claude, Amazon Q, and Obsidian.)

## 8. Importing Your Data
(This section remains the same as v3, detailing the `basic-memory import` commands.)

## 9. Advanced Topics: How It Works

### The `memory.json` File
This file is a line-delimited JSON backup of the knowledge graph.
*   **How it Works:** Each line is a complete JSON object, either an "entity" (node) or a "relation" (edge). The `memory-json` importer reads this file line-by-line to reconstruct a knowledge base.
*   **Purpose:** Backups, migrations, and project seeding.

### The Legal & IP Inventory System
These are administrative scripts for tracking contributions. The main script is `scripts/generate_legal_inventory.py`.

#### How the Exhibit Generation Scripts Work
The scripts `create_individual_exhibits.py` and `create_csv_exhibits.py` generate reports for specific contributors.
1.  **Load Master Inventory:** They read the master JSON inventory file generated by the main script.
2.  **Define Targets:** They have a hard-coded list of contributors to generate reports for.
3.  **Filter Contributions:** They loop through all files in the inventory and select only those a target person contributed to.
4.  **Generate Output:** `create_individual_exhibits.py` creates a formal Markdown "Exhibit A" document. `create_csv_exhibits.py` creates a data-centric CSV file.
5.  **Save Files:** The final reports are saved into the `legal_exhibits/` directory.

### Understanding the Search Engine
`basic-memory` uses the **FTS5 full-text search engine** from SQLite.
*   **How it Works:** The file `src/basic_memory/models/search.py` defines the schema for a virtual search table. When you sync, the content of your notes is tokenized and put into this table.
*   **Path Searching:** The engine is configured to treat slashes (`/`) as part of a word, so you can search for paths like `search_notes("imported/chatgpt/*")`.

### Understanding Prompts and Templates
The `src/basic_memory/templates/prompts` directory contains Handlebars templates (`.hbs`).
*   **How they Work:** These templates dynamically generate the prompts for the AI assistant, inserting information from your notes.
*   **Purpose:** They are designed to be proactive, encouraging the AI to not just retrieve information but also to help you capture new knowledge.
