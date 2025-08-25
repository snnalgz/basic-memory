# Basic Memory: The Comprehensive "Full+Full" Setup Guide (v3)

Welcome to the ultimate guide for `basic-memory`. This guide provides a complete, in-depth walkthrough of every feature, from installation and client configuration to advanced topics like data importing and the legal inventory system. This is the "ultradeep" guide you requested, with detailed explanations of *how* key components work.

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
7.  [Importing Your Data](#7-importing-your-data)
    *   [Importing from ChatGPT](#importing-from-chatgpt)
    *   [Importing from Claude](#importing-from-claude)
    *   [Importing from a `basic-memory` JSON Backup](#importing-from-a-basic-memory-json-backup)
8.  [Advanced Topics: How It Works](#8-advanced-topics-how-it-works)
    *   [The `memory.json` File](#the-memoryjson-file)
    *   [The Legal & IP Inventory System](#the-legal--ip-inventory-system)
    *   [Understanding the Search Engine](#understanding-the-search-engine)
    *   [Understanding Prompts and Templates](#understanding-prompts-and-templates)

---

## 1. Introduction: What is Basic Memory?

Basic Memory is a powerful tool that allows you to build a persistent, local-first knowledge base using simple Markdown files. It acts as a long-term memory for your interactions with Large Language Models (LLMs), enabling them to read from and write to your local notes.

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

*   **Projects:** A directory on your filesystem that contains a collection of notes.
*   **Markdown as Source of Truth:** Your knowledge is stored in standard Markdown files.
*   **Semantic Markdown:** `basic-memory` understands special syntax (`- [category] ...` for observations, `[[Note Title]]` for relations) to build a knowledge graph.
*   **MCP Server:** A local server (`basic-memory mcp`) that exposes tools to AI clients.
*   **Sync Service:** A background service (`basic-memory sync --watch`) that keeps your knowledge graph and search index up-to-date with your files in real-time.

## 4. Getting Started: Your First Project

1.  **Create a Project:** `basic-memory init`
2.  **Start the MCP Server:** (In a new terminal) `basic-memory mcp`
3.  **Run the Real-time Sync Service:** (In another new terminal) `basic-memory sync --watch`

## 5. Client Configuration

### Visual Studio Code
Add this to your `settings.json`:
```json
"mcp": { "servers": { "basic-memory": { "command": "uvx", "args": ["basic-memory", "mcp"] } } }
```

### Claude Desktop
Add this to `claude_desktop_config.json`:
```json
{ "mcpServers": { "basic-memory": { "command": "uvx", "args": ["basic-memory", "mcp"] } } }
```

### Amazon Q Developer CLI
This repository contains a pre-built agent configuration at `.amazonq/agent.json`.
```bash
mkdir -p ~/.amazonq/agents
cp .amazonq/agent.json ~/.amazonq/agents/basic-memory-agent.json
q chat --agent basic-memory-agent
```

### Obsidian (Best Practices)
1.  **Open as Vault:** In Obsidian, use "Open folder as vault" and select your `basic-memory` project directory.
2.  **Enable Core Plugins:** Enable `Canvas`, `Daily Notes`, and `Tags` in Obsidian's settings for the best experience.
3.  **Real-time Sync is Key:** Always run `basic-memory sync --watch`.
4.  **Embrace Wiki-Links:** Use `[[Note Title]]` to create relations.

## 6. The Full Feature Guide: Mastering the Tools
(For a full list of tools and their parameters, please refer to the tool definitions in `src/basic_memory/mcp/tools/`.)

The tools allow you to manage notes (`write_note`, `read_note`, `edit_note`, `delete_note`, `move_note`), manage projects (`list_memory_projects`, `switch_project`, etc.), and discover information (`search_notes`, `list_directory`, `recent_activity`, `build_context`).

## 7. Importing Your Data

Use the `basic-memory import` CLI command to import data.

### Importing from ChatGPT
```bash
basic-memory import chatgpt /path/to/your/conversations.json --destination-folder "imported/chatgpt"
```

### Importing from Claude
**Conversations:**
```bash
basic-memory import claude-conversations /path/to/your/claude_conversations.json --destination-folder "imported/claude"
```
**Projects:**
```bash
basic-memory import claude-projects /path/to/your/claude_projects.json --destination-folder "imported/claude-projects"
```

### Importing from a `basic-memory` JSON Backup
```bash
basic-memory import memory-json /path/to/your/memory.json
```

## 8. Advanced Topics: How It Works

### The `memory.json` File
This file is a line-delimited JSON file that acts as a raw, portable dump of the entire knowledge graph.

*   **How it Works:** Each line in the file is a complete JSON object, representing either an "entity" (a node in the graph) or a "relation" (an edge connecting two nodes). This format is efficient for streaming and allows the `memory-json` importer to read the file line-by-line to reconstruct a knowledge base without consuming excessive memory.
*   **Purpose:** Its primary purpose is for backups, migrations between projects, or for seeding a new `basic-memory` instance with a pre-existing knowledge graph.

### The Legal & IP Inventory System
This is a suite of administrative scripts for tracking intellectual property and contributions. As an end-user, you won't need to run these, but understanding them reveals the project's maturity. The main script is `scripts/generate_legal_inventory.py`, which produces reports like the one in `legal_inventory_sample.md`.

#### How the Exhibit Generation Scripts Work
The scripts `create_individual_exhibits.py` and `create_csv_exhibits.py` are used to generate reports for specific contributors. They function as follows:

1.  **Load Master Inventory:** Both scripts start by loading a master JSON inventory file (generated by the main inventory script), which contains a list of every file in the repo and all of its contributors.
2.  **Define Targets:** They have a hard-coded list of `target_contributors` they need to generate reports for.
3.  **Filter Contributions:** For each target contributor, the scripts iterate through the entire master file list and select only those files that the target person contributed to.
4.  **Generate Output:**
    *   `create_individual_exhibits.py` constructs a formal **Markdown document** for each person, formatted as a legal "Exhibit A".
    *   `create_csv_exhibits.py` constructs a **CSV file** for each person, suitable for analysis in a spreadsheet.
5.  **Save Files:** The final Markdown or CSV files are saved into the `legal_exhibits/` directory.

### Understanding the Search Engine
`basic-memory` uses the powerful **FTS5 full-text search engine** from SQLite.
*   **How it Works:** The file `src/basic_memory/models/search.py` defines the schema for a virtual table called `search_index`. When you sync your project, the content of your notes is tokenized (broken into words) and inserted into this table.
*   **What's Indexed:** The `title`, `content`, and `permalink` of your notes are indexed for fast searching.
*   **Path Searching:** The search engine is specifically configured to treat forward slashes (`/`) as part of a token. This is why you can effectively search for file paths (e.g., `search_notes("imported/chatgpt/*")`).

### Understanding Prompts and Templates
The directory `src/basic_memory/templates/prompts` contains Handlebars templates (`.hbs` files).
*   **How they Work:** These templates are used to generate the prompts for the AI assistant. For example, `continue_conversation.hbs` is used to structure the response when you ask the AI to continue a discussion. They dynamically insert information about your notes into the prompt text.
*   **Purpose:** They are designed to be proactive and helpful, encouraging the AI to not just retrieve information but also to help you capture new knowledge, making `basic-memory` a more interactive partner.
