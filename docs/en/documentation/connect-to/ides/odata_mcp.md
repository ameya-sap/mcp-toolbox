---
title: "OData Service using MCP"
type: docs
weight: 10
description: >
  Connect your IDE or AI application to an OData service using MCP Toolbox.
---

## Prerequisites

1. Download or build the `toolbox` binary.
2. Prepare your OData configuration file (e.g. `tools.yaml`).

---

## 1. Run the Toolbox server

Start the server, pointing to your configuration file:

```bash
./toolbox --config "tools.yaml"
```

---

## 2. Configure Your IDE or AI Environment

Add the server configuration to your IDE or MCP client:

```json
{
  "mcpServers": {
    "odata-toolbox": {
      "command": "/path/to/toolbox",
      "args": ["--config", "/path/to/tools.yaml"]
    }
  }
}
```

---

## 3. Test Interaction

Once connected, ask your AI assistant:

* *"Retrieve the top 5 Sales Orders where SalesOrganization equals '1010'"*
* *"Create a new Sales Order for customer 10100001 with order type OR"*
