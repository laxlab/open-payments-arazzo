# open-payments-arazzo

Arazzo Specification 1.0.1 workflow files documenting the [Open Payments](https://openpayments.dev) API flows.

Submitted as part of the [Interledger Foundation SDK Grant Program](https://interledger.org/grant/open-payments-sdk) — Arazzo documentation track.

---

## What is Arazzo?

[Arazzo](https://www.openapis.org/arazzo-specification) is an OpenAPI Initiative standard for describing sequences of API calls as machine-readable workflows. Where OpenAPI describes what a single endpoint does, Arazzo describes how multiple endpoints work together to achieve a real outcome.

This repo takes the three Open Payments OpenAPI specifications and documents the complete multi-step payment flows in Arazzo format — something that does not yet exist anywhere in the Interledger ecosystem.

---

## Source specifications

The `specs/` directory contains the three official Open Payments OpenAPI specifications:

| File | Server | Purpose |
|------|--------|---------|
| `wallet-address-server.yaml` | Wallet Address Server | Public account info, JWKS |
| `auth-server.yaml` | Authorization Server | GNAP grant requests and continuations |
| `resource-server.yaml` | Resource Server | Incoming payments, quotes, outgoing payments |

---

## Workflows

| File | Flow | Steps |
|------|------|-------|
| `01-create-incoming-payment.arazzo.yaml` | Create incoming payment | Get wallet address to request grant to create resource |
| `02-create-quote.arazzo.yaml` | Create quote | Request grant to create quote to read quote |
| `03-end-to-end-payment.arazzo.yaml` | Full payment | 8-step flow across both wallets with interactive grant |
| `04-list-and-read-resources.arazzo.yaml` | List and read | List incoming payments, outgoing payments, get single resources |
| `05-complete-incoming-payment.arazzo.yaml` | Complete incoming payment | Explicit manual completion for fixed-send flows |

---

## Validation

All workflow files are validated against the Arazzo 1.0.1 schema using Redocly CLI.

```bash
npm install
npm run validate
```

---

## Author

Victor — [Lax Lab](https://github.com/laxlab)
