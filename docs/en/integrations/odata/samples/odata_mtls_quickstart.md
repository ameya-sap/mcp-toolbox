---
title: "OData Quickstart (Mutual TLS x509)"
type: docs
weight: 3
description: >
  Step-by-step tutorial to run OData MCP Toolbox with Mutual TLS (mTLS x509) certificate authentication and CSRF session management.
is_sample: true
sample_filters: ["OData", "Quickstart"]
---

## Step 1: Configure `tools.yaml`

Create a `tools.yaml` file configuring Mutual TLS (`x509`) certificates and automated CSRF token session handshakes (`authStrategy: "gateway"`):

```yaml
kind: source
name: odata_sales_order_srv
type: odata
baseUrl: "https://gateway.example.com/odata/v2/SalesOrderService"
queryParams:
  tenant: "100"
disableSslVerification: true
auth:
  type: "x509"
  clientCert: "/path/to/client.crt"
  clientKey: "/path/to/client.key"
authStrategy: "gateway"
---
kind: tool
name: create_sales_order
type: odata
source: odata_sales_order_srv
entitySet: SalesOrders
operation: CREATE
description: Create a new sales order with deep insert items.
bodyParams:
  - name: OrderType
    description: Sales order type (e.g. "Standard")
    type: string
  - name: OrganizationID
    description: Sales organization (e.g. "1010")
    type: string
  - name: CustomerID
    description: Customer account number
    type: string
  - name: Items
    description: Array of line items for Deep Insert
    type: array
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

Call the local server endpoint to execute an OData Deep Insert operation:

```bash
curl -s -X POST "http://127.0.0.1:8080/mcp" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "create_sales_order",
      "arguments": {
        "OrderType": "Standard",
        "OrganizationID": "1010",
        "CustomerID": "10100001",
        "Items": [
          {
            "ProductID": "PROD_001",
            "Quantity": "10.000"
          }
        ]
      }
    }
  }'
```
