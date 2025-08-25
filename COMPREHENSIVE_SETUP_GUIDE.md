# Basic Memory: The Comprehensive "Full+Full" Setup Guide (v2)

Welcome to the ultimate guide for `basic-memory`. This guide provides a complete, in-depth walkthrough of every feature, from installation and client configuration to advanced topics like data importing and the legal inventory system. This is the "ultradeep" guide you requested.

## Table of Contents
1.  [Introduction: What is Basic Memory?](#1-introduction-what-is-basic-memory)
2.  [Installation](#2-installation)
3.  [Core Concepts](#3-core-concepts)
4.  [Getting Started: Your First Project](#4-getting-started-your-first-project)
5.  [Client Configuration](#5-client-configuration)
    *   [Visual Studio Code](#visual-studio-code)
    *   [Claude Desktop](#claude-desktop)
    *   [Amazon Q Developer CLI](#amazon-q-developer-cli)
    *   [Obsidian (Best Practices)](#obsidian-best-practices)
6.  [The Full Feature Guide: Mastering the Tools](#6-the-full-feature-guide-mastering-the-tools)
    *   [Core Note Management](#core-note-management)
    *   [Project Management](#project-management)
    *   [Navigation and Discovery](#navigation-and-discovery)
    *   [Advanced Tools](#advanced-tools)
7.  [Importing Your Data](#7-importing-your-data)
    *   [Importing from ChatGPT](#importing-from-chatgpt)
    *   [Importing from Claude](#importing-from-claude)
    *   [Importing from a `basic-memory` JSON Backup](#importing-from-a-basic-memory-json-backup)
8.  [Advanced Topics](#8-advanced-topics)
    *   [Understanding the Search Engine](#understanding-the-search-engine)
    *   [The Legal & IP Inventory System](#the-legal--ip-inventory-system)
    *   [Understanding Prompts and Templates](#understanding-prompts-and-templates)

---

## 1. Introduction: What is Basic Memory?

Basic Memory is a powerful tool that allows you to build a persistent, local-first knowledge base using simple Markdown files. It acts as a long-term memory for your interactions with Large Language Models (LLMs), enabling them to read from and write to your local notes.

**Key Features:**

*   **Local-First:** All your data lives in Markdown files on your computer. You are in full control.
*   **Bi-Directional:** Both you and your AI assistant can read and write to the same knowledge base.
*   **Knowledge Graph:** `basic-memory` automatically builds a semantic knowledge graph from your notes using simple wiki-links (`[[Note Title]]`) and other conventions.
*   **Client Agnostic:** It can be integrated with any client that supports the Model Context Protocol (MCP), including VS Code, Claude Desktop, and Amazon Q.

## 2. Installation

`basic-memory` is a Python application. The recommended way to install it is using `uv`, a fast Python package installer.

```bash
# Install uv (if you don't have it)
pip install uv

# Install basic-memory as a command-line tool
uv tool install basic-memory
```
Verify the installation by running: `basic-memory --version`.

## 3. Core Concepts

*   **Projects:** A project is a directory on your filesystem that contains a collection of notes. You can have multiple projects (e.g., for work, personal, etc.).
*   **Markdown as Source of Truth:** Your knowledge is stored in standard Markdown files, making them portable, human-readable, and easy to edit with any tool.
*   **Semantic Markdown:** `basic-memory` understands special syntax within your Markdown files to build the knowledge graph:
    *   **Observations:** `- [category] Fact or idea #tag (context)`
    *   **Relations:** `- relates_to [[Other Note Title]]`
*   **The MCP Server:** A local server (`basic-memory mcp`) that exposes tools to AI clients.
*   **The Sync Service:** A background service (`basic-memory sync --watch`) that keeps your knowledge graph and search index up-to-date with your files in real-time.

## 4. Getting Started: Your First Project

1.  **Create a Project:**
    ```bash
    # This interactive command is the easiest way to start
    basic-memory init
    ```
2.  **Start the MCP Server:** (In a new terminal)
    ```bash
    basic-memory mcp
    ```
3.  **Run the Real-time Sync Service:** (In another new terminal)
    ```bash
    basic-memory sync --watch
    ```
You are now fully set up and ready to connect a client.

## 5. Client Configuration

### Visual Studio Code
Add the following to your `settings.json` (`Ctrl+Shift+P` -> `Preferences: Open User Settings (JSON)`):
```json
{
  "mcp": {
    "servers": {
      "basic-memory": { "command": "uvx", "args": ["basic-memory", "mcp"] }
    }
  }
}
```

### Claude Desktop
Add the following to your `claude_desktop_config.json` (e.g., `~/Library/Application Support/Claude/claude_desktop_config.json` on Mac):
```json
{
  "mcpServers": {
    "basic-memory": { "command": "uvx", "args": ["basic-memory", "mcp"] }
  }
}
```

### Amazon Q Developer CLI
This repository now contains a pre-built agent configuration at `.amazonq/agent.json`.
```bash
# Copy the agent configuration to the Amazon Q agents directory
mkdir -p ~/.amazonq/agents
cp .amazonq/agent.json ~/.amazonq/agents/basic-memory-agent.json

# Start a chat session using the new agent
q chat --agent basic-memory-agent
```

### Obsidian (Best Practices)

`basic-memory` and Obsidian are a perfect match.
1.  **Open as Vault:** In Obsidian, use "Open folder as vault" and select your `basic-memory` project directory.
2.  **Enable Core Plugins:** For the best experience, ensure these core plugins are enabled in Obsidian's settings:
    *   **Canvas:** To view and edit `.canvas` files created by the `canvas` tool.
    *   **Daily Notes:** Useful for capturing daily thoughts that can be linked into your knowledge graph.
    *   **Tags:** The tags you use in `basic-memory` frontmatter will be recognized by Obsidian.
3.  **Real-time Sync is Key:** Always have the `basic-memory sync --watch` command running. When your AI assistant creates a note, it will appear in your vault instantly.
4.  **Embrace Wiki-Links:** The `[[Note Title]]` syntax is the primary way to create relations in `basic-memory`. Use them liberally in Obsidian to build your knowledge graph. `basic-memory` will automatically resolve these links, even if the target note doesn't exist yet (a "forward reference").

## 6. The Full Feature Guide: Mastering the Tools

This is a reference for all tools exposed by the `basic-memory` MCP server.

### Core Note Management

| Tool | Description |
| :--- | :--- |
| **`write_note`** | Creates or updates a note. The primary tool for adding knowledge. |
| **`read_note`** | Reads the full Markdown content of a single note by its title or permalink. |
| **`edit_note`** | Performs targeted edits on an existing note (append, prepend, find/replace). Requires an *exact* identifier. |
| **`delete_note`** | Deletes a note from the knowledge base and the filesystem. |
| **`move_note`** | Moves a note to a new file path, updating all links and references in the database. |

### Project Management

| Tool | Description |
| :--- | :--- |
| **`list_memory_projects`** | Lists all configured projects. |
| **`get_current_project`** | Shows the name and stats for the currently active project. |
| **`switch_project`** | Switches the active context to a different project. |
| **`create_memory_project`** | Creates and registers a new project. |
| **`set_default_project`** | Sets a project as the default for when the server starts. |
| **`delete_project`** | Removes a project from the configuration (does not delete files). |

### Navigation and Discovery

| Tool | Description |
| :--- | :--- |
| **`search_notes`** | Performs a powerful search. See the "Understanding the Search Engine" section for details. |
| **`list_directory`** | Lists the files and directories in your project, like `ls`. |
| **`recent_activity`** | Shows the most recently created or modified notes and relations. |
| **`build_context`** | Gathers context around a topic, traversing the knowledge graph to find related information. |

### Advanced Tools

| Tool | Description |
| :--- | :--- |
| **`canvas`** | Creates an Obsidian Canvas file (`.canvas`) from a list of nodes and edges. |
| **`read_content`** | Reads the raw content of any file, including images and other binary files. |
| **`view_note`** | A wrapper around `read_note` that formats the output as a special "artifact" for clients like Claude Desktop. |
| **`sync_status`** | Checks the status of the file synchronization service. |

## 7. Importing Your Data

`basic-memory` includes a powerful CLI for importing your existing data from other services. You use the `basic-memory import` command.

### Importing from ChatGPT
1.  Request your data export from ChatGPT. You will receive an email with a link to download a `.zip` file.
2.  Unzip the file and locate `conversations.json`.
3.  Run the following command:
    ```bash
    basic-memory import chatgpt /path/to/your/conversations.json --destination-folder "imported/chatgpt"
    ```

### Importing from Claude
Claude offers two types of exports: conversations and projects.

**To import conversations:**
1.  Export your conversations from Claude's settings. You will receive a `claude_conversations.json` file.
2.  Run the command:
    ```bash
    basic-memory import claude-conversations /path/to/your/claude_conversations.json --destination-folder "imported/claude"
    ```

**To import projects:**
1.  Export your projects from Claude. You will receive a `claude_projects.json` file.
2.  Run the command:
    ```bash
    basic-memory import claude-projects /path/to/your/claude_projects.json --destination-folder "imported/claude-projects"
    ```

### Importing from a `basic-memory` JSON Backup
The `memory-json` importer can restore a knowledge base from a `memory.json` file. This is useful for backups or migrating between projects.
```bash
basic-memory import memory-json /path/to/your/memory.json
```

## 8. Advanced Topics

### Understanding the Search Engine
`basic-memory` uses the powerful **FTS5 full-text search engine** built into SQLite. This provides fast and advanced search capabilities.
*   **What's Indexed:** The `title`, `content`, and `permalink` of your notes are indexed for searching.
*   **Path Searching:** The search engine is specifically configured to treat forward slashes (`/`) as part of a word. This allows you to effectively search for file paths, e.g., `search_notes("imported/chatgpt/*")`.
*   **Prefix Searching:** The index is optimized for prefix searches (e.g., `search*`), which makes finding files in a specific directory very fast.

### The Legal & IP Inventory System
You may have noticed several scripts in the repository related to "legal inventory" (e.g., `scripts/generate_legal_inventory.py`).
*   **Purpose:** These are administrative tools for the project maintainers to track contributions and manage intellectual property (IP) for legal purposes like copyright assignment.
*   **Functionality:** The main script scans the entire repository using `git` history, identifies every file and every contributor to that file, and generates a detailed report in various formats (CSV, JSON, Markdown).
*   **Relevance to You:** As an end-user, you will likely never need to run these scripts. However, their existence demonstrates the project's maturity and commitment to proper open-source management.

### Understanding Prompts and Templates
The directory `src/basic_memory/templates/prompts` contains Handlebars templates (`.hbs` files) that are used to generate the prompts for the AI assistant. For example, `continue_conversation.hbs` is used to structure the response when you ask the AI to continue a discussion. These templates are designed to be proactive and helpful, encouraging the AI to not just retrieve information but also to help you capture new knowledge. They are a key part of what makes `basic-memory` feel like a true partner in knowledge creation.
