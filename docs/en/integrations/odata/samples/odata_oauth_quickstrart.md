---
title: "OData Quickstart (Dynamic User OAuth)"
type: docs
weight: 2
description: >
  Step-by-step tutorial to run OData MCP Toolbox with Dynamic User OAuth (Principal Propagation).
is_sample: true
sample_filters: ["OData", "OAuth", "Quickstart"]
---

## Step 1: Configure `tools.yaml`

Create a `tools.yaml` file enabling `useClientOauth`. Toolbox will forward incoming OAuth access tokens received from client applications directly to the OData backend service:

```yaml
kind: source
name: odata_sales_order_srv
type: odata
baseUrl: "https://gateway.example.com/odata/v2/SalesOrderService"
queryParams:
  tenant: "100"
disableSslVerification: true
useClientOauth: "true" # Dynamically extracts token from Authorization header
---
kind: tool
name: read_sales_order
type: odata
source: odata_sales_order_srv
entitySet: SalesOrders
operation: READ
description: Retrieve sales order records using principal propagation.
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

When making calls with Dynamic User OAuth, pass the user's OAuth access token in the `Authorization: Bearer <token>` header of the request to Toolbox:

```bash
curl -s -X POST "http://127.0.0.1:8080/mcp" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_USER_OAUTH_TOKEN" \
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
