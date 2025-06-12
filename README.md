<p align="center">
  <img src="./icons/logo-p4mcp-reg.svg" alt="Perforce P4 MCP Server" width="480" />
</p><br>


<div align="center">

![Support](https://img.shields.io/badge/Support-Community-yellow.svg)

![GitHub release](https://img.shields.io/github/v/release/perforce/p4mcp-server?color=blue)

<h1>Perforce P4 MCP Server</h1>

<p>
  <strong>Perforce P4 MCP Server is a Model Context Protocol (MCP) server that integrates with the Perforce P4 version control system. It is built on FastMCP with direct P4 Python bindings to expose safe, structured read/write tools for changelists, files, shelves, workspaces, jobs, reviews, and server metadata. It also integrates with P4 DAM through its REST API to manage <a href="#features">resources</a>.</strong>
</p>

<nav aria-label="Quick navigation">
  <p align="center">
    <a href="#features">Features</a> ·
    <a href="#prerequisites">Prerequisites</a> ·
    <a href="#system-requirements">System Requirements</a> ·
    <a href="#local-p4-mcp-server-installation">Install</a> ·
    <a href="#deployment">Deployment</a> ·
    <a href="#mcp-client-configuration">Client Configurations</a> ·
    <a href="#p4-configuration">P4 Configurations</a> ·
    <a href="#available-tools">Tools</a> 
  </p>
   <p align="center">
    <a href="#logging-and-usage-data">Logging</a> ·
    <a href="#troubleshooting">Troubleshoot</a> ·
    <a href="#support">Support</a> ·
    <a href="#contributions">Contributions</a> ·
    <a href="#license">License</a>
  </p>
</nav>
</div>

## Features

- **Comprehensive P4 integration**: Read/write tools across files, changelists, shelves, workspaces, jobs, reviews, streams, and server information.
- **Code review workflows**: P4 Code Review support for review discovery, voting, state transitions, commenting, and participant management.
- **P4 DAM integration**: Access P4 DAM assets and metadata through MCP-compatible AI assistants. Search for assets, retrieve metadata, manage reviews, create asset bundles, update asset tags, and browse projects. To learn more, see [Configure P4 DAM access for P4 MCP Server](https://help.perforce.com/helix-core/helix-dam/current/Content/HTH-Admin/DAM/configure-p4dam-to-p4-mcp.html) in the P4 DAM documentation.
- **Safety first**: Read-only mode by default, ownership checks, interactive MCP elicitation (PROCEED/CANCEL) for destructive delete and obliterate operations.
- **Flexible toolsets**: Configure which tool categories to enable: server, files, changelists, shelves, workspaces, jobs, reviews, streams, and p4dam.
- **Robust logging**: Application and session logging to the `logs/` directory.
- **Optional telemetry**: Consent-gated usage statistics. Disabled by default.
- **Cross platform**: Supported on macOS, Linux and Windows with pre-built binaries.

## Prerequisites

- **P4 Server access**: Connection to a P4 Server with proper credentials
- **Authentication**: Valid P4 login (ticket-based or password)

## System Requirements

| Component | Supported Versions |
|-----------|-------------------|
| **Operating Systems** | Windows 10+<br>macOS 12+<br>Linux (glibc 2.34+, e.g. Ubuntu 22.04+, Rocky Linux 9+) |
| **Perforce P4 Server** | 2026.1 *(earlier versions untested)* |
| **Python** | 3.11+ *(required only for building from source)* |

## Local P4 MCP Server Installation
<details><summary><b>uvx (easiest, no install required)</b></summary>

If you have [uv](https://docs.astral.sh/uv/) installed, you can run P4 MCP Server directly without any manual installation:

```bash
# Run the server
uvx p4mcp-server

# Check version
uvx p4mcp-server --version

# Run with arguments
uvx p4mcp-server --readonly --allow-usage
```

This automatically fetches and runs the latest version from PyPI. No Python virtual environment setup or dependency management needed.

**Requirements:**
- [uv](https://docs.astral.sh/uv/) installed on your system
- Python 3.11+ (uv will handle this automatically)

</details>

<details><summary><b>Pre-built binaries (recommended for offline/air-gapped)</b></summary>

Download the appropriate binary for your operating system:
- **macOS**: [p4-mcp-server-mac.zip](https://github.com/perforce/p4mcp-server/releases/latest/download/p4-mcp-server-mac.zip)
- **Windows**: [p4-mcp-server-win.zip](https://github.com/perforce/p4mcp-server/releases/latest/download/p4-mcp-server-win.zip)
- **Linux**: [p4-mcp-server-linux.zip](https://github.com/perforce/p4mcp-server/releases/latest/download/p4-mcp-server-linux.zip)

Extract and use the executable directly. No Python installation is required.

```bash
# macOS / Linux
unzip p4-mcp-server-mac.zip   # or p4-mcp-server-linux.zip
./p4-mcp-server --help
```

```powershell
# Windows
Expand-Archive p4-mcp-server-win.zip -DestinationPath .
.\p4-mcp-server.exe --help
```

</details>

<details><summary><b>Build from source</b></summary>

Requirements:
  - Python 3.11+ (with Tkinter)

Build:
  - macOS:  <code>chmod +x build.sh &amp;&amp; ./build.sh package</code>
  - Linux:  <code>chmod +x build.sh &amp;&amp; ./build.sh package</code>
  - Windows: <code>build.bat package</code>

Output:
  - macOS & Linux: <code>p4-mcp-server-&lt;version&gt;.tgz</code>
  - Windows: <code>p4-mcp-server-&lt;version&gt;.zip</code>

</details>

## Deployment

### STDIO-based deployment

<details><summary><b>Local</b></summary>

Run the P4 MCP Server directly on your machine using the default STDIO transport.

Add the following to your `mcp.json`:

```json
{
  "mcpServers": {
    "perforce-p4-mcp": {
      "command": "/absolute/path/to/p4-mcp-server",
      "env": {
        "P4PORT": "ssl:perforce.example.com:1666",
        "P4USER": "your_username",
        "P4CLIENT": "your_workspace",
        "P4DAM_API_KEY": "your_api_key"
      },
      "args": [
        "--readonly", "--allow-usage"
      ]
    }
  }
}
```

> **Note:** This example shows explicit `env` values. If `P4CONFIG` is set, you can omit them and use the generic configuration example in the `MCP client configuration` section instead.

</details>

<details><summary><b>Docker</b></summary>

Run the P4 MCP Server from a Docker container with STDIO transport, allowing MCP clients to manage the container lifecycle.

> **Note:** Docker-based execution is currently supported on macOS and Linux only.

**Prerequisites**

- Docker installed and running
- Valid P4 credentials and access to a P4 server

**Pull the Docker image**

```bash
docker pull ghcr.io/perforce/p4mcp-server:latest
```

<details><summary><b>Build from source instead</b></summary>

```bash
cd /path/to/p4mcp-server
docker build -t ghcr.io/perforce/p4mcp-server .
```

</details>

**Configure MCP Client**

Add the following to your `mcp.json`:

```json
{
    "servers": {
        "perforce-p4mcp-docker": {
            "command": "docker",
            "args": [
                "run", "-i", "--rm",
                "--hostname", "your-hostname",
                "-e", "P4PORT=ssl:perforce.example.com:1666",
                "-e", "P4USER=your_username",
                "-e", "P4CLIENT=your_workspace",
                "-e", "P4DAM_API_KEY=your_api_key",
                "-v", "/Users/your_username/.p4tickets:/home/mcpuser/.p4tickets:ro",
                "ghcr.io/perforce/p4mcp-server:latest"
            ]
        }
    }
}
```

**Configuration Options**

| Flag | Description |
|------|-------------|
| `-i` | Interactive mode (required for STDIO) |
| `--rm` | Remove container when stopped |
| `--hostname` | Match workspace host restriction |
| `-e P4PORT` | P4 server address |
| `-e P4USER` | P4 username |
| `-e P4CLIENT` | Workspace name |
| `-v` | Mount P4 tickets file |

**Authentication**

Using P4 tickets:
```bash
# macOS/Linux
-v /Users/your_username/.p4tickets:/home/mcpuser/.p4tickets:ro
```

> **Note:** Use the full path to your tickets file (not `~`). After running `p4 login`, restart the MCP server to pick up the new ticket.

> **Note:** By default the container starts as root, copies the bind-mounted tickets file to a location readable by the non-root `mcpuser`, then drops privileges (via `gosu`) so the server runs as `mcpuser`. This auto-copy lets an owner-only (`0600`) tickets file owned by a foreign uid work on runtimes that preserve host ownership (native Linux Docker/podman). A `:ro` bind mount of the ticket is supported — the file is copied, not modified in place (this is distinct from a read-only container root filesystem; see below). Note the auto-copy runs **only when the container starts as root** (the default); forcing a non-root uid or a read-only root filesystem changes this behavior, as described below.

> **Kubernetes / restricted (non-root-enforced) profiles:** the image intentionally starts as root and drops to `mcpuser` (uid 1000) via `gosu`, so the server always runs non-root — both paths are supported. For `runAsNonRoot` / restricted PodSecurity / OpenShift restricted SCC, set `securityContext.runAsUser: 1000` (and `fsGroup: 1000` when mounting the ticket). In that mode the entrypoint's root-only ticket copy is skipped, so the mounted ticket must already be readable by uid 1000 (owner-matched mount, `-e P4PASSWD="your_ticket_content"`, or `podman -v <host>:/home/mcpuser/.p4tickets:U`).

> **Read-only root filesystem (`--read-only` / `readOnlyRootFilesystem`):** provide a writable/tmpfs `/home/mcpuser` (e.g. `--tmpfs /home/mcpuser`) so the ticket copy can be written; otherwise the ticket copy fails and P4 will report a normal auth error.

Using a password:
```bash
-e P4PASSWD="your_password"
```

**Workspace Host Restrictions**

> ⚠️ **Important:** Docker containers have their own hostname, which differs from your local machine. If your P4 workspace is restricted to a specific host, operations like `sync` will fail.

To resolve this, set the container hostname to match your workspace's host restriction:

```bash
--hostname your-hostname
```

To find your workspace host name:
```bash
# macOS/Linux
p4 client -o your_workspace | grep "^Host:"
```

**Mounting Client Root for Write Operations**

> ⚠️ **Important:** By default, the Docker container cannot access your local workspace files. For write operations like `sync`, `submit`, or `reconcile`, you must mount your client root directory into the container at the **same path**.

Add a volume mount for your client root:
```bash
-v /path/to/your/client/root:/path/to/your/client/root
```

Example configuration with client root mounted:
```json
{
    "servers": {
        "perforce-p4mcp-docker": {
            "command": "docker",
            "args": [
                "run", "-i", "--rm",
                "--hostname", "your-hostname",
                "-e", "P4PORT=ssl:perforce.example.com:1666",
                "-e", "P4USER=your_username",
                "-e", "P4CLIENT=your_workspace",
                "-e", "P4DAM_API_KEY=your_api_key",
                "-v", "/Users/your_username/.p4tickets:/home/mcpuser/.p4tickets",
                "-v", "/path/to/client/root:/path/to/client/root",
                "ghcr.io/perforce/p4mcp-server:latest"
            ]
        }
    }
}
```

To find your client root:
```bash
p4 client -o your_workspace | grep "^Root:"
```

> **Note:** The mount path inside the container must match the client root path exactly, as P4 tracks files by their absolute paths.

</details>

### HTTP-based deployment

<details><summary><b>VM</b></summary>

Run the MCP server on a VM using the HTTP transport, allowing clients to connect over the network.

**Start the server on the VM:**

```bash
P4PORT=ssl:perforce.example.com:1666 P4USER=your_username P4PASSWD=YOUR_TICKET ./p4-mcp-server --readonly --transport http --port 8000
```

**Configure the MCP client:**

Add the following to your `mcp.json`:

```json
{
    "servers": {
        "perforce-p4-mcp": {
            "type": "http",
            "url": "http://<ip-or-hostname>:8000/mcp"
        }
    }
}
```

> **Note:** Ensure the VM's firewall allows inbound connections on the chosen port. For production use, consider placing the server behind a reverse proxy with TLS.

</details>

<details><summary><b>Docker</b></summary>

Run the MCP server in a Docker container using HTTP transport and expose the MCP endpoint over a host port.

**Start the container:**

```bash
docker run --rm -p 8000:8000 \
  -e P4PORT=ssl:perforce.example.com:1666 \
  -e P4USER=your_username \
  -e P4PASSWD=YOUR_TICKET \
  -e P4DAM_API_KEY=your_api_key \
  ghcr.io/perforce/p4mcp-server:latest \
  python3 -m p4mcp.main --readonly --transport http --port 8000
```

**Configure the MCP client:**

Add the following to your `mcp.json`:

```json
{
  "servers": {
    "perforce-p4-mcp": {
      "type": "http",
      "url": "http://<ip-or-hostname>:8000/mcp"
    }
  }
}
```

> **Note:** Docker supports HTTP-based deployment as well. The container image defaults to STDIO transport, so the HTTP startup command must explicitly override the default command. If you need write operations, also mount the client root and ticket file paths into the container.

</details>

## MCP client configuration

> **Note:** In all configuration examples below, if `P4CONFIG` is set, you do not need to set any environment variables in the `env` block. The server will use the configuration from the specified P4CONFIG file instead.

> **Tip:** If you have [uv](https://docs.astral.sh/uv/) installed, you can use `uvx p4mcp-server` instead of `/absolute/path/to/p4-mcp-server` in the `command` field. This eliminates the need to download or build binaries manually.
> <details> <summary><strong>Server configuration example using uvx</strong></summary>
>
>  ```json
>{
>   "mcpServers": {
>      "perforce-p4-mcp": {
>         "command": "uvx",
>         "args": [
>            "p4mcp-server",
>            "--readonly", "--allow-usage"
>         ],
>         "env": {
>            "P4PORT": "ssl:perforce.example.com:1666",
>            "P4USER": "your_username",
>            "P4CLIENT": "your_workspace",
>            "P4DAM_API_KEY": "your_api_key"
>         }
>      }
>    }
>}
>```
></details>
>
> <details> <summary><strong>Server configuration example using binary path</strong></summary>
>
>  ```json
>{
>   "mcpServers": {
>      "perforce-p4-mcp": {
>         "command": "/absolute/path/to/p4-mcp-server",
>         "env": {
>         },
>         "args": [
>            "--readonly", "--allow-usage"
>         ]
>      }
>    }
>}
>```
</details>

<br>

<details>
  <summary><strong>JetBrains IDEs (IntelliJ IDEA, Rider, PyCharm, etc.)</strong></summary>
 
See the [JetBrains AI Assistant VCS Integration documentation](https://www.jetbrains.com/help/ai-assistant/ai-in-vcs-integration.html#configure-a-perforce-mcp-server) for detailed configuration steps.

</details>

<details>
  <summary><strong>Claude Code</strong></summary>

See the [Claude Code MCP docs](https://docs.anthropic.com/en/docs/claude-code/mcp) for more information.

**Using uvx (no installation required):**
```json
{
  "mcpServers": {
    "perforce-p4-mcp": {
      "command": "uvx",
      "args": [
        "p4mcp-server",
        "--readonly", "--allow-usage"
      ],
      "env": {
        "P4PORT": "ssl:perforce.example.com:1666",
        "P4USER": "your_username",
        "P4CLIENT": "your_workspace",
        "P4DAM_API_KEY": "your_api_key"
      }
    }
  }
}
```

**Using pre-built binary:**
```json
{
  "mcpServers": {
    "perforce-p4-mcp": {
      "command": "/absolute/path/to/p4-mcp-server",
      "env": {
        "P4PORT": "ssl:perforce.example.com:1666",
        "P4USER": "your_username",
        "P4CLIENT": "your_workspace",
        "P4DAM_API_KEY": "your_api_key"
      },
      "args": [
        "--readonly", "--allow-usage"
      ]
    }
  }
}
```
</details>

<details>
  <summary><strong>Cursor</strong></summary>

See the [Cursor MCP documentation](https://docs.cursor.com/en/context/mcp) for more information.

```json
{
  "mcpServers": {
    "perforce-p4-mcp": {
      "command": "/absolute/path/to/p4-mcp-server",
      "env": {
        "P4PORT": "ssl:perforce.example.com:1666",
        "P4USER": "your_username",
        "P4CLIENT": "your_workspace",
        "P4DAM_API_KEY": "your_api_key"
      },
      "args": [
        "--readonly", "--allow-usage"
      ]
    }
  }
}
```
</details>

<details>
  <summary><strong>Eclipse</strong></summary>

See the [Eclipse MCP documentation](https://eclipse.dev/lmos/docs/arc/mcp) for more information.

```json
{
  "servers": {
    "perforce-p4-mcp": {
      "command": "/absolute/path/to/p4-mcp-server",
      "env": {
        "P4PORT": "ssl:perforce.example.com:1666",
        "P4USER": "your_username",
        "P4CLIENT": "your_workspace",
        "P4DAM_API_KEY": "your_api_key"
      },
      "args": [
        "--readonly", "--allow-usage"
      ]
    }
  }
}
```
</details>

<details><summary><strong>Kiro</strong></summary>

See the [Kiro MCP documentation](https://kiro.dev/docs/mcp/configuration/) for more information.

```json
{
  "mcpServers": {
    "perforce-p4-mcp": {
      "command": "/absolute/path/to/p4-mcp-server",
      "env": {
        "P4PORT": "ssl:perforce.example.com:1666",
        "P4USER": "your_username",
        "P4CLIENT": "your_workspace",
        "P4DAM_API_KEY": "your_api_key"
      },
      "args": [
        "--readonly", "--allow-usage"
      ]
    }
  }
}
```

</details>

<details>
  <summary><strong>VS Code</strong></summary>

See the [VS Code documentation](https://code.visualstudio.com/docs/copilot/customization/mcp-servers) for more information.

```json
{
  "servers": {
    "perforce-p4-mcp": {
      "command": "/absolute/path/to/p4-mcp-server",
      "env": {
        "P4PORT": "ssl:perforce.example.com:1666",
        "P4USER": "your_username",
        "P4CLIENT": "your_workspace",
        "P4DAM_API_KEY": "your_api_key"
      },
      "args": [
        "--readonly", "--allow-usage"
      ]
    }
  }
}
```
</details>

<details>
  <summary><strong>Windsurf</strong></summary>

See the [Windsurf MCP documentation](https://docs.windsurf.com/windsurf/cascade/mcp) for more information.

```json
{
  "mcpServers": {
    "perforce-p4-mcp": {
      "command": "/absolute/path/to/p4-mcp-server",
      "env": {
        "P4PORT": "ssl:perforce.example.com:1666",
        "P4USER": "your_username",
        "P4CLIENT": "your_workspace",
        "P4DAM_API_KEY": "your_api_key"
      },
      "args": [
        "--readonly", "--allow-usage"
      ]
    }
  }
}
```
</details>

### P4 Environment Variables
- `P4PORT` - P4 Server address. Examples: `ssl:perforce.example.com:1666`, `localhost:1666`
- `P4USER` - Your P4 username
- `P4CLIENT` - Your current P4 workspace. Optional, but recommended

### Result limit environment variables
- `P4MCP_MAX_RESULTS` - Cap on the number of rows the P4 server returns per command (`p4.maxresults`). Default: `10000`. Set to `0` to disable the limit (server default in effect). When a command would exceed this limit the server aborts it with an error rather than truncating results, so keep the value generous. Can be overridden by the `--max-results` CLI argument. Must be a non-negative integer; an invalid value fails fast at startup before any P4 connection is attempted.
- `P4MCP_MAX_SCAN_ROWS` - Cap on the number of rows the P4 server scans per command (`p4.maxscanrows`). Unset by default, so admin/group policy governs scan limits. Can be overridden by the `--max-scan-rows` CLI argument. Must be a non-negative integer when supplied.

### Logging environment variables
- `P4MCP_LOG_DIR` - Directory for log files. Default: `logs/` in the server executable's directory. Can be overridden by the `--log-dir` CLI argument.

### SSL/TLS environment variables
- `P4MCP_TLS_CA_MODE` - TLS certificate source mode.
  - `system` (default): use OS trust store via `truststore`. **Note:** In this mode, `truststore` overrides the `verify=` parameter — custom CA bundles set via `P4MCP_CA_BUNDLE` or `--ca-bundle` are ignored. To use a custom CA bundle, set `P4MCP_TLS_CA_MODE=certifi`.
  - `certifi`: disable `truststo