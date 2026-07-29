---
title: "OData using MCP"
type: docs
weight: 1
description: >
  Connect AI applications and LLM agents to OData services using MCP Toolbox.
---

## Overview

This pre-built configuration connects MCP Toolbox to live OData Gateway services using **Mutual TLS (x509 mTLS)** and automated **CSRF Token Gateway handshakes**.

---

## Complete Manifest (`tools.yaml`)

```yaml
kind: source
name: odata_sales_order_srv
type: odata
baseUrl: "https://gateway.example.com/odata/v2/SalesOrderService"
queryParams:
  tenant: "100"
disableSslVerification: true
auth:
  type: x509
  clientCert: "/path/to/client.crt"
  clientKey: "/path/to/client.key"
authStrategy: "gateway"
---
kind: source
name: odata_purchase_order_srv
type: odata
baseUrl: "https://gateway.example.com/odata/v2/PurchaseOrderService"
queryParams:
  tenant: "100"
disableSslVerification: true
auth:
  type: x509
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
    description: Value for OrderType (e.g. "Standard")
    type: string
  - name: OrganizationID
    description: Value for OrganizationID (e.g. "1010")
    type: string
  - name: ChannelID
    description: Value for ChannelID (e.g. "10")
    type: string
  - name: DivisionID
    description: Value for DivisionID (e.g. "00")
    type: string
  - name: CustomerID
    description: Customer account number (e.g. "10100001")
    type: string
  - name: CustomerReference
    description: Customer purchase order reference string
    type: string
  - name: Items
    description: Array of line items for Deep Insert
    type: array
---
kind: tool
name: read_sales_order
type: odata
source: odata_sales_order_srv
entitySet: SalesOrders
operation: READ
description: Retrieve sales order records.
---
kind: tool
name: post_purchase_order
type: odata
source: odata_purchase_order_srv
entitySet: PurchaseOrders
operation: CREATE
description: Create a Purchase Order with nested items (OData Deep Insert).
bodyParams:
  - name: PurchaseOrderType
    description: Purchase Order Type (e.g. "Standard")
    type: string
  - name: CompanyID
    description: Company Code (e.g. "1010")
    type: string
  - name: PurchasingOrg
    description: Purchasing Organization (e.g. "1010")
    type: string
  - name: PurchasingGroup
    description: Purchasing Group (e.g. "001")
    type: string
  - name: SupplierID
    description: Supplier ID (e.g. "17300001")
    type: string
  - name: DocumentCurrency
    description: Document Currency (e.g. "EUR", "USD")
    type: string
```
