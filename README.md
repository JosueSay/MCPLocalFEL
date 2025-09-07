# FEL MCP Server (Local)

Local **Model Context Protocol (MCP)** server for **Guatemalan FEL** invoices.  
Exposes tools to **validate FEL XML** and **render branded PDFs**. Designed for **WSL (Ubuntu)** or Linux and can be consumed by MCP hosts (e.g., Claude Desktop).

> This repository focuses on the **FEL MCP server**. It’s often used alongside a chatbot client; see “Related repositories”.

## 🔗 Related repositories

- [Chatbot (CLI / UI)](https://github.com/JosueSay/ChatBotMCP) — client that can drive this MCP server.
- [Reference: OpenAI Chat API Example](https://github.com/JosueSay/Selectivo_IA/blob/main/docs_assistant/README.md) — reference only (patterns for API connectivity and instruction context).

## ✨ Tools (MCP)

- `fel_validate`  
  **required**: `xml_path` (absolute WSL path)  
  Checks required fields and totals (subtotal, VAT 12%, total).

- `fel_render`  
  **required**: `xml_path`, `out_path` (absolute WSL paths)  
  Renders a branded PDF from the FEL XML.

- `fel_batch`  
  **required**: `dir_xml`, `out_dir` (absolute WSL paths)  
  Processes all `*.xml` in `dir_xml`, produces one PDF per XML in `out_dir`, and writes `manifest.json`.

> Use **absolute WSL paths** like `/mnt/d/...` in all arguments and environment variables.

## ⚙️ Requirements

- Python **3.12**
- **WSL Ubuntu 22.04** (or Linux)
- Virtual environment (recommended)

## 🔧 Installation

```bash
git clone <REPO_URL>
cd <REPO_DIR>

python3.12 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Create your environment file (use **absolute WSL paths**):

```bash
cp .env.example .env
```

Key variables used by the server (placeholders shown):

```env
# I/O
FEL_XML_PATH=/ABSOLUTE/PATH/TO/REPO/data/xml/factura.xml
FEL_OUTPUT_PDF=/ABSOLUTE/PATH/TO/REPO/data/out/factura.pdf
FEL_BATCH_OUT_DIR=/ABSOLUTE/PATH/TO/REPO/data/out
# optional logo
FEL_LOGO_PATH=/ABSOLUTE/PATH/TO/REPO/data/logos/logo.jpg

# Fonts
FEL_ACTIVE_FONT=1
FEL_FONT_DIR_MONTSERRAT=/ABSOLUTE/PATH/TO/REPO/assets/fonts/Montserrat/static
FEL_FONT_DIR_ROBOTOMONO=/ABSOLUTE/PATH/TO/REPO/assets/fonts/Roboto_Mono/static

# Theme/Layout
FEL_THEME=light
FEL_QR_SIZE=150
FEL_TOP_BAR_HEIGHT=20
```

## ▶️ Run the server (STDIO)

```bash
source venv/bin/activate
python servers/fel_mcp_server/server_stdio.py
```

## 🧪 Quick CLI tests (JSON-RPC over stdin)

**List tools:**

```bash
printf '%s\n' \
'{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}' \
'{"jsonrpc":"2.0","id":2,"method":"tools/list"}' \
| python servers/fel_mcp_server/server_stdio.py
```

**Validate one XML:**

```bash
printf '%s\n' \
'{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}' \
'{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"fel_validate","arguments":{"xml_path":"/ABSOLUTE/PATH/TO/REPO/data/xml/factura.xml"}}}' \
| python servers/fel_mcp_server/server_stdio.py
```

**Render one PDF (no logo):**

```bash
printf '%s\n' \
'{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}' \
'{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"fel_render","arguments":{"xml_path":"/ABSOLUTE/PATH/TO/REPO/data/xml/factura.xml","out_path":"/ABSOLUTE/PATH/TO/REPO/data/out/testing.pdf"}}}' \
| python servers/fel_mcp_server/server_stdio.py
```

**Batch:**

```bash
printf '%s\n' \
'{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}' \
'{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"fel_batch","arguments":{"dir_xml":"/ABSOLUTE/PATH/TO/REPO/data/xml","out_dir":"/ABSOLUTE/PATH/TO/REPO/data/out/batch"}}}' \
| python servers/fel_mcp_server/server_stdio.py
```

## 🖥️ Use with Claude Desktop (MCP)

1. Install Claude Desktop: [https://claude.ai/download](https://claude.ai/download)
2. Open **Settings -> Developer -> Edit Config**, then edit `claude_desktop_config.json`:

    ```json
    {
      "mcpServers": {
        "FEL": {
          "command": "wsl.exe",
          "args": [
            "-e",
            "/ABSOLUTE/PATH/TO/REPO/venv/bin/python",
            "/ABSOLUTE/PATH/TO/REPO/servers/fel_mcp_server/server_stdio.py"
          ]
        }
      }
    }
    ```

    - Use **absolute WSL paths** in `args`.
    - If the host logs show errors like `"'xml_path'"`, it means the call was sent **without arguments**; re-run with a prompt that includes the **exact JSON arguments** block.

3. Restart Claude Desktop (PowerShell):

```powershell
Stop-Process -Name "Claude" -Force; Start-Process "<ABSOLUTE_WINDOWS_PATH_TO_Claude.exe>"
```

## 📚 References

- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Anthropic API](https://docs.anthropic.com/en/api)
- [JSON-RPC 2.0](https://www.jsonrpc.org/)
