# Self Hosted Version of Twenty CRM

- **Web App**: https://twenty.pegasus.togosolutions.co.za/
- **MCP Endpoint**: `https://twenty.pegasus.togosolutions.co.za/mcp`
- **Server SSH**: `root@157.180.20.132`
- **Server Directory**: `/opt/twenty-crm`

---

## MCP Server Configuration

The self-hosted Twenty MCP server has been configured in:
- **Global Antigravity Configuration**: `~/.gemini/config/mcp_config.json`
- **Workspace Antigravity Configuration**: `.agents/mcp_config.json`
- **Cursor Configuration**: `.cursor/mcp.json`

### Endpoint & Protocol Details
- **URL**: `https://twenty.pegasus.togosolutions.co.za/mcp`
- **Transport**: Streamable HTTP / Server-Sent Events (SSE) via `mcp-remote` bridge
- **Protocol Version**: `2025-06-18` (Twenty MCP Server v0.1.0)
- **Available CRM Objects (29)**: `companies`, `people`, `opportunities`, `tasks`, `notes`, `dashboards`, `messages`, `calendar_events`, `workspace_members`, etc.
- **Available Meta-Tools**: `execute_tool`, `learn_tools`, `load_skills`, `list_object_metadata_names`, `list_skills`, `get_tool_catalog`, `search_help_center`
- **Available Skills (16)**: `crm-hygiene`, `deal-review`, `meeting-prep`, `enrich`, `metadata-building`, `workflow-building`, `dashboard-building`, `view-building`, `roles`, `workspace-demo-seeding`, `data-manipulation`, `code-interpreter`, `docx`, `xlsx`, `pptx`, `pdf`

### Activating in Antigravity IDE
To reload the active tool palette and connect directly to the newly configured server, restart the IDE window or open a new conversation.