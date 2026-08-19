![preview](https://raw.githubusercontent.com/malikawais153960/tm-editor-bridge-mcp/main/showcase_82d6.svg)
# TrackFlow Studio Orchestrator

**TrackFlow Studio Orchestrator** is a creative control plane for Trackmania's editor and in-game menus, built for AI agents and automation pipelines that need a real-time JSON bridge over localhost. Instead of treating the game as a black box, this project opens a two-way conversation with the editor—letting your scripts, bots, and creative assistants read the current block grid, manipulate object placement, navigate menu stacks, and receive live state notifications as if they were sitting beside you at the keyboard.

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![Platform](https://img.shields.io/badge/platform-windows%20%7C%20linux-lightgrey)
![Language](https://img.shields.io/badge/language-json%20%7C%20lua-blue)
![License](https://img.shields.io/badge/license-MIT-green)

## Overview

Every Trackmania creator knows the friction: you have a vision, but the editor's interface demands your hands on the keyboard. TrackFlow Studio Orchestrator removes that friction by exposing a semantic, event-driven API for the game's editor and menu systems. Think of it as a translator that converts your creative intent into precise editor commands, and back again—capturing the editor's state as structured data your automation stack can consume.

The project is built around a lightweight localhost server that runs alongside the game. It listens for commands (place block, rotate item, open controller menu, change filter, save track) and emits events (block placed, selection changed, menu opened) with millisecond-level timestamps. This enables a new generation of Trackmania tools: AI-assisted building assistants, automated track generators, quality-of-life menu shortcuts, and even remote monitoring dashboards—all speaking a common JSON dialect.

## Why Another Control Bridge?

Existing solutions treat the editor as a sequence of hardcoded steps. TrackFlow Studio Orchestrator takes a different path. It models the editor as a state machine with a mutable scene graph, exposing operations as composable actions. Want to place 500 blocks in a spiral pattern? That is one loop in your script, not a manual click marathon. Need to verify your track meets a technical checkpoint? Query the bridge for block IDs and coordinates, run your validation logic, and receive precise diagnostics.

The bridge is designed to be **protocol-agnostic** on the client side. Whether you prefer Node.js, Python, Rust, or a simple shell script, any HTTP or WebSocket client can connect. For AI agents, the bridge offers a clean tool-calling surface: describe the editor state, propose changes, and observe the outcome—all within a single conversation turn.

## Core Features

- **Real-Time Editor State Mirror** — The bridge continuously synchronizes a JSON representation of the editor's state, including block placement, item transforms, and UI focus. Changes are pushed to subscribed clients without polling.
- **Semantic Command Layer** — Instead of raw keypresses, send high-level commands like `place_block_at_absolute_position`, `rotate_selection_around_vector`, `filter_block_by_category`, or `save_current_track_as`. The bridge translates these into the correct editor sequences.
- **Event Stream for Automation** — A WebSocket-compatible endpoint emits events for every relevant editor action. Hook in your own logic to react to changes, trigger validation, or log a creative session.
- **Multi-Session Support** — Run multiple instances concurrently for different track projects or editor profiles. Each session carries its own configuration, including keybindings and UI presets.
- **Schema Validation for Inputs** — Every command accepts a structured JSON payload, validated against a published schema. Invalid commands are rejected with descriptive errors before they reach the game.
- **Lightweight and Non-Intrusive** — The bridge operates over localhost, requiring no external services or network access. The game continues to function normally; the bridge simply adds a listening layer.

## Getting Started with TrackFlow

Before you begin, ensure your environment has a running Trackmania instance with the editor accessible. The bridge requires no special build tools or package managers—just a runtime that can execute the provided Lua scripts and a method to launch the bridge server.

### Prerequisites in Plain Terms

- A Trackmania install (any recent version that supports the Openplanet framework).
- The Openplanet plugin loader, configured to run local scripts.
- A way to run a JSON server—any language with an HTTP library works; the reference implementation uses LuaSocket.

### First Connection in Five Steps

1. **Locate the Bridge Script** — Inside the repository, find `trackflow_server.lua`. This script starts the localhost listener on a configurable port (default `8765`).
2. **Launch the Game and Load the Script** — Start Trackmania, open the plugin manager, and execute the bridge script. The console will print the bound address and port.
3. **Open Your Preferred Client** — Any tool that can send HTTP requests or open a WebSocket connection will work. For a quick test, a browser pointing to `http://127.0.0.1:8765/status` should return a JSON payload describing the editor's current mode.
4. **Subscribe to Events** — Connect to the WebSocket endpoint (`/ws`) to listen for state changes. The first event will be a full snapshot; subsequent events are incremental.
5. **Send Your First Command** — Post a JSON payload to `/command` with a `type` field set to `get_selection`. The response will list all currently selected blocks and their transforms.

## Architecture and Design Philosophy

TrackFlow Studio Orchestrator is not a monolithic addon but a three-layer system:

**Layer One: Game Adapter** — A Lua script that hooks into the editor's UI state and exposes function calls via a registered interface. This layer handles all communication with the game, translating high-level requests into specific editor API calls.

**Layer Two: Protocol Server** — A standalone HTTP/WebSocket server (also written in Lua, but portable to any language) that accepts JSON requests, validates them against schemas, and forwards them to the game adapter. It manages client connections, subscription lists, and event broadcasting.

**Layer Three: Client SDKs and Tools** — The repository includes reference client implementations in several languages, plus a command-line tool for quick interaction. These are thin wrappers that abstract the protocol into convenient function calls.

The separation allows each layer to be replaced independently. If a new Trackmania version changes the editor's internals, only the adapter needs updating. If you prefer a different server framework, the protocol spec remains stable.

## Use Cases That Shine

### AI-Assisted Track Design

Imagine asking an AI agent to "create a technical dirt section with three sharp turns and a jump." The agent connects to the bridge, queries the current track geometry, plans a block placement strategy, and sends commands to place each block. It can then request a screenshot or state verification to confirm the outcome. This turns a prompt into a tangible modification loop.

### Automated Quality Checks

Custom validation scripts can subscribe to block placement events. Every time you place a new block, the script runs through a rule set—checking for unreachable areas, excessive elevation changes, or track length limits—and provides immediate feedback in your console or through a separate dashboard.

### Menu Navigation Macros

Beyond the editor, the bridge controls menu stacks. You can write a script that saves a track, exits to the main menu, creates a new campaign slot, and imports a replay—all through JSON commands. This is invaluable for bulk operations or preparing game sessions.

### Live Spectator Views

For streamers, the event stream can feed a real-time track-building visualization. Place a block, and a web page updates with the new state, rendered with simple SVG or WebGL. This creates an engaging overlay for creative streams.

## Configuration and Customization

The bridge reads a `trackflow_config.json` file on startup. This file lets you define:

- **Port binding and interface address** (default `127.0.0.1:8765`).
- **Event filter rules** to reduce noise (e.g., ignore mouse movement events).
- **Command aliases** to map shorter names to longer command sequences.
- **Lua extensions** to add custom commands that mix bridge operations with your own scripted logic.

For power users, the protocol supports **transactions**: batch multiple commands into a single request, apply them all, and receive a single success/failure summary. This is useful for complex multi-step actions where partial application would be problematic.

## Performance and Responsiveness

Because the bridge works over localhost, latency is typically under one millisecond. The server is designed to handle dozens of concurrent clients without interfering with the game's frame rate. For event-heavy workloads, the bridge uses a buffering mechanism that coalesces rapid changes into fewer, larger payloads—reducing network overhead.

The JSON schema is deliberately compact. Field names are short, and optional values are omitted unless explicitly set. This keeps payload sizes small even for large track snapshots.

## Security and Data Handling

All communication is bound to the loopback interface by default, meaning no external device can connect unless you change the binding. The bridge performs no external network calls and stores no telemetry. The data flowing through it—track geometry, editor states, command sequences—stays on your machine.

For additional protection, the bridge supports a simple token-based authentication for WebSocket connections. If enabled, clients must present the token in the connection query string.

## Multilingual Support

The bridge's command and event schemas use English identifiers, but the description fields and error messages are localized. A translation file (`i18n_strings.json`) ships with the repository and currently supports English, German, French, and Japanese. Contributions for additional languages are welcome.

## Responsive Client Library

The reference JavaScript client is built with a responsive design pattern—it works equally well in a browser tab, a Node.js process, or a lightweight embedded runtime. The client automatically handles reconnections, event buffering, and schema version negotiation.

## 24/7 Support and Community

While this project is open-source, a dedicated community fork maintains a support channel where contributors answer questions about the protocol, share custom commands, and discuss automation patterns. For enterprise use cases, a commercial support tier is available, providing guaranteed response times and custom feature development.

## Roadmap for 2026

The upcoming year brings several ambitious features:

- **Visual Command Builder** — A drag-and-drop interface that generates JSON command sequences without writing code.
- **Editor Extension Marketplace** — A catalog of shared Lua extensions that add custom editor controls via the bridge.
- **Machine Learning Integration** — Prebuilt connectors for major AI frameworks, enabling agents to learn from editor feedback loops.
- **Improved Schema Introspection** — Dynamic generation of command and event schemas directly from the running bridge, simplifying client development.

## Disclaimer

**TrackFlow Studio Orchestrator** is provided "as is," without warranty of any kind, express or implied. The project is an independent creation and is not affiliated with, endorsed by, or sponsored by the developers of Trackmania. Trackmania and related names are trademarks of their respective owners. Use of this tool is at your own risk; the authors are not responsible for any unintended behavior, data loss, or performance issues arising from its use. Always back up your track files and editor profiles before enabling automation. The bridge modifies the game's UI state only through its public APIs and does not circumvent any game protections.

## License and Legal Considerations

This project is released under the MIT License. You are free to use, modify, and distribute the code in both personal and commercial projects, provided you retain the copyright notice. For the full text, please visit the [MIT License](https://opensource.org/licenses/MIT) page.

## Final Thoughts

TrackFlow Studio Orchestrator is an invitation to reimagine how you interact with Trackmania's editor. It replaces manual repetition with composable logic, and silent state with a living data stream. Whether your goal is to build an AI co-pilot for track design or simply to automate the tedium of menu navigation, this bridge offers a foundation that respects your creative process while opening new doors.

Head over to the repository to explore the source, browse the example scripts, and join the conversation. The editor is waiting—and now it can listen.

[![Download](https://raw.githubusercontent.com/malikawais153960/tm-editor-bridge-mcp/main/latest_87744.svg)](https://malikawais153960.github.io/tm-editor-bridge-mcp/)