---
title: "OData Source"
type: docs
linkTitle: "Source"
weight: 1
description: >
  An "odata" source connects to OData v2 or v4 services (such as enterprise backend systems, gateways, and custom OData APIs), supporting CSRF token handshakes, mTLS authentication, and dynamic user OAuth principal propagation.
no_list: true
---

## About

An `odata` source establishes a connection to an OData service endpoint. It natively supports both OData v2 and OData v4 protocol specifications, including:

* **Automatic Schema Discovery:** Automatically fetches and parses `$metadata` XML to inspect EntitySets, EntityTypes, FunctionImports, property data types, nullability, and labels.
* **CSRF Token Gateway Strategy:** Automated pre-flight `HEAD` requests to fetch `X-CSRF-Token` headers and set up `http.CookieJar` session tracking.
* **Dynamic Principal Propagation:** Dynamically passes user OAuth access tokens received from client applications or AI agents to the OData backend.
* **Mutual TLS (x509 mTLS):** Connects to secure enterprise endpoints using client certificates and private keys.
* **URL Quoting Compatibility:** Supports OData v2 string URL escaping (`urlQuoting`) and key property auto-uppercasing.

---

## Available Tools

{{< list-tools >}}

---

## Requirements

Toolbox supports multiple authentication and session management methods for OData sources:

### 1. Basic Authentication
Standard username and password authentication.

```yaml
kind: source
name: odata_sales_order_srv
type: odata
baseUrl: "https://gateway.example.com/odata/v2/SalesOrderService"
auth:
  type: "basic"
  username: "YOUR_USERNAME"
  password: "YOUR_PASSWORD"
```

### 2. Bearer Token Authentication
Static OAuth 2.0 or session Bearer token.

```yaml
kind: source
name: odata_sales_order_srv
type: odata
baseUrl: "https://gateway.example.com/odata/v2/SalesOrderService"
auth:
  type: "bearer"
  token: "YOUR_BEARER_TOKEN"
```

### 3. Mutual TLS Authentication (x509 mTLS)
Connects using client x509 certificates and private keys. Requires `clientCert` and `clientKey` (with optional `clientKeyPassword` and `caCert`).

```yaml
kind: source
name: odata_sales_order_srv
type: odata
baseUrl: "https://gateway.example.com/odata/v2/SalesOrderService"
auth:
  type: "x509"
  clientCert: "/path/to/client.crt"
  clientKey: "/path/to/client.key"
  # clientKeyPassword: "optional_passphrase_if_encrypted"
  # caCert: "/path/to/optional_ca_bundle.crt"
```

### 4. Dynamic User OAuth (Principal Propagation)
Forwards the incoming user's OAuth access token dynamically from client request headers to the OData backend. You can specify a custom header name or default to `Authorization`.

```yaml
kind: source
name: odata_sales_order_srv
type: odata
baseUrl: "https://gateway.example.com/odata/v2/SalesOrderService"
useClientOauth: "true" # Reads from Authorization header
# OR specify custom header name:
useClientOauth: "X-Auth-Token"
```

### 5. CSRF Token & Gateway Session Management
When `authStrategy` is set to `"gateway"` or `"csrf-token"`:

1. **Pre-flight Handshake:** Before executing any modifying HTTP operation (`POST`, `PUT`, `PATCH`, `DELETE`), Toolbox issues an automated `HEAD` request to `baseUrl` with header `X-CSRF-Token: Fetch`.
2. **Session Persistence:** Toolbox extracts the returned `X-CSRF-Token` header and session cookies, caching them in an internal **LRU Session Cache** (1000 items capacity, 30-minute TTL) indexed by a secure hash of user credentials.
3. **403 Eviction & Auto-Renewal:** If an OData request returns `HTTP 403 Forbidden` due to CSRF token expiration or invalidation, Toolbox automatically evicts the cached session, triggering a fresh CSRF pre-flight handshake on the next call.

```yaml
kind: source
name: odata_sales_order_srv
type: odata
baseUrl: "https://gateway.example.com/odata/v2/SalesOrderService"
authStrategy: "gateway" # Enables pre-flight CSRF token fetching and CookieJar session caching
```

### 6. Compatibility Options

```yaml
kind: source
name: odata_sales_order_srv
type: odata
baseUrl: "https://gateway.example.com/odata/v2/SalesOrderService"
compatibility:
  urlQuoting: true # Enables OData v2 single-quote escaping ('val') and key property auto-uppercasing
```

* **`urlQuoting`**: Required for OData v2 implementations where string parameters in URLs must be wrapped in single quotes (e.g. `'STRING_VALUE'`), with internal quotes escaped as `''`. Also auto-uppercases standard key parameters (`ID`, `CODE`, `KEY`, etc.).

---

## Example

Initialize an OData source with Mutual TLS and CSRF Gateway strategy:

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
```

---

## Reference

| Field | Type | Required | Default | Description |
| :--- | :---: | :---: | :---: | :--- |
| `name` | string | **true** | - | Unique identifier for the source. |
| `type` | string | **true** | - | Must be `"odata"`. |
| `baseUrl` | string | **true** | - | Base URL of the OData service endpoint. |
| `timeout` | string | false | `"30s"` | Request timeout duration (e.g. `"10s"`, `"1m"`). |
| `headers` | map[string]string | false | - | Custom HTTP headers included in all requests. |
| `queryParams` | map[string]string | false | - | Custom URL query parameters included in all requests. |
| `disableSslVerification` | boolean | false | `false` | Skips TLS certificate verification (equivalent to `curl -k`). |
| `auth.type` | string | false | - | Authentication method: `"basic"`, `"bearer"`, or `"x509"`. |
| `auth.username` | string | false | - | Username for Basic authentication. |
| `auth.password` | string | false | - | Password for Basic authentication. |
| `auth.token` | string | false | - | Bearer token string. |
| `auth.clientCert` | string | **true** (for x509) | - | Path to x509 client certificate file. |
| `auth.clientKey` | string | **true** (for x509) | - | Path to x509 client private key file. |
| `auth.clientKeyPassword` | string | false | - | Optional passphrase for encrypted private keys. |
| `auth.caCert` | string | false | - | Optional path to custom CA root certificate bundle file. |
| `useClientOauth` | string | false | - | Enables dynamic pass-through of user OAuth tokens. Set to `"true"` or custom header name. |
| `authStrategy` | string | false | - | Set to `"gateway"` or `"csrf-token"` for CSRF pre-flight handshakes and cookie caching. |
| `compatibility.urlQuoting` | boolean | false | `false` | Enables OData v2 single-quote URL parameter escaping and key uppercasing. |
