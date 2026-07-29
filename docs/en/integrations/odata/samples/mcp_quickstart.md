---
title: "OData Quickstart (Basic Auth)"
type: docs
weight: 1
description: >
  Step-by-step tutorial to run OData MCP Toolbox with Basic Username/Password authentication.
is_sample: true
sample_filters: ["OData", "Quickstart"]
---

## Step 1: Configure `tools.yaml`

Create a `tools.yaml` file defining your OData source with Basic Username/Password authentication:

```yaml
kind: source
name: odata_sales_order_srv
type: odata
baseUrl: "https://gateway.example.com/odata/v2/SalesOrderService"
queryParams:
  tenant: "100"
disableSslVerification: true
auth:
  type: "basic"
  username: "YOUR_USERNAME"
  password: "YOUR_PASSWORD"
---
kind: tool
name: read_sales_order
type: odata
source: odata_sales_order_srv
entitySet: SalesOrders
operation: READ
description: Retrieve sales order records from OData backend.
```

---

## Step 2: Run the Toolbox server

Run the Toolbox server, pointing to the `tools.yaml` file created earlier:

```bash
./toolbox --config "tools.yaml"
```

The server will start listening on `http://127.0.0.1:8080/mcp`.

---

## Step 3: Execute JSON-RPC Tool Request

Call the local server endpoint:

```bash
curl -s -X POST "http://127.0.0.1:8080/mcp" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "read_sales_order",
      "arguments": {
        "top": 5,
        "select": "OrderID,OrderType,CustomerID"
      }
    }
  }'
```
