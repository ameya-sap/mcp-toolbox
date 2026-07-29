# MCP Toolbox for OData

MCP Toolbox for OData is an open-source [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server that connects AI applications and LLM agents directly to **OData v2 and v4 services** (including enterprise backend systems, gateways, and custom OData APIs).

---

## 🚀 Key Features

* **Complete OData Operations:** Full support for `READ`, `CREATE`, `UPDATE`, `DELETE`, and `FUNCTION_IMPORT`.
* **OData Deep Inserts:** Create complex nested entity structures (e.g. Sales Order Header with multiple Line Items) in a single atomic payload.
* **CSRF Token Gateway Strategy:** Automated pre-flight `HEAD` handshakes, `X-CSRF-Token` fetching, cookie-jar session persistence per user, and 403 eviction/token renewal.
* **Flexible Enterprise Authentication:**
  * **Basic Authentication** (Username/Password)
  * **Static Bearer Token** (OAuth 2.0 / JWT)
  * **Mutual TLS (x509 mTLS):** Password-encrypted or unencrypted client certificates (`clientCert`, `clientKey`, `caCert`).
  * **Dynamic User OAuth (Principal Propagation):** Dynamically forwards the user's OAuth access token from client applications (`Authorization` or custom headers like `X-OData-Token`, `X-Auth-Token`).
* **Automated Metadata Discovery:** Parses `$metadata` XML to dynamically enrich parameter descriptions (`$filter`, `$select`) with property names and data types.
* **Server-Side Pagination:** Automatically handles `$skiptoken`, `$top`, and `$skip` parameters with built-in truncation notifications (`_NOTICE`).

---

## 📥 Quick Start

### 1. Download or Build the Binary

Build locally using Go:

```bash
go build -o toolbox main.go
```

### 2. Create Configuration File (`tools.yaml`)

Define your OData source and MCP tools in a YAML configuration file:

```yaml
kind: source
name: odata_sales_order_srv
type: odata
baseUrl: "https://gateway.example.com/odata/v2/SalesOrderService"
queryParams:
  tenant: "100"
disableSslVerification: false
auth:
  type: x509
  clientCert: "/path/to/client.crt"
  clientKey: "/path/to/client.key"
authStrategy: "gateway"
---
kind: tool
name: read_sales_order
type: odata
source: odata_sales_order_srv
entitySet: SalesOrders
operation: READ
description: Retrieve sales orders from backend.
---
kind: tool
name: create_sales_order
type: odata
source: odata_sales_order_srv
entitySet: SalesOrders
operation: CREATE
description: Create a sales order with deep insert line items.
bodyParams:
  - name: OrderType
    description: Sales order type (e.g. "Standard")
    type: string
  - name: OrganizationID
    description: Organization ID (e.g. "1010")
    type: string
  - name: CustomerID
    description: Customer account number
    type: string
  - name: Items
    description: Array of line items for Deep Insert
    type: array
```

### 3. Run the Toolbox server

Run the Toolbox server, pointing to the `tools.yaml` file created earlier:

```bash
./toolbox --config "tools.yaml"
```

The server will start listening on `http://127.0.0.1:8080/mcp`.

---

## 🔐 Authentication Strategies

| Auth Type | Field | Description |
| :--- | :--- | :--- |
| **Basic** | `auth.type: "basic"` | Standard username/password authentication (`auth.username`, `auth.password`). |
| **Bearer** | `auth.type: "bearer"` | Static OAuth 2.0 Bearer token (`auth.token`). |
| **x509 (mTLS)** | `auth.type: "x509"` | Mutual TLS certificate authentication (`auth.clientCert`, `auth.clientKey`, optional `auth.caCert`). |
| **Dynamic User OAuth** | `useClientOauth: "true"` | Pass-through user OAuth token forwarded from client request headers. |
| **CSRF Gateway** | `authStrategy: "gateway"` | Automated pre-flight `X-CSRF-Token: Fetch` handshake and cookie session cache. |

---

## 📖 Documentation

* [OData Source Configuration Guide](en/integrations/odata/source.md)
* [OData Tool Definition Reference](en/integrations/odata/tools/odata.md)
* [Pre-built OData Configurations](en/integrations/odata/prebuilt-configs/odata.md)
* [Connecting IDEs to OData MCP Server](en/documentation/connect-to/ides/odata_mcp.md)

---

## 📄 License

Apache License 2.0 - See [LICENSE](../LICENSE) for details.
