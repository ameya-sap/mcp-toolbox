---
title: "odata"
type: docs
weight: 1
description: >
  An "odata" tool executes CRUD operations (READ, CREATE, UPDATE, DELETE) and function imports against OData v2 and v4 services, supporting deep inserts, automated pagination, and metadata schema inspection.
---

## About

An `odata` tool maps OData entity sets and operations to MCP tool definitions. It allows AI applications and LLM agents to seamlessly interact with backend OData services (including enterprise backend systems, gateways, and custom OData APIs).

### Supported Operations

| Operation | HTTP Method | Description |
| :--- | :---: | :--- |
| `READ` | `GET` | Queries entity sets. Automatically adds `$filter`, `$select`, `$top`, `$skip`, and `$skiptoken` parameters. |
| `CREATE` | `POST` | Creates a new entity. Supports **OData Deep Inserts** for creating nested entity trees. |
| `UPDATE` | `PUT` | Updates an existing entity. |
| `DELETE` | `DELETE` | Removes an entity. |
| `FUNCTION_IMPORT` | `POST` | Invokes custom OData function import endpoints. |

### Features & Capabilities

#### 1. Automated `READ` Query Parameters & Metadata Introspection
When `operation` is set to `READ`, Toolbox automatically exposes the standard OData query parameters to the LLM:

* **`filter`**: OData `$filter` expression (e.g. `OrderType eq 'Standard' and CustomerID eq '10100001'`).
* **`select`**: OData `$select` comma-separated field list (e.g. `OrderID,CustomerID,NetAmount`).
* **`top`**: OData `$top` integer limit.
* **`skip`**: OData `$skip` integer offset.
* **`skiptoken`**: OData `$skiptoken` string for server-side pagination.

If `$metadata` XML was successfully parsed by the source, parameter descriptions are dynamically enriched with available entity property names and types, guiding LLMs to generate syntactically correct `$filter` and `$select` strings.

#### 2. Server-Side Pagination & Truncation Notifications
When an OData service returns a next-page link (`__next` in OData v2, `@odata.nextLink` in OData v4), Toolbox automatically injects a helpful `_NOTICE` field in the response JSON:

```json
{
  "_NOTICE": "Results truncated by OData Server Pagination. To get the next set, call again with $skiptoken extracted from this URL: ...&$skiptoken=100",
  "d": {
    "results": [...]
  }
}
```

#### 3. OData Deep Inserts (`CREATE`)
OData `CREATE` operations support **Deep Inserts**, allowing an AI agent to create a parent entity alongside multiple child entities in a single atomic request (e.g., Sales Order Header with Line Items).

Define array or object types in `bodyParams`:

```yaml
bodyParams:
  - name: OrderType
    type: string
  - name: CustomerID
    type: string
  - name: Items
    type: array
    description: Nested line items for Deep Insert
```

Toolbox serializes nested arrays and objects directly into the JSON body sent to the backend OData service.

---

## Compatible Sources

{{< compatible-sources >}}

---

## Example

### Example 1: `READ` Query Tool

```yaml
kind: tool
name: read_sales_order
type: odata
source: odata_sales_order_srv
entitySet: SalesOrders
operation: READ
description: Retrieve sales orders with optional filtering, selection, and pagination.
```

### Example 2: `CREATE` Tool with Deep Insert (Sales Order)

```yaml
kind: tool
name: create_sales_order
type: odata
source: odata_sales_order_srv
entitySet: SalesOrders
operation: CREATE
description: Create a new sales order with deep insert items.
bodyParams:
  - name: OrderType
    description: Type of sales order (e.g. "Standard")
    type: string
  - name: OrganizationID
    description: Organization code (e.g. "1010")
    type: string
  - name: CustomerID
    description: Customer account number
    type: string
  - name: CustomerReference
    description: Customer purchase reference string
    type: string
  - name: Items
    description: Array of line items (ProductID, Quantity, Price)
    type: array
```

### Example 3: `UPDATE` Tool

```yaml
kind: tool
name: update_purchase_order
type: odata
source: odata_purchase_order_srv
entitySet: PurchaseOrders
operation: UPDATE
description: Update an existing purchase order header.
queryParams:
  - name: PurchaseOrderID
    description: Key Purchase Order ID
    type: string
bodyParams:
  - name: SupplierID
    description: Updated supplier ID
    type: string
  - name: PurchasingGroup
    description: Updated purchasing group
    type: string
```

### Example 4: `FUNCTION_IMPORT` Tool

```yaml
kind: tool
name: check_flight_availability
type: odata
source: odata_flight_srv
entitySet: CheckFlightAvailability
operation: FUNCTION_IMPORT
description: Invoke flight availability function import.
queryParams:
  - name: AirlineCode
    description: Two-character airline code (e.g. "LH")
    type: string
  - name: FlightDate
    description: Flight date (YYYY-MM-DD)
    type: string
```

---

## Reference

| Field | Type | Required | Default | Description |
| :--- | :---: | :---: | :---: | :--- |
| `name` | string | **true** | - | Unique name of the tool exposed to LLMs. |
| `type` | string | **true** | - | Must be `"odata"`. |
| `source` | string | **true** | - | Name of the configured OData source. |
| `entitySet` | string | **true** | - | Name of the target OData EntitySet or FunctionImport. |
| `operation` | string | **true** | - | Operation type: `"READ"`, `"CREATE"`, `"UPDATE"`, `"DELETE"`, or `"FUNCTION_IMPORT"`. |
| `description` | string | false | - | Human-readable explanation of what the tool does. |
| `contentType` | string | false | `"application/json"` | Content-Type header override. |
| `queryParams` | array | false | - | List of explicit query parameters or URL key parameters. |
| `bodyParams` | array | false | - | List of body parameters serialized into the JSON payload for `CREATE`/`UPDATE`. |
| `annotations` | object | false | `ReadOnly` (`READ`) / `Destructive` (`CREATE`/`UPDATE`/`DELETE`) | Tool annotations (see [Tool Annotations](../../../documentation/configuration/tools/_index.md#specifying-annotations)). |
